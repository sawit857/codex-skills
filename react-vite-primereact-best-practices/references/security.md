# React + Vite + PrimeReact Security Reference

## Token Boundaries

Keep these roles separate:

- authentication token
  - used for protected API calls
  - usually sent as `Authorization: Bearer <token>` when required by backend contract
- route parameter or navigation token
  - used to open or resolve UI flow context
  - do not treat it as a replacement for bearer auth
- temporary session identifier
  - used only for short-lived flow or recovery state when the application explicitly needs it
  - keep it separate from authentication state

## PrimeReact Boundary

PrimeReact is a presentation and interaction layer. It must not silently redefine:

- auth storage strategy
- token passing rules
- upload contract
- error mapping contract
- request headers required by the backend

## Network-Capable Components

Treat components with built-in request behavior carefully.

Examples:

- upload widgets
- lazy-loading tables with server callbacks
- components that assume a default request format

For these components, verify:

- request path and method match backend contract
- authorization headers are attached correctly
- multipart or JSON payload shape matches backend expectation
- error responses are mapped without leaking internal details

## Dependency Security Checks

Before merge:

- pin exact versions in `package.json`
- commit lock file
- run `npm audit`
- review direct and transitive dependencies, especially `react-transition-group`
- avoid stacking multiple large UI libraries together without explicit need

## Sensitive Data Rules

- do not log tokens, passwords, OTPs, or secret values in the browser console
- do not persist sensitive values unless the backend contract explicitly requires it
- prefer short-lived in-memory flow state when possible
- keep route-bound tokens or session identifiers only for the flows that explicitly require them

## Offline-Only Asset Loading

The client browser may not have outbound internet access. All assets must be self-contained.

- do not load any script, stylesheet, font, or icon from an external CDN or remote URL
- do not use Google Fonts, unpkg, cdnjs, jsdelivr, or similar services
- PrimeIcons and all icon sets must be imported from locally installed npm packages
- font files must be hosted inside the project (`public/fonts/` or bundled via CSS)
- after build, verify the output directory has no references to external domains
- treat any external runtime asset loading as a security and availability risk
