# Web Conventions

The web-specific layer on top of the universal foundations. The craft rules (grid, proximity, hierarchy in `foundations.md`, plus `typography.md`, `color-and-depth.md`, `motion.md`) are the bulk of web design and apply directly. This file covers what differs on web: pointer input, larger canvas, navigation, responsiveness, and interaction states. Our web stack is ShadCN + Tailwind + React - render these as ShadCN primitives; see `ui/shadcn`.

Note: this layer leans on the universal foundations plus the web-native patterns/conversion material (much of UX Peak's examples are web e-commerce and SaaS). It is principles-first; deepen it if a dedicated web-layout course is added later.

## The Grid on Web

- The 8pt grid maps to Tailwind spacing: `1`=4px, `2`=8px, `4`=16px, `6`=24px, `8`=32px, `12`=48px. Compose spacing from scale steps, not arbitrary pixel values - it keeps a large, multi-section page consistent.
- Web pages are wider than the content should be. Constrain reading and content columns (`max-w-prose` ~65ch for text; a container max-width like `max-w-6xl` for page sections) and center them. The #1 web layout mistake is letting content stretch full-bleed on large screens.
- Use a real column grid for multi-column layouts (CSS grid / Tailwind `grid-cols-*`) and keep gutters on the spacing scale.

## Input: Pointer, Not Finger

- A mouse is precise, so click targets can be smaller than mobile's 44pt - but keep **at least ~24px**, and honor 44px for touch-screen laptops and accessibility. Do not make tiny click targets just because you can.
- **Hover and focus states are web-only and mandatory.** Every interactive element needs a visible hover state (see the card/button hover in `motion.md`) and a clear keyboard-focus ring. ShadCN components ship these - do not strip them. Never rely on hover alone to convey information (it does not exist on touch).
- Cursor affordance: `cursor-pointer` on clickable non-link elements; do not fake links on non-clickable text.

## Navigation

- **Top navigation** is the web default (the mobile bottom tab bar is not a web pattern). A horizontal nav with the wordmark left, primary links center/left, and the main action right covers most apps and marketing sites.
- Keep primary destinations visible; use a menu/disclosure only for secondary items. On small breakpoints, collapse to a menu - but that is a responsive fallback, not the desktop default.
- For app shells, a left sidebar is the common pattern for many destinations; keep it to a scannable, grouped list.

## Responsive Design

- **The rules do not change with screen size, only the values.** Maintain hierarchy, proximity, and 45-90 character line length at every breakpoint.
- Design mobile-first, then add breakpoints (Tailwind `sm md lg xl`) that reflow multi-column layouts to single-column and adjust type/spacing - do not just shrink a desktop layout.
- Constrain with `max-width` so content never stretches on wide screens; let it fill naturally on narrow ones.
- Test the real breakpoints, not just the extremes - the awkward middle (tablet) is where layouts break.

## Screen States (web)

Design loading, empty, error, and success, not only the populated view. Use skeletons for loading and helpful empty states with a path forward (see `patterns.md`). For the component-level motion of these (skeleton shimmer, toasts, modal/dropdown entrance), use `ui/animation-patterns`.

## Rendering in the Stack

Map the patterns and conversion tactics onto ShadCN primitives rather than hand-rolling:

- Selection cards / options -> Card + RadioGroup; visible option swatches -> ToggleGroup.
- Just-in-time reassurance -> Tooltip / HoverCard.
- Tiered disclosure -> Accordion / Collapsible.
- Forms -> Form + Input with visible labels, inline validation, and the field states from `patterns.md`.

Keep component setup and configuration in `ui/shadcn`; this file stays about the design decision, not the install.

## How to Verify

- Spacing uses the Tailwind scale; content columns are max-width-constrained and centered, never full-bleed on wide screens.
- Every interactive element has visible hover AND keyboard-focus states; no information conveyed by hover alone.
- Top nav (or grouped sidebar) with primary destinations visible; menus only for secondary/responsive collapse.
- Layout reflows at real breakpoints to single-column; line length stays 45-90 chars throughout.
- Loading/empty/error/success states are all designed; components are ShadCN primitives.
