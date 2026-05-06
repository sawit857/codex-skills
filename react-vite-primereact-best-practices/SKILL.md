---
name: react-vite-primereact-best-practices
description: แนวทางปฏิบัติที่แนะนำสำหรับการสร้าง รีวิว หรือรีแฟกเตอร์งาน React + Vite + PrimeReact โดยอ้างอิงโครงสร้างมาตรฐานเดียวกับฝั่ง Next คือ `src/app` + `src/resources/<group>` แต่คงความเบาแบบ Vite SPA เหมาะเมื่อ Codex ต้องตั้งค่า/ตรวจสอบโปรเจกต์ที่ใช้ PrimeReact styled mode, react-hook-form + zod, client-side routing, และ service boundary ที่ชัดเจน.
---

# React Vite PrimeReact Best Practices

## ภาพรวม (Overview)

ใช้สกิลนี้เมื่อเป็นงาน `React SPA` ที่สร้างด้วย `Vite` และใช้ `PrimeReact` เป็น component layer หลัก โดยให้ยึดแนว production-oriented setup, styled mode เป็น visual system, dependency เท่าที่จำเป็น, และแยกขอบเขต UI, routing, API, และ domain logic ให้ชัดเจนตามมาตรฐานเดียวกับฝั่ง Next

## ขั้นตอนทำงาน (Workflow)

1. ยืนยันว่าโปรเจกต์เป็น `React + Vite` หรือควรปรับเป็นรูปแบบนี้
2. เลือกโครงสร้างหลักให้ใช้ `src/app` + `src/resources/<group>` เป็นค่าเริ่มต้น
3. ล็อก PrimeReact stack, local asset strategy, และ theme mode ก่อนทำ UI
4. แยก routing, app composition, shared providers, และ domain logic ออกจากกัน
5. รวมศูนย์ API client, error handling, form patterns, และ service modules
6. รีวิว dependency, security, และ offline asset loading ก่อน merge

## กฎโครงสร้าง (Structure Rules)

- ใช้ `src/app` สำหรับ bootstrap, router composition, providers, layouts, และ application shell
- ใช้ `src/resources/<group>` สำหรับ reusable หรือ domain logic
- Treat page/screen components as orchestration layers, not the permanent home of API, validation, or message infrastructure
- Prefer resource-group structure:
  - `src/app`
  - `src/resources/<group>/assets`
  - `src/resources/<group>/contexts`
  - `src/resources/<group>/components`
  - `src/resources/<group>/hooks`
  - `src/resources/<group>/layouts`
  - `src/resources/<group>/lib`
  - `src/resources/<group>/models`
  - `src/resources/<group>/services`
  - `src/resources/<group>/tests`
- ถ้าโปรเจกต์เล็กมาก สามารถใช้ top-level shared folders แบบย่อได้ แต่ไม่ควรปล่อยให้ feature โตจนกระจายปนกันใน `components/`, `hooks/`, `services/` แบบไร้ขอบเขต

Read `references/project-structure.md` first when the task involves setting up folders, moving code, or deciding where modules should live.

## กฎ Stack พื้นฐาน (Frontend Foundation)

ค่าเริ่มต้นควรเป็น stack นี้ เว้นแต่รีโปหรือผู้ใช้กำหนดไว้ต่างจากนี้ชัดเจน:

- React SPA
- TypeScript
- Vite
- PrimeReact `10.9.7`
- PrimeIcons `7.0.0`
- PrimeReact styled mode
- PrimeFlex `4.0.0` only when a utility/grid layer is needed
- `react-router-dom` เมื่อแอปมีหลาย screen/flow
- `react-hook-form + zod` สำหรับ production forms
- หลีกเลี่ยง Tailwind หาก PrimeReact styled mode เป็น visual system หลักของโปรเจกต์

Read `references/stack.md` when you need the package/dependency checklist.

## กฎ PrimeReact และฟอร์ม (PrimeReact and Form Rules)

- Use PrimeReact as the main component library.
- Use styled mode as the default theming path.
- Avoid mixing PrimeReact styled mode with another full UI system unless the repo already does so.
- Search PrimeReact first before writing custom UI.
- Keep PrimeReact provider, theme CSS, PrimeReact base CSS, and PrimeIcons imports in one shared app entrypoint.
- Treat components with built-in request behavior, such as upload widgets, as presentation helpers first; verify or replace their request logic if the backend contract is strict.
- Use `react-hook-form + zod` for production forms.
- Keep manual `useState` per field only for demo code or truly trivial forms.

Read `references/component-selection.md` for the full PrimeReact component inventory organized by category.
Read `references/shared-patterns.md` for shared UI and form patterns.

## กฎ Layout และ Responsive Design

- Keep responsive behavior required even without Tailwind.
- Choose one responsive strategy as the primary one:
  - PrimeFlex
  - project-owned CSS with media queries
- Do not combine Tailwind and PrimeReact styled mode by default.
- Keep the visual system coherent: one theme source, one spacing strategy, one layout utility approach.

## กฎขอบเขต Routing และ Composition

- Keep routing and app-shell composition under `src/app`.
- Keep screen components thin: read route params, call services/hooks, and hand prepared data to UI components.
- Do not let route definitions become the permanent home of API calls or domain transforms.
- Prefer nested layouts and route wrappers only when they reduce duplication; do not over-abstract a small SPA.

## กฎขอบเขต API และ Service (API and Service Boundary Rules)

- Do not hardcode backend origins in screen files.
- Keep environment-based configuration in one place.
- Centralize API clients and request defaults in shared modules.
- Keep service modules under `src/resources/<group>/services` when they represent domain operations.
- Keep lower-level wrappers or clients under `src/resources/<group>/lib` when they support multiple services or features.
- Avoid duplicated inline error extraction and repeated page-local toast/message refs.
- Keep UI components from becoming the permanent home of network configuration or request orchestration.

## กฎ Authentication และ Security Boundary

- Keep authentication tokens, navigation parameters, and temporary session identifiers as separate concerns.
- Send protected API calls with `Authorization: Bearer <token>` when that is the backend contract.
- Do not let PrimeReact or any UI helper redefine auth storage, token flow, API shape, upload format, or error handling rules.

Read `references/security.md` when the task touches auth, upload, token handling, or dependency review.

## กฎ Shared Patterns และ Reuse

- Do not repeat the same UI pattern, hook logic, or layout structure across multiple screens.
- If the same pattern appears in two or more screens, extract it into shared modules.
- Shared patterns should live in `src/app` only when app-wide; otherwise place them in `src/resources/<group>/...`.
- Every serious project should standardize at least: toast/message handling, page loading, API error display, repeated form field layout, and centralized API client.

Read `references/shared-patterns.md` for code examples and placement rules.

## กฎ Offline Asset Loading

All libraries, fonts, icons, and static assets must be bundled into the production build output. The client browser must never need outbound internet access to load the application.

- Do not load scripts, stylesheets, fonts, or icons from external CDNs at runtime.
- Do not use Google Fonts, unpkg, cdnjs, jsdelivr, or any remote asset URL in HTML or CSS.
- PrimeIcons and any icon set must be installed via npm and imported from `node_modules`.
- Font files must be self-hosted inside the project.
- Vite build must produce a self-contained output with no external runtime dependencies.
- If a third-party library attempts to load remote resources at runtime, replace or configure it to use local assets instead.

## กฎการดูแลระบบงาน (Operational Rules)

- Pin package versions explicitly.
- Commit the lock file.
- Review direct and transitive dependencies.
- Run `npm audit` when a frontend package manifest exists.
- Check that PrimeReact-related packages are version-compatible.
- Avoid adding overlapping UI libraries without a strong reason.
- Verify that UI code does not log or persist sensitive token or secret values.
- Verify that no `<script src>`, `<link href>`, `@import`, or `url()` references point to external domains in the final build output.

## รายการตรวจสอบ (Review Checklist)

Use this checklist when reviewing an existing implementation:

- Is the stack actually React + Vite + PrimeReact, without a conflicting UI system?
- Is `src/app` used for app-shell, providers, and router composition rather than mixing everything into `main.tsx` or page files?
- Is `src/resources/<group>` used consistently for shared or domain logic?
- Is PrimeReact using styled mode consistently?
- Is responsive layout handled by PrimeFlex or project CSS, but not a second visual system?
- Are screen files thin enough, or are they becoming the permanent home of API calls and domain transforms?
- Are protected API calls still aligned with backend contract?
- Are authentication state, route params, and temporary session state kept distinct?
- Are package versions pinned and lock files committed?
- Are network-capable components checked against real API requirements?
- Does the production build load all assets locally without any external CDN or remote URL dependency?
- Is every UI element using a PrimeReact component when one exists, instead of a custom replacement?
- Are custom components following PrimeReact design tokens for visual consistency?
- Are shared patterns in place (global Toast, PageLoading, ApiErrorMessage, FormField, useApiCall)?
- Is there duplicated UI logic across pages that should be extracted into a shared hook or component?
- Are forms using `react-hook-form` + `zod` for validation instead of manual state per field?

## แนวทางจากโครงสร้างอ้างอิง (Repo-Derived Guidance)

แนวทางของสกิลนี้อ้างอิงจาก `react-next-primereact-best-practices` เป็นหลัก โดยปรับให้เหมาะกับ Vite:

- Preserve these strengths:
  - `src/app` for application composition
  - `src/resources/<group>` for domain separation
  - centralized models, services, components, and shared patterns
  - PrimeReact-first UI decisions
  - `react-hook-form + zod` for production forms
- Adapt these differences for Vite:
  - replace App Router with `react-router-dom` or equivalent client-side routing
  - move server-first route concerns into app composition, loaders, hooks, or services that fit SPA behavior
  - keep the structure lightweight; do not import Next-specific files or conventions that Vite does not use

## เอกสารอ้างอิง (References)

- `references/project-structure.md` — primary structure reference and placement rules
- `references/stack.md` — package set, compatibility notes, and layout choices
- `references/component-selection.md` — PrimeReact component inventory and per-screen suggestions
- `references/shared-patterns.md` — centralized components, hooks, and providers with code examples
- `references/security.md` — token boundaries, upload/network cautions, and dependency security checks
- `references/coding-standards-example.md` — full code examples and project structure
