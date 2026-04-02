---
name: react-vite-primereact-best-practices
description: Best practices for building, reviewing, or refactoring React + Vite + PrimeReact frontends. Use when Codex needs to set up a React SPA with Vite, choose or review PrimeReact integration, implement PrimeReact styled mode, define responsive layout strategy without Tailwind, or keep frontend auth/API behavior aligned with backend contracts.
---

# React Vite PrimeReact Best Practices

## Overview

Use this skill when the work involves a React SPA built with Vite and PrimeReact. Default to a production-oriented setup: PrimeReact as the main component layer, styled mode as the visual system, minimal extra dependencies, and frontend behavior that follows backend auth and API contracts instead of inventing its own abstractions.

## Workflow

1. Confirm the frontend foundation.
2. Lock the PrimeReact stack and theme approach.
3. Keep UI concerns separate from auth and network concerns.
4. Review dependencies and security before implementation or merge.

## Frontend foundation

Default to this stack unless the repository or user explicitly overrides it:

- React SPA
- TypeScript
- Vite
- PrimeReact `10.9.7`
- PrimeIcons `7.0.0`
- PrimeReact styled mode
- PrimeFlex `4.0.0` only if a utility/grid layer is needed
- No Tailwind CSS when PrimeReact styled mode is the chosen visual system

## Offline-only asset loading

All libraries, fonts, icons, and static assets must be bundled into the production build output. The client browser must never need outbound internet access to load the application.

- Do not load scripts, stylesheets, fonts, or icons from external CDNs at runtime.
- Do not use Google Fonts, unpkg, cdnjs, jsdelivr, or any remote asset URL in HTML or CSS.
- PrimeIcons and any icon set must be installed via npm and imported from `node_modules`.
- Font files must be self-hosted inside the project (e.g. `public/fonts/` or imported via CSS from the bundle).
- Vite build must produce a fully self-contained output with no external runtime dependencies.
- If a third-party library attempts to load remote resources at runtime, replace or configure it to use local assets instead.

Read `references/stack.md` when you need the package/dependency checklist.
Read `references/package-template.md` when you need a starting `package.json`.
Read `references/setup-checklist.md` when you need a step-by-step setup pass.

## PrimeReact integration rules

- Use PrimeReact as the main component library.
- Use styled mode as the default theming path.
- Avoid mixing PrimeReact styled mode with another full UI system unless the repo already does so.
- Treat components with built-in request behavior, such as upload widgets, as presentation helpers first; verify or replace their request logic if the backend contract is strict.

### PrimeReact-first component selection

When implementing any UI element, follow this decision order:

1. **Search PrimeReact first.** Check `references/component-selection.md` for the full component inventory. If PrimeReact has a component that covers the need, use it. Do not write a custom replacement.
2. **Customize PrimeReact if close but not exact.** If PrimeReact has a component that almost fits, use it and customize via props, `className`, `style`, `pt` (passthrough), or CSS overrides on PrimeReact design tokens. Do not rewrite the component from scratch.
3. **Write custom only when PrimeReact cannot cover it.** If no PrimeReact component exists for the need, write a custom component. Custom components must follow PrimeReact design tokens for colors, spacing, and typography so the UI stays visually coherent.
4. **Never duplicate what PrimeReact already provides.** Do not create a custom `<AppButton>`, `<AppDialog>`, `<AppTooltip>`, or similar wrapper that only re-exports a PrimeReact component with no added logic. Import and use PrimeReact directly.

Read `references/component-selection.md` for the full PrimeReact component inventory organized by category.

## Styling and responsive layout

- Keep responsive behavior required even without Tailwind.
- Choose one responsive strategy as the primary one:
  - PrimeFlex
  - project-owned CSS with media queries
- Do not combine Tailwind and PrimeReact styled mode by default.
- Keep the visual system coherent: one theme source, one spacing strategy, one layout utility approach.

## Auth and API boundary

- Keep `accessToken`, `url token`, and `signingSessionId` as separate concerns.
- Send protected API calls with `Authorization: Bearer <access_token>` when that is the backend contract.
- Use `url token` only to open and resolve screen context such as `ggaureg-ui`, `certreg-ui`, or `signdoc-ui`.
- Use `signingSessionId` only for `signdoc` session reuse.
- Do not let a component library redefine auth storage, token flow, API shape, upload format, or error handling rules.

Read `references/security.md` when the task touches auth, upload, token handling, or dependency review.

## Code reuse and centralization

Do not repeat the same UI pattern, hook logic, or layout structure across multiple pages. Extract shared concerns into centralized modules.

### When to extract

- If the same pattern appears in two or more pages, extract it into `components/common/`, `hooks/`, or `providers/`.
- If a PrimeReact component always needs the same wrapper setup (e.g., Toast requires a ref and show/hide methods), wrap it in a shared hook or context provider.
- If multiple forms repeat label + input + validation error layout, extract a shared `<FormField>` wrapper.

### Required shared patterns

Every project should have these centralized patterns instead of duplicating them per page:

- **Global Toast** — `AppToastProvider` + `useAppToast()` hook. One `<Toast>` in the layout, every page calls `useAppToast().show()`.
- **Page loading** — `<PageLoading />` component wrapping `ProgressSpinner` with centered layout.
- **API error display** — `<ApiErrorMessage />` component wrapping `Message` with standard error extraction from Axios errors.
- **Form field wrapper** — `<FormField>` component that renders label, children (any input), and validation error message in a consistent layout.
- **Confirm dialog** — `useConfirmDialog()` hook wrapping PrimeReact `ConfirmDialog` with a promise-based API.
- **API call state** — `useApiCall()` hook that manages `loading`, `error`, `data` state for a single async operation.

### What not to extract

- Do not create thin wrappers that only re-export a PrimeReact component with no added logic.
- Do not create a generic `<Form>` component that tries to handle every form shape — use `react-hook-form` + `zod` schemas instead.
- Do not extract page-specific business logic into shared hooks.

Read `references/shared-patterns.md` for full code examples of each pattern.

## Dependency and security review

Before finalizing a React + Vite + PrimeReact setup or change:

- Pin package versions explicitly.
- Commit the lock file.
- Review direct and transitive dependencies.
- Run `npm audit` when a frontend package manifest exists.
- Check that PrimeReact-related packages are version-compatible.
- Avoid adding overlapping UI libraries without a strong reason.
- Verify that UI code does not log or persist sensitive token or secret values.
- Verify that no `<script src>`, `<link href>`, `@import`, or `url()` references point to external domains in the final build output.

## Review checklist

Use this checklist when reviewing an existing implementation:

- Is the stack actually React + Vite + PrimeReact, without a conflicting UI system?
- Is PrimeReact using styled mode consistently?
- Is responsive layout handled by PrimeFlex or project CSS, but not a second visual system?
- Are protected API calls still aligned with backend contract?
- Are token roles kept distinct?
- Are package versions pinned and lock files committed?
- Are network-capable components checked against real API requirements?
- Does the production build load all assets locally without any external CDN or remote URL dependency?
- Is every UI element using a PrimeReact component when one exists, instead of a custom replacement?
- Are custom components following PrimeReact design tokens for visual consistency?
- Are shared patterns in place (global Toast, PageLoading, ApiErrorMessage, FormField, useApiCall)?
- Is there duplicated UI logic across pages that should be extracted into a shared hook or component?
- Are forms using `react-hook-form` + `zod` for validation instead of manual state per field?

## References

- `references/stack.md` — package set, compatibility notes, and layout choices
- `references/package-template.md` — starting `package.json` example
- `references/setup-checklist.md` — setup sequence for React + Vite + PrimeReact
- `references/component-selection.md` — PrimeReact component inventory and per-screen suggestions
- `references/shared-patterns.md` — centralized components, hooks, and providers with code examples
- `references/security.md` — token boundaries, upload/network cautions, and dependency security checks
- `references/coding-standards-example.md` — full code examples and project structure
