# Bluesky in sootsim

Browser-based rendering of the Bluesky app via sootsim's canvas engine.

## How to run

```sh
# 1. compile translations (one-time, or after updating locale files)
yarn intl:compile

# 2. start metro bundler on port 8082
npx expo start --port 8082

# 3. in soot repo, start sootsim dev server
cd ~/soot && bun dev

# 4. open in browser
open "http://localhost:5173/?bundle=http%3A%2F%2Flocalhost%3A8082%2Findex.bundle%3Fplatform%3Dios%26dev%3Dtrue%26minify%3Dfalse"
```

## What works

- **Login screen**: fully rendered with form fields, keyboard input, and navigation
- **Authentication**: login via email/password works, session tokens persist in localStorage
- **Post-login shell**: mobileShellView renders with the full app structure
- **Bottom tab bar**: all 5 tabs render (Home, Search, Messages, Notifications, Profile)
- **Notification badge**: shows unread count
- **Drawer menu**: full navigation drawer with Explore, Home, Chat, Notifications, Feeds, Lists, Saved, Profile, Settings
- **Text rendering**: all text content renders on canvas
- **SVG icons**: bottom bar icons and UI icons render correctly
- **Form interaction**: text inputs, keyboard, tap handlers all work
- **API calls**: CORS is fully open on bsky.social, all XRPC calls work from browser
- **Session persistence**: session data stored in localStorage via MMKV/AsyncStorage stubs

## What doesn't work

- **Home feed**: followingFeedPage renders at 0 height, feed items don't appear. The Animated.FlatList needs better integration with sootsim's FlatList component (data/renderItem passthrough).
- **Images/avatars**: expo-image stub renders placeholder views, not actual images from CDN
- **Age assurance flow**: bypassed via bundle patch (requires native browser redirects)
- **Push notifications**: stubbed, no real push tokens in browser
- **Drawer interaction**: DrawerLayout bypassed via bundle patch (native Reanimated animation issues), drawer renders but can't be swiped
- **Video playback**: no video support in canvas renderer
- **Camera/media picker**: stubbed, returns empty results

## Known issues

1. Feed content has 0 height: the feed page renders but its FlatList content doesn't size. Likely a props passthrough issue in AnimatedFlatList -> sootsim FlatList.
2. "Please verify your email" dialog appears as an overlay — doesn't block the app but shows in the tree.
3. Some Reanimated worklet functions need `_WORKLET` global (fixed in sootsim).

## sootsim patches applied (automatic)

These patches are applied by sootsim's ExternalApp.tsx bundle transformer at load time. No changes to the Bluesky source code are needed.

- Skip Splash screen animation (renders children directly)
- Bypass AgeAssurance NoAccessScreen and RedirectOverlay
- Force useAgeAssuranceState to return Full access
- Skip DrawerLayout (renders children directly)
- Skip Statsig loading wait
- Bypass PersistGate bootstrapping
- Guard LiveEventsProvider data.feeds access
- Noop Animated.addWhitelistedUIProps/NativeProps
- Replace import.meta.url with location.href
- Fix React CJS production/development branching
- Skip storybook initialization
- Stub Shopify performance module
- Skip useStarterPackEntry wait
- Start directly at login screen (skip landing page)
- Bypass Apollo client null gate
- Fix lingui I18nProvider null rendering

## Architecture notes

sootsim loads the metro bundle (`index.bundle?platform=ios`) and executes it in a browser `<script>` tag. Before execution, it:

1. Injects sootsim's react-native module stubs (View, Text, ScrollView, etc.)
2. Patches the bundle text for known compatibility issues
3. Binds to the app's React instance
4. Runs deferred entry points
5. Captures the root component from AppRegistry

The app uses @atproto/api for authentication (JWT-based, not cookie-based). All API calls go directly to bsky.social which has `Access-Control-Allow-Origin: *`.

Storage is handled via sootsim stubs:
- MMKV -> localStorage
- AsyncStorage -> localStorage
- expo-secure-store -> localStorage
