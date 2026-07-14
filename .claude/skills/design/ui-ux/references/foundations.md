# Foundations: Grid, Alignment, Hierarchy

The single most important part of interface design, and the one juniors most often get wrong, is layout. Good layout is mostly aligning simple rectangles on a consistent grid and letting the distances between them create grouping and hierarchy. Master this before anything else.

Source: Michal Malewicz / Hype4 Academy. The "red square method" is his technique.

## Everything Is a Rectangle

Every object on a screen - text blocks, buttons, icons, images, avatars - is a rectangle for layout purposes. A line of text occupies a box. An icon sits in a box. When you think in rectangles, you stop eyeballing positions and start aligning edges. This one mental shift fixes most alignment problems.

## The 8pt Grid

Use a base-8 spacing system. Every size and distance is a multiple of 8 (with 4 as the half-step for tight spots). The working set of spacing values:

**4, 8, 16, 24, 32, 48** (add 56/64 for larger gaps).

- Pick heights and gaps you can divide by 8 wherever possible. Occasionally an element needs an odd value (a 50px button, a 44px touch target) - that is fine, force the exception deliberately, do not let everything drift.
- **Tailwind mapping (web):** `1`=4px, `2`=8px, `4`=16px, `6`=24px, `8`=32px, `12`=48px. The grid *is* the spacing scale - use scale steps, not arbitrary values.
- **React Native (mobile):** points map 1:1 - `padding: 16`, `gap: 24`, etc.

Why base-8: it divides cleanly, scales across screen densities, and removes hundreds of micro-decisions. Constraints make you faster and more consistent.

## The Rule of Proximity

Objects close together are read as a group; objects far apart are read as separate. This is the lever that creates hierarchy without borders or color.

**The core rule: the distance *between* groups must be larger than the distance *within* a group.**

- A label and its input field belong together - small gap (e.g. 8).
- Two separate form fields are different groups - larger gap (e.g. 24).
- The screen margin is a bigger container still (e.g. 32).
- An action (a button) is its own group - separate it from the content above it (e.g. 32-48).

Map spacing values to hierarchy levels: small squares (4, 8) for within-group spacing, large squares (24, 32, 48) for between-group spacing. If two things look wrongly grouped, you almost always have the gap sizes inverted.

## The Red Square Method

An alignment technique: create a set of colored helper squares sized to your grid, and physically place them between elements to set and verify every distance. It turns "does this look about right" into "is the square exactly touching both edges."

### Setup

Create squares at each grid value: **4x4, 8x8, 16x16, 24x24, 32x32, 48x48.** Make them a bright, obvious color (red on most UIs; use a different hue if the design is red). Label each with its size. Keep them off-canvas and drag copies in as needed. (In the design tools, starter files ship with these; in code, they are a mental tool - reason in the same fixed steps.)

### Using them

1. **Margins first.** Place the 32 square against the left screen edge and another against the right. Every element aligns to the inside edge of these - that is your content column.
2. **Set each gap with the matching square.** Drop the 24 between two form groups; nudge the lower group until it touches the square exactly. Drop the 8 between a label and its field. The square either fits perfectly or it does not - no guessing.
3. **Center things both axes.** To center a label inside a button, place a small square between the text and each edge (top/bottom, then left/right). All four gaps on an axis must be equal. This is how you catch text that sits 1px too low - which happens constantly because fonts render off-center.
4. **Verify heights against the grid.** Stack squares to size a photo or card so its height stays on the grid.
5. **Final pass.** Before calling a screen done, run the squares back over every distance. Elements drift when you edit - re-checking catches it. Precision is the job.

### The invisible tap-target trick

Small interactive elements (a "forgot password" link, a close X, an icon button) need a large touch area even when they look small. Wrap the element in a transparent rectangle sized to the touch target (44x44 on mobile), center the visible element inside it, set the rectangle opacity to 0, and group them. The link stays visually small but becomes comfortably tappable. Never place a small tap target right next to a form field it could be mis-tapped for - give it its own 44pt row.

## Block Framing

Before committing to real content, rough the whole screen as flat colored rectangles - one per text block, image, button, control - at exact grid sizes. This is not wireframing; it is precise. It forces you to solve layout, grouping, and hierarchy without the distraction of real copy or images. Then raise fidelity: swap rectangles for real elements, keeping every position. Because you thought in rectangles from the start, the real screen inherits the alignment.

A practical fidelity ladder for a card:
1. Background rectangle on the grid, corners rounded to your system radius (see `color-and-depth.md`).
2. Photo rectangle - round only the top corners if it sits flush at the card top.
3. Text blocks as rectangles at the intended font heights (a 24px title block, a 16px subtitle block).
4. Button rectangle with its label rectangle centered inside.
5. Apply real content and colors last.

## Hierarchy

Hierarchy is what the eye reads first, second, third. Build it with three tools, in this order:

1. **Space (proximity).** Grouping via distance does most of the work. Get this right first.
2. **Size and weight.** Bigger and heavier reads as more important - titles larger/bolder than body. Do not overdo it (see `typography.md` for the restrained scale).
3. **Color.** Lighter/more-muted text recedes; the accent color pulls the eye to actions. Secondary info (item counts, timestamps) goes a step lighter than primary (see `color-and-depth.md`).

If a screen feels flat or confusing, it is almost always a hierarchy problem - usually spacing, not size or color. Fix the gaps before you touch font sizes.

## Content Realism

Never lay out with Lorem Ipsum or repeated placeholder numbers. Fake content produces lazy, unrealistic layouts and hides real problems (a title that wraps, a price that is too long, five identical "13 items" rows that look obviously fake). Write content that actually fits the product, and vary it - different names, different prices, different counts. People subconsciously notice when content is fake.

## Quick Checklist

- Every distance is a grid value (4/8/16/24/32/48), set with the red square method.
- Between-group gaps are larger than within-group gaps (proximity holds).
- Screen margins are consistent (commonly 32 on mobile) and everything aligns to that column.
- Button labels are centered on both axes (verified, not eyeballed).
- Small tap targets have an invisible 44pt hit area.
- Content is real and varied, not placeholder.
- You re-ran the squares after editing - nothing drifted.
