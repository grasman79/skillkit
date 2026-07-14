# UX Patterns: Adaptive UI, States, Inputs, Flows

Craft makes a screen look right; patterns make the product *behave* right. These are the recurring UX decisions that separate a functional interface from one that feels smart and considerate. All apply to web and mobile.

Source: UX Peak (uxpeak.com).

## Adapt the UI to the User's Journey Stage

Not all users are the same, yet most designs show everyone the identical screen. Meet people where they are.

- **New user** - keep it simple. Welcome, one clear first action (set a goal), a little to explore, nothing overwhelming.
- **Returning user** - skip onboarding; lead with the daily task (today's plan, the thing they came to do), with useful detail.
- **Power user** - go advanced: stats, optimization, personalized recommendations. They do not need help starting; they want to go deeper.

Small adaptations - a personalized greeting, tailored content, progress tracking, a different default screen by stage - make a product feel human and valuable. Design at least the new/returning states explicitly; do not ship one generic screen.

## Design the Empty and Search States (not just the happy path)

The blank state is a design surface, not a dead end. A tapped search bar is a moment of *intent* - the user wants something but may not know exactly what.

- **Don't ship a blank search screen.** On focus, offer: recent searches (pick up where they left off), popular items (what others find useful), personalized suggestions (from past behavior). These support without getting in the way - a user who knows what they want just types past them.
- Apply the same thinking to every empty state (empty list, no results, first run): show a path forward, not a void.

## Match the Input Method to the Task, Not Just the Data

Two numeric inputs can look identical and need opposite controls. What matters is *frequency and precision*, not the data type.

- **One-time, low-effort, known-range setup** (age, height, weight at signup) -> **sliders or scroll wheels.** Fast to get through, no typing, low cognitive effort.
- **Frequent, precise, repeated entry** (logging food quantity, calories, macros) -> **text fields, steppers, or number inputs.** Sliders and wheels are too slow and fiddly when used all day.

Rule of thumb: casual + one-time = slider/wheel; frequent + precise = text field/stepper. Choose to reduce friction for the case that repeats, not for which control looks nicer.

## List and Category Screens Need Hierarchy and Image Consistency

Category and list screens seem trivial but heavily shape navigation. Three quality levels:

1. **Plain text list** - clean, but every row looks identical, so it puts all the scanning effort on the user. No rhythm, no cues.
2. **Image-heavy "junior" version** - looks designed but usually fails: contrast problems (even with a dark overlay, text stays hard to read), and mismatched imagery (one bright, one moody, one magazine-style) that reads as random stock, not one brand. Visually busy = slower to parse.
3. **The good version** - color-coded cards with soft solid backgrounds and clean, stylistically **unified** imagery. Cohesive and branded, with a scannable rhythm you can read in seconds.

Takeaways: give list rows visual hierarchy and cues; keep imagery stylistically consistent (same treatment, same mood); ensure text contrast passes regardless of the image behind it (see `color-and-depth.md`). Vary the real content (different counts per category) so it does not look fake (see `foundations.md`).

## Design the Post-Purchase Moment

Post-purchase experience matters as much as the purchase. Between payment and delivery, the user is full of expectation and uncertainty - great design reduces stress and answers questions before they are asked.

A weak order screen is a data dump: order number, a plain item list, courier text, a row of dates. Everything is present but unstructured, so the user has to interpret it.

An upgraded order screen:

- **A confident status message** up front ("Your order is on the way") with clear delivery details and icons for the time window and address.
- **A humanized courier section** - photo, name, and quick call/message buttons - which adds trust and responsiveness instead of a bare name and number.
- **A visual timeline** for order history (stages), so the user sees the current stage instantly instead of parsing dates.

This is a different experience, not just a nicer UI: it answers "where is my thing and when will it arrive" before the user asks.

## How to Verify

- The first screen differs for a new vs returning user (at minimum).
- No blank search/empty states - each offers a path forward (recent, popular, suggested).
- Input controls match frequency: wheels/sliders for one-time setup, text/steppers for repeated precise entry.
- List/category imagery is stylistically unified; rows have hierarchy and cues; text passes contrast over any image.
- Post-purchase/status screens lead with a clear status, humanize the human elements, and use a visual timeline over a date dump.
