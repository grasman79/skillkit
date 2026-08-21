---
name: app-store
description: Apple App Store asset/product page best practices AND app review/approval guidance - app icon, screenshots, app previews, product page header, search results creative, in-app events, review notes, dark patterns to avoid, expedited review, and scope management for a fast App Review. Activate when the user is preparing App Store Connect assets, designing app icons, writing screenshot captions, creating app preview videos, setting up a product page, writing App Review notes, preparing an app submission, asking how to get an app approved faster, or asking how to avoid App Store rejection for assets OR for the app itself.
---

# App Store Assets & Review

Guidance for creating App Store Connect assets that follow Apple's official best practices - drawn from [Apple's App Store asset best practices](https://developer.apple.com/app-store/asset-best-practices/) page - plus practical guidance for getting the app itself through App Review quickly. Covers what to build, how to design it, what gets rejected, and how to submit so the review is fast and boring.

## When to Use This Skill

- Designing or reviewing an app icon
- Writing screenshot sequences or captions for the product page
- Scripting/storyboarding an app preview video
- Setting up a product page header or search results creative asset
- Building In-App Events cards
- Sanity-checking assets before submission to avoid rejection
- Preparing an app submission for App Review (review notes, demo video, avoiding dark patterns)
- Trying to get an app approved faster, or recovering from a rejection loop

## Content Rules (Apply to Every Asset)

**Never include:**
- Pricing, discounts, or website URLs
- Copyright symbols (©)
- Unverified claims - awards or recognition the app hasn't actually received
- Logos or references to other platforms/marketplaces
- Apple-designated recognitions the app doesn't hold (Editor's Choice, App of the Day, Apple Design Award)

**Design for all customers:**
- All assets must meet a 4+ age rating, even if the app itself is rated higher. Avoid explicit/harmful imagery, profanity, drug references, self-harm, hate speech, stereotypes, sexually suggestive content, or inflammatory religious content.
- Games can show action imagery but should avoid gore, graphic violence, or weapons pointed at a person/the viewer.
- Localize metadata, images, and video for each supported region rather than shipping one locale everywhere.

**Stay on-topic:** every asset must depict something directly related to what the app or game actually does. Don't pad with lifestyle stock imagery unrelated to the product.

## App Icon

The icon is the most persistent brand touchpoint - it lives on the Home Screen after download, not just on the product page.

- Make it unique and expressive of the app's purpose/personality
- Design for instant recognition at a glance, and legibility at every size (App Store, Spotlight, Home Screen)
- Use [Icon Composer](https://developer.apple.com/icon-composer/) for layered icons with Liquid Glass support across iPhone, iPad, Mac, and Apple Watch
- Reference: [Human Interface Guidelines - App Icons](https://developer.apple.com/design/human-interface-guidelines/app-icons)

## Product Page Header

First thing a visitor sees on the product page.

- Focus on ONE clear idea - avoid visual clutter or stacking multiple messages into one image
- Design for first-time visitors, not existing users - lead with what's most compelling to someone who's never seen the app
- The same creative can double as the search results asset for consistent brand recognition, or you can differentiate them
- Test alternate headers with product page optimization before committing
- Official starting templates: [Figma (Universal)](https://developer.apple.com/go/?id=figma-creative-assets-templates), [Photoshop (Header)](https://developer.apple.com/go/?id=photoshop-product-page-header-template), [Photoshop (Universal)](https://developer.apple.com/go/?id=photoshop-universal-asset-template), [Pixelmator (Header)](https://developer.apple.com/go/?id=pixelmator-product-page-header-template), [Pixelmator (Universal)](https://developer.apple.com/go/?id=pixelmator-universal-asset-template)

## Search Results Asset

The first thing users see when they find the app via Search - appears before they reach the product page.

- State the obvious: the app/game's purpose must be clear at a glance
- Show the firsthand experience - actual interface or gameplay, not abstract branding
- If no dedicated search asset is uploaded, In-App Events, app previews, or screenshots are shown as fallback
- Official starting templates: [Figma (Universal)](https://developer.apple.com/go/?id=figma-creative-assets-templates), [Photoshop (Search results)](https://developer.apple.com/go/?id=photoshop-search-results-template), [Photoshop (Universal)](https://developer.apple.com/go/?id=photoshop-universal-asset-template), [Pixelmator (Search results)](https://developer.apple.com/go/?id=pixelmator-search-results-template), [Pixelmator (Universal)](https://developer.apple.com/go/?id=pixelmator-universal-asset-template)

## Screenshots

- Up to 10 on the product page; up to 3 may surface in search results depending on orientation
- Show what people expect: the app in actual use, giving a clear detailed view of the experience
- Lead with the strongest features first - sequence tells a cohesive story of how someone would actually use the app
- Don't burn an entire screenshot slot on a generic accolade banner or a bare call-to-action - every slot should show product

## App Previews (Video)

- Up to 3 on the product page; up to 3 may surface in search results
- Use real UI or real gameplay - this is a demo, not a trailer
- Pick a poster frame that's visually compelling and clearly represents the app on its own (it's what shows before playback)
- Start strong: lead with the value proposition or the single most compelling feature in the first few seconds
- Pace it for comprehension - avoid quick cuts and fast-moving visuals; give the viewer time to understand what they're seeing
- Structure it like a story - a beginning, middle, and end (e.g. rank features by popularity, or walk through game progression)
- Loop seamlessly - previews autoplay and repeat, so the last frame should flow back into the first without a visible jump
- Audio is muted by default: the video must communicate everything visually first. Treat sound as brand texture, not information delivery - favor a short music loop over sound effects, avoid sudden loud moments, and make sure the audio loop itself is clean

## In-App Events

For timely, limited-time moments - competitions, premieres, live experiences, seasonal content.

- Design for both required aspect ratios: the event card is 16:9 landscape, the event details page is 9:16 portrait. Both must independently communicate what the event is.
- Use consistent colors/illustration style across the two so they read as the same event
- Pick the right badge (Challenge, Competition, Live Event, Major Update, New Season, Premiere, Special Event) and design the media to complement it, not repeat it in the artwork

## Submission Workflow

- Submit assets via App Store Connect alongside a new app version, or upload directly to the Asset Library - or automate via the App Store Connect API
- Use the **Product Page Preview Tool** before publishing to see exactly how the header, app name, description, screenshots, and search results asset will render
- Use **Product Page Optimization** to A/B test creative variants (headers, screenshots, icons) and keep the best performer
- Use **[Custom Product Pages](https://developer.apple.com/app-store/custom-product-pages/)** to build alternate versions of the product page (different creative assets, different messaging) for specific ad campaigns or audience segments - each gets its own trackable URL, distinct from Product Page Optimization which A/B tests variants of the single default page

## Pre-Submission Checklist

- [ ] No pricing, URLs, copyright symbols, or unverified award claims in any asset
- [ ] All assets clear the 4+ age rating regardless of the app's actual rating
- [ ] Icon is legible and recognizable at the smallest rendered size
- [ ] Screenshots lead with the strongest feature and tell a sequential story
- [ ] App preview works with sound off and loops without a visible seam
- [ ] Metadata and imagery localized for each target region
- [ ] Checked against [App Review Guidelines Section 2.3](https://developer.apple.com/app-store/review/guidelines/) (Accurate Metadata) and [Apple's Advertising Policies](https://developer.apple.com/app-store/marketing/guidelines/)

## App Review (Getting the App Itself Approved)

Distinct from the asset guidance above - this covers getting the app binary through App Review, not the marketing creative around it. Reviews are deliberately subjective (a human judgment call, not a checklist pass/fail), but a submission can be structured to maximize the odds of a fast, clean approval.

### Pre-Submission Fundamentals

- **Quality reads as trust.** A polished, useful app gets more benefit of the doubt from reviewers than something that feels unfinished - scrutiny scales inversely with perceived quality.
- **UI must work across devices** - proper adaptation across iPhone/iPad sizes, native keyboard/sheets/navigation/safe-area handling, Dynamic Type and accessibility defaults. Building natively (SwiftUI) gives more surface area for this than cross-platform frameworks, though frameworks like Expo can still clear review.
- **Privacy policy and terms** as static HTML, reachable without login.
- **Request permissions properly** - ask at the point of use with a clear purpose string, not upfront during onboarding.
- **Social/UGC apps need:** report and blocking functionality, a contact-support path, and some system to moderate user-generated content (even a lightweight automated moderation pass is expected).
- **Everything must actually work** before submission - payments, every advertised feature, no exceptions.

### Dark Patterns to Avoid (Initial Submission)

Never do these - they're the fastest way to a rejection or a flagged account:
- No unfinished or "coming soon" screens - every screen must be fully functional
- Making a remote paywall non-compliant *after* the version is approved (i.e. switching pricing/behavior server-side post-review)
- Prompting for an App Store review during onboarding (Apple has closed this pattern)
- Fake social proof ("#1 on the App Store" claims that aren't verified)
- Unverified or controversial claims ("fixes 100% of X")

**Higher-risk-but-technically-allowed patterns** - legal per guidelines but reviewer discretion is subjective enough that they can tank an *initial* review if a strict reviewer flags them. Don't include these in the first submission; add them in a later version once the app has an approval history:
- **Transaction abandon flows** (interrupting a cancellation with a retention offer)
- **Exit offers** on the manage-subscription flow (deep-linking to the App Store subscription page is standard; showing a cheaper-plan questionnaire first is allowed but riskier)

**Don't 1:1 copy an existing app.** Inspiration from general patterns, core ideas, and positioning is fine; a direct clone invites rejection.

### Writing Review Notes

Detailed but simple review notes are one of the highest-ROI things in a submission - possibly required for an initial submission to avoid auto-rejection (unconfirmed but consistent with observed patterns). Write for a total beginner reading it cold, in concise bullet points - not AI-polished prose, just clarity.

**What to include:**
- One-line description of what the app does and whether login is required
- Exactly how to test any paid/subscription flow (sandbox environment, where "Restore Purchases" lives)
- A numbered walkthrough of the main review flow - the exact taps a reviewer needs to make to see the app work
- For any social/multiplayer feature: a dedicated reviewer test account (pre-configured to accept requests immediately) and the exact steps to test it
- Photo/data handling: what's optional, what permission is requested and when, what happens to uploaded data (retention period, any third-party processing like automated moderation) - full transparency here specifically, since misleading data handling is a common rejection trigger
- Where account/legal controls live (manage subscription, restore purchases, support, privacy policy, terms, delete account) and confirm they're reachable even without an active subscription. "Delete account" must be a genuine backend deletion (an edge function/API route that actually removes the user's data), not just a client-side sign-out or a deactivation flag - Guideline 5.1.1(v) requires full account deletion capability, and a UI button that doesn't back onto real deletion is a rejection risk on its own.

**Also add a screen-recorded demo video** with text overlay explaining each section as it plays. High ROI relative to the effort - treat it as expected for an initial submission, not optional.

**Goal:** make the review effortless and boring. A reviewer who never has to guess what something does or hunt for a feature is a reviewer who approves quickly.

### Getting a Faster Review

**Expedited review** can be requested (no reason required) and is intended for: critical bugs (follow-up reviews only), timed events/launches, marketing moments, and plausibly a genuine viral moment ahead of an initial launch (not explicitly listed in Apple's criteria, so not guaranteed).

**Requesting a call from Apple** is underused - reviewers report almost nobody asks for one. It likely routes the app into a separate, less-contested review queue.

**Avoiding the rejection loop:** a common failure pattern is fix-one-issue → resubmit → get rejected for a *different* issue → repeat. When rejected, explicitly ask for all rejection reasons at once rather than fixing and resubmitting reactively. Requesting a call is the most reliable way to break this loop if it's already happening.

### Scope Creep Is the Biggest Speed Killer

The single biggest mistake for anyone wanting a fast initial review: shipping too much in the first submission. Reviews have tiers:
1. **Straightforward reviews** - first pile, typically resolved within ~48 hours
2. **Complex issues/questions** - escalated to a senior specialist, can mean a ~14 day wait

Unless the app has genuine inherent complexity (banking, health/biometric data, a marketplace), keep the **initial submission an MVP** and add features in later versions once the app has an approval history. More features in submission #1 means more surface area for a reviewer to escalate rather than approve outright.

### Principle for Every Submission

Apple's actual bar is whether the user is treated well - if the app misleads or confuses a user at any point, that's the rejection trigger, regardless of which specific guideline gets cited. For an initial submission, play it safe and avoid every higher-risk pattern above. For later updates, it's reasonable to be bolder (test a transaction-abandon flow, etc.) - but always disclose exactly what's happening in the review notes and back it with a demo. Never try to get something past the review team quietly; clarity and honesty in the submission itself is what actually earns approval, subjective reviewer variance notwithstanding.

## Related Skills

- `platform/app-onboarding` - for the first-run experience inside the app itself, once someone has already downloaded it
- `platform/expo` - for building and shipping the app via EAS Build/Submit
