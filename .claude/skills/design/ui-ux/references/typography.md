# Typography (web + mobile)

Typography is hard even after decades of practice. For your first year or two, the winning move is restraint: few fonts, a tight size range, no thin weights. This reference merges web typography rules (Butterick's Practical Typography) with mobile app type rules (Michal Malewicz / Hype4 Academy).

Sources: Butterick's Practical Typography (practicaltypography.com); Michal Malewicz / Hype4 Academy.

## The Constraints That Make You Better

- **One font per product.** Avoid font pairing until you are comfortable. One typeface with weight and size variation carries almost any interface. Only exceed this if a client requires a brand font.
- **Avoid thin and light weights.** Start with Regular for body and a heavier weight (Semibold/Bold) for titles. Thin fonts are hard to read, especially on mobile and on light backgrounds.
- **Bold OR italic, never both.** They are mutually exclusive emphasis tools. In sans-serif, skip italic (the slant is too subtle) - use bold. In serif, italic for gentle emphasis, bold for stronger.
- **Emphasize sparingly.** If everything is emphasized, nothing is. Never emphasize whole paragraphs.

## Size

Different fonts render at different visual sizes at the same point value - always check the actual rendered size, do not trust the number alone.

### Mobile (app UI)

- **Range: 12-40pt.** Most of the interface lives in **12-24**. Reserve **32-40** for onboarding titles or a single very important headline per screen.
- **Body / larger text blocks: 14-16pt.**
- **Never go below 12pt** for readable content - it is hard to read on a phone and looks bad. The only exceptions: tab bar labels can go to **10pt**, and a notification-bubble number can dip to ~10-11pt. Nothing else.
- Fonts render off-center and off-size; you will often set 14 where you planned 12, or 17 to center perfectly in a 56pt button. Adjust visually.

### Web

- **Body: 15-25px.** Good starting points: 18px most fonts, 16px for large x-height fonts (Inter), 20px for small ones (Garamond). Tailwind: `text-base` (16) to `text-lg` (18); use `text-[17px]` if needed. Never below 16px on any device.
- **Headings: smallest increment that creates hierarchy.** If body is 18px, an H3 at 20-22px works. Avoid the web default of headings at 200% of body.

## Line Height (leading)

The single biggest readability lever.

- **Body: 120-145% of font size.** Unitless in CSS: `line-height: 1.4` for body, `1.2` for headings. Tailwind: `leading-snug` (1.375) or `leading-[1.4]`; `leading-normal` (1.5) is usually too loose.
- Longer lines need more line height; larger sizes need proportionally less; dark backgrounds need slightly more.
- On mobile, tighten line height for small stacked text (e.g. a two-line subtitle) so lines do not float apart - bring the blocks close.

## Line Length (measure) - web

- **45-90 characters per line, including spaces.** The #1 web typography mistake is text stretching edge to edge. Constrain with `max-width: 65ch` (Tailwind `max-w-prose`). Maintain the range at every breakpoint.
- Quick test: type the alphabet 2-3 times on a line. Fewer than 2 alphabets = too short; more than 3 = too long.

## Font Choice

- **Serif or sans both work** on modern screens. Quality matters more than category.
- Free web fonts that hold up: **Sans** - Inter, IBM Plex Sans, Source Sans, DM Sans. **Serif** - Source Serif, IBM Plex Serif, Charter, Lora. **Mono** - JetBrains Mono, IBM Plex Mono, Fira Code.
- Mobile: pick from a small safe list and stick with it. (Malewicz's course ships a curated list; any of the above sans faces work.)
- Avoid: Comic Sans, Papyrus, Arial, default Times New Roman.

## Font Pairing (only once comfortable)

- Most projects need at most 2 fonts; many need 1. Almost none can handle 4+.
- Same-designer pairings are reliable (Inter + Inter Tight, IBM Plex Sans + IBM Plex Serif).
- Give each font one role (headings vs body) and never mix roles. Serif + sans is not required - two sans or two serifs can pair well.

## Color of Text

Never pure black on screens (see `color-and-depth.md` for the full rule). Use a dark gray tinted with a hint of the accent color for body text so it blends with the interface. Web quick values: `#374151` / `#1f2937` (Tailwind `text-gray-700`/`800`). Reserve color for clickable elements. Use a lighter shade for secondary text (counts, timestamps, captions) to show hierarchy.

## Headings

- Use the smallest size jump that creates clear hierarchy; limit to 2-3 levels.
- **Space above a heading matters more than its size** - generous `margin-top` makes a heading stand out even without a big size jump.
- Bold, not italic. No underline. No all-caps for headings longer than one line. Suppress hyphenation (`hyphens: none`). Keep headings with their content (`break-after: avoid`).

## Paragraphs (web)

Choose ONE separation method, never both:
1. Space between paragraphs: 50-100% of body size (`margin-bottom: 1em`). Most common on web.
2. First-line indent: 1-4x font size, no space between. For long-form/books. Do not indent the first paragraph.

The `@tailwindcss/typography` `prose` class handles this well out of the box.

## All Caps / Small Caps

Acceptable for less than one line (labels, nav, short headings). **Always add letter-spacing** (0.05-0.12em / Tailwind `tracking-wide`) or caps look cramped. Reduce size slightly. Real small caps need font support (`font-variant: small-caps`); otherwise the browser fakes them badly.

## Letter Spacing

- Kerning always on (`text-rendering: optimizeLegibility`).
- All caps / small caps: +0.05 to 0.12em. Regular lowercase body: leave at 0. Large lowercase headlines: tighten slightly (-0.01 to -0.02em).

## Alignment

- **Left-aligned is the default and safest** for body text, web and mobile.
- **Centered: short blocks only** - headings, hero copy, a short profile bio, a short popup message. Never center body paragraphs or anything past ~3 lines.
- Justified needs hyphenation (`hyphens: auto`) or word spacing gets ugly; browser justification is mediocre - avoid unless you have a reason.

## Aligning Type on the Grid

Text is a rectangle (see `foundations.md`). Align to the **cap height / baseline**, not the bounding box - a lowercase letter with an ascender (k, l) can sit higher than the cap line, so align to the capital letter. When centering text in a button or field, verify with the red square method; expect to nudge 1px because fonts render off-center.

## Responsive Typography (web)

Rules do not change with screen size, only values. Keep 45-90 characters at every breakpoint. Fluid sizing: `font-size: clamp(16px, 1vw + 14px, 20px)`. Mobile screens are held closer, so relatively smaller text still reads - but never below 16px on web / 12pt in native app content.

## Punctuation Quick Reference (web)

| Instead of | Use |
|---|---|
| Straight quotes | Curly quotes |
| Two spaces after period | One space |
| Hyphens for dashes | En dash (ranges) or em dash (breaks) |
| Three periods | Ellipsis character |
| Multiple exclamation marks | One, maximum |

## How to Verify

- Body is 14-16 (mobile) / 15-25px (web); nothing readable below 12pt / 16px.
- Line height 1.2-1.45; not the browser default or word-processor 1.5.
- Web: no line exceeds ~90 characters.
- Headings only slightly larger than body, with more space above than below.
- One font (or a deliberate, role-separated pair). No thin weights.
- Text is dark gray tinted with accent, not pure black. Secondary text is lighter than primary.
- Caps have letter-spacing. Type is centered/aligned to the cap line, verified with squares.
