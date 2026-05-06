# Next.js + PrimeReact stack reference

Use this file when choosing packages, reviewing dependencies, or normalizing a Next.js + PrimeReact codebase to the team standard.

## Recommended stack

- Next.js App Router
- React
- React DOM
- TypeScript
- PrimeReact `10.9.7`
- PrimeIcons `7.0.0`
- PrimeReact styled mode
- PrimeFlex `4.0.0` only when a utility/grid layer is needed

## Recommended dependencies

| Package | Purpose |
| --- | --- |
| `axios` | HTTP client with centralized defaults and error handling |
| `react-hook-form` | Form state management for production forms |
| `@hookform/resolvers` | Bridge from RHF to zod |
| `zod` | Schema validation and typed form parsing |

Optional:

- `primeflex` when the project wants PrimeFlex as the layout utility layer
- test tooling such as `vitest`, `@testing-library/react`, and `jsdom`

## Why this stack fits the team structure

- `Next.js App Router` gives clear route composition in `src/app`
- `PrimeReact` accelerates form-heavy and CRUD-heavy screens
- `react-hook-form + zod` reduces repeated field-level state and validation code
- `axios` supports shared clients and request defaults inside `src/resources/<group>/lib`
- `PrimeIcons` and local theme imports keep the PrimeReact visual layer self-contained

## Dependency rules

- Do not add a second HTTP client when `axios` is already the standard.
- Do not add `formik` or `yup` when `react-hook-form + zod` is already the form standard.
- Do not mix PrimeReact styled mode with another full visual system unless the repo already committed to that path.
- Do not normalize unused dependencies into the standard just because one repo experimented with them.

## Theme and styling rules

- Use PrimeReact styled mode as the primary visual system.
- Import the theme, PrimeReact base CSS, and PrimeIcons in a shared layout/provider entry.
- Use PrimeFlex only if the project explicitly wants it as the main layout utility layer.
- Otherwise use project-owned CSS and media queries.
- Do not scatter theme or base CSS imports across page files.

## Environment and network rules

- Keep backend origins in environment variables.
- Do not hardcode `localhost` origins in page files.
- Centralize shared API configuration in one module.
- Avoid scattering direct `axios.create()` instances across the app.

## App Router boundary rules

- Default route entrypoints and layouts to `Server Components`.
- Add `"use client"` only where interactive browser behavior is required.
- Keep PrimeReact interactive widgets inside client components, but keep surrounding route composition on the server when possible.
- Do not move initial data loading to the client when the same data can be prepared on the server.

## Mutation and transport rules

- Use server actions for same-app form mutations when the App Router flow fits.
- Use `route.ts` when the app needs a real HTTP endpoint.
- Use shared clients and `services` for external backend communication.
- Keep transport, validation, and error normalization centralized near the service boundary.

## Offline-friendly asset loading

The browser should not need outbound internet access to render the app.

- Import PrimeIcons from npm.
- Host custom fonts locally if needed.
- Do not use external CDN script or stylesheet tags.
- Do not reference remote font providers by default.

## Suggested checkpoints

- Confirm the project is actually using App Router.
- Confirm PrimeReact versions are compatible.
- Confirm environment variables are used for backend configuration.
- Confirm repeated API logic is moving into shared modules.
- Confirm the chosen dependencies fit the repo’s actual usage rather than speculative future use.
