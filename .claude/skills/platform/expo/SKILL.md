---
name: expo
description: Expo mobile development for React Native apps. Trigger words - expo, mobile app, ios app, android app, ota update, expo router, expo camera, push notification mobile
---

# Expo

Streamlined React Native development with managed workflow and built-in features.

## When to Use This Skill

- Rapid mobile app development
- Don't want native configuration complexity
- Need common features (camera, location, notifications)
- Want over-the-air updates
- Building MVP or prototype quickly

## When NOT to Use

- Need very specific native modules not in Expo
- Require deep native customization
- Building complex Bluetooth apps

## Setup

```bash
npx create-expo-app my-app
cd my-app
npx expo start
```

Scan QR code with Expo Go app to test on device.

## Project Structure

```
my-app/
├── app/                # Expo Router screens
│   ├── (tabs)/         # Tab navigator
│   │   ├── index.tsx   # Home tab
│   │   └── _layout.tsx # Tab layout
│   ├── [id].tsx        # Dynamic route
│   └── _layout.tsx     # Root layout
├── components/
├── assets/
└── app.json
```

## Expo Router Navigation

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="[id]" options={{ title: 'Details' }} />
    </Stack>
  );
}
```

```tsx
// app/index.tsx
import { Link } from 'expo-router';
import { View, Text } from 'react-native';

export default function Home() {
  return (
    <View>
      <Text>Welcome</Text>
      <Link href="/user/123">View User</Link>
    </View>
  );
}
```

```tsx
// app/[id].tsx
import { useLocalSearchParams } from 'expo-router';
import { Text } from 'react-native';

export default function Detail() {
  const { id } = useLocalSearchParams();
  return <Text>ID: {id}</Text>;
}
```

### Programmatic Navigation

```tsx
import { router } from 'expo-router';

router.push('/about');
router.push({ pathname: '/user/[id]', params: { id: '123' } });
router.back();
router.replace('/home');
```

## Camera

```bash
npx expo install expo-camera
```

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';

export function Camera() {
  const [permission, requestPermission] = useCameraPermissions();

  if (!permission?.granted) {
    return <Button onPress={requestPermission} title="Grant Permission" />;
  }

  return <CameraView style={{ flex: 1 }} />;
}
```

## Location

```bash
npx expo install expo-location
```

```tsx
import * as Location from 'expo-location';

async function getLocation() {
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') return;

  const location = await Location.getCurrentPositionAsync({});
  console.log(location.coords.latitude, location.coords.longitude);
}
```

## Image Picker

```bash
npx expo install expo-image-picker
```

```tsx
import * as ImagePicker from 'expo-image-picker';

async function pickImage() {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    quality: 1,
  });

  if (!result.canceled) {
    setImage(result.assets[0].uri);
  }
}
```

## Secure Storage

```bash
npx expo install expo-secure-store
```

```tsx
import * as SecureStore from 'expo-secure-store';

// Save
await SecureStore.setItemAsync('authToken', token);

// Get
const token = await SecureStore.getItemAsync('authToken');

// Delete
await SecureStore.deleteItemAsync('authToken');
```

## Tab Navigator

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color }) => <Text style={{ color }}>🏠</Text>,
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color }) => <Text style={{ color }}>👤</Text>,
        }}
      />
    </Tabs>
  );
}
```

## EAS Build & Deploy

```bash
npm install -g eas-cli
eas login

# Build
eas build --platform ios
eas build --platform android

# Submit to stores
eas submit --platform ios
eas submit --platform android

# OTA update
eas update --branch production --message "Bug fixes"
```

## Environment Variables

```typescript
// app.config.ts
export default ({ config }) => ({
  ...config,
  extra: {
    apiUrl: process.env.API_URL,
  },
});
```

```typescript
import Constants from 'expo-constants';
const API_URL = Constants.expoConfig?.extra?.apiUrl;
```

## Tips

- Use `npx expo install` for compatible package versions
- Test on real device with Expo Go app
- Use EAS Build for app store releases
- Expo Router provides file-based routing like Next.js

---

## Advanced Expo Router Features

### 1. API Routes

Create serverless API endpoints inside your Expo app. Code runs on the server, not in the app.

```
app/
├── api/
│   └── hello+api.ts    # +api suffix makes it a server route
```

```tsx
// app/api/hello+api.ts
export function GET(request: Request) {
  return Response.json({ time: new Date().toISOString() });
}

export function POST(request: Request) {
  const body = await request.json();
  return Response.json({ received: body });
}
```

```tsx
// Fetching from your app
const response = await fetch('/api/hello');
const data = await response.json();
```

Use cases:
- AI completions with secret API keys
- Server-side web scraping
- Database operations with admin tokens
- Any code requiring secrets (safe on server)

Deploy with EAS Hosting. Set origin in app.json for production.

### 2. Bottom Sheets with Form Sheet

Native bottom sheets without external packages.

```tsx
// app/_layout.tsx
<Stack.Screen
  name="modal"
  options={{
    presentation: 'formSheet',
    sheetCornerRadius: 24,
    sheetGrabberVisible: true,
    sheetAllowedDetents: [0.5, 1.0],  // 50% and 100% height
  }}
/>
```

Other presentation options:
- `modal` - Standard modal
- `transparentModal` - Modal with transparent background
- `fullScreenModal` - Covers entire screen
- `formSheet` - Native bottom sheet (recommended)

### 3. Route Groups and Array Syntax

Group related files without affecting URLs using parentheses:

```
app/
├── (auth)/           # Group - won't appear in URL
│   ├── login.tsx     # /login
│   └── register.tsx  # /register
├── (tabs)/
│   └── index.tsx     # /
```

Reuse pages across multiple routes with array syntax:

```
app/
├── (home,profile)/   # Array syntax
│   └── [id].tsx      # Accessible as /home/123 AND /profile/123
```

```tsx
// Check which route was used
import { useSegments } from 'expo-router';

export default function Page() {
  const segments = useSegments();
  // segments[0] will be 'home' or 'profile'
}
```

### 4. Protected Routes (Expo Router v5+)

No more nested layout redirects. Use protected groups:

```tsx
// app/_layout.tsx
import { Stack, useRootNavigation } from 'expo-router';

export const unstable_settings = {
  initialRouteName: '(public)',
};

// Define guards for route groups
export function useProtectedRoute(user: User | null) {
  const segments = useSegments();
  const router = useRouter();

  useEffect(() => {
    const inAuthGroup = segments[0] === '(auth)';
    if (!user && !inAuthGroup) {
      router.replace('/login');
    } else if (user && inAuthGroup) {
      router.replace('/');
    }
  }, [user, segments]);
}
```

Better approach with Expo Router v5 guards:

```tsx
// app/(protected)/_layout.tsx
import { Redirect, Slot } from 'expo-router';
import { useAuth } from '@/hooks/useAuth';

export default function ProtectedLayout() {
  const { user } = useAuth();

  if (!user) {
    return <Redirect href="/login" />;
  }

  return <Slot />;
}
```

Also useful for:
- Onboarding flows (check if user completed intro)
- Feature flags
- Subscription gates

### 5. Link Preview (iOS, Expo Router v6+)

Show native link previews on long press (iOS only):

```tsx
import { Link } from 'expo-router';

<Link href="/details/123" asChild>
  <LinkTrigger>
    <Text>View Details</Text>
  </LinkTrigger>

  <LinkPreview>
    <CustomPreviewContent />
  </LinkPreview>

  <LinkMenu>
    <LinkMenuButton title="Share" icon="square.and.arrow.up" />
    <LinkMenuButton title="Copy" icon="doc.on.doc" />
  </LinkMenu>
</Link>
```

```tsx
// Detect if rendering in preview mode
import { useIsPreview } from 'expo-router';

export default function DetailsPage() {
  const isPreview = useIsPreview();

  return (
    <View>
      {isPreview ? (
        <Text>Preview content</Text>
      ) : (
        <Text>Full page content</Text>
      )}
    </View>
  );
}
```

### 6. Native Tab Bars

Use platform-native tabs instead of JavaScript tabs. Critical for iOS 26 liquid glass support.

```tsx
// app/(tabs)/_layout.tsx
import { unstable_NativeTabs as NativeTabs } from 'expo-router';

export default function TabLayout() {
  return (
    <NativeTabs>
      <NativeTabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: { sfSymbol: 'house.fill' },  // SF Symbols on iOS
          tabBarBadge: '3',  // Badge support
        }}
      />
      <NativeTabs.Screen
        name="search"
        options={{
          title: 'Search',
          tabBarIcon: { sfSymbol: 'magnifyingglass' },
        }}
      />
    </NativeTabs>
  );
}
```

Benefits:
- Native glass effects on iOS 26
- Smooth native animations
- SF Symbol support
- Native badges
- Native search integration

### 7. React Server Components (SDK 52+, Beta)

Run React components on the server. Stream results to client.

```tsx
// app/components/ServerQuote.tsx
'use server';
import 'server-only';

export default async function ServerQuote() {
  const quote = await fetchQuoteFromAPI();  // Uses secret key
  return <Text>{quote}</Text>;
}
```

```tsx
// app/index.tsx
import { Suspense } from 'react';
import ServerQuote from './components/ServerQuote';

export default function Home() {
  return (
    <View>
      <Suspense fallback={<Text>Loading...</Text>}>
        <ServerQuote />
      </Suspense>
    </View>
  );
}
```

Use cases:
- Secret API keys (safe on server)
- Dynamic content updates without app store
- Heavy data transformations
- Database queries

Deploy with EAS Hosting.

### 8. Custom Headless Tab Bars

Full control over tab bar styling with headless UI:

```tsx
// app/(tabs)/_layout.tsx
import { Tabs, TabSlot, TabList, TabTrigger } from 'expo-router/ui';

export default function TabLayout() {
  return (
    <Tabs>
      <TabSlot />  {/* Renders current tab content */}

      <TabList style={styles.tabBar}>
        <TabTrigger name="index" style={styles.tab}>
          {({ isFocused }) => (
            <View style={[styles.tabContent, isFocused && styles.focused]}>
              <HomeIcon color={isFocused ? '#007AFF' : '#8E8E93'} />
              <Text>Home</Text>
            </View>
          )}
        </TabTrigger>

        <TabTrigger name="profile" style={styles.tab}>
          {({ isFocused }) => (
            <View style={[styles.tabContent, isFocused && styles.focused]}>
              <ProfileIcon color={isFocused ? '#007AFF' : '#8E8E93'} />
              <Text>Profile</Text>
            </View>
          )}
        </TabTrigger>
      </TabList>
    </Tabs>
  );
}
```

Use when you need:
- Floating tab bars
- Custom animations
- Non-standard layouts
- Center action buttons

### 9. Static Site Generation (Web)

Build static websites with Expo Router:

```tsx
// app/topics/[slug].tsx
export async function generateStaticParams() {
  return [
    { slug: 'react-native' },
    { slug: 'expo-router' },
    { slug: 'typescript' },
  ];
}

export default function TopicPage() {
  const { slug } = useLocalSearchParams();
  return <Text>Topic: {slug}</Text>;
}
```

```bash
# Export static site
npx expo export --platform web

# Creates dist/ folder with:
# - index.html
# - topics/react-native.html
# - topics/expo-router.html
# - topics/typescript.html
```

Use for:
- Marketing pages with SEO
- Documentation sites
- Landing pages
- Blog posts

### 10. Deep Link Rewriting

Intercept and transform incoming deep links:

```tsx
// app/+native-intent.tsx
import { useRouter } from 'expo-router';
import { useEffect } from 'react';

export default function NativeIntent({ path }: { path: string }) {
  const router = useRouter();

  useEffect(() => {
    // Transform incoming URLs
    if (path.includes('/invite/')) {
      const code = path.split('/invite/')[1];
      router.replace(`/redeem?code=${code}`);
      return;
    }

    if (path.includes('/share/')) {
      // Handle share extension
      showToast('Content shared!');
    }

    // Default: navigate to path
    router.replace(path);
  }, [path]);

  return null;
}
```

Triggered by:
- Deep links (yourapp://path)
- Universal links (yourapp.com/path)
- Share extensions
- Widgets

### 11. In-App Purchases with RevenueCat

Fast setup using RevenueCat's Test Store and MCP integration.

#### Why RevenueCat + Test Store

- **Test Store**: Built-in testing environment - no App Store Connect or Play Store setup needed
- **Works in simulator**: Unlike native StoreKit testing
- **MCP Integration**: AI agents can create products, offerings, and entitlements automatically

#### Setup

```bash
# Install the package
npx expo install react-native-purchases
```

```tsx
// lib/revenue-cat.ts
import Purchases from 'react-native-purchases';

const API_KEY = process.env.EXPO_PUBLIC_REVENUECAT_API_KEY;

export async function initRevenueCat() {
  await Purchases.configure({ apiKey: API_KEY });
}

export async function getOfferings() {
  const offerings = await Purchases.getOfferings();
  return offerings.current;
}

export async function purchasePackage(pkg: PurchasesPackage) {
  const { customerInfo } = await Purchases.purchasePackage(pkg);
  return customerInfo;
}

export async function restorePurchases() {
  const customerInfo = await Purchases.restorePurchases();
  return customerInfo;
}

export async function checkEntitlement(entitlementId: string) {
  const customerInfo = await Purchases.getCustomerInfo();
  return customerInfo.entitlements.active[entitlementId] !== undefined;
}
```

```tsx
// app/_layout.tsx
import { useEffect } from 'react';
import { initRevenueCat } from '@/lib/revenue-cat';

export default function RootLayout() {
  useEffect(() => {
    initRevenueCat();
  }, []);

  return <Stack />;
}
```

#### RevenueCat Test Store Setup

1. Create project at revenuecat.com
2. Go to API Keys - Test Store key is auto-generated
3. Use Test Store API key during development (works in simulator)
4. No need to connect App Store Connect or Play Store initially

#### MCP Integration for AI-Assisted Setup

RevenueCat provides an MCP (Model Context Protocol) with 35+ tools. AI agents can:
- Create products, offerings, entitlements
- Configure iOS and Android apps
- Mirror test store config to production

```bash
# In Cursor/VS Code with RevenueCat MCP extension
# Set your secret API key (create in RevenueCat dashboard with v2 API, full read/write access)
revenuecat set project secret key <YOUR_SECRET_KEY>
```

**Important**: Add `.cursor/mcp.json` to `.gitignore` - contains secret key.

Example prompts for AI:
- "Create products for a 100 coin pack consumable and premium lifetime non-consumable"
- "Create a new iOS app and attach existing products from test store"
- "Get my RevenueCat project details"

#### Product Types

```tsx
// Consumable (coins, credits)
const CONSUMABLE_PRODUCT = 'rc_consumable_100_coins';

// Non-consumable (lifetime unlock)
const LIFETIME_PRODUCT = 'rc_premium_lifetime';

// Subscription
const MONTHLY_SUB = 'rc_monthly_premium';
```

#### Purchase UI Example

```tsx
import { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';
import Purchases, { PurchasesOffering } from 'react-native-purchases';

export default function PurchaseScreen() {
  const [offering, setOffering] = useState<PurchasesOffering | null>(null);
  const [isPremium, setIsPremium] = useState(false);

  useEffect(() => {
    async function load() {
      const offerings = await Purchases.getOfferings();
      setOffering(offerings.current);

      const info = await Purchases.getCustomerInfo();
      setIsPremium(info.entitlements.active['premium'] !== undefined);
    }
    load();
  }, []);

  async function handlePurchase(pkg: PurchasesPackage) {
    try {
      const { customerInfo } = await Purchases.purchasePackage(pkg);
      setIsPremium(customerInfo.entitlements.active['premium'] !== undefined);
    } catch (e) {
      if (!e.userCancelled) console.error(e);
    }
  }

  return (
    <View>
      <Text>{isPremium ? 'Premium User' : 'Free User'}</Text>

      {offering?.availablePackages.map((pkg) => (
        <Button
          key={pkg.identifier}
          title={`${pkg.product.title} - ${pkg.product.priceString}`}
          onPress={() => handlePurchase(pkg)}
        />
      ))}
    </View>
  );
}
```

#### Going to Production

1. Create iOS app in RevenueCat, connect to App Store Connect
2. Create Android app, connect to Play Store (use RevenueCat's Google Cloud script)
3. Use MCP to mirror test store products to production apps
4. RevenueCat can create products directly in App Store Connect

---

## How to Verify

### Quick Checks
- `npx expo start` opens QR code
- App loads in Expo Go
- Navigation between screens works

### Common Issues
- "Module not found": Use `npx expo install` not `npm install`
- Permission denied: Request permission before using feature
- Build fails: Check app.json configuration
