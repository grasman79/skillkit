# Motion: Whether and How to Animate

Motion should guide attention, not steal it. Most interfaces are over-animated. This reference covers the tasteful defaults for web marketing/page motion (Supafast's analysis of 200+ SaaS sites) plus mobile motion principles. For **app component** animation (buttons, loaders, toasts, modals, number tickers, spring physics), use the dedicated `ui/animation-patterns` skill - this file is the design judgment about motion, that skill is the component how-to.

Sources: Supafast (web motion rules); Michal Malewicz / Hype4 Academy (mobile depth/motion feel).

## The Animation Test

Every animation must pass all four:

1. Remove it. Does the screen still make sense? If yes, keep considering it.
2. Does it guide the eye toward the primary action? If no, delete it.
3. Does it add more than ~0.5s to perceived load? If yes, delete it.
4. Would anyone notice if it were gone? If no, it was never needed.

## Anti-Patterns (never do these)

- Hero text bouncing in letter by letter.
- Parallax on everything.
- Auto-playing carousels.
- Floating/pulsing CTA buttons.
- Page transitions of 2+ seconds.
- Scroll-jacking (hijacking native scroll).
- Animating every element on scroll.
- Bounce or overshoot easing on functional UI.

## The 5 Places to Animate (web pages)

Everything else stays static.

### 1. Hero - fade in on load
Fade in headline, then CTA. One or two elements, not the whole hero.
- Duration 0.4s ease-out; 0.1s stagger between the two elements.
- CSS: `opacity 0 -> 1` with `translateY(10px) -> 0`. Tailwind: `animate-in fade-in slide-in-from-bottom-2 duration-400`.

### 2. Cards and buttons - hover lift
Subtle lift + shadow increase on hover.
- Duration 0.2s ease. Combine `translateY(-2px)` with `scale(1.02)` - never above 1.05, that looks cheap. Transition `transform` and `box-shadow` together.
- Tailwind: `transition-all duration-200 hover:-translate-y-0.5 hover:scale-[1.02] hover:shadow-lg`.

### 3. Scroll reveals - once
Sections fade up as they enter the viewport, one time.
- `translateY(20px)` max (not 50-100px); duration 0.5s; trigger at 20% visible (IntersectionObserver `threshold: 0.2`); `unobserve` after firing. Never re-animate on scroll up.

### 4. FAQ accordions - expand/collapse
Smooth height + chevron rotate. Duration 0.3s ease-in-out, no bounce. Use the `grid-template-rows: 0fr -> 1fr` trick for smooth height (not max-height hacks); rotate the chevron 180deg.

### 5. Form validation - state cue
Green checkmark fades in on valid (0.2s opacity); red shake on error (3px horizontal, 0.3s). One cue per state change.

### Quick reference (web)

| Element | Duration | Easing | Transform |
|---|---|---|---|
| Hero fade-in | 0.4s | ease-out | translateY(10px) + opacity |
| Card hover | 0.2s | ease | scale(1.02) + translateY(-2px) |
| Scroll reveal | 0.5s | ease | translateY(20px) + opacity |
| Accordion | 0.3s | ease-in-out | grid-template-rows |
| Form shake | 0.3s | ease | translateX(3px) |
| Checkmark | 0.2s | ease | opacity |

## Mobile Motion

Native apps lean on depth and small, purposeful transitions rather than page-load choreography.

- **Depth is the primary "motion" cue.** A shadow that makes a button look pressable does more than an animation (see `color-and-depth.md`). Tie a matching micro-interaction (icon swap, gentle scale) to real state changes only - e.g. the eye icon opening/closing when a password reveals, placed in the exact same spot so the swap reads as a transition.
- **Respect platform expectations.** iOS and Android have native push/modal/tab transitions - use the framework's defaults rather than reinventing them. In React Native/Expo, lean on the navigator's built-in transitions.
- **Reduced motion:** honor the OS "reduce motion" setting; fall back to a plain fade or no motion.
- For the concrete component recipes (spring presets, skeleton shimmer, toasts, number tickers, layout animations, `useReducedMotion` setup), use `ui/animation-patterns`.

## Performance

- Animate only `transform` and `opacity` - they skip layout recalculation. Never animate `width`, `height`, `top`, `left`, or `margin` (use `transform`).
- Use `will-change: transform` sparingly, only on elements about to animate.
- Test on a low-end device and a 4G connection; if it stutters or delays meaningful content, simplify or cut. Check Lighthouse CLS for animation-caused layout shift.

## How to Verify

- Remove all animation - the screen still works.
- Watch someone use it; they should not notice the motion.
- Nothing bounces or overshoots on functional UI; no auto-carousels, no scroll-jacking.
- Only `transform`/`opacity` animate; no jank on a slow device.
- Mobile transitions are the platform defaults; reduce-motion is honored.
