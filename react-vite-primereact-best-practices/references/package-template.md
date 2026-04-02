# React + Vite + PrimeReact package template

Use this as a starting point when the repo needs a minimal package manifest for a React + Vite + PrimeReact app.

## Example dependencies

```json
{
  "name": "frontend",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "axios": "1.9.0",
    "dompurify": "3.2.6",
    "primeicons": "7.0.0",
    "primereact": "10.9.7",
    "react": "19.2.0",
    "react-dom": "19.2.0",
    "react-hook-form": "7.56.4",
    "react-router-dom": "7.6.1",
    "zod": "3.25.36",
    "@hookform/resolvers": "5.0.1"
  },
  "devDependencies": {
    "@types/dompurify": "3.2.0",
    "@types/react": "19.2.2",
    "@types/react-dom": "19.2.2",
    "typescript": "5.9.3",
    "vite": "7.1.12"
  }
}
```

## Dependency purposes

| Package | Purpose |
|---|---|
| `axios` | HTTP client with interceptors for centralized auth and error handling |
| `dompurify` | HTML sanitization for safe rendering of untrusted content |
| `react-hook-form` | Form state management — replaces manual `useState` per field |
| `@hookform/resolvers` | Bridge between `react-hook-form` and validation libraries (zod) |
| `zod` | Schema-based validation — type-safe, works with react-hook-form |
| `react-router-dom` | Client-side routing |

## Notes

- Pin versions explicitly.
- Add `primeflex` only if the project chooses it as the responsive utility layer.
- Keep extra UI dependencies out unless the repo has a clear need.
- If the repo already pins React/Vite versions elsewhere, follow the repo instead of forcing this example verbatim.
- Every dependency used in `coding-standards-example.md` and `shared-patterns.md` must appear here.
