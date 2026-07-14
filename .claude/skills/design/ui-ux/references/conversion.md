# Conversion and Decision Design

The interface expression of persuasion. Same product, same price, same traffic - a paywall or product page can convert at 2% or 7% depending entirely on how the screen is framed. The organizing idea: **every element on the screen asks the user a question, and the question you ask determines whether they act or hesitate.**

Source: UX Peak (uxpeak.com).

## Important: this cross-links the copy skills - do not re-derive the psychology

The underlying persuasion mechanics (anchoring, specificity, social proof, framing, the belief-shift model) are owned by the copywriting system. Use those for the *why* and for writing the actual words:

- `copy-chief` - diagnose why copy/a screen is not converting; the proof modalities.
- `research-b2c-buyer-persona` / `research-market` - the awareness stage and objections that decide the framing.
- `copy-facebook-ads`, `copy-saas`, `copy-homepage` - the copy blocks themselves.

This file is only the *interface* layer: how these ideas render as screen structure and components. Keep persuasion doctrine in the copy skills.

## Ask an Easy Question, Not a Hard One

Every screen implicitly asks the user something. Hard questions get "I'll think about it later" (= never); easy questions get fast yeses.

- A paywall headlined "Get access - $19/mo" asks *"Is this worth $19 a month?"* - brutal for someone 30 seconds into the app who has felt no value. Reframe to *"How your free trial works"*, which asks *"Can I try this free?"* - an obvious yes. Same price, same trial, easier question.
- A ride screen showing price *ranges* ("$13-17") asks *"How much am I willing to risk?"* (the brain anchors to the high number and multiplies uncertainty across options). Showing one number per option asks *"Which one do you want?"* - evaluative ease, a 2-second comparison.

Before finalizing any decision screen, name the question it asks. If it is hard, reframe.

## Show a Timeline, Not a Feature List (trials)

For a free trial, a 3-step timeline beats feature bullets:
- **Today** - you unlock everything.
- **Day 5** - we remind you the trial is ending.
- **Day 7** - first charge; cancel anytime before.

The day-5 reminder line does more work than any feature bullet: proactively revealing the upcoming charge triggers **transparency bias** - a company that reveals a downside is trusted more, not less. The best paywall makes the user feel safe, not sold.

## Words Carry Weight

- **"Start" beats "Subscribe."** "Subscribe" = recurring, locked-in; "Start" = a beginning. "Start my free trial" (note "my," not "your" - ownership before the tap).
- **Specificity is trust.** "Start in two taps" beats "quick setup"; "delivery in 23 minutes" beats "fast delivery." A number does the convincing and kills uncertainty. Round numbers (100, 500) feel like estimates or fakes - "4.9 stars, 221 reviews" feels authentic.
- **Journey copy beats transactional.** "Add to cart - start my journey" softens the spend vs a bare "Add to cart."
- **Micro-labels do the thinking.** A small "cheaper" or "-31%" badge categorizes the option as the smart choice before the user reasons it out.

## Show What They Get, and Anchor in Their Favor

- **Show the product, not decoration.** Real game characters / the mixed drink in a glass beats abstract hero art - "you can't commit to something you can't visualize." Close the **imagination gap**: for anything physical, show it in use, not isolated on a plain background.
- **Anchor with a reference price.** A crossed-out higher price next to the real one, plus a "-31%" badge, makes the same number feel like a deal.
- **Status badges (halo effect).** A "bestseller / top-rated" badge above the title frames perceived value before any detail is read.
- **Reduce mental math.** Day names ("Friday, March 28"), a night count ("5 nights"), the total on the button ("Reserve - $445 total"), and a free-cancellation line answer the top objection before it is asked.

## Component Tactics (how these render)

- **Swatches beat dropdowns.** Surface options as visible swatches with icons; a dropdown forces click -> scroll -> read just to see basic choices.
- **Selection cards beat radio buttons for upsell.** Two equal radio buttons default users to the lowest-risk option. A pre-selected, gently tinted card with a "most popular" tag and the reassurance *inside* it (save 15%, cancel anytime) guides the choice.
- **Progressive disclosure for tiers.** Keep the initial UI clean; when the user picks the cheaper option, expand to reveal bundle tiers (1/2/3-month at increasing discount). Rewards the click, incentivizes the larger purchase, preserves autonomy.
- **Micro-interaction as just-in-time reassurance.** A hover/press tooltip answering the specific fear ("light, tart, not overly sweet") at the exact moment of hesitation.
- **Specific trust badges beat generic.** "Third-party tested for heavy metals," "60-day guarantee" answer real unspoken questions; "free shipping" is expected and ignored.

(For building these in the stack: `ui/shadcn` on web - Card, RadioGroup, ToggleGroup, Tooltip; a pressable + tinted card on mobile via `platform/expo`.)

## How to Verify

- Name the question each decision screen asks; the hard ones are reframed to easy ones.
- Trials show a transparency timeline (including a pre-charge reminder), not just features.
- Copy uses "start"/ownership/specific numbers over "subscribe"/vague claims.
- Prices are single numbers where possible, anchored against a reference; totals shown before commit.
- Options are swatches/selection-cards, not dropdowns/equal-radios; upsell uses progressive disclosure.
- Trust badges are specific; the product is shown in use, not as decoration.
- The persuasion wording itself was written or checked with the copy skills, not invented here.
