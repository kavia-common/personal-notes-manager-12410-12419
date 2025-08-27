# Notes Frontend (SvelteKit)

A modern, minimalistic Svelte-based SPA for managing personal notes with authentication and full CRUD. Uses a light theme and the following colors:
- primary: #1976d2
- secondary: #424242
- accent: #ffb300

## Features
- User authentication (login and signup)
- Create, list, search, edit, and delete notes
- Responsive layout
  - Sidebar for navigation and user status
  - Notes list on the left
  - Editor/viewer on the right
- Environment-configurable API base

## Configuration
Create a .env file (see .env.example):
```
VITE_API_BASE=/api
```
Point it to your backend (e.g., https://api.example.com) when integrating.

## Scripts
- npm run dev — start dev server
- npm run build — build for production
- npm run preview — preview built app
- npm run check — type and Svelte checks
- npm run lint — lint

## Project structure
- src/lib/services/api.ts — API client (auth + notes CRUD)
- src/lib/stores/auth.ts — auth store and helpers
- src/lib/stores/notes.ts — notes store and helpers
- src/routes/+layout.svelte — app layout with sidebar and auth summary
- src/routes/+page.svelte — main notes UI (search, list, editor)
- src/routes/auth/+page.svelte — login/signup

## Backend integration
This frontend expects a backend with:
- POST /auth/login { email, password } -> { token, user: { id, email } }
- POST /auth/signup { email, password } -> { token, user }
- GET /notes?q=... -> Note[]
- GET /notes/:id -> Note
- POST /notes -> Note
- PUT /notes/:id -> Note
- DELETE /notes/:id -> { success: boolean }

Attach the token via Authorization: Bearer <token> (handled by api client).

## Notes type
```
type Note = {
  id: string;
  title: string;
  content: string;
  tags?: string[];
  updatedAt: string; // ISO
  createdAt: string; // ISO
}
```

## Styling
The light theme and minimal style system is defined in src/app.css using CSS variables, with primary/secondary/accent colors applied to components.
