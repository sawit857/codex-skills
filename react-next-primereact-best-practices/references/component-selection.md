# PrimeReact-first component selection for Next.js projects

Use this file before writing custom UI. Default to PrimeReact when it already covers the need.

## Decision order

1. Search PrimeReact first.
2. Customize PrimeReact if it is close but not exact.
3. Write custom UI only when PrimeReact does not cover the need.
4. Do not create thin wrappers that only rename a PrimeReact component.

## Common form and CRUD components

| Need | Prefer |
| --- | --- |
| Text input | `InputText` |
| Numeric input | `InputNumber` |
| Date input | `Calendar` |
| Single select | `Dropdown` |
| Toggle options | `SelectButton`, `RadioButton` |
| Submit and action buttons | `Button` |
| Inline validation or status | `Message` |
| Toast-style feedback | `Toast` |
| Grouped list or table | `DataTable` + `Column` |
| Card section | `Card` |
| Section separators | `Divider`, `Fieldset`, `Panel` |

## Form guidance

- Use `register` for PrimeReact inputs that work with standard input props.
- Use `Controller` for components with custom value or event signatures such as `Calendar`, `Dropdown`, `InputNumber`, and similar widgets.
- Keep validation with `react-hook-form + zod`.

## CRUD guidance

- Use `DataTable` for tabular data when sort, filter, or repeated row actions matter.
- Keep query and edit concerns split when the screen grows.
- Reuse shared message, error, and field layout patterns instead of recreating them per CRUD page.

## Custom component rule

Write custom components only when PrimeReact cannot express the need. When custom code is necessary, keep visual decisions aligned with PrimeReact theme tokens and the project’s chosen spacing/layout strategy.
