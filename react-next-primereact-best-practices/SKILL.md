---
name: react-next-primereact-best-practices
description: แนวทางปฏิบัติที่แนะนำสำหรับการสร้าง รีวิว หรือรีแฟกเตอร์งาน Next.js App Router + PrimeReact ตามโครงสร้างมาตรฐานของทีม เหมาะเมื่อ Codex ต้องตั้งค่า/ตรวจสอบโปรเจกต์ที่ใช้ App Router, route groups, PrimeReact styled mode, react-hook-form + zod, และสถาปัตยกรรม `src/app` + `src/resources/<group>`.
---

# React Next PrimeReact Best Practices

## ภาพรวม (Overview)

ใช้สกิลนี้สำหรับงาน `Next.js + PrimeReact` ที่ต้องพร้อมใช้งานระดับ production และยึดโครงสร้างมาตรฐานของทีม โดยให้ใช้ `Next.js App Router`, `PrimeReact` เป็น component layer หลัก, และโครงสร้าง `src/app` + `src/resources/<group>` เว้นแต่ในรีโปมีแนวทางเดิมที่ชัดเจนอยู่แล้ว

## ขั้นตอนทำงาน (Workflow)

1. ยืนยันว่ารีโปเป็น `Next.js App Router` หรือควรปรับเป็นรูปแบบนี้
2. เลือกโครงสร้างให้เหมาะระหว่าง `Enterprise Structure` และ `Global Standard Structure`
3. ค่าเริ่มต้นให้ใช้ `Enterprise Structure` สำหรับระบบที่มีหลายโดเมน/หลายทีม
4. กำหนด PrimeReact stack, local asset strategy, และ theme mode ก่อนเริ่มทำ UI
5. แบ่งขอบเขต `Server Component` และ `Client Component` ให้ชัดก่อนเพิ่ม page logic
6. วาง route ไว้ใน `src/app` และวาง shared/domain logic ไว้ใน `src/resources/<group>`
7. เลือกเส้นทาง mutation ให้ชัดเจน: server actions, `route.ts`, หรือ external backend API
8. รวมศูนย์ API client, error handling, และ form patterns ที่ใช้ซ้ำ ไม่กระจายไว้ในหลาย page
9. รีวิวจุดเสี่ยง: วางไฟล์ผิดที่, hardcoded backend URL, logic ซ้ำใน page, cache ผิดพลาด, และ tutorial pattern ที่หลุดเข้า production

## กฎโครงสร้าง (Structure Rules)

- Keep `route` concerns in `src/app`: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `route.ts`.
- Keep reusable or domain logic in `src/resources/<group>`.
- Treat page files as orchestration layers, not the place to duplicate API, validation, and message infrastructure.
- Prefer `Enterprise Structure`:
  - `src/app` for routing and composition
  - `src/resources/<group>/assets`
  - `src/resources/<group>/contexts`
  - `src/resources/<group>/components`
  - `src/resources/<group>/hooks`
  - `src/resources/<group>/layouts`
  - `src/resources/<group>/lib`
  - `src/resources/<group>/models`
  - `src/resources/<group>/services`
  - `src/resources/<group>/tests`
- Treat `Global Standard Structure` as the simpler fallback for small or medium apps.

Read `references/project-structure.md` first when the task involves setting up folders, moving code, or deciding where modules should live.

## กฎ PrimeReact และฟอร์ม (PrimeReact and Form Rules)

- Use PrimeReact as the main component library.
- Use styled mode as the default theming path.
- Avoid mixing PrimeReact styled mode with another full visual system unless the repository already does so.
- Search PrimeReact first before writing custom UI.
- Keep the PrimeReact provider, theme CSS, PrimeReact base CSS, and PrimeIcons imports in one shared layout/provider entrypoint.
- Mark PrimeReact-heavy interactive components with `"use client"` only where browser APIs, event handlers, refs, or client hooks are actually needed.
- Do not turn an entire route tree into a client boundary just to host one form, dialog, or table.
- Use `react-hook-form + zod` for production forms.
- Keep manual `useState` per field only for teaching/demo code or truly trivial forms.

Read `references/component-selection.md` for PrimeReact-first component choices.
Read `references/shared-patterns.md` for shared UI and form patterns.

## กฎขอบเขต Server/Client (Server and Client Boundary Rules)

- Default to `Server Components` for route entrypoints, layout composition, and initial data loading.
- Add `"use client"` only for modules that need browser-only behavior such as event handlers, refs, local interactive state, PrimeReact widgets, or client hooks.
- Keep data fetching on the server when the page can render from request-time or build-time data without browser interaction.
- Keep page files thin: read params, trigger server-side loaders, and hand prepared data to client components when interactivity is needed.
- Do not move server-safe logic into client components just to simplify imports.
- Do not import server-only modules into client components.
- Prefer small client islands inside server-rendered routes instead of promoting the full page to the client.

Read `references/coding-standards-example.md` for route and component boundary examples.

## กฎขอบเขต API และ Service (API and Service Boundary Rules)

- Do not hardcode backend origins such as `http://localhost:8080` inside page files.
- Keep environment-based configuration in one place.
- Centralize API clients and request defaults in shared modules.
- Keep service modules under `src/resources/<group>/services` when they represent domain operations.
- Keep lower-level wrappers or clients under `src/resources/<group>/lib` when they support multiple services or features.
- Avoid duplicated inline error extraction and repeated page-local message refs.
- Keep page files and client components from becoming the permanent home of network configuration or request orchestration.

## กฎ Authentication และ Security Boundary

- Keep authentication tokens, route parameters, cookies, and temporary session state as separate concerns.
- Send protected API calls with `Authorization: Bearer <token>` only when that is the backend contract.
- Keep secrets and protected values out of client logs, persistent browser storage, and accidental UI state leaks.
- Do not let PrimeReact or any UI helper redefine auth storage, token flow, upload format, API shape, or error handling rules.

Read `references/security.md` when the task touches auth, upload, token handling, or dependency review.

## กฎการเลือก Mutation Strategy

- Use `server actions` for form-style mutations that belong to the same Next application boundary and fit the App Router workflow.
- Use `route.ts` when the application needs an HTTP endpoint, webhook receiver, file upload endpoint, or public/internal API surface inside the Next app.
- Use a shared API client plus `services` when the Next frontend talks to an external backend.
- Keep mutation-specific validation, DTO transforms, and error normalization close to the domain service instead of duplicating them in each page or component.
- Do not call external APIs directly from many client components when the same operation can be centralized.
- When a client component submits a mutation, keep it focused on collecting input and rendering state; move domain operations into `services` or server actions.

Read `references/stack.md` for stack and dependency decisions.
Read `references/coding-standards-example.md` for Next-specific code examples using the team structure.

## กฎ Data Fetching และ Cache

- Treat `page.tsx` and route layouts as server-first entrypoints for initial data loading.
- Choose static, dynamic, or revalidated behavior intentionally; do not leave cache behavior accidental.
- Keep fetch or API usage consistent within a feature so stale and fresh data expectations are understandable.
- Avoid duplicating the same initial fetch in both the route file and the first client component render.
- When a page becomes highly interactive after the initial render, separate initial server-loaded data from client-side refresh or mutation flows.
- Document or make explicit any non-default cache assumptions when they affect correctness.

## กฎการดูแลระบบงาน (Operational Rules)

- Pin major frontend package versions intentionally and keep the lock file committed.
- Keep PrimeReact-related package versions compatible with each other.
- Prefer local bundled assets for fonts, icons, and stylesheets; do not rely on external CDNs by default.
- Keep the browser able to render the application without outbound runtime requests for UI assets.
- Keep theme and visual imports owned by the application rather than scattered across route files.
- Review dependency additions for overlap before adding another UI, form, or HTTP library.

## รายการตรวจสอบ (Review Checklist)

Use this checklist when reviewing an existing implementation:

- Is the stack actually `Next.js App Router + PrimeReact`, without a conflicting UI system?
- Are `Server Components` the default and `"use client"` boundaries kept as small as practical?
- Are PrimeReact providers and CSS imports placed in one shared layout/provider entrypoint?
- Is `src/app` used for route composition rather than storing all business logic?
- Is the repository using `src/resources/<group>` consistently for shared or domain logic?
- Is `Enterprise Structure` used when the project is large enough to justify domain separation?
- Are forms using `react-hook-form + zod` where validation and repeated field handling matter?
- Is the mutation path chosen intentionally between `server actions`, `route.ts`, and external backend services?
- Is data fetching performed on the server by default when interactivity is not required?
- Is cache or revalidation behavior explicit enough to avoid stale-data surprises?
- Are API calls centralized instead of repeated directly in page files?
- Are authentication state, route params, cookies, and temporary session state kept distinct?
- Are backend URLs environment-driven rather than hardcoded?
- Are repeated message, error, and loading patterns extracted into shared modules?
- Are assets, fonts, icons, and theme files loaded locally rather than from remote runtime URLs?
- Are tutorial/demo patterns clearly separated from production patterns?

## แนวทางจากรีโปอ้างอิง (Repo-Derived Guidance)

This skill is grounded in the patterns used by the `next-primereact-web` project:

- Preserve these strengths:
  - App Router route composition in `src/app`
  - PrimeReact provider and theme setup inside route-group layouts
  - separation of models, services, and shared components inside `src/resources/standard`
  - progression from simple React forms to `react-hook-form + zod` and CRUD examples
- Correct these before treating them as standards:
  - hardcoded backend URLs
  - duplicated page-level API calls
  - repeated inline message/error handling
  - tutorial-style manual state used in places where production code should centralize form logic

Simple tutorial chapters may intentionally violate the production rules for teaching purposes. Do not normalize those teaching shortcuts into the standard architecture.

## เอกสารอ้างอิง (References)

- `references/project-structure.md` — primary structure reference and placement rules
- `references/stack.md` — package choices, compatibility, and dependency rules
- `references/security.md` — authentication, token, upload, and dependency security rules
- `references/shared-patterns.md` — shared components and patterns to centralize
- `references/component-selection.md` — PrimeReact-first component selection for form and CRUD screens
- `references/coding-standards-example.md` — Next-specific code examples using the team structure
