---
name: animation-patterns
description: Use this skill when adding animations to web app UI components - buttons, icons, loaders, modals, toasts, dropdowns, number tickers, skeleton screens, spring physics, or state transitions. Activate when the user mentions button press feedback, icon animation, loading spinner, skeleton loader, shimmer, toast animation, modal entrance, number counter, typewriter, spring animation, layout animation, or micro-interactions inside an app. This is for interactive app components, not marketing page scroll reveals (use design/animation for those).
---

# UI Animation Patterns

Practical animation patterns for web app components. Covers the full vocabulary of UI motion: press feedback, icon states, loaders, notifications, overlays, data display, and spring physics. Framework examples use React with Framer Motion for complex animations and CSS for simple ones.

## When to Use This Skill

- Adding feedback animations to buttons, inputs, or icons
- Building loaders, skeletons, or progress indicators
- Animating modals, drawers, toasts, dropdowns, or tooltips
- Implementing number tickers, typewriters, or text counters
- Applying spring physics or layout animations (Framer Motion)
- Transitioning between UI states (success, error, loading, empty)

**Use `design/animation` instead** for marketing page animations: hero fade-ins, scroll reveals, parallax, or FAQ accordions.

## Library Decision

| Complexity | Library | When |
|------------|---------|------|
| Simple hover/press | CSS transitions | Single-property changes, no physics |
| Enter/exit + springs | `framer-motion` | Components that mount/unmount, spring physics, layout shifts |
| Loaders | CSS `@keyframes` | Purely decorative, no JS needed |

Install Framer Motion when needed:
```bash
[runner] add framer-motion
```

## Accessible Motion - Always Include This

Wrap or gate any motion behind the reduced-motion media query. In React:

```typescript
import { useReducedMotion } from 'framer-motion';

function useMotionSafe() {
  const reduce = useReducedMotion();
  return (full: object, reduced: object = {}) => (reduce ? reduced : full);
}
```

For CSS animations, always add:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Spring Presets

Use these instead of duration-based easing for interactive elements. Springs feel physical - they respond to interruption naturally.

```typescript
import { type Spring } from 'framer-motion';

export const spring = {
  snappy:  { type: 'spring', stiffness: 400, damping: 30 } satisfies Spring,
  bouncy:  { type: 'spring', stiffness: 300, damping: 20 } satisfies Spring,
  gentle:  { type: 'spring', stiffness: 200, damping: 25 } satisfies Spring,
  stiff:   { type: 'spring', stiffness: 600, damping: 40 } satisfies Spring,
};
```

- **snappy** - default for most UI interactions (buttons, toggles)
- **bouncy** - confirmation success states, checkmarks
- **gentle** - panels, drawers, non-urgent elements
- **stiff** - tooltips, dropdowns (should feel instant)

## Button & Press Feedback

Scale down on press, spring back on release. Never just change color alone - physical feedback requires a transform.

```typescript
import { motion } from 'framer-motion';
import { spring } from '@/lib/motion';

function Button({ children, onClick }: { children: React.ReactNode; onClick?: () => void }) {
  return (
    <motion.button
      onClick={onClick}
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.96 }}
      transition={spring.snappy}
      className="px-4 py-2 bg-blue-600 text-white rounded-lg"
    >
      {children}
    </motion.button>
  );
}
```

**Rules:**
- `whileTap` scale: 0.94-0.97 (never below 0.9 - looks broken)
- `whileHover` scale: 1.01-1.03 (subtle lift)
- Use `spring.snappy` - duration-based easing feels sluggish on press
- Destructive buttons (delete): skip the hover lift, only do press-down

## Icon Animations

### Loading Spinner

CSS-only. No library needed.

```tsx
function Spinner({ size = 20 }: { size?: number }) {
  return (
    <svg
      width={size}
      height={size}
      viewBox="0 0 24 24"
      fill="none"
      className="animate-spin"
      aria-label="Loading"
    >
      <circle cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="3" strokeOpacity="0.25" />
      <path
        d="M12 2a10 10 0 0 1 10 10"
        stroke="currentColor"
        strokeWidth="3"
        strokeLinecap="round"
      />
    </svg>
  );
}
```

Tailwind's `animate-spin` is 1s linear infinite - do not change the duration.

### Checkmark Success

Scale in with a bounce after an async operation completes.

```tsx
import { motion, AnimatePresence } from 'framer-motion';

function SuccessIcon() {
  return (
    <motion.svg
      initial={{ scale: 0, opacity: 0 }}
      animate={{ scale: 1, opacity: 1 }}
      transition={{ type: 'spring', stiffness: 300, damping: 18 }}
      width="24"
      height="24"
      viewBox="0 0 24 24"
      fill="none"
    >
      <motion.path
        d="M5 13l4 4L19 7"
        stroke="currentColor"
        strokeWidth="2.5"
        strokeLinecap="round"
        strokeLinejoin="round"
        initial={{ pathLength: 0 }}
        animate={{ pathLength: 1 }}
        transition={{ duration: 0.3, delay: 0.1 }}
      />
    </motion.svg>
  );
}
```

### Icon State Transition (e.g. play/pause)

```tsx
import { AnimatePresence, motion } from 'framer-motion';

function PlayPauseButton({ isPlaying }: { isPlaying: boolean }) {
  return (
    <button>
      <AnimatePresence mode="wait" initial={false}>
        <motion.span
          key={isPlaying ? 'pause' : 'play'}
          initial={{ scale: 0.7, opacity: 0 }}
          animate={{ scale: 1, opacity: 1 }}
          exit={{ scale: 0.7, opacity: 0 }}
          transition={{ duration: 0.12 }}
        >
          {isPlaying ? <PauseIcon /> : <PlayIcon />}
        </motion.span>
      </AnimatePresence>
    </button>
  );
}
```

`mode="wait"` ensures exit completes before enter starts - prevents icon overlap.

## Skeleton / Shimmer Loader

CSS shimmer animation. No JS needed. Match the shape of the real content exactly.

```css
@keyframes shimmer {
  0%   { background-position: -400px 0; }
  100% { background-position:  400px 0; }
}

.skeleton {
  background: linear-gradient(
    90deg,
    hsl(var(--muted)) 25%,
    hsl(var(--muted-foreground) / 0.15) 50%,
    hsl(var(--muted)) 75%
  );
  background-size: 800px 100%;
  animation: shimmer 1.4s ease-in-out infinite;
  border-radius: 4px;
}
```

```tsx
function CardSkeleton() {
  return (
    <div className="p-4 space-y-3">
      <div className="skeleton h-4 w-3/4" />
      <div className="skeleton h-4 w-1/2" />
      <div className="skeleton h-32 w-full rounded-lg" />
    </div>
  );
}
```

**Rules:**
- Match skeleton dimensions to the real content (use fixed heights that match)
- Show skeleton immediately - never show a blank white area first
- Fade out skeleton when content arrives with a 0.2s opacity transition
- No skeleton for operations under 300ms - just show a spinner inline

## Toast / Notification

Slide in from the side or bottom, spring out. Use `AnimatePresence` so exits animate.

```tsx
import { motion, AnimatePresence } from 'framer-motion';

const toastVariants = {
  initial: { opacity: 0, y: 20, scale: 0.95 },
  animate: { opacity: 1, y: 0, scale: 1, transition: { type: 'spring', stiffness: 350, damping: 28 } },
  exit:    { opacity: 0, y: 10, scale: 0.95, transition: { duration: 0.15 } },
};

function Toast({ message, visible }: { message: string; visible: boolean }) {
  return (
    <AnimatePresence>
      {visible && (
        <motion.div
          variants={toastVariants}
          initial="initial"
          animate="animate"
          exit="exit"
          className="fixed bottom-4 right-4 bg-gray-900 text-white px-4 py-3 rounded-lg shadow-xl"
        >
          {message}
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

**Rules:**
- Enter: spring (feels responsive)
- Exit: duration-based, faster than enter (0.1-0.15s)
- Position: bottom-right for most apps, bottom-center for mobile
- Never stack more than 3 toasts - queue them

## Modal / Dialog

Backdrop fades in. Panel scales up from 95% with a spring.

```tsx
import { motion, AnimatePresence } from 'framer-motion';

const overlayVariants = {
  initial: { opacity: 0 },
  animate: { opacity: 1 },
  exit:    { opacity: 0 },
};

const panelVariants = {
  initial: { opacity: 0, scale: 0.95, y: 8 },
  animate: { opacity: 1, scale: 1, y: 0, transition: { type: 'spring', stiffness: 400, damping: 30 } },
  exit:    { opacity: 0, scale: 0.96, y: 4, transition: { duration: 0.15 } },
};

function Modal({ open, onClose, children }: { open: boolean; onClose: () => void; children: React.ReactNode }) {
  return (
    <AnimatePresence>
      {open && (
        <>
          <motion.div
            variants={overlayVariants}
            initial="initial"
            animate="animate"
            exit="exit"
            transition={{ duration: 0.2 }}
            onClick={onClose}
            className="fixed inset-0 bg-black/50"
          />
          <motion.div
            variants={panelVariants}
            initial="initial"
            animate="animate"
            exit="exit"
            className="fixed inset-0 flex items-center justify-center pointer-events-none"
          >
            <div className="bg-white rounded-xl p-6 shadow-2xl pointer-events-auto max-w-md w-full mx-4">
              {children}
            </div>
          </motion.div>
        </>
      )}
    </AnimatePresence>
  );
}
```

## Dropdown / Popover

Scale from transform-origin top. Faster than modals - should feel instant.

```tsx
const dropdownVariants = {
  initial: { opacity: 0, scale: 0.95, y: -4 },
  animate: { opacity: 1, scale: 1, y: 0, transition: { type: 'spring', stiffness: 500, damping: 35 } },
  exit:    { opacity: 0, scale: 0.97, y: -2, transition: { duration: 0.1 } },
};

function Dropdown({ open, children }: { open: boolean; children: React.ReactNode }) {
  return (
    <AnimatePresence>
      {open && (
        <motion.div
          variants={dropdownVariants}
          initial="initial"
          animate="animate"
          exit="exit"
          style={{ transformOrigin: 'top center' }}
          className="absolute top-full mt-1 bg-white border rounded-lg shadow-lg overflow-hidden"
        >
          {children}
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

Always set `transformOrigin` to the direction the dropdown appears from (top for downward, bottom for upward).

## Number Ticker

Animate a number rolling up/down to a new value. Useful for dashboards, stats, scores.

```tsx
import { useEffect, useRef } from 'react';
import { animate } from 'framer-motion';

function NumberTicker({
  value,
  duration = 0.8,
  format = (n: number) => Math.round(n).toLocaleString(),
}: {
  value: number;
  duration?: number;
  format?: (n: number) => string;
}) {
  const ref = useRef<HTMLSpanElement>(null);
  const prev = useRef(0);

  useEffect(() => {
    const from = prev.current;
    prev.current = value;

    const controls = animate(from, value, {
      duration,
      ease: [0.16, 1, 0.3, 1],
      onUpdate: (latest) => {
        if (ref.current) ref.current.textContent = format(latest);
      },
    });

    return controls.stop;
  }, [value, duration, format]);

  return <span ref={ref}>{format(value)}</span>;
}
```

Use `tabular-nums` CSS class to prevent layout shift as digits change width:

```tsx
<span className="tabular-nums font-mono">
  <NumberTicker value={count} />
</span>
```

## Typewriter Effect

```tsx
import { useEffect, useState } from 'react';

function Typewriter({ text, speed = 40 }: { text: string; speed?: number }) {
  const [displayed, setDisplayed] = useState('');

  useEffect(() => {
    setDisplayed('');
    let i = 0;
    const id = setInterval(() => {
      setDisplayed(text.slice(0, ++i));
      if (i >= text.length) clearInterval(id);
    }, speed);
    return () => clearInterval(id);
  }, [text, speed]);

  return (
    <span>
      {displayed}
      <span className="animate-pulse">|</span>
    </span>
  );
}
```

`speed` is ms per character. 30-50ms feels natural. Below 20ms looks instant. Above 80ms feels sluggish.

## Layout Animations

When elements shift position due to state changes, use Framer Motion's `layout` prop so the movement animates instead of snapping.

```tsx
import { motion, AnimatePresence } from 'framer-motion';

function FilteredList({ items }: { items: { id: string; name: string }[] }) {
  return (
    <ul>
      <AnimatePresence>
        {items.map((item) => (
          <motion.li
            key={item.id}
            layout
            initial={{ opacity: 0, scale: 0.9 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.9 }}
            transition={{ type: 'spring', stiffness: 350, damping: 25 }}
          >
            {item.name}
          </motion.li>
        ))}
      </AnimatePresence>
    </ul>
  );
}
```

The `layout` prop makes the item animate its position when the list reorders. Always pair with `AnimatePresence` for add/remove.

## State Transitions (Loading → Content → Error)

```tsx
import { AnimatePresence, motion } from 'framer-motion';

type State = 'loading' | 'content' | 'error';

const fade = {
  initial: { opacity: 0 },
  animate: { opacity: 1, transition: { duration: 0.2 } },
  exit:    { opacity: 0, transition: { duration: 0.15 } },
};

function AsyncView({ state, data, error }: { state: State; data?: unknown; error?: string }) {
  return (
    <AnimatePresence mode="wait">
      {state === 'loading' && (
        <motion.div key="loading" {...fade}><Skeleton /></motion.div>
      )}
      {state === 'content' && (
        <motion.div key="content" {...fade}>{/* render data */}</motion.div>
      )}
      {state === 'error' && (
        <motion.div key="error" {...fade}><ErrorState message={error} /></motion.div>
      )}
    </AnimatePresence>
  );
}
```

`mode="wait"` ensures the exiting state fades out before the entering state fades in - prevents visual overlap.

## Rubber-banding / Drag Constraints

When dragging past a boundary, show resistance rather than hard-stopping.

```tsx
import { motion, useDragControls } from 'framer-motion';

function DraggableSheet({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      drag="y"
      dragConstraints={{ top: 0, bottom: 0 }}
      dragElastic={0.2}
      onDragEnd={(_, info) => {
        if (info.offset.y > 100) {
          // dismiss
        }
      }}
    >
      {children}
    </motion.div>
  );
}
```

`dragElastic` controls rubber-band resistance. 0 = no elasticity (hard stop), 1 = full elasticity (no resistance). 0.15-0.25 feels natural for bottom sheets.

## Performance Rules

- Only animate `transform` (translate, scale, rotate) and `opacity` - these run on the GPU compositor thread
- Never animate `width`, `height`, `top`, `left`, `padding`, `margin` - these trigger layout recalc every frame
- Use `will-change: transform` sparingly - only on elements that animate repeatedly (infinite loaders). Overuse causes memory pressure
- Framer Motion's `layout` prop is the exception to the height/width rule - it uses FLIP to compute transforms internally
- Test on a throttled CPU (Chrome DevTools: Performance > CPU throttling) before shipping

## Common Gotchas

1. **AnimatePresence requires a key** - the child must have a stable `key` prop. Without it, React won't unmount/remount, and exit animations never fire.

2. **mode="wait" vs mode="popLayout"** - `wait` is sequential (exit then enter). `popLayout` is concurrent but removes the exiting element from layout immediately. Use `popLayout` for lists, `wait` for full-view state swaps.

3. **Exit animations require the parent to keep rendering** - `AnimatePresence` must wrap the conditional render, not be inside it. Common mistake: putting `AnimatePresence` inside the `{condition && ...}`.

4. **Springs don't have a fixed duration** - they settle based on physics. Don't set `duration` on a spring transition - it's ignored. Use `stiffness`, `damping`, `mass` instead.

5. **Skeleton dimensions must match real content** - if the skeleton is a different height than the content it replaces, you'll get a layout shift on load. Build the skeleton by inspecting the rendered real component.

6. **Number ticker flickers on fast updates** - debounce the value if it changes faster than the animation duration. Animate only the final value in a burst.

7. **Typewriter effect breaks on fast text changes** - the `useEffect` cleanup cancels the interval, but `setDisplayed('')` runs after the new effect starts. Add a guard: `if (i > text.length) return` inside the interval callback.

8. **`transform-origin` defaults to center** - dropdowns, tooltips, and context menus that grow from an edge need explicit `transformOrigin` or they'll scale from the wrong point.

## Source of Truth

- Framer Motion docs: https://www.framer.com/motion/
- animations.dev vocabulary: https://animations.dev/vocabulary
