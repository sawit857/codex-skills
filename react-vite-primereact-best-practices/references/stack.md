# React + Vite + PrimeReact stack reference

## Recommended stack

- React
- React DOM
- TypeScript
- Vite
- PrimeReact `10.9.7`
- PrimeIcons `7.0.0`
- PrimeReact styled mode
- PrimeFlex `4.0.0` only when a utility/grid layer is needed

## Recommended dependencies

| Package | Version | Purpose |
|---|---|---|
| `axios` | `1.9.0` | HTTP client with interceptors for centralized auth/error handling |
| `react-router-dom` | `7.6.1` | Client-side routing |
| `react-hook-form` | `7.56.4` | Form state management — replaces manual useState per field |
| `@hookform/resolvers` | `5.0.1` | Bridge react-hook-form ↔ zod |
| `zod` | `3.25.36` | Schema validation — type-safe, runtime-safe |
| `dompurify` | `3.2.6` | HTML sanitization for safe rendering of untrusted content |

### Why these packages

- **axios** over native `fetch`: interceptor pattern allows centralized Bearer token attachment and 401 redirect without repeating per call.
- **react-hook-form + zod** over manual state: eliminates repeated `useState` + `onChange` + validation per field across every form. Zod schemas also validate API response shapes.
- **react-router-dom**: required for multi-page SPA routing (`ggaureg-ui`, `certreg-ui`, `signdoc-ui`).
- **dompurify**: required by `SafeHtml` component for sanitizing untrusted HTML.

### What not to add

- Do not add `formik` or `yup` when `react-hook-form` + `zod` is already in the stack.
- Do not add a second HTTP client (e.g., `ky`, `got`) when `axios` is the standard.
- Do not add Tailwind CSS when PrimeReact styled mode is the visual system.

## Compatibility notes

- PrimeReact `10.9.7` depends on React and React DOM as peer dependencies.
- PrimeReact `10.9.7` also brings `react-transition-group` as a runtime dependency.
- PrimeIcons is required for many PrimeReact components to render icons correctly.
- PrimeFlex is optional; if not used, the project must provide responsive layout via custom CSS.

## Theme and styling decisions

- Use PrimeReact styled mode as the primary visual system.
- Do not add Tailwind CSS when the project has already committed to PrimeReact styled mode.
- Keep theme ownership in one place to avoid specificity and maintenance issues.

## Offline-only asset loading

The application must work without outbound internet access from the client browser.

- All npm packages are installed locally and bundled by Vite — no CDN script/link tags.
- PrimeIcons must be imported from the npm package (`import 'primeicons/primeicons.css'`), not from a CDN.
- If custom fonts are needed, host the font files inside `public/fonts/` or import them from a local package.
- Do not reference Google Fonts, unpkg, cdnjs, jsdelivr, or any external URL in `index.html`, CSS `@import`, or `url()` values.
- After `vite build`, verify the `dist/` output contains all required `.css`, `.js`, `.woff2`, and image files with no external fetch at runtime.

## Suggested setup checkpoints

- Import PrimeReact theme resources correctly.
- Import PrimeIcons CSS.
- Initialize app providers needed by PrimeReact.
- Keep Vite dev proxy pointed at backend `/api` routes.
- Ensure production build output is bundled into the WAR when the project uses Tomcat packaging.
- Confirm no `<link>` or `<script>` tags in `index.html` point to external domains.
