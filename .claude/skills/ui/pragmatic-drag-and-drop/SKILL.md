---
name: pragmatic-drag-and-drop
description: Use this skill when implementing drag-and-drop interactions. Activate when the user mentions drag and drop, drag-and-drop, DnD, sortable list, kanban board, reorder, drag to reorder, drop zone, file drop, pragmatic-drag-and-drop, @atlaskit/pragmatic-drag-and-drop, or wants to migrate from react-beautiful-dnd.
---

# Pragmatic Drag and Drop

Atlassian's framework-agnostic drag-and-drop toolkit. Powers Trello, Jira, and Confluence. Tiny core (~4.7kB), headless, fully accessible, works with React, Vue, Svelte, or vanilla JS. `react-beautiful-dnd` is deprecated - pragmatic is its successor.

## When to Use This Skill

- User wants drag-and-drop in any framework
- Building a sortable list, kanban board, tree, or file drop zone
- Migrating from `react-beautiful-dnd` (deprecated) or `dnd-kit`
- User mentions "drag", "reorder", "drop zone", or "draggable"

**Do NOT reach for this when:** the user only needs click-based reordering with up/down buttons. A DnD library adds complexity that isn't needed for simple list manipulation without pointer interaction.

## Why Pragmatic over Alternatives

- Tiny bundle: ~4.7kB core vs react-beautiful-dnd (20kB+) and dnd-kit (12kB+)
- Headless: you own 100% of the rendering and styling
- Framework-agnostic: same primitives in React, Vue, Svelte, or plain JS
- `react-beautiful-dnd` is officially deprecated; pragmatic is the Atlassian-recommended replacement
- Full cross-browser support: Chrome, Firefox, Safari, iOS, Android

## Install

```bash
[runner] add @atlaskit/pragmatic-drag-and-drop
```

Add optional packages as needed (see Optional Packages section below).

## Core Concepts

**Adapter** - teaches the library what kind of entity is being dragged (elements, text selections, or external items like files). Each adapter provides a `draggable()`, `dropTargetFor*()`, and `monitorFor*()` function.

**Draggable** - registers an HTML element as something the user can pick up and move. Returns a cleanup function.

**Drop target** - registers an HTML element as an area that can receive a dropped item. Returns a cleanup function.

**Monitor** - observes drag events globally (across all draggables and drop targets in scope). Useful for state updates at the container level. Returns a cleanup function.

## The Three Adapters

### Element adapter (most common)

Dragging DOM elements - cards, list items, rows, tree nodes.

```typescript
import {
  draggable,
  dropTargetForElements,
  monitorForElements,
} from '@atlaskit/pragmatic-drag-and-drop/element/adapter';
```

### Text selection adapter

Dragging selected text.

```typescript
import {
  dropTargetForTextSelection,
  monitorForTextSelection,
} from '@atlaskit/pragmatic-drag-and-drop/text-selection/adapter';
```

### External adapter

Drops arriving from outside the window - OS file system, other browser tabs.

```typescript
import {
  dropTargetForExternal,
  monitorForExternal,
} from '@atlaskit/pragmatic-drag-and-drop/external/adapter';
```

## Cleanup Pattern

Every primitive returns a cleanup function. In React, return it from `useEffect`. Use `combine` to merge multiple cleanups into one.

```typescript
import { combine } from '@atlaskit/pragmatic-drag-and-drop/combine';

useEffect(() => {
  return combine(
    draggable({ element: ref.current! }),
    dropTargetForElements({ element: ref.current! }),
  );
}, []);
```

Never call `useEffect` with missing dependencies or you will register duplicate listeners on every render. Always tie the effect to the refs and data that the registration actually needs.

## React Pattern - Draggable + Drop Target

```typescript
import { useEffect, useRef } from 'react';
import {
  draggable,
  dropTargetForElements,
} from '@atlaskit/pragmatic-drag-and-drop/element/adapter';
import { combine } from '@atlaskit/pragmatic-drag-and-drop/combine';

function Card({ id, label }: { id: string; label: string }) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;

    return combine(
      draggable({
        element: el,
        getInitialData: () => ({ type: 'card', id }),
        onDragStart: () => el.style.opacity = '0.4',
        onDrop: () => el.style.opacity = '1',
      }),
      dropTargetForElements({
        element: el,
        canDrop: ({ source }) => source.data.type === 'card' && source.data.id !== id,
        onDrop: ({ source }) => {
          console.log(`Dropped ${source.data.id} onto ${id}`);
        },
      }),
    );
  }, [id]);

  return <div ref={ref}>{label}</div>;
}
```

**Type guarding source data** - `source.data` is `Record<string, unknown>`. Always validate before reading:

```typescript
function isCardData(data: Record<string, unknown>): data is { type: 'card'; id: string } {
  return data.type === 'card' && typeof data.id === 'string';
}
```

## Monitor Pattern

Use a monitor when you need to respond to drag events at a higher level than the individual elements - for example, updating global state after a drop.

```typescript
import { monitorForElements } from '@atlaskit/pragmatic-drag-and-drop/element/adapter';

useEffect(() => {
  return monitorForElements({
    onDrop: ({ source, location }) => {
      const target = location.current.dropTargets[0];
      if (!target) return;
      if (!isCardData(source.data)) return;
      // update your state here
    },
  });
}, []);
```

## Sortable List Pattern

Install the hitbox optional package for closest-edge detection:

```bash
[runner] add @atlaskit/pragmatic-drag-and-drop-hitbox
```

```typescript
import { attachClosestEdge, extractClosestEdge } from '@atlaskit/pragmatic-drag-and-drop-hitbox/closest-edge';
import { monitorForElements } from '@atlaskit/pragmatic-drag-and-drop/element/adapter';

// In each list item's useEffect:
dropTargetForElements({
  element: el,
  getData: ({ input, element }) =>
    attachClosestEdge({ index }, { input, element, allowedEdges: ['top', 'bottom'] }),
  onDrop: ({ self }) => {
    const edge = extractClosestEdge(self.data);
    // edge is 'top' or 'bottom' - use it to determine insertion position
  },
});

// In the list container:
monitorForElements({
  onDrop: ({ source, location }) => {
    const target = location.current.dropTargets[0];
    if (!target) return;

    const srcIndex = source.data.index as number;
    const dstIndex = target.data.index as number;
    const edge = extractClosestEdge(target.data);

    const insertAt = edge === 'bottom' ? dstIndex + 1 : dstIndex;
    setItems(reorder(items, srcIndex, insertAt));
  },
});
```

Add drop indicator lines with:

```bash
[runner] add @atlaskit/pragmatic-drag-and-drop-react-drop-indicator
```

```tsx
import { DropIndicator } from '@atlaskit/pragmatic-drag-and-drop-react-drop-indicator/box';

// Render inside each list item (conditionally when dragging over):
<DropIndicator edge={closestEdge} gap="8px" />
```

## Kanban Board Pattern

Cards are draggable. Columns are drop targets. A top-level monitor moves items between columns.

```typescript
// Card component - draggable with column + card identity
draggable({
  element: cardRef.current!,
  getInitialData: () => ({ type: 'card', cardId, columnId }),
});

// Column component - drop target for cards
dropTargetForElements({
  element: columnRef.current!,
  canDrop: ({ source }) => source.data.type === 'card',
  getData: () => ({ columnId }),
});

// Board-level monitor
monitorForElements({
  onDrop: ({ source, location }) => {
    const target = location.current.dropTargets[0];
    if (!target || !isCardData(source.data)) return;

    const { cardId, columnId: srcColumnId } = source.data;
    const { columnId: dstColumnId } = target.data as { columnId: string };

    if (srcColumnId !== dstColumnId) {
      moveCard(cardId, srcColumnId, dstColumnId);
    }
  },
});
```

## File Drop Pattern

```typescript
import {
  dropTargetForExternal,
  monitorForExternal,
} from '@atlaskit/pragmatic-drag-and-drop/external/adapter';
import { containsFiles, getFiles } from '@atlaskit/pragmatic-drag-and-drop/external/file';

useEffect(() => {
  return combine(
    dropTargetForExternal({
      element: dropZoneRef.current!,
      canDrop: containsFiles,
      onDrop: ({ source }) => {
        const files = getFiles({ source });
        handleFiles(files);
      },
    }),
    monitorForExternal({
      canMonitor: containsFiles,
      onDragStart: () => setIsDraggingFile(true),
      onDrop: () => setIsDraggingFile(false),
    }),
  );
}, []);
```

## Optional Packages

| Package | What it adds | When to use |
|---------|-------------|-------------|
| `@atlaskit/pragmatic-drag-and-drop-hitbox` | Closest edge and corner detection for precise drop targeting | Sortable lists, trees |
| `@atlaskit/pragmatic-drag-and-drop-react-drop-indicator` | Renders drop indicator lines between items | Sortable lists, kanban |
| `@atlaskit/pragmatic-drag-and-drop-flourish` | Flash-on-drop and other polish effects | When you want delightful micro-interactions |
| `@atlaskit/pragmatic-drag-and-drop-auto-scroll` | Smooth viewport/container autoscroll while dragging near edges | Long lists, infinite scrollers |
| `@atlaskit/pragmatic-drag-and-drop-react-accessibility` | Keyboard drag-and-drop controls for React | When you need accessible DnD without building it yourself |
| `@atlaskit/pragmatic-drag-and-drop-live-region` | Screen reader announcements during drag operations | Accessibility requirement |
| `@atlaskit/pragmatic-drag-and-drop-react-beautiful-dnd-migration` | Drop-in migration layer from `react-beautiful-dnd` | Migrating large existing codebases |
| `@atlaskit/pragmatic-drag-and-drop-unit-testing` | Test helpers for simulating drag events | Unit testing |

## Accessibility

Browser-native drag-and-drop is not keyboard-accessible by default. Two options:

**Option A - Use the `react-accessibility` package** (recommended for most projects)

```bash
[runner] add @atlaskit/pragmatic-drag-and-drop-react-accessibility
```

Provides opinionated React components that add keyboard controls on top of your existing drag-and-drop setup without restructuring it.

**Option B - Build your own keyboard alternative**

Provide a separate set of keyboard controls (up/down arrow reorder buttons) alongside the drag handles. The drag handles are pointer-only; the buttons cover keyboard users.

**Screen reader announcements** - always add them for reorder operations:

```bash
[runner] add @atlaskit/pragmatic-drag-and-drop-live-region
```

```typescript
import { announce } from '@atlaskit/pragmatic-drag-and-drop-live-region';

// After a successful drop:
announce(`Moved "${item.label}" to position ${newIndex + 1} of ${items.length}`);
```

## Common Gotchas

1. **Always return cleanup from `useEffect`** - every primitive leaks listeners if not cleaned up. Use `combine` so you never forget one.

2. **Payload is untyped** - `source.data` and `target.data` are `Record<string, unknown>`. Write type guard functions and call them before reading properties.

3. **`canDrop` runs hot** - it fires continuously while the user drags. Keep it cheap: a simple type check is fine, database lookups are not.

4. **Avoid re-registering on every render** - if `id` changes on every render (e.g. from an object literal in JSX), your effect re-runs and you stack up listeners. Stabilize values with `useMemo` or `useCallback`.

5. **Files need the external adapter** - the element adapter never sees drops from the OS file system. You need `dropTargetForExternal` + `containsFiles`.

6. **Touch/mobile requires `touch-action: none`** - add this CSS to any element you want to be draggable on mobile, otherwise the browser's default scroll gesture wins:

   ```css
   .draggable {
     touch-action: none;
   }
   ```

7. **`location.current.dropTargets` can be empty** - when a drop lands outside all registered targets. Always null-check `dropTargets[0]` before reading its data.

8. **Migrating from `react-beautiful-dnd`** - use the `react-beautiful-dnd-migration` package for large codebases to avoid a big-bang rewrite. It lets you migrate component-by-component.

## Source of Truth

- Docs: https://atlassian.design/components/pragmatic-drag-and-drop
- Repository: https://github.com/atlassian/pragmatic-drag-and-drop
