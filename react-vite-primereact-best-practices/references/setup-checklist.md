# React + Vite + PrimeReact setup checklist

## Initial setup

- Create the Vite React + TypeScript app scaffold.
- Install `primereact`, `primeicons`, and optionally `primeflex`.
- Confirm React and React DOM versions are compatible with the selected PrimeReact version.
- Add or verify `tsconfig.json`, `vite.config.ts`, and `src/main.tsx`.

## Styling setup

- Import the chosen PrimeReact theme.
- Import PrimeIcons CSS from the npm package (`import 'primeicons/primeicons.css'`), not from a CDN.
- Import PrimeFlex only if the project has selected it.
- Keep PrimeReact styled mode as the primary visual system.
- Do not add Tailwind unless the project explicitly changes direction.
- Do not add Google Fonts or any external font CDN link. If custom fonts are needed, place font files in `public/fonts/` and reference them via local CSS `@font-face`.

## App wiring

- Wrap the app with the provider PrimeReact needs for project-wide configuration.
- Keep routing separate from component-library configuration.
- Keep API client/auth logic outside component-library setup.

## Vite integration

- Configure dev proxy for backend `/api` routes when local development requires it.
- Ensure build output path matches the project packaging model.
- If the app is bundled into a WAR, confirm the built assets are copied into the web root used by the server.

## Offline-only verification

- After `vite build`, inspect `dist/index.html` and all generated CSS/JS for external URLs.
- Confirm no `<script src="https://...">`, `<link href="https://...">`, `@import url('https://...')`, or `url('https://...')` references exist.
- Run the app with network disabled (browser DevTools offline mode) to verify it loads completely.
- If any library loads remote resources at runtime (e.g., font or analytics), replace it with a local alternative or configure it for offline use.

## Security checks

- Verify protected API calls still send `Authorization: Bearer <access_token>` when required.
- Keep `url token` flow separate from bearer auth.
- Keep `signingSessionId` scoped to `signdoc` only.
- Do not log or persist sensitive values unintentionally.
- Run `npm audit` after dependency installation or version changes.
