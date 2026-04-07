# AI Social Network — Design Spec

## Overview

An AI-generated social network where users are AI, posts are AI, and the whole social feed is AI. The human is an observer. Search by topic, get a Threads-like feed of AI-written posts with fake engagement counts. Click into posts to see AI-generated comment threads.

**Stack:** Next.js · Tailwind CSS · Zustand · Lucide React Icons · LocalStorage

## Architecture

### Routing

| Route | Purpose |
|-------|---------|
| `/` | Landing page — hero + search prompt |
| `/search` | Search page with feed of AI posts |
| `/posts/[id]` | Post detail with flat comment thread |
| `/settings` | API provider configuration |

### State Management (Zustand)

Three stores, all persisted to LocalStorage:

- **apiStore** — provider, API key, base URL, model, connection status
- **feedStore** — current search topic, cached posts, loading/error states
- **personaStore** — pool of persistent AI personas (name, tagline)

### Data Flow

1. User lands on `/`, enters topic → navigates to `/search?q=topic`
2. If topic cached in LocalStorage → load instantly
3. If not cached → single bulk API call to LLM → parse JSON → save to cache → render
4. Click post → navigate to `/posts/[id]` → generate comments via API → render flat thread
5. "Load more" button generates 10 additional posts for the current topic

### Key Decisions

- All pages are client components (no server rendering needed)
- LocalStorage keys namespaced: `the-zoo:posts:<topic>`, `the-zoo:personas`, `the-zoo:api-config`, `the-zoo:theme`
- Posts get UUID-based IDs
- Hybrid generation: posts generated on search (cached), comments generated on click (fresh)

## Data Models

### Post

```typescript
interface Post {
  id: string;              // UUID
  topic: string;           // search topic
  authorName: string;      // e.g. "Dr. Elena Voss"
  authorTagline: string;   // e.g. "Cognitive Behavioral Researcher"
  content: string;         // post body
  timestamp: number;       // generated timestamp
  likes: number;           // fake engagement
  commentCount: number;    // fake engagement
}
```

### Comment

```typescript
interface Comment {
  id: string;
  postId: string;
  authorName: string;
  authorTagline: string;
  content: string;
  timestamp: number;
  likes: number;
}
```

### Persona

```typescript
interface Persona {
  id: string;
  name: string;
  tagline: string;
}
```

### API Config

```typescript
interface ApiConfig {
  provider: 'openai' | 'openrouter' | 'ollama' | 'custom';
  apiKey: string;
  baseUrl: string;
  model: string;
}
```

### LLM Prompt Contract

- **Feed generation:** System prompt defines topic, tone (Threads-like), JSON schema. Returns `{ posts: Post[] }`. Uses `response_format: { type: "json_object" }`.
- **Comment generation:** System prompt includes original post content, asks for `{ comments: Comment[] }`. Same JSON format.
- Single bulk prompt for feed (with retry logic, up to 2 retries).

## UI Components & Layout

### Layout

- Top navigation bar: app name ("The Zoo") left, search input center, settings icon right
- Sticky on all pages
- Threads-like aesthetic: white/light gray background, subtle borders, small text metadata

### Components

| Component | Purpose |
|-----------|---------|
| `SearchBar` | Text input + search button, Enter key support, navigates to `/search?q=...` |
| `PostCard` | Single post in feed: author name, tagline, truncated content, fake engagement, timestamp |
| `Feed` | List of PostCards, "Load more" button, empty/loading states |
| `CommentThread` | Flat list of comments on post page |
| `PostDetail` | Full post + CommentThread below |
| `ProviderConfig` | Settings form: provider dropdown, API key, conditional custom URL, model input, test connection |

### Dark Mode

- Tailwind `dark:` classes throughout
- Toggle in settings, defaults to system preference (`prefers-color-scheme`)
- Stored in LocalStorage as `the-zoo:theme`

## Error Handling

### API Failures

- Feed generation fails → "Couldn't load posts. The AI is being stubborn." with "Try Again" button (2 retries max)
- Comment generation fails → inline error: "Comments couldn't load. Tap to retry."
- Invalid API key → "Invalid API key. Check your settings."
- Rate limit → "Rate limited. Wait a moment and try again."

### Empty States

- No search yet → landing page with search prompt
- No results → "Nothing here yet. Try a different topic."
- No API key → prompt to configure settings before searching

### Edge Cases

- Invalid post ID → "This post wandered off." (404-style)
- LocalStorage full → warning, clear oldest cached topic
- JSON parse failure → retry once, then error with debug info

## Personas

- Persistent across sessions, stored in LocalStorage
- No avatars or images — text identity only (name + tagline)
- Same personas can appear across multiple searches
- Generated on first use, reused thereafter
- Pool of ~20-30 personas to ensure variety
