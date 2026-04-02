# React + Vite + PrimeReact setup checklist

## Initial setup

- Create the Vite React + TypeScript app scaffold.
- Install `primereact`, `primeicons`, and optionally `primeflex`.
- Confirm React and React DOM versions are compatible with the selected PrimeReact version.
- Add or verify `tsconfig.json`, `vite.config.ts`, and `src/main.tsx`.

## Styling setup

- Import the chosen PrimeReact theme.
- Import PrimeIcons CSS.
- Import PrimeFlex only if the project has selected it.
- Keep PrimeReact styled mode as the primary visual system.
- Do not add Tailwind unless the project explicitly changes direction.

## App wiring

- Wrap the app with the provider PrimeReact needs for project-wide configuration.
- Keep routing separate from component-library configuration.
- Keep API client/auth logic outside component-library setup.

## Vite integration

- Configure dev proxy for backend `/api` routes when local development requires it.
- Ensure build output path matches the project packaging model.
- If the app is bundled into a WAR, confirm the built assets are copied into the web root used by the server.

## Security checks

- Verify protected API calls still send `Authorization: Bearer <access_token>` when required.
- Keep `url token` flow separate from bearer auth.
- Keep `signingSessionId` scoped to `signdoc` only.
- Do not log or persist sensitive values unintentionally.
- Run `npm audit` after dependency installation or version changes.
