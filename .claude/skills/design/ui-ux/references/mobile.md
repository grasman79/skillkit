# Mobile Conventions

The mobile-specific layer on top of the universal foundations. Touch, screen chrome, and navigation differ from web; everything in `foundations.md`, `typography.md`, `color-and-depth.md`, and `motion.md` still applies underneath. Our mobile stack is React Native + Expo - for the technical implementation of any of this, see `platform/expo` and `platform/react-native`.

Source: Michal Malewicz / Hype4 Academy.

## Touch Targets

- **Minimum 44x44pt** for anything tappable - that is the comfortable finger size. A 40x20 button frustrates users.
- **Buttons: 44-60pt tall.** 48 is a safe default, 56 is generous. Below 44 or above 60 tends to feel wrong.
- **Icons fit within 44x44 and should not exceed half of it** (~16-24pt visible), with rare exceptions.
- **Invisible hit area for small controls:** wrap a small link/icon in a transparent 44pt rectangle and center the visible element in it (see the tap-target trick in `foundations.md`). Never put a small tap target within a finger's width of a form field it could be mis-tapped for - give it its own 44pt row.
- When in doubt on mobile, make things bigger and spacing larger - it is the safer error.

## Screen Chrome and Safe Areas

A phone screen has reserved zones. Build a helper group (call it "OS helpers") to keep them straight, then hide it:

- **Status bar** (top, ~44pt on modern iPhones) - do not place content in it.
- **Nav bar** (below the status bar, ~44-64pt) - screen title, back arrow, top actions.
- **Tab bar** (bottom) - primary navigation.
- **Home indicator / safe area** (very bottom, ~22-34pt) - leave clear; do not put tappable content there.

Design to the safe content column between these. In React Native/Expo use `SafeAreaView` / safe-area insets rather than hardcoding - device notches and indicators vary.

Standard side margin on mobile is commonly **32pt**; align all content (fields, buttons, titles, list content) to that column.

## Navigation Patterns

### Tab bar (preferred, top-level navigation)

- Bottom bar, always visible, **2-5 tabs.** One tab means you need no nav; more than five usually means a flow problem - if you cannot condense to five, reconsider structure.
- Each tab = icon + label. Label may go to **10pt** (the one place below 12 is allowed) in a medium weight - bold looks bad here.
- The tab bar is the **highest level of navigation**. Top-level tab screens have **no back arrow** (there is nowhere above them). Deeper screens (a category, a detail page) get a back arrow and no tab bar. Only your top-level screens (five, if five tabs) show the tab bar.
- Selected state: color the active icon (often the accent, or white on a colored pill) and its label; mute the inactive ones (desaturated accent). A small pill/indicator behind the active item works well.
- Sizing tabs: divide the screen width by tab count; if it does not divide evenly, round the tab width down and distribute the remaining points as equal gaps (e.g. on a 390pt screen, four tabs at 96pt = 384, leaving three 2pt gaps).

### Hamburger menu (avoid when possible)

Hamburger menus are one of the weakest patterns: poor discoverability (users may not know the menu exists) and they hide the app's structure behind a tap. They are mostly used to cram in more than five destinations - which is the smell of a flow that should be simplified. Design one only when genuinely necessary; otherwise use tabs. If you must: keep menu items on a 48pt touch height, icons in a consistent safe box (24pt box, ~16pt icon), and put logout at the bottom (and ideally not in the menu at all, to avoid accidental taps).

## Platform Conventions: iOS vs Android

- Know the platform defaults even when you deviate. iOS commonly centers the nav-bar title; Material (Android) left-aligns it. iOS uses the system share icon, back chevron, and specific transition feels; Material uses its own elevation, FAB, and ripple.
- You do not always have to follow platform rules (e.g. left-aligning an iOS title for a custom look), but decide deliberately - know the convention before breaking it.
- Use the framework's native components and transitions (Expo Router stack/tab navigators, native modals) rather than reinventing them - they carry the right platform feel for free. For the SwiftUI/Jetpack Compose native-UI layer, see `platform/expo` (Expo UI).

## Icons

All icons from **one set**, with consistent corner radius and stroke weight. Mixing sets (even similar ones) reads as inconsistent. Recolor icons to the app's desaturated accent for consistency rather than leaving them default black.

## Lists and Cards (mobile)

- **Tile lists** (bordered/elevated rows) suit lighter content; **full-width separator lists** (edge-to-edge, hairline dividers, larger internal margins ~24/32) fit denser, data-heavy screens.
- **Cards** foreground the photo - bigger imagery, minimal text - for browse/discovery. Compute card width from the grid: screen width minus both margins minus the inter-card gap, split by column count (e.g. 390 - 32 - 32 - 24 = 302, /2 = 151pt cards).
- Let a card row peek off the screen edge to signal horizontal scroll; use a smaller inter-item gap (8-16) when you want to reveal the next item.

## Screen States and Flows

Design the full flow, not just the success screen. For a login/registration flow that means: empty, filled/validated (green field + checkmark), password strength (weak/strong indicators), error (red field + message), and edge cases (account exists, passwords do not match). For a product it means loading, empty, and error states, not only the populated view. Reuse components across states (see `foundations.md` on block framing) so the flow stays consistent and fast to build.

## How to Verify

- Everything tappable is >= 44pt; small controls have invisible 44pt hit areas.
- Content respects status bar / nav bar / tab bar / home-indicator safe areas (via safe-area insets, not hardcoded).
- 2-5 tabs; top-level screens have no back arrow; deeper screens have back arrow and no tab bar.
- One icon set, consistent stroke/radius, recolored to the accent.
- Platform title alignment / controls are a deliberate choice; native navigators used for transitions.
- The whole flow is designed - empty, filled, error, edge cases - not just the happy path.
