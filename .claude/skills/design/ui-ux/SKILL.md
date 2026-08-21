---
name: ui-ux
description: Master UI/UX design skill for web and mobile apps. Use when designing or reviewing any interface - layout, spacing, hierarchy, typography, color, motion, screen states, navigation, or conversion. Covers the visual craft (grid, proximity, type, color, depth), UX patterns (adaptive UI, empty states, input methods, post-purchase), conversion/decision design (framing, anchoring, trust), platform conventions (iOS/Material for mobile, responsive/hover for web), and pulling real style references (Refero Styles) for a niche before designing. Trigger words - UI design, UX design, interface design, layout, spacing, visual hierarchy, typography, font sizes, line height, color palette, contrast, accessibility, shadows, animation, motion, micro-interactions, empty state, loading state, onboarding screen, app screen, mobile app design, screen design, redesign a screen, make this look better, design review, touch targets, tab bar, navigation design, paywall, pricing screen, product page, conversion, why isn't this converting, website examples, design examples, style reference, show me examples.
---

# UI/UX Design

The single entry point for interface design across web and mobile. It teaches how to *think* like a designer - precise layout, clear hierarchy, tasteful restraint - and then routes to the right reference for the specific decision.

This skill covers the "what good looks like and why." For the "how to build it in our stack," it points at the technical skills: `ui/shadcn` and `ui/animation-patterns` (web components), `platform/expo` and `platform/react-native` (mobile). It never duplicates their component or config docs.

## When to Use This Skill

- Designing a new screen, page, or flow (web or mobile)
- Reviewing an existing interface for quality ("make this look better," "why does this feel off")
- Any decision about spacing, alignment, hierarchy, type, color, shadows, or motion
- Choosing a UX pattern (search, empty state, input method, navigation, post-purchase)
- Improving conversion on a paywall, pricing page, product page, or booking flow
- Deciding whether something should be animated at all

## Our Stack (read before implementing)

- **Web:** ShadCN + Tailwind + React. The 8pt grid maps directly onto Tailwind spacing (`2`=8px, `4`=16px, `6`=24px, `8`=32px, `12`=48px). Render patterns as ShadCN primitives - see `ui/shadcn`.
- **Mobile:** React Native + Expo. Design in points; the mobile rules here map 1:1 to RN units. See `platform/expo`.

## Style Reference Search (Refero)

Before designing a new screen/page from scratch, or when the user wants to see real examples for a niche or vibe ("show me some real estate website examples," "what do fintech landing pages look like"), search [Refero Styles](https://styles.refero.design) - a curated library of 2,000+ AI-readable `DESIGN.md` design systems (colors, typography, spacing, components) extracted from real product websites.

**Query pattern:** append the search term to the URL as a `q` param, spaces as `+`:

```
https://styles.refero.design/?q=real+estate
https://styles.refero.design/?q=fintech
https://styles.refero.design/?q=saas+dashboard
```

Fetch the URL, present the matching styles/sites to the user, and pull the actual `DESIGN.md` for whichever one they pick (or the closest match to the brief) as a concrete style anchor before starting layout work - real extracted systems (actual color values, actual type scales) beat inventing a palette from a vague brand-vibe description. Use this as a starting reference, not a template to copy wholesale - still apply this skill's own craft/pattern/conversion judgment on top of whatever style gets pulled in.

## Design Philosophy: Build Experiences, Not Screens

Before the tactics, the frame. The apps that survive don't win on features - competitors copy features in a weekend. They win on experiences that compound over years. Three meta-patterns (Sips):

1. **The Human pattern - your data isn't always the product.** Sometimes the user's *emotional read* of the data is the product. Carrot Weather shipped the same forecast data as Apple Weather, but the headline was a character reacting to the weather - you felt the forecast before reading a number. Put personality at the *core surface*, not bolted onto the edges as confetti and mascots.

2. **The Behavior pattern - put the stake on screen, not the metric.** For habit, focus, and fitness apps, don't just track the behavior; make its *absence* cost something the user feels. Forest grows a tree during a focus session and kills it if you leave - the dying tree does the discipline work. A visible stake, a streak that breaks, a status that drops. (Deeper mechanics live in `platform/app-onboarding` and the copy skills' psychology.)

3. **The Craft pattern - when the data layer is commoditized, enrich around it.** Day One charged for journaling against free Apple Notes by surrounding the same text with auto-captured weather, location, a photo, a timeline, real search. Moleskine vs IKEA notebook - same paper, considered details. Craft is the work users won't see but will feel: animation timing, font weight, metadata in the right place.

Keep this as the north star. Everything below is how you actually earn it.

## The Three Tactical Pillars

Load the reference for the decision in front of you. Do not read them all at once.

## Scoped Redesign Triage (limited budget/time)

When the ask is "make this app feel significantly better" but a full redesign isn't in scope, don't spread effort evenly across every screen - a handful of screens carry disproportionate weight on how polished the whole app feels. Prioritize in this order:

1. **Onboarding** - decides whether the user understands the product and wants to continue. Keep it short and personal; show value instead of explaining everything; ask only questions that visibly change what comes next. See `platform/app-onboarding` for the full screen-by-screen blueprint.
2. **Home screen** - should make the next action obvious, not showcase every feature at once. Decide what matters most to this user and build the hierarchy around that single thing - it should feel like the app knows why they came, not like a dashboard. See `references/foundations.md` for hierarchy and `references/patterns.md` for adaptive-by-journey-stage guidance.
3. **The core action screen** - every app has the one thing people actually open it to do (booking, tracking, creating, generating, messaging). Identify it and spend the most effort simplifying it: cut unnecessary taps, choices, and steps. If the core action feels effortless, the whole product reads as polished, disproportionate to how much else got touched.
4. **The paywall** - explain the outcome the upgrade produces, not just a feature list against a price. See `references/conversion.md` for framing/anchoring tactics, and note that *when* the paywall shows is often as consequential as how it's designed - see `platform/app-onboarding`'s permission/paywall sequencing.
5. **Empty, success, and progress states** - the easiest screens to skip, and the ones that most reveal whether an app was actually finished with care. Replace "no data yet" with a path forward; acknowledge completions; show progress. See `references/patterns.md` for the empty-state pattern.

A full redesign still wins when it's in scope - it fixes the experience consistently across every screen and flow rather than in five high-leverage spots. But for a scoped engagement, this order is where the visible improvement per hour of work is highest.

### Pillar 1 - Craft (visual precision)

The foundation. Most junior work fails here, on layout and spacing, not on ideas.

| Reference | Use when |
|---|---|
| [`references/foundations.md`](references/foundations.md) | Laying out any screen - the 8pt grid, the red square method, block framing, the rule of proximity, hierarchy. **Start here for any new screen.** |
| [`references/typography.md`](references/typography.md) | Font sizes, line height, line length, weights, pairing, hierarchy - web and mobile scales. |
| [`references/color-and-depth.md`](references/color-and-depth.md) | Building a palette, avoiding clashing colors, contrast/accessibility, shadows, gradients, elevation. |
| [`references/motion.md`](references/motion.md) | Whether and how to animate - the tasteful defaults, anti-patterns, exact durations. |

### Pillar 2 - Patterns (UX behavior)

How screens adapt, guide, and flow.

| Reference | Use when |
|---|---|
| [`references/patterns.md`](references/patterns.md) | Adaptive UI by user journey stage, search/empty states, input-method selection, list/category screens, post-purchase/order-tracking. |

### Pillar 3 - Conversion (decision design)

The interface expression of persuasion. Every element asks the user a question; the question decides whether they act.

| Reference | Use when |
|---|---|
| [`references/conversion.md`](references/conversion.md) | Paywalls, pricing, product pages, booking, checkout. Framing, anchoring, specificity, trust, and the component tactics (badges, swatches, selection cards, progressive disclosure). Cross-links the copy skills for the underlying psychology - do not re-derive it here. |

### Platform layers

Load the one that matches the target after the pillars.

| Reference | Use when |
|---|---|
| [`references/mobile.md`](references/mobile.md) | Mobile-specific: 44pt touch targets, tab bars, safe areas, iOS HIG vs Material, navigation patterns. Pairs with `platform/expo`. |
| [`references/web.md`](references/web.md) | Web-specific: click targets, top nav, responsive breakpoints, hover/focus states. Pairs with `ui/shadcn`. |

## The One Habit That Matters Most

Whatever the platform, design in aligned rectangles on an 8pt grid, and let the distance between things do the grouping. Precision is the job. Before calling any screen done, run the red square method over it (see `foundations.md`) and check contrast (see `color-and-depth.md`).

## Sources

This skill consolidates and credits several third-party sources. Full attribution in the repo README's Credits section.

- **Craft (grid, red square method, block framing, mobile conventions):** Michal Malewicz / Hype4 Academy - [hype4.academy](https://hype4.academy). The "red square method" is his.
- **UX patterns and conversion/decision design:** UX Peak - [uxpeak.com](https://uxpeak.com).
- **Design philosophy (experiences-not-screens meta-patterns):** Tim, Sips (Sips App).
- **Web typography rules:** Butterick's Practical Typography - [practicaltypography.com](https://practicaltypography.com).
- **Web motion rules:** Supafast - [withsupafast.com](https://withsupafast.com).
