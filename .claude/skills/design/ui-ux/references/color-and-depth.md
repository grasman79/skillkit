# Color, Contrast, and Depth

Color, like typography, is hard, and it is where junior work most often goes wrong - too many colors, too saturated, or too pale to read. The fix is the same as everywhere else: constraints. Depth (shadows, gradients, elevation) is the finishing layer that makes buttons feel tappable and cards feel real.

Sources: Michal Malewicz / Hype4 Academy (palette, shadows, gradients); accessibility per WCAG.

## The Starter Palette: Three Colors

Your first apps should use exactly three:

1. **One accent color** (e.g. blue, green) - the brand color, used for primary actions and emphasis.
2. **White (or a near-white background).** A very light, slightly-off-white version of the accent (HSL lightness 80-90, saturation ~80 gives a bright, barely-tinted white) makes the whole app feel cohesive.
3. **A dark gray tinted with the accent** for text - never pure black.

Only once that feels comfortable, add a **second accent** for a specific job (e.g. a "cart"/notification color distinct from the primary action). Keep the second accent near the first on the wheel and reserve it for one consistent purpose across the app.

## Never Pure Black

Pure `#000` text on a light screen is harsh - screens emit light and the contrast is jarring, and pure black never appears in a colored interface so it stands out as foreign. Use a dark gray with a hint of the accent hue. In HSL: saturation ~20, lightness ~40 gives a readable near-black that blends. Web quick values: `#374151` / `#1f2937`.

The same logic applies to borders, icons, and shadows: derive them from the accent, not from neutral gray or black, so every element shares a common undertone.

## Deriving the Palette (HSL)

Work in HSL - it makes systematic variation easy.

- **Accent:** pick one safe color. Keep saturation in the middle 70-80% of the range at first; extreme saturation clashes and pale pastels fail contrast.
- **Background:** same hue, lightness 80-90, saturation ~80 - a bright off-white.
- **Field borders / subtle lines:** accent hue, saturation dropped to ~20, lightness raised to ~80 (60-70 for higher contrast).
- **Body text:** accent hue, saturation ~20, lightness ~40.
- **Secondary text:** same, lightness raised ~20 points (to ~60) so it recedes.
- **Secondary/ghost buttons:** accent hue at high lightness (~80), or a slightly saturated gray.

To make a variant, drag one HSL slider and leave the others - e.g. shift only lightness for a lighter tint, or only hue for the second accent. To make a "wrong" state (error), drag hue toward red without touching saturation/lightness, then fine-tune.

## Avoid Clashing Colors

Some color pairs vibrate at their shared edge - the border between them looks fuzzy (classic: saturated red next to saturated green or blue). If two colors clash, do not use them adjacent. Stay in the middle 70-80% of the spectrum until you are experienced; experts can make almost any color work, but it is not easy, so take the safe path first.

Helper squares (red square method) are intentionally the most-saturated, most-clashing color on purpose - so they are unmistakably visible over the UI and you never confuse them with real elements. That is the one place clashing is a feature.

## Corner Radius Is a System Decision

Pick one radius scale up front and apply it consistently:

- **0** for sharp, **4** for subtle, **8** for friendly, **16+** for soft/playful. For a first app, keep it low (0-8).
- Nested elements get a smaller radius than their container (a card at 8, its inner button/photo at 4) so the inner corner sits nicely inside the outer.
- When a rounded photo sits inside a rounded card, reduce the photo's mask radius (e.g. card 8, photo mask 2-4) so the white gap around the photo stays visually uniform on the diagonal.
- **Avoid pill shapes early:** fully-rounded buttons force fully-rounded fields, and a top-aligned field label no longer belongs to any edge, so it looks off. Sharp or 0-8 avoids this.

## Depth: Shadows

A shadow is what makes a rectangle read as a *button* rather than a block. The default tool shadow always looks bad - build it deliberately.

**Rules for a good shadow:**

1. **Color from the accent, never black/gray.** Use the color picker on the button's own color, then darken (drop lightness ~35) so the shadow feels like it comes from the element.
2. **Blur is at least 2x the Y offset.** This is the core ratio for a soft, natural shadow.
3. **Small button shadow:** Y = 4, blur = 8, spread = -2. **Larger/floating:** Y = 8, blur = 16, or Y = 16, blur = 32 for a hero/logo that should feel lifted.
4. **Negative spread** pulls the shadow under the element so it does not leak past the edges.
5. **Low opacity** (~40-60%). Tweak lightness/saturation of the shadow color until it reads as depth, not a gray smudge.
6. **X offset stays 0** in almost all cases - light comes from above.
7. Smaller shadow means darker/more-visible: when you shrink Y and blur, drop the shadow lightness a bit so it stays perceptible.

Copy the shadow style once it is right and paste it onto every element of the same type (all primary buttons, all cards) for consistency. In code, this becomes one reusable token/variant.

**Web (Tailwind/CSS):** the same principles - prefer a colored, layered shadow over the flat default `shadow`. For app component elevation specifics, see `ui/animation-patterns` and `ui/shadcn`.

## Depth: Gradients

- Use a **subtle** gradient on primary buttons/logos for a slightly more real feel. Pick the same color on both ends, then drop one end's lightness ~3-4 points and raise the other ~3-4 points. Anything stronger takes over the screen.
- **Diagonal** reads more natural than top-to-bottom: darker in the bottom-left, lighter in the top-right. The general rule: darker toward the bottom, lighter toward the top (that is how the brain expects light).
- A near-invisible background gradient (white to the very light accent, top to bottom) adds life without drawing attention.

## Photos and Color Matching

When a photo carries a strong color, add a very low-opacity accent-color fill over it (or over its mask, ~5-30%) so it shares the app's undertone and does not clash. For profile/avatar imagery, a light accent-tinted overlay makes mismatched user photos feel consistent.

## Contrast and Accessibility (do not skip)

Before shipping any screen, check contrast on everything important.

- Pick the background color, then the foreground (text/icon) color, and run the pair through a contrast checker (WebAIM's, or a design-tool plugin). Aim for **WCAG AA or better**. Below AA, adjust the saturation/lightness of at least one color.
- Check button-label color against the button background the same way.
- Decorative or subtle borders (e.g. a faint card outline) do not need to pass, as long as the card's structure is clear without them.
- **Readability pairs with contrast:** essential text stays >= 12pt (>= 10pt only for tab labels), no thin/light weights (see `typography.md`).

## How to Verify

- Three colors max to start (accent, tinted-white bg, accent-tinted dark-gray text); a second accent only for one consistent job.
- No pure black anywhere; text and shadows carry the accent undertone.
- No adjacent clashing colors; saturation in the safe middle range.
- One corner-radius system, nested radii step down, no accidental pills.
- Shadows are accent-colored, blur >= 2x Y, negative spread, low opacity - and consistent across same-type elements.
- Gradients are subtle and diagonal, darker at the bottom.
- Every important foreground/background pair passes WCAG AA.
