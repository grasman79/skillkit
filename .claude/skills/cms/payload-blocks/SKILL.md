---
name: payload-blocks
description: Use when building flexible page layouts with Payload's blocks field - defining blocks, adding blocks fields to collections, configuring admin options, and rendering blocks on the frontend. Trigger words - blocks field, payload blocks, block layout, flexible layouts, page builder, content blocks, block types, blockType, blockName, initCollapsed, isSortable, minRows, maxRows, imageURL, imageAltText
---

# Payload Blocks

The blocks field stores an array of typed objects (blocks) that you define once and reuse anywhere. Each block has its own schema. Use blocks to build flexible, page-builder-style layouts where editors can mix and reorder different content structures within a single document.

## Quick Reference

| Task | Solution |
|------|----------|
| Define a block | `export const MyBlock: Block = { slug, fields }` |
| Add blocks to a collection | `{ type: 'blocks', name: 'layout', blocks: [MyBlock] }` |
| Restrict block count | `minRows`, `maxRows` on the field |
| Disable drag-to-sort | `admin.isSortable: false` |
| Start blocks collapsed | `admin.initCollapsed: true` |
| Custom block drawer image | `imageURL` + `imageAltText` on block config |
| Render blocks on frontend | Map over array, switch on `blockType` |
| TypeScript types | `interfaceName` on block config for named interface |

## Block Definition

Create a `src/blocks/` directory. Each block gets its own file.

```ts
// src/blocks/content-with-media.ts
import type { Block } from 'payload'

export const ContentWithMedia: Block = {
  slug: 'contentWithMedia',           // Required - becomes blockType in API
  interfaceName: 'ContentWithMediaBlock', // Optional - generates named TS interface
  labels: {                           // Optional - auto-generated from slug if omitted
    singular: 'Content with Media Block',
    plural: 'Content with Media Blocks',
  },
  imageURL: 'https://example.com/block-preview.png', // Optional - custom drawer image
  imageAltText: 'Content with media layout',          // Optional - drawer image alt
  fields: [
    {
      type: 'richText',
      name: 'content',
    },
    {
      type: 'upload',
      name: 'image',
      relationTo: 'media',
    },
    {
      type: 'radio',
      name: 'textPosition',
      options: ['left', 'right'],
      defaultValue: 'left',
    },
  ],
}
```

**Required:** `slug`, `fields`
**Auto-generated in API response:** `blockType` (equals `slug`), `blockName` (editor-set label in admin)

## Block Field in Collection

```ts
// src/collections/Pages.ts
import type { CollectionConfig } from 'payload'
import { ContentWithMedia } from '../blocks/content-with-media'
import { HeroBlock } from '../blocks/hero'
import { CTABlock } from '../blocks/cta'

export const Pages: CollectionConfig = {
  slug: 'pages',
  fields: [
    { name: 'title', type: 'text', required: true },
    {
      name: 'layout',
      type: 'blocks',
      blocks: [HeroBlock, ContentWithMedia, CTABlock], // Order sets drawer order
      minRows: 1,     // Optional - minimum blocks required
      maxRows: 20,    // Optional - cap how many editors can add
      label: false,   // Optional - hide field label ("layout"), or set custom labels
      required: false,
      admin: {
        initCollapsed: false, // Default - blocks expanded on load
        isSortable: true,     // Default - editors can drag to reorder
      },
    },
  ],
}
```

## Common Block Patterns

### Hero Block

```ts
// src/blocks/hero.ts
import type { Block } from 'payload'

export const HeroBlock: Block = {
  slug: 'hero',
  fields: [
    { name: 'heading', type: 'text', required: true },
    { name: 'subheading', type: 'text' },
    { name: 'backgroundImage', type: 'upload', relationTo: 'media' },
    {
      name: 'cta',
      type: 'group',
      fields: [
        { name: 'label', type: 'text' },
        { name: 'url', type: 'text' },
      ],
    },
  ],
}
```

### Call to Action Block

```ts
// src/blocks/cta.ts
import type { Block } from 'payload'

export const CTABlock: Block = {
  slug: 'cta',
  labels: { singular: 'Call to Action', plural: 'Calls to Action' },
  fields: [
    { name: 'heading', type: 'text', required: true },
    { name: 'body', type: 'richText' },
    { name: 'buttonLabel', type: 'text', required: true },
    { name: 'buttonUrl', type: 'text', required: true },
    {
      name: 'style',
      type: 'select',
      options: ['primary', 'secondary', 'outline'],
      defaultValue: 'primary',
    },
  ],
}
```

### Rich Text Block

```ts
// src/blocks/rich-text.ts
import type { Block } from 'payload'

export const RichTextBlock: Block = {
  slug: 'richText',
  fields: [
    { name: 'content', type: 'richText', required: true },
    {
      name: 'maxWidth',
      type: 'select',
      options: ['narrow', 'normal', 'wide'],
      defaultValue: 'normal',
    },
  ],
}
```

## Full Field Config Reference

```ts
{
  name: 'layout',
  type: 'blocks',
  blocks: [HeroBlock, ContentWithMedia], // Required - array of block definitions
  label: 'Page Layout',                  // Field label - set false to hide
  minRows: 1,
  maxRows: 20,
  required: false,
  hidden: false,      // Set true to hide from API and admin
  access: {
    create: ({ req }) => req.user?.role === 'admin',
    read: () => true,
    update: ({ req }) => req.user?.role === 'editor',
  },
  admin: {
    initCollapsed: false,  // Start blocks expanded (default)
    isSortable: true,      // Allow drag-to-reorder (default)
    description: 'Build the page layout by adding blocks',
    condition: (data) => data.enableCustomLayout === true,
  },
}
```

## Block Config Reference

```ts
export const MyBlock: Block = {
  slug: 'myBlock',              // Required - used as blockType in API
  interfaceName: 'MyBlock',     // Optional - named TypeScript interface in payload-types.ts
  labels: {
    singular: 'My Block',
    plural: 'My Blocks',
  },
  imageURL: 'https://...',      // Optional - custom image in block picker drawer
  imageAltText: 'My block',     // Optional - alt text for imageURL
  fields: [],                   // Required
}
```

## API Response Shape

When a document has a blocks field, each block in the array includes:

```json
{
  "layout": [
    {
      "blockType": "hero",       // Always equals the block's slug
      "blockName": "Homepage Hero", // Editor-set label (empty string if not set)
      "id": "64abc123...",
      "heading": "Welcome",
      "backgroundImage": { ... }
    },
    {
      "blockType": "contentWithMedia",
      "blockName": "",
      "id": "64abc456...",
      "content": { ... },
      "textPosition": "left"
    }
  ]
}
```

## Frontend Rendering

Switch on `blockType` to render the correct component. With Next.js + Payload:

```tsx
// src/components/blocks/render-blocks.tsx
import type { Page } from '@/payload-types'

type LayoutBlock = Page['layout'][number]

export function RenderBlocks({ blocks }: { blocks: LayoutBlock[] }) {
  return (
    <div>
      {blocks.map((block, i) => {
        switch (block.blockType) {
          case 'hero':
            return <HeroBlock key={i} {...block} />
          case 'contentWithMedia':
            return <ContentWithMediaBlock key={i} {...block} />
          case 'cta':
            return <CTABlock key={i} {...block} />
          case 'richText':
            return <RichTextBlock key={i} {...block} />
          default:
            return null
        }
      })}
    </div>
  )
}
```

```tsx
// src/app/(frontend)/[slug]/page.tsx
import { getPayload } from 'payload'
import config from '@payload-config'
import { RenderBlocks } from '@/components/blocks/render-blocks'

export default async function Page({ params }: { params: { slug: string } }) {
  const payload = await getPayload({ config })

  const { docs } = await payload.find({
    collection: 'pages',
    where: { slug: { equals: params.slug } },
    depth: 2,
  })

  const page = docs[0]
  if (!page) return notFound()

  return <RenderBlocks blocks={page.layout} />
}
```

## TypeScript Types

Generate types with `payload generate:types`. With `interfaceName` set, blocks get named interfaces:

```ts
import type { HeroBlock, ContentWithMediaBlock } from '@/payload-types'

// Or use the union type from the collection
import type { Page } from '@/payload-types'
type AnyBlock = Page['layout'][number]
```

## Directory Structure

```txt
src/
├── blocks/
│   ├── hero.ts              // Block definition (+ optional hero.tsx component)
│   ├── content-with-media.ts
│   ├── cta.ts
│   └── rich-text.ts
├── components/
│   └── blocks/
│       ├── render-blocks.tsx
│       ├── hero-block.tsx
│       ├── content-with-media-block.tsx
│       └── cta-block.tsx
└── collections/
    └── Pages.ts
```

Since Next.js and Payload are co-located, you can keep the component in the same `blocks/` directory as the definition:

```txt
src/
└── blocks/
    ├── hero/
    │   ├── config.ts     // Block definition
    │   └── component.tsx // React component
    └── cta/
        ├── config.ts
        └── component.tsx
```

## Common Gotchas

**Slug as blockType** - The `slug` value becomes `blockType` in the API. If you rename the slug, existing saved data will not match. Migrate data or keep slugs stable.

**depth: 2 for media** - Upload fields inside blocks are relationships. Set `depth: 2` when querying to get the full media object instead of just an ID.

**Type narrowing** - When using the union type, narrow by `blockType` before accessing block-specific fields:

```ts
if (block.blockType === 'hero') {
  // TypeScript now knows block is HeroBlock
  console.log(block.heading)
}
```

**interfaceName is global** - Use distinct `interfaceName` values across all blocks to avoid conflicts in `payload-types.ts`.

**Block groups** - Payload supports block groups to organize large numbers of blocks in the picker drawer. See [Payload docs](https://payloadcms.com/docs/fields/blocks).

## Resources

- Docs: <https://payloadcms.com/docs/fields/blocks>
- Guide: <https://payloadcms.com/posts/guides/how-to-build-flexible-layouts-with-payload-blocks>
- Frontend rendering video: <https://www.youtube.com/watch?v=SMOULBdsKCM>
