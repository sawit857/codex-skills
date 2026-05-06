# React + Vite + PrimeReact Stack Reference

Use this file when choosing packages, reviewing dependencies, or normalizing a React + Vite + PrimeReact codebase to the team standard.

This reference follows the same design intent as the Next-based skill, but adapts it for a lightweight SPA.

## Recommended stack

- React SPA
- React DOM
- TypeScript
- Vite
- PrimeReact `10.9.7`
- PrimeIcons `7.0.0`
- PrimeReact styled mode
- `react-router-dom` for client-side routing when the app has multiple screens
- PrimeFlex `4.0.0` only when a utility/grid layer is needed

## Recommended dependencies

| Package | Purpose |
| --- | --- |
| `axios` | HTTP client with centralized defaults and error handling |
| `react-router-dom` | Client-side routing and route composition |
| `react-hook-form` | Form state management for production forms |
| `@hookform/resolvers` | Bridge from RHF to zod |
| `zod` | Schema validation and typed form parsing |

Optional:

- `dompurify` when the UI must render untrusted HTML safely
- `primeflex` when the project wants PrimeFlex as the layout utility layer
- test tooling such as `vitest`, `@testing-library/react`, and `jsdom`

## Why this stack fits the team structure

- `Vite` keeps the frontend lightweight and fast for SPA development
- `src/app` cleanly owns app composition, providers, and routing
- `src/resources/<group>` keeps domain logic separate from app bootstrap code
- `PrimeReact` accelerates form-heavy and CRUD-heavy screens
- `react-hook-form + zod` reduces repeated field-level state and validation code
- `axios` supports shared clients and request defaults inside `src/resources/<group>/lib`
- `PrimeIcons` and local theme imports keep the PrimeReact visual layer self-contained

## Dependency rules

- Do not add a second HTTP client when `axios` is already the standard.
- Do not add `formik` or `yup` when `react-hook-form + zod` is already the form standard.
- Do not mix PrimeReact styled mode with another full visual system unless the repo already committed to that path.
- Do not normalize unused dependencies into the standard just because one project experimented with them.
- Do not add Tailwind CSS by default when PrimeReact styled mode is the chosen visual system.

## Theme and styling rules

- Use PrimeReact styled mode as the primary visual system.
- Import the theme, PrimeReact base CSS, and PrimeIcons in a shared app entry.
- Use PrimeFlex only if the project explicitly wants it as the main layout utility layer.
- Otherwise use project-owned CSS and media queries.
- Do not scatter theme or base CSS imports across multiple screens.

## Environment and network rules

- Keep backend origins in environment variables or Vite env configuration.
- Do not hardcode `localhost` origins inside screen files.
- Centralize shared API configuration in one module.
- Avoid scattering direct `axios.create()` instances across the app.

## Routing and service boundary rules

- Keep route composition in `src/app`.
- Keep screen components thin and domain services under `src/resources/<group>/services`.
- Keep lower-level clients, DTO mappers, and helpers under `src/resources/<group>/lib`.
- Do not move domain operations into route definitions just because the app is client-rendered.

## Offline-friendly asset loading

The browser should not need outbound internet access to render the app.

- Import PrimeIcons from npm.
- Host custom fonts locally if needed.
- Do not use external CDN script or stylesheet tags.
- Do not reference remote font providers by default.

## Suggested checkpoints

- Confirm the project is actually using Vite.
- Confirm PrimeReact versions are compatible.
- Confirm route composition lives in `src/app`.
- Confirm `src/resources/<group>` is used for shared and domain logic.
- Confirm environment variables are used for backend configuration.
- Confirm repeated API logic is moving into shared modules.
- Confirm the chosen dependencies fit the repo’s actual usage rather than speculative future use.
