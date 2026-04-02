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
    "primeicons": "7.0.0",
    "primereact": "10.9.7",
    "react": "19.2.0",
    "react-dom": "19.2.0"
  },
  "devDependencies": {
    "@types/react": "19.2.2",
    "@types/react-dom": "19.2.2",
    "typescript": "5.9.3",
    "vite": "7.1.12"
  }
}
```

## Notes

- Pin versions explicitly.
- Add `primeflex` only if the project chooses it as the responsive utility layer.
- Keep extra UI dependencies out unless the repo has a clear need.
- If the repo already pins React/Vite versions elsewhere, follow the repo instead of forcing this example verbatim.
