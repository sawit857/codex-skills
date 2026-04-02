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

## Compatibility notes

- PrimeReact `10.9.7` depends on React and React DOM as peer dependencies.
- PrimeReact `10.9.7` also brings `react-transition-group` as a runtime dependency.
- PrimeIcons is required for many PrimeReact components to render icons correctly.
- PrimeFlex is optional; if not used, the project must provide responsive layout via custom CSS.

## Theme and styling decisions

- Use PrimeReact styled mode as the primary visual system.
- Do not add Tailwind CSS when the project has already committed to PrimeReact styled mode.
- Keep theme ownership in one place to avoid specificity and maintenance issues.

## Suggested setup checkpoints

- Import PrimeReact theme resources correctly.
- Import PrimeIcons CSS.
- Initialize app providers needed by PrimeReact.
- Keep Vite dev proxy pointed at backend `/api` routes.
- Ensure production build output is bundled into the WAR when the project uses Tomcat packaging.
