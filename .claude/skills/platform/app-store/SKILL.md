---
name: app-store
description: Apple App Store asset and product page best practices - app icon, screenshots, app previews, product page header, search results creative, and in-app events. Activate when the user is preparing App Store Connect assets, designing app icons, writing screenshot captions, creating app preview videos, setting up a product page, or asking how to avoid App Store rejection for marketing assets.
---

# App Store Assets

Guidance for creating App Store Connect assets that follow Apple's official best practices - drawn from [Apple's App Store asset best practices](https://developer.apple.com/app-store/asset-best-practices/) page. Covers what to build, how to design it, and what gets assets rejected.

## When to Use This Skill

- Designing or reviewing an app icon
- Writing screenshot sequences or captions for the product page
- Scripting/storyboarding an app preview video
- Setting up a product page header or search results creative asset
- Building In-App Events cards
- Sanity-checking assets before submission to avoid rejection

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

## Search Results Asset

The first thing users see when they find the app via Search - appears before they reach the product page.

- State the obvious: the app/game's purpose must be clear at a glance
- Show the firsthand experience - actual interface or gameplay, not abstract branding
- If no dedicated search asset is uploaded, In-App Events, app previews, or screenshots are shown as fallback

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

## Pre-Submission Checklist

- [ ] No pricing, URLs, copyright symbols, or unverified award claims in any asset
- [ ] All assets clear the 4+ age rating regardless of the app's actual rating
- [ ] Icon is legible and recognizable at the smallest rendered size
- [ ] Screenshots lead with the strongest feature and tell a sequential story
- [ ] App preview works with sound off and loops without a visible seam
- [ ] Metadata and imagery localized for each target region
- [ ] Checked against [App Review Guidelines Section 2.3](https://developer.apple.com/app-store/review/guidelines/) (Accurate Metadata) and [Apple's Advertising Policies](https://developer.apple.com/app-store/marketing/guidelines/)

## Related Skills

- `platform/app-onboarding` - for the first-run experience inside the app itself, once someone has already downloaded it
- `platform/expo` - for building and shipping the app via EAS Build/Submit
