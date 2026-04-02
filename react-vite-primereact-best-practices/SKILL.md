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

Read `references/stack.md` when you need the package/dependency checklist.
Read `references/package-template.md` when you need a starting `package.json`.
Read `references/setup-checklist.md` when you need a step-by-step setup pass.

## PrimeReact integration rules

- Use PrimeReact as the main component library.
- Use styled mode as the default theming path.
- Avoid mixing PrimeReact styled mode with another full UI system unless the repo already does so.
- Prefer PrimeReact components for inputs, dialogs, steps, messages, overlays, and feedback UI.
- Treat components with built-in request behavior, such as upload widgets, as presentation helpers first; verify or replace their request logic if the backend contract is strict.

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

## Dependency and security review

Before finalizing a React + Vite + PrimeReact setup or change:

- Pin package versions explicitly.
- Commit the lock file.
- Review direct and transitive dependencies.
- Run `npm audit` when a frontend package manifest exists.
- Check that PrimeReact-related packages are version-compatible.
- Avoid adding overlapping UI libraries without a strong reason.
- Verify that UI code does not log or persist sensitive token or secret values.

## Review checklist

Use this checklist when reviewing an existing implementation:

- Is the stack actually React + Vite + PrimeReact, without a conflicting UI system?
- Is PrimeReact using styled mode consistently?
- Is responsive layout handled by PrimeFlex or project CSS, but not a second visual system?
- Are protected API calls still aligned with backend contract?
- Are token roles kept distinct?
- Are package versions pinned and lock files committed?
- Are network-capable components checked against real API requirements?

## References

- `references/stack.md` — package set, compatibility notes, and layout choices
- `references/package-template.md` — starting `package.json` example
- `references/setup-checklist.md` — setup sequence for React + Vite + PrimeReact
- `references/component-selection.md` — suggested PrimeReact component groups per screen
- `references/security.md` — token boundaries, upload/network cautions, and dependency security checks
