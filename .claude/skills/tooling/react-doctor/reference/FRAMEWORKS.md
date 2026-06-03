# React Doctor - Framework Configurations

Reference for setting up React Doctor per framework, including CLI config suppressions.

## Web React (Vite / React Router / TanStack Start)

### ESLint Config
```js
import reactDoctor from "eslint-plugin-react-doctor";

export default [
  reactDoctor.configs.recommended,
  reactDoctor.configs["tanstack-start"],  // remove if not using TanStack
  reactDoctor.configs["tanstack-query"],  // remove if not using TanStack Query
];
```

### CLI Config (`react-doctor.config.json`)
No suppressions needed for web. Use rule overrides only for project-specific exceptions:

```json
{
  "ignore": {
    "overrides": [
      {
        "files": ["src/routes/**/*.tsx"],
        "rules": ["react-doctor/only-export-components"]
      }
    ]
  }
}
```

## Next.js

### ESLint Config
```js
import reactDoctor from "eslint-plugin-react-doctor";

export default [
  reactDoctor.configs.recommended,
  reactDoctor.configs.next,
];
```

### CLI Config
Suppress TanStack rules if not in use:

```json
{
  "ignore": {
    "rules": [
      "react-doctor/tanstack-start-loader-parallel-fetch",
      "react-doctor/tanstack-start-no-anchor-element",
      "react-doctor/tanstack-start-no-navigate-in-render",
      "react-doctor/tanstack-start-no-useeffect-fetch"
    ]
  }
}
```

## React Native / Expo

### ESLint Config
```js
import reactDoctor from "eslint-plugin-react-doctor";

export default [
  reactDoctor.configs.recommended,
  reactDoctor.configs["react-native"],
];
```

The `react-native` config automatically enables the 28 RN-specific rules and suppresses web-only rules.

### CLI Config (`react-doctor.config.json`)
When using the CLI scanner (`npx react-doctor@latest`) on a React Native project, suppress web-only rules manually:

```json
{
  "ignore": {
    "rules": [
      "react-doctor/anchor-has-content",
      "react-doctor/anchor-is-valid",
      "react-doctor/iframe-missing-sandbox",
      "react-doctor/rendering-hydration-mismatch-time",
      "react-doctor/rendering-hydration-no-flicker",
      "react-doctor/server-sequential-independent-await",
      "react-doctor/tanstack-start-loader-parallel-fetch",
      "react-doctor/tanstack-start-no-anchor-element",
      "react-doctor/tanstack-start-no-navigate-in-render",
      "react-doctor/tanstack-start-no-useeffect-fetch",
      "react-doctor/prefer-dynamic-import",
      "react-doctor/no-gradient-text",
      "react-doctor/no-pure-black-background",
      "react-doctor/client-passive-event-listeners",
      "react-doctor/html-no-nested-interactive",
      "react-doctor/interactive-supports-focus",
      "react-doctor/no-noninteractive-element-interactions",
      "react-doctor/no-noninteractive-tabindex",
      "react-doctor/no-static-element-interactions",
      "react-doctor/prefer-tag-over-role"
    ]
  }
}
```

### React Native Specific Rules (28 rules)
The RN config enforces mobile-specific patterns:

- **Animated properties** - GPU-friendly transforms over layout animations
- **List virtualization** - FlatList/SectionList patterns, FlashList recommendations
- **Gesture handling** - Proper gesture responder usage
- **Reanimated integration** - useAnimatedStyle, withTiming, withSpring patterns
- **Platform-specific code** - Platform.select, platform file extensions
- **StyleSheet usage** - StyleSheet.create over inline styles for performance

## Expo-Specific Notes

Expo projects use the same `react-native` config. Additional suppressions for Expo Router:

```json
{
  "ignore": {
    "overrides": [
      {
        "files": ["app/**/*.tsx"],
        "rules": ["react-doctor/only-export-components"]
      }
    ]
  }
}
```

## All Frameworks - Common Suppressions

These are typically safe to disable project-wide when they conflict with shadcn/ui or similar component libraries:

```json
{
  "ignore": {
    "rules": [
      "react-doctor/no-array-index-as-key",
      "react-doctor/button-has-type"
    ],
    "overrides": [
      {
        "files": ["components/ui/**/*.tsx"],
        "rules": [
          "react-doctor/only-export-components",
          "react-doctor/no-multi-comp"
        ]
      }
    ]
  }
}
```
