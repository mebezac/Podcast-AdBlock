# Frontend Architecture

## Overview

Podly's frontend is a **React + TypeScript** single-page application built with **Vite**.

## Technology Stack

- **Framework**: React 18
- **Language**: TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **HTTP Client**: Native fetch API
- **Icons**: Lucide React

## Project Structure

```
frontend/
├── src/
│   ├── components/        # Reusable UI components
│   ├── pages/            # Page-level components
│   ├── hooks/            # Custom React hooks
│   ├── services/         # API service functions
│   ├── types/            # TypeScript interfaces
│   ├── utils/            # Utility functions
│   ├── context/          # React contexts
│   └── App.tsx           # Root component
├── public/               # Static assets
├── index.html            # HTML entry point
├── vite.config.ts        # Vite configuration
└── tailwind.config.js    # Tailwind configuration
```

## Key Components

### Layout Components
- `Layout.tsx` - Main app shell with navigation
- `Sidebar.tsx` - Navigation sidebar
- `Header.tsx` - Top header bar

### Page Components

**Feed Management:**
- `FeedsPage.tsx` - List all podcast feeds
- `FeedDetailPage.tsx` - View feed details and episodes
- `AddFeedModal.tsx` - Add new podcast feed

**Episode Management:**
- `PostsPage.tsx` - List episodes with filters
- `PostDetailPage.tsx` - Episode details and actions
- `AudioPlayer.tsx` - Built-in audio player

**Configuration:**
- `ConfigPage.tsx` - Main settings page
- `LLMSettings.tsx` - LLM configuration
- `WhisperSettings.tsx` - Transcription settings
- `ProcessingSettings.tsx` - Processing options
- `OutputSettings.tsx` - Audio output settings
- `AppSettings.tsx` - Application settings

**Jobs:**
- `JobsPage.tsx` - Background job monitoring
- `JobDetail.tsx` - Individual job status

**Auth:**
- `LoginPage.tsx` - Login form
- `UserManagement.tsx` - Admin user management

### Shared Components
- `Button.tsx` - Styled button component
- `Card.tsx` - Card container
- `Modal.tsx` - Dialog modal
- `Toast.tsx` - Notification toasts
- `LoadingSpinner.tsx` - Loading indicator
- `ErrorBoundary.tsx` - Error handling

## State Management

**Approach**: React Context + useState (no Redux)

**Contexts:**
- `AuthContext` - Authentication state
- `ConfigContext` - Application settings
- `ToastContext` - Notifications

**Local State:**
- Component-level useState for UI state
- useEffect for data fetching
- Custom hooks for reusable logic

## API Integration

All API calls use native fetch with async/await:

```typescript
// services/api.ts
export async function fetchFeeds(): Promise<Feed[]> {
  const response = await fetch('/api/feeds', {
    credentials: 'include'  // Send cookies
  });
  if (!response.ok) throw new Error('Failed to fetch feeds');
  return response.json();
}
```

## Key Features

### Real-time Updates
- Polling for job status updates
- WebSocket not implemented (HTTP polling used)
- Refresh buttons for manual updates

### Responsive Design
- Mobile-first with Tailwind breakpoints
- Collapsible sidebar on mobile
- Touch-friendly controls

### Audio Player
- Custom HTML5 audio wrapper
- Playback controls (play, pause, skip)
- Progress bar with seeking
- Speed control (0.5x - 2x)
- Episode artwork display

### Form Handling
- Controlled inputs with React state
- Client-side validation
- Error message display
- Submit loading states

## Routing

React Router handles navigation:

```typescript
// App.tsx routes
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Layout />}>
      <Route index element={<Dashboard />} />
      <Route path="feeds" element={<FeedsPage />} />
      <Route path="feeds/:id" element={<FeedDetailPage />} />
      <Route path="posts" element={<PostsPage />} />
      <Route path="config" element={<ConfigPage />} />
      <Route path="jobs" element={<JobsPage />} />
      <Route path="login" element={<LoginPage />} />
    </Route>
  </Routes>
</BrowserRouter>
```

## Build & Development

**Development:**
```bash
cd frontend
npm install
npm run dev  # Starts Vite dev server on :5173
```

**Production Build:**
```bash
npm run build  # Outputs to dist/
```

**Integration:**
- Flask serves static files from `src/app/static/`
- Vite build output copied to static folder
- Flask template renders index.html
- API requests proxied to Flask backend

## Environment Variables

```env
VITE_API_URL=http://localhost:5001  # API base URL
```

## TypeScript Types

Key interfaces defined in `src/types/`:

```typescript
// types/feed.ts
interface Feed {
  id: number;
  title: string;
  description?: string;
  rss_url: string;
  image_url?: string;
  posts: Post[];
}

// types/post.ts
interface Post {
  id: number;
  title: string;
  description?: string;
  release_date: string;
  duration: number;
  processed_audio_path?: string;
  whitelisted: boolean;
}
```

## Error Handling

- API errors displayed as toast notifications
- ErrorBoundary catches React errors
- Network errors retry with backoff
- Form validation prevents invalid submissions

## Accessibility

- Semantic HTML elements
- ARIA labels for screen readers
- Keyboard navigation support
- Color contrast compliance
- Focus management in modals
