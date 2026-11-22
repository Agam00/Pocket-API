# 📒 Pocket API — Learning Notes

> My personal notes while building Pocket API (see `POCKET_API_BLUEPRINT.md`).
> Each section = one phase. Concepts are explained using **real code from this project**.
> Revisit the **Self-check** at the end of each phase before interviews.

---

## Table of Contents

- [Phase 0 — Setup & Fundamentals](#phase-0--setup--fundamentals)
  - [0.1 What is React Native?](#01-what-is-react-native)
  - [0.2 What is Expo?](#02-what-is-expo)
  - [0.3 How code reaches the phone (Metro, Hermes, New Architecture)](#03-how-code-reaches-the-phone)
  - [0.4 Expo Go vs Development Build vs Production Build](#04-expo-go-vs-development-build-vs-production-build)
  - [0.5 Continuous Native Generation (why there's no android/ or ios/ folder)](#05-continuous-native-generation-cng)
  - [0.6 Project file structure](#06-project-file-structure)
  - [0.7 package.json explained](#07-packagejson-explained)
  - [0.8 app.json explained](#08-appjson-explained)
  - [0.9 tsconfig.json & TypeScript strict mode](#09-tsconfigjson--typescript-strict-mode)
  - [0.10 ESLint & Prettier](#010-eslint--prettier)
  - [0.11 Expo Router basics (file-based routing)](#011-expo-router-basics)
  - [0.12 How the app boots — step by step](#012-how-the-app-boots--step-by-step)
  - [0.13 Styling & Flexbox basics](#013-styling--flexbox-basics)
  - [0.14 Platform-specific code](#014-platform-specific-code)
  - [0.15 Hooks & custom hooks (first look)](#015-hooks--custom-hooks-first-look)
  - [0.16 Commands cheat sheet](#016-commands-cheat-sheet)
  - [0.17 Glossary](#017-glossary)
  - [0.18 Self-check (with answers)](#018-self-check-with-answers)
  - [0.19 Phase 0 status](#019-phase-0-status)
  - [0.20 Deep dive: `"main": "expo-router/entry"`](#020-deep-dive-main-expo-routerentry)
  - [0.21 Resetting the project (blank slate)](#021-resetting-the-project-blank-slate)

---

# Phase 0 — Setup & Fundamentals

## 0.1 What is React Native?

**React Native (RN)** is a framework by Meta that lets you build **mobile apps using React and JavaScript/TypeScript**.

The key idea: **your UI is real native UI**, not a web page.

| You write (React Native) | Android shows            | iOS shows                |
| ------------------------ | ------------------------ | ------------------------ |
| `<View>`                 | `android.view.ViewGroup` | `UIView`                 |
| `<Text>`                 | `TextView`               | `UILabel`-like text view |
| `<TextInput>`            | `EditText`               | `UITextField`            |
| `<ScrollView>`           | `ScrollView`             | `UIScrollView`           |
| `<Image>`                | `ImageView`              | `UIImageView`            |

So RN is **not** a WebView/hybrid app (like Cordova/Ionic). That's why RN apps feel native.

**What you write:**

- **Logic** (state, networking, data) → JavaScript/TypeScript
- **UI description** → React components (JSX)
- **Rendering** → done by the native platform

**Versions in this project** (from `package.json`):

- `react-native`: **0.88.1**
- `react`: **19.3.0**

---

## 0.2 What is Expo?

**Expo** is a framework + set of tools built **on top of** React Native. Think of it like this:

```
React        → the UI library (components, hooks, state)
React Native → React for mobile (native views)
Expo         → batteries-included toolkit around React Native
```

What Expo gives you:

1. **Expo SDK** — a big set of ready-made native modules, all version-matched to each other:
   - `expo-device` (device info), `expo-image` (fast images), `expo-splash-screen`,
   - later in this project: `expo-sqlite`, `expo-secure-store`, `expo-clipboard`, `expo-haptics`, `expo-local-authentication`, etc.
2. **Expo Router** — file-based navigation (like Next.js, but for mobile).
3. **Expo CLI** — `npx expo start`, `npx expo install`, `npx expo lint`.
4. **EAS (Expo Application Services)** — cloud builds (`eas build`), store submission (`eas submit`), over-the-air updates (`eas update`). No need for Android Studio / Xcode on your machine to ship.

**SDK versions:** Expo releases a new **SDK** a few times a year. Every `expo-*` package version is tied to the SDK.

- This project: **Expo SDK 58** (`"expo": "~58.0.23"`).
- That's why you **always install packages with `npx expo install <pkg>`** instead of `npm install` — it picks the version compatible with SDK 58.

> ⚠️ Expo changes fast. When looking things up, use the docs for **v58**: `https://docs.expo.dev/versions/v58.0.0/`

---

## 0.3 How code reaches the phone

### The full pipeline

```
 ┌──────────────────────────── YOUR PC ────────────────────────────┐
 │                                                                  │
 │   src/app/index.tsx, src/components/*.tsx, ...                   │
 │              │                                                   │
 │              ▼                                                   │
 │   ┌─────────────────── METRO BUNDLER ───────────────────┐        │
 │   │ 1. Resolution  – start at entry, follow every import │        │
 │   │ 2. Transform   – Babel: TS/JSX → plain JavaScript    │        │
 │   │ 3. Serialize   – combine all modules into ONE bundle │        │
 │   │ 4. Serve       – HTTP dev server (port 8081)         │        │
 │   └──────────────────────────────────────────────────────┘        │
 └──────────────────────────────┬───────────────────────────────────┘
                                │  Wi-Fi / USB (QR code)
 ┌──────────────────────────────▼──────────── PHONE ────────────────┐
 │  Expo Go  /  Development build                                   │
 │                                                                  │
 │   HERMES (JS engine)  ── runs the bundle                         │
 │        │                                                         │
 │        ▼                                                         │
 │   React builds a tree of components                              │
 │        │                                                         │
 │        ▼  (New Architecture: JSI — direct JS ↔ C++ calls)         │
 │   FABRIC renderer → creates real native views on screen          │
 │   TURBO MODULES  → native features (camera, storage, etc.)       │
 └──────────────────────────────────────────────────────────────────┘
```

### Metro — the bundler

**Metro** is React Native's JavaScript bundler. It runs on your computer when you do `npx expo start`.

Its 4 jobs:

1. **Resolution** — Starts at the **entry file** (`expo-router/entry`) and follows every `import`/`require` to find all files your app needs. It also picks platform files (`.ios.tsx`, `.android.tsx`, `.web.tsx`) and handles aliases like `@/`.
2. **Transformation** — Uses **Babel** to convert TypeScript + JSX into plain JavaScript the phone can run. (Note: Babel **strips types**, it doesn't check them. That's why `tsc --noEmit` is a separate step.)
3. **Serialization** — Combines all the transformed modules into **one bundle**.
4. **Serving** — Runs an HTTP server that the phone downloads the bundle from.

**Fast Refresh:** Metro watches your files. When you save, it re-transforms **only the changed file** and pushes it to the phone. React re-renders that component and **keeps its state** (e.g., text you typed stays). If you edit a non-component file, it may do a full reload instead.

> 🎤 **Interview answer — "What does Metro do?"**
> "Metro is the JavaScript bundler for React Native. Starting from the entry file, it resolves every import, transforms TypeScript and JSX into plain JavaScript with Babel, and serializes everything into a single bundle that it serves to the device. In development it watches files and uses Fast Refresh to update only what changed while keeping component state."

### Hermes — the JavaScript engine

- **Hermes** is a JS engine made by Meta specifically for React Native.
- It's the **default** engine. It runs your bundle on the phone.
- Benefits: faster startup, lower memory. In production builds, JS is pre-compiled to **bytecode** so the phone doesn't parse raw JS at startup.

### The New Architecture (JSI, Fabric, TurboModules)

Old RN had a **"Bridge"**: JS and native talked by sending **JSON messages asynchronously** across a queue. Slow for heavy communication.

The **New Architecture** (default in modern RN) replaces that:

| Piece                          | What it does                                                                                                                     |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **JSI** (JavaScript Interface) | Lets JS hold references to C++ objects and **call native functions directly** (even synchronously). No JSON serialization.       |
| **Fabric**                     | The new **rendering system**. Takes React's component tree and creates/updates native views. Supports concurrent React features. |
| **TurboModules**               | The new **native module system**. Modules are loaded lazily (only when used) and called through JSI.                             |
| **Codegen**                    | Generates type-safe glue code between JS and native from TypeScript specs.                                                       |

> 🎤 **Short version:** "The New Architecture replaces the async JSON bridge with JSI, so JavaScript can call native code directly. Fabric is the new renderer and TurboModules are the new lazy-loaded native modules."

### Threads (good to know)

- **JS thread** — runs your React code and business logic.
- **UI / main thread** — draws native views and handles touches.
- If the JS thread is busy (e.g., parsing a 5 MB JSON), your **React** UI updates freeze. That's why Phase 6 (large responses) and Reanimated (animations on the UI thread) matter.

---

## 0.4 Expo Go vs Development Build vs Production Build

|               | **Expo Go**                                  | **Development Build**                                                  | **Production Build**                   |
| ------------- | -------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------- |
| What          | A ready-made app from Play Store / App Store | **Your own app**, with dev tools inside (`expo-dev-client`)            | Your final app for users               |
| Who builds it | Expo                                         | You (`eas build --profile development` or `npx expo run:android`)      | You (`eas build --profile production`) |
| Native code   | **Only** what Expo bundled into Expo Go      | **Any** native library you install                                     | Any                                    |
| Loads JS from | Metro (your PC)                              | Metro (your PC)                                                        | Bundle embedded inside the app         |
| Fast Refresh  | ✅                                           | ✅                                                                     | ❌                                     |
| App name/icon | "Expo Go"                                    | Your app                                                               | Your app                               |
| Use when      | Learning, prototyping                        | You add a library with custom native code, or need exact native config | Shipping to users / stores             |

**Rule of thumb:**

- Start with **Expo Go** (fastest).
- When you install a library that isn't included in Expo Go → you **need a development build**. Expo Go can't magically gain new native code; native code has to be compiled into the app.

---

## 0.5 Continuous Native Generation (CNG)

You'll notice there's **no `android/` or `ios/` folder** in this project. That's intentional.

- Expo **generates** those native projects when needed (on `npx expo prebuild`, `npx expo run:*`, or during EAS Build).
- The source of truth is **`app.json` + config plugins**, not the native folders.
- **Config plugins** = small scripts that modify the generated native project (e.g., add permissions to `AndroidManifest.xml`, add keys to `Info.plist`).

**Rule:** never create or edit `android/`/`ios/` by hand. Change `app.json` or add a config plugin instead. Benefits: upgrading SDKs is much easier, native config is reproducible.

---

## 0.6 Project file structure

```
pocket-api/
├── package.json        ← dependencies, scripts, entry point
├── package-lock.json   ← exact installed versions (commit this!)
├── app.json            ← app config: name, icon, scheme, plugins
├── tsconfig.json       ← TypeScript settings
├── eslint.config.js    ← lint rules
├── .prettierrc.json    ← formatting rules
├── .prettierignore     ← files Prettier skips
├── .gitignore          ← files Git skips (node_modules, .expo, ...)
├── expo-env.d.ts       ← Expo type declarations (auto-generated, don't edit)
├── AGENTS.md           ← instructions for AI coding assistants
├── README.md           ← project readme (still template)
├── LICENSE
├── .expo/              ← local Expo cache (generated, git-ignored)
├── .vscode/            ← editor settings
├── node_modules/       ← installed packages (never edit, never commit)
├── assets/             ← images, icons, splash images, fonts
├── scripts/
│   └── reset-project.js← moves starter code out, leaves blank src/app
└── src/
    ├── global.css               ← global styles (used on web)
    ├── app/                     ← ⭐ ROUTES. Every file here = a screen
    │   ├── _layout.tsx          ← root layout (wraps all screens)
    │   ├── index.tsx            ← "/"  → Home tab
    │   └── explore.tsx          ← "/explore" → Explore tab
    ├── components/              ← reusable UI (NOT routes)
    │   ├── app-tabs.tsx         ← native bottom tabs (Android/iOS)
    │   ├── app-tabs.web.tsx     ← tabs version for web
    │   ├── animated-icon.tsx (+ .web.tsx, .module.css)
    │   ├── themed-text.tsx      ← Text that follows light/dark theme
    │   ├── themed-view.tsx      ← View that follows light/dark theme
    │   ├── hint-row.tsx, external-link.tsx, web-badge.tsx
    │   └── ui/collapsible.tsx
    ├── constants/
    │   └── theme.ts             ← Colors, Spacing, BottomTabInset, MaxContentWidth
    └── hooks/
        ├── use-color-scheme.ts (+ .web.ts)
        └── use-theme.ts
```

**Golden rule:** `src/app/` holds **only screens/layouts**. Components, hooks, logic, utils go **outside** `src/app/`. Otherwise Expo Router will treat them as routes.

> 📝 The blueprint shows `app/` at the project root. This project uses **`src/app/`** (newer Expo default). Both work; Expo Router supports either. Everything else in the blueprint's structure goes under `src/`.

---

## 0.7 package.json explained

```json
{
  "name": "pocket-api",
  "main": "expo-router/entry",
  "dependencies": { ... },
  "devDependencies": { ... },
  "scripts": { ... }
}
```

### `"main": "expo-router/entry"` — the entry point

This is **where Metro starts**. `expo-router/entry`:

1. Registers the root component of the app.
2. Scans `src/app/` and builds the navigation tree from your files.

So you never write a manual `App.tsx` — the **file system is the app's navigation**.

> 🔎 Full step-by-step breakdown with the real library code: see [0.20 Deep dive](#020-deep-dive-main-expo-routerentry).

### `dependencies` vs `devDependencies`

- **dependencies** — code that ships **inside the app** (react, react-native, expo-router, ...).
- **devDependencies** — tools used only while developing (typescript, eslint, prettier, @types/react).

### Version symbols

- `~58.0.23` → allow **patch** updates only (58.0.x).
- `^9.0.0` → allow **minor + patch** updates (9.x.x).
- `19.3.0` → exact version.

### Notable packages already installed

| Package                                             | Purpose                                                       |
| --------------------------------------------------- | ------------------------------------------------------------- |
| `expo`                                              | Core Expo SDK                                                 |
| `expo-router`                                       | File-based navigation                                         |
| `react-native-screens`                              | Native screen containers (used by navigation for performance) |
| `react-native-safe-area-context`                    | Know where the notch / status bar / home indicator is         |
| `react-native-gesture-handler`                      | Native gestures (swipes, drags) — used in Phase 6             |
| `react-native-reanimated` + `react-native-worklets` | UI-thread animations — Phase 6                                |
| `expo-splash-screen`                                | Control the splash screen                                     |
| `expo-image`                                        | Fast, cached images                                           |
| `expo-device`                                       | Device info (is it a real device? model?)                     |
| `expo-linking`                                      | Deep links — Phase 7                                          |
| `react-native-web` + `react-dom`                    | Run the same app in a browser                                 |

### Scripts

```
npm start          → expo start          (start Metro dev server)
npm run android    → expo start --android
npm run ios        → expo start --ios
npm run web        → expo start --web
npm run lint       → expo lint            (ESLint)
npm run format     → prettier --write .   (auto-format all files)
npm run format:check → prettier --check . (just check, don't change)
npm run typecheck  → tsc --noEmit         (type-check, don't output files)
npm run reset-project → move starter code to /example, blank src/app
```

---

## 0.8 app.json explained

```json
{
  "expo": {
    "name": "pocket-api",
    "slug": "pocket-api",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/icon.png",
    "scheme": "pocketapi",
    "userInterfaceStyle": "automatic",
    "ios": { "icon": "./assets/expo.icon" },
    "android": {
      "adaptiveIcon": { ... },
      "predictiveBackGestureEnabled": false
    },
    "web": { "output": "server", "favicon": "./assets/images/favicon.png" },
    "plugins": ["expo-router", ["expo-splash-screen", { ... }]],
    "experiments": { "typedRoutes": true, "reactCompiler": true }
  }
}
```

| Key                               | Meaning                                                                                                                                                                                |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                            | Name shown under the app icon                                                                                                                                                          |
| `slug`                            | URL-friendly id used by Expo/EAS                                                                                                                                                       |
| `version`                         | User-facing version (1.0.0)                                                                                                                                                            |
| `orientation`                     | Lock to portrait                                                                                                                                                                       |
| `icon`                            | App icon                                                                                                                                                                               |
| `scheme`                          | **Deep link prefix** → `pocketapi://...`. Needed for Phase 7 (`pocketapi://import?curl=...`). Already set ✅                                                                           |
| `userInterfaceStyle: "automatic"` | App follows the phone's light/dark setting                                                                                                                                             |
| `android.adaptiveIcon`            | Android icon layers (foreground/background/monochrome)                                                                                                                                 |
| `plugins`                         | **Config plugins** that modify native projects (see CNG above). `expo-router` sets up linking; `expo-splash-screen` configures the splash color/image                                  |
| `experiments.typedRoutes`         | TypeScript knows all your routes → `router.push('/hsitory')` (typo) becomes a **type error**                                                                                           |
| `experiments.reactCompiler`       | **React Compiler** auto-memoizes components/values at build time, reducing unnecessary re-renders (you'll still learn `useMemo`/`useCallback`/`React.memo` to understand what it does) |

Later (Phase 10) the blueprint moves this to **`app.config.ts`** — same config but in TypeScript, so it can use variables/env vars.

---

## 0.9 tsconfig.json & TypeScript strict mode

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "paths": {
      "@/*": ["./src/*"],
      "@/assets/*": ["./assets/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

### `extends: "expo/tsconfig.base"`

Starts from Expo's recommended settings (JSX mode, module resolution, etc.) and adds ours on top.

### `strict: true` ✅

Turns on a group of strict checks. Most important ones:

| Check                          | Catches                                                  |
| ------------------------------ | -------------------------------------------------------- |
| `strictNullChecks`             | Using something that might be `null`/`undefined`         |
| `noImplicitAny`                | Variables/params with no type that silently become `any` |
| `strictFunctionTypes`          | Unsafe function parameter types                          |
| `strictPropertyInitialization` | Class fields never initialized                           |

Example of what strict mode catches:

```ts
function getLength(s: string | undefined) {
  return s.length; // ❌ Error: 's' is possibly 'undefined'
}

function getLength2(s: string | undefined) {
  return s?.length ?? 0; // ✅ handled
}
```

### `paths` — import aliases

```ts
// Without alias (fragile, ugly):
import { Colors } from '../../../constants/theme';

// With alias:
import { Colors } from '@/constants/theme'; // @/ = src/
```

### Type checking vs running

- Metro/Babel **removes** types but does **not check** them → the app may run even with type errors.
- `npm run typecheck` (`tsc --noEmit`) **checks** types without producing files. Always run it before calling something "done".

---

## 0.10 ESLint & Prettier

|                          | **ESLint**                                                                                       | **Prettier**                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| Job                      | Finds **bugs & bad patterns**                                                                    | Fixes **formatting** only                                     |
| Examples                 | unused variables, hooks called conditionally, missing `useEffect` dependencies, unreachable code | quotes, semicolons, indentation, line length, trailing commas |
| Changes meaning of code? | Can flag logic issues                                                                            | Never — only whitespace/style                                 |

### `eslint.config.js`

```js
const { defineConfig } = require('eslint/config');
const expoConfig = require('eslint-config-expo/flat');
const eslintPluginPrettierRecommended = require('eslint-plugin-prettier/recommended');

module.exports = defineConfig([
  expoConfig, // Expo's rules (React, React Hooks, RN, TS)
  eslintPluginPrettierRecommended, // run Prettier as an ESLint rule + disable conflicting rules
  { ignores: ['dist/*', '.expo/*', 'node_modules/*'] },
]);
```

- This is the new **"flat config"** format (ESLint 9): an array of config objects.
- `eslint-plugin-prettier` → formatting problems show up as lint errors, so **one `npm run lint` checks both**.
- `eslint-config-prettier` (installed) → turns off ESLint style rules that would fight Prettier.

### `.prettierrc.json` — our style

```json
{
  "singleQuote": true, // 'hello' not "hello"
  "trailingComma": "all", // trailing commas everywhere → cleaner git diffs
  "printWidth": 100, // wrap lines longer than 100 chars
  "semi": true, // always semicolons
  "endOfLine": "auto" // keep existing line endings (Windows CRLF vs LF)
}
```

---

## 0.11 Expo Router basics

**File-based routing:** the file path inside `src/app/` **is** the URL/route.

| File                       | Route          | Notes                                                            |
| -------------------------- | -------------- | ---------------------------------------------------------------- |
| `src/app/index.tsx`        | `/`            | Default screen of a folder                                       |
| `src/app/explore.tsx`      | `/explore`     |                                                                  |
| `src/app/history.tsx`      | `/history`     | Creating the file creates the route                              |
| `src/app/request/[id].tsx` | `/request/123` | **Dynamic route** — `[id]` is a parameter                        |
| `src/app/(tabs)/index.tsx` | `/`            | **Group** — `(name)` organizes files without adding to the URL   |
| `src/app/_layout.tsx`      | —              | **Layout** — defines a navigator that wraps sibling/child routes |
| `src/app/+not-found.tsx`   | any unknown    | 404 screen                                                       |

### Layouts (`_layout.tsx`)

- A layout is **not a screen**; it wraps the screens next to it and below it.
- It decides the **navigator type**: Stack (push/pop), Tabs (bottom bar), Drawer, etc.
- The **root** `src/app/_layout.tsx` wraps the **whole app** → perfect place for providers (theme, database init, etc.).

### Navigating (we'll use this in Phase 1)

```tsx
import { Link, router, useLocalSearchParams } from 'expo-router';

<Link href="/history">Go to history</Link>; // declarative
router.push('/request/42'); // imperative
router.back();

// inside src/app/request/[id].tsx
const { id } = useLocalSearchParams<{ id: string }>(); // "42"
```

### Why file-based routing is great

- Every screen automatically gets a **URL** → **deep links work for free** (`pocketapi://request/42`).
- Structure of the app is visible just by looking at folders.
- With `typedRoutes`, wrong links are type errors.

---

## 0.12 How the app boots — step by step

```
1. Phone opens app → Hermes loads the JS bundle
2. package.json "main": "expo-router/entry"
3. Expo Router scans src/app/ and builds the route tree
4. Renders src/app/_layout.tsx (root layout)
5. Root layout renders <AppTabs /> (NativeTabs navigator)
6. NativeTabs shows the tab for the current route "/" → src/app/index.tsx
7. Splash overlay animates away → Home screen visible
```

### Step 4 — `src/app/_layout.tsx`

```tsx
import { DarkTheme, DefaultTheme, ThemeProvider } from 'expo-router';
import * as SplashScreen from 'expo-splash-screen';
import { useColorScheme } from 'react-native';

import { AnimatedSplashOverlay } from '@/components/animated-icon';
import AppTabs from '@/components/app-tabs';

SplashScreen.preventAutoHideAsync(); // (a)

export default function TabLayout() {
  const colorScheme = useColorScheme(); // (b)
  return (
    <ThemeProvider value={colorScheme === 'dark' ? DarkTheme : DefaultTheme}>
      {' '}
      {/* (c) */}
      <AnimatedSplashOverlay /> {/* (d) */}
      <AppTabs /> {/* (e) */}
    </ThemeProvider>
  );
}
```

- **(a)** `preventAutoHideAsync()` runs **once at module load** (outside the component). It keeps the native splash screen visible so the app can prepare (later: open the database, load settings) before showing UI. Something must later call `SplashScreen.hideAsync()`.
- **(b)** `useColorScheme()` — a React Native hook returning `'light'` or `'dark'` from the phone's settings. When the user switches theme, the component **re-renders** automatically.
- **(c)** `ThemeProvider` — a **React Context provider**. Every navigation component inside (headers, tab bars) reads colors from it. Context = a way to pass data deep down the tree without passing props through every level.
- **(d)** An animated overlay shown during startup.
- **(e)** The tab navigator.
- `export default` — Expo Router requires route/layout files to **default-export** a component.

### Step 5 — `src/components/app-tabs.tsx`

```tsx
import { NativeTabs } from 'expo-router/native-tabs';
import { useColorScheme } from 'react-native';
import { Colors } from '@/constants/theme';

export default function AppTabs() {
  const scheme = useColorScheme();
  const colors = Colors[scheme === 'dark' ? 'dark' : 'light'];

  return (
    <NativeTabs
      backgroundColor={colors.background}
      indicatorColor={colors.backgroundElement}
      labelStyle={{ selected: { color: colors.text } }}
    >
      <NativeTabs.Trigger name="index">
        {' '}
        {/* → src/app/index.tsx */}
        <NativeTabs.Trigger.Label>Home</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon
          src={require('@/assets/images/tabIcons/home.png')}
          renderingMode="template"
        />
      </NativeTabs.Trigger>

      <NativeTabs.Trigger name="explore">
        {' '}
        {/* → src/app/explore.tsx */}
        ...
      </NativeTabs.Trigger>
    </NativeTabs>
  );
}
```

- **`NativeTabs`** uses the **platform's real tab bar** (Material bottom navigation on Android, `UITabBar` on iOS) instead of one drawn in JS → native look & feel, better performance.
- **`name="index"`** must match a **route file name** in `src/app/`.
- **`require('...png')`** — static image import; Metro bundles the image.
- **`renderingMode="template"`** — icon is treated as a shape and tinted with the theme color.
- This is a **component** (in `components/`), not a route, even though the layout uses it — keeping `src/app/` clean.

### Step 6 — `src/app/index.tsx` (Home screen)

```tsx
export default function HomeScreen() {
  return (
    <ThemedView style={styles.container}>
      <SafeAreaView style={styles.safeArea}>
        <ThemedView style={styles.heroSection}>
          <AnimatedIcon />
          <ThemedText type="title" style={styles.title}>
            Welcome to&nbsp;Expo
          </ThemedText>
        </ThemedView>
        ...
        {Platform.OS === 'web' && <WebBadge />}
      </SafeAreaView>
    </ThemedView>
  );
}
```

- **`SafeAreaView`** (from `react-native-safe-area-context`) adds padding so content doesn't go under the **notch, status bar, or home indicator**.
- **`ThemedView` / `ThemedText`** — wrappers around `View` / `Text` that pick colors from the current theme.
- **`{condition && <X />}`** — conditional rendering: renders `<X />` only if condition is true.
- **`Platform.OS`** — `'ios' | 'android' | 'web'`.
- `getDevMenuHint()` uses `Device.isDevice` (from `expo-device`) to show a different hint on a real phone vs emulator.

---

## 0.13 Styling & Flexbox basics

### StyleSheet

```tsx
const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', flexDirection: 'row' },
  title: { textAlign: 'center' },
});

<View style={styles.container} />
<Text style={[styles.title, { color: 'red' }]} />   // array = merge, later wins
```

- Styles are **JavaScript objects**, not CSS files.
- Property names are **camelCase** (`backgroundColor`, not `background-color`).
- Numbers have **no units** — they're **density-independent pixels (dp)**, so `16` looks the same physical size on all screens.
- No cascading/inheritance like CSS (except text styles nested inside `<Text>`).

### Flexbox in React Native — differences from web

|                         | Web CSS                         | React Native                           |
| ----------------------- | ------------------------------- | -------------------------------------- |
| Default `flexDirection` | `row`                           | **`column`**                           |
| `display: flex` needed? | yes                             | **no — every View is flex**            |
| `flex`                  | shorthand for grow/shrink/basis | `flex: 1` = "take all available space" |

### Most used layout props

| Prop                   | Axis                      | Meaning                                                                             |
| ---------------------- | ------------------------- | ----------------------------------------------------------------------------------- |
| `flexDirection`        | —                         | `column` (top→bottom) or `row` (left→right)                                         |
| `justifyContent`       | **main** axis             | `flex-start`, `center`, `flex-end`, `space-between`, `space-around`, `space-evenly` |
| `alignItems`           | **cross** axis            | `stretch` (default), `flex-start`, `center`, `flex-end`                             |
| `alignSelf`            | cross axis, for one child | override parent's `alignItems`                                                      |
| `flex: 1`              | —                         | grow to fill space                                                                  |
| `gap`                  | —                         | space between children                                                              |
| `padding*` / `margin*` | —                         | inner / outer spacing                                                               |

```
flexDirection: 'column' (default)      flexDirection: 'row'
main axis ↓, cross axis →              main axis →, cross axis ↓
┌──────────┐                           ┌──────────────────┐
│ [A]      │                           │ [A] [B] [C]      │
│ [B]      │                           └──────────────────┘
│ [C]      │
└──────────┘
```

### Design tokens (from `src/constants/theme.ts`)

The template already uses **design tokens** — named constants instead of magic numbers:

```ts
paddingHorizontal: Spacing.four,      // not 24
gap: Spacing.three,
paddingBottom: BottomTabInset + Spacing.three,
maxWidth: MaxContentWidth,
```

Same idea for `Colors.light.text`, `Colors.dark.background`, etc. Change the token once → whole app updates. We'll build a full theme system on this idea in Phase 5.

---

## 0.14 Platform-specific code

Two ways to write different code per platform:

### 1. `Platform.OS` / `Platform.select` (small differences)

```tsx
import { Platform } from 'react-native';

if (Platform.OS === 'android') { ... }

const shortcut = Platform.OS === 'android' ? 'cmd+m (or ctrl+m)' : 'cmd+d';

const styles = { padding: Platform.select({ ios: 12, android: 8, default: 10 }) };
```

### 2. Platform file extensions (big differences)

```
app-tabs.tsx        ← Android + iOS
app-tabs.web.tsx    ← Web
use-color-scheme.ts / use-color-scheme.web.ts
animated-icon.tsx   / animated-icon.web.tsx
```

You import **without** the extension: `import AppTabs from '@/components/app-tabs'`.
**Metro picks the right file** for the platform it's bundling. Also available: `.ios.tsx`, `.android.tsx`, `.native.tsx` (iOS + Android).

---

## 0.15 Hooks & custom hooks (first look)

**Hooks** are functions that let components use React features (state, effects, context).

Rules of hooks:

1. Only call hooks **at the top level** of a component or custom hook (not inside `if`, loops, or nested functions).
2. Only call hooks from **React components** or **other hooks**.
3. Name custom hooks starting with **`use`**.

Hooks seen so far:

| Hook               | From                         | Returns                                    |
| ------------------ | ---------------------------- | ------------------------------------------ |
| `useColorScheme()` | `react-native`               | `'light' \| 'dark'` (re-renders on change) |
| `useTheme()`       | our `src/hooks/use-theme.ts` | the color set for current scheme           |

### Our custom hook — `src/hooks/use-theme.ts`

```ts
import { Colors } from '@/constants/theme';
import { useColorScheme } from '@/hooks/use-color-scheme';

export function useTheme() {
  const scheme = useColorScheme();
  const theme = scheme === 'dark' ? 'dark' : 'light';
  return Colors[theme];
}
```

- A **custom hook** = a normal function that **uses other hooks** and packages logic for reuse.
- Now any component can do `const colors = useTheme();` instead of repeating the light/dark logic.
- Note: it imports `useColorScheme` from **our own** `@/hooks/use-color-scheme` (which has a `.web.ts` version), not directly from `react-native` — so web gets special handling.

Coming in later phases: `useState`, `useEffect`, `useRef`, `useMemo`, `useCallback`, and our own `useSendRequest`, `useHistory`, `useDebounce`.

---

## 0.16 Commands cheat sheet

```bash
# Run
npx expo start              # start Metro; scan QR with Expo Go
npx expo start --clear      # start with cleared Metro cache (fixes weird bundling issues)
#   in the Metro terminal: a = open Android, w = open web, r = reload, m = dev menu, j = debugger

# Install packages (ALWAYS this, not npm install)
npx expo install <package>  # picks SDK-58-compatible version
npx expo install --fix      # fix mismatched versions

# Quality
npm run lint                # ESLint + Prettier check
npm run format              # auto-format everything
npm run typecheck           # TypeScript check

# Diagnose
npx expo-doctor             # check project health/dependency problems

# Builds (later)
npx expo run:android        # local development build (needs Android SDK)
npx eas-cli@latest build --profile development --platform android   # cloud dev build
```

**Dev menu on phone:** shake the device (or press `m` in the Metro terminal) → Reload, Toggle element inspector, Open debugger, Performance monitor.

---

## 0.17 Glossary

| Term                  | Meaning                                                                |
| --------------------- | ---------------------------------------------------------------------- |
| **Bundle**            | Single JS file containing all your app code, produced by Metro         |
| **Metro**             | React Native's JS bundler + dev server                                 |
| **Babel**             | Compiler that converts TS/JSX/modern JS → JS the engine runs           |
| **Hermes**            | JS engine that runs the bundle on the device                           |
| **JSI**               | JavaScript Interface — direct JS ↔ native (C++) calls                  |
| **Fabric**            | New Architecture renderer                                              |
| **TurboModules**      | New Architecture native modules (lazy, via JSI)                        |
| **Fast Refresh**      | Instant reload of edited components while keeping state                |
| **Expo SDK**          | Versioned set of Expo native modules (we're on 58)                     |
| **Expo Go**           | Prebuilt sandbox app for running projects without building             |
| **Development build** | Your own app binary with dev tools; supports any native code           |
| **CNG**               | Continuous Native Generation — `android/`/`ios/` generated from config |
| **Config plugin**     | Script that modifies generated native projects                         |
| **EAS**               | Expo Application Services: Build, Submit, Update                       |
| **Route**             | A screen reachable by a URL path                                       |
| **Layout**            | `_layout.tsx` — a navigator wrapping routes                            |
| **Dynamic route**     | `[id].tsx` — route with a URL parameter                                |
| **Group**             | `(name)/` folder — organizes routes without affecting URL              |
| **Deep link**         | URL that opens a specific screen in the app (`pocketapi://...`)        |
| **Scheme**            | The `pocketapi://` prefix defined in `app.json`                        |
| **Safe area**         | Screen region not covered by notch / status bar / home indicator       |
| **dp**                | Density-independent pixels — RN's unit for sizes                       |
| **Design tokens**     | Named constants for colors/spacing/typography                          |
| **Context**           | React feature to share data deep in the tree without prop drilling     |
| **Hook**              | Function starting with `use` that taps into React features             |
| **Strict mode (TS)**  | Group of strict type checks (`strictNullChecks`, `noImplicitAny`, ...) |
| **Lint**              | Static analysis to catch bugs/bad patterns (ESLint)                    |
| **Formatter**         | Tool that rewrites code style consistently (Prettier)                  |

---

## 0.18 Self-check (with answers)

<details>
<summary><b>1. What does <code>"main": "expo-router/entry"</code> do?</b></summary>

It's the entry point Metro starts bundling from. `expo-router/entry` registers the app's root component and builds the navigation tree by scanning the files in `src/app/`. So the folder structure _is_ the app's navigation — no manual `App.tsx` needed.
</details>

<details>
<summary><b>2. If you create <code>src/app/history.tsx</code>, what happens?</b></summary>

Expo Router automatically creates a new route at `/history`. You can navigate to it with `<Link href="/history">` or `router.push('/history')`, and it's deep-linkable (`pocketapi://history`). To show it as a **tab**, you'd also add a `<NativeTabs.Trigger name="history">` in the tab layout. The file must `export default` a component.
</details>

<details>
<summary><b>3. What's the difference between ESLint and Prettier?</b></summary>

ESLint is a **linter** — it analyzes code for bugs and bad patterns (unused vars, hook rule violations, missing effect dependencies). Prettier is a **formatter** — it only rewrites style (quotes, semicolons, line width) and never changes behavior. In this project, `eslint-plugin-prettier` runs Prettier inside ESLint so `npm run lint` reports both.
</details>

<details>
<summary><b>4. When do you need a development build instead of Expo Go?</b></summary>

When you add a library with **native code that isn't included in Expo Go**, or need custom native configuration (config plugins, custom app icon/scheme behaviour to test exactly, etc.). Expo Go can't add new native code at runtime — native code must be compiled into the app binary, which is what a development build is.
</details>

<details>
<summary><b>5. What does Metro do?</b></summary>

Resolves all imports starting from the entry file, transforms TS/JSX to JS with Babel, serializes everything into one bundle, and serves it to the device. In development it watches files and supports Fast Refresh.
</details>

<details>
<summary><b>6. Why does the app still run when there are TypeScript errors?</b></summary>

Because Babel (inside Metro) only **strips** types — it doesn't check them. Type-checking is a separate step: `npm run typecheck` (`tsc --noEmit`).
</details>

<details>
<summary><b>7. Why shouldn't you edit <code>android/</code> or <code>ios/</code> folders?</b></summary>

Because of Continuous Native Generation: they're generated from `app.json` + config plugins and can be regenerated at any time, wiping manual changes. Native config belongs in `app.json` / config plugins.
</details>

<details>
<summary><b>8. What's the default <code>flexDirection</code> in React Native, and how is that different from the web?</b></summary>

`column` in React Native; `row` on the web. Also every `View` is a flex container by default.
</details>

<details>
<summary><b>9. What is the New Architecture, in one sentence?</b></summary>

It replaces the old asynchronous JSON bridge with JSI so JS can call native code directly, using Fabric as the renderer and TurboModules for lazy-loaded native modules.
</details>

<details>
<summary><b>10. Why does <code>_layout.tsx</code> call <code>SplashScreen.preventAutoHideAsync()</code>?</b></summary>

To keep the native splash screen visible until the app is ready (fonts, database, settings loaded), avoiding a flash of empty/unstyled UI. Something must later call `SplashScreen.hideAsync()`.
</details>

---

## 0.19 Phase 0 status

| Task                                     | Status               |
| ---------------------------------------- | -------------------- |
| Create Expo project (SDK 58)             | ✅                   |
| TypeScript strict mode                   | ✅                   |
| ESLint + Prettier configured             | ✅                   |
| App runs on phone (Expo Go)              | ❓ confirm           |
| Can explain what Metro does              | 📖 learned (see 0.3) |
| Git init + first commit + push to GitHub | ❌ next              |
| Own README                               | ❌ next              |

---

## 0.20 Deep dive: `"main": "expo-router/entry"`

### 🧒 In simple words (read this first)

**Analogy — a book:**

- Your app is a **book** with many pages (files).
- `"main"` is the note on the cover saying **"start reading from page 1 here"**.
- `"expo-router/entry"` is the name of that **page 1** — but **Expo Router wrote it for you**.

**What that page 1 does — just 3 things:**

1. 👀 **Looks inside your `src/app/` folder** and lists every file.
2. 🗺️ **Turns each file into a screen**:
   - `index.tsx` → Home screen
   - `explore.tsx` → Explore screen
   - `_layout.tsx` → the frame (tabs) around them
3. ▶️ **Tells the phone:** "this is the app, show it!"

**Why it's nice:**

- **Old way:** you had to write the start file yourself AND manually list every screen.
- **Now:** you just **create a file** in `src/app/` → it becomes a screen automatically.

**Remember just this one line:**

> `"main": "expo-router/entry"` = **"Start the app using Expo Router, and let it turn my `src/app/` files into screens."**

**3 rules that come from it:**

1. Put **screens** in `src/app/`.
2. Put **everything else** (components, helpers) **outside** `src/app/` — otherwise it becomes a screen too.
3. **Don't touch** this line.

### 🚶 How Metro walks from `package.json` to `src/app/` (step by step)

**First: what Metro does, in one line.**
Metro is a **file collector**. It opens a file, looks at its `import` lines, then opens those files, and keeps going. At the end it glues every file it found into **one big file** (the bundle) and sends it to the phone.

**Tiny example first (not our app):**

```js
// a.js
import './b';
// b.js
import './c';
// c.js
console.log('hi');
```

If Metro starts at `a.js` → sees `import './b'` → opens `b.js` → sees `import './c'` → opens `c.js` → no more imports → done.
Bundle = `a.js + b.js + c.js`. Like following arrows. ➡️➡️➡️

**Now our real app — same idea, just follow the arrows:**

| Step | Metro opens…                                | What it sees inside                                       | So it goes to…                        |
| ---- | ------------------------------------------- | --------------------------------------------------------- | ------------------------------------- |
| 1    | `package.json`                              | `"main": "expo-router/entry"`                             | `node_modules/expo-router/entry.js`   |
| 2    | `node_modules/expo-router/entry.js`         | `import 'expo-router/entry-classic'`                      | `entry-classic.js`                    |
| 3    | `node_modules/expo-router/entry-classic.js` | `import { App } from 'expo-router/build/qualified-entry'` | `qualified-entry.js`                  |
| 4    | `qualified-entry.js`                        | `import { ctx } from 'expo-router/_ctx'`                  | `_ctx.android.js`                     |
| 5    | `_ctx.android.js`                           | `require.context(<src/app folder>, ...)` ← **special!**   | **every file in `src/app/`** 🎯       |
| 6    | `src/app/_layout.tsx`, `index.tsx`, …       | `import AppTabs from '@/components/app-tabs'` etc.        | `src/components/...`, `src/hooks/...` |

**The special trick is step 5.**
A normal `import` brings in **one** file. `require.context` says **"bring in ALL files from this folder"**. That's the bridge from Expo Router's code (inside `node_modules`) to **your** code (in `src/app/`).

**How does it know the folder is `src/app`?**
When Metro bundles, Expo checks: _"does `src/app` exist? → use it. Otherwise use `app/`."_ It writes that folder path into `_ctx` (that's the `process.env.EXPO_ROUTER_APP_ROOT` you see in the code).

**Then, on the phone:**
`entry-classic.js` calls `renderRootComponent(App)` → the phone shows `App` → `App` contains the list of your `src/app/` files → it shows `_layout.tsx` (tabs) → which shows `index.tsx` (Home).

**Whole thing in one picture:**

```
package.json ──main──► entry.js ──import──► entry-classic.js ──import──► qualified-entry.js
                                                                              │ import
                                                                              ▼
                                                                         _ctx.android.js
                                                                              │ require.context("src/app")
                                                   ┌──────────────────────────┼─────────────────┐
                                                   ▼                          ▼                 ▼
                                           src/app/_layout.tsx        src/app/index.tsx   src/app/explore.tsx
                                                   │ import                   │ import
                                                   ▼                          ▼
                                           src/components/...          src/components/...
```

### 🎯 The exact line that "jumps" into `src/` — fill in the blank

The file `node_modules/expo-router/_ctx.android.js`:

```js
export const ctx = require.context(
  process.env.EXPO_ROUTER_APP_ROOT, // 1. WHICH folder
  true, // 2. look inside sub-folders too?
  /^...\.[tj]sx?$/, // 3. WHICH files (a filter)
  process.env.EXPO_ROUTER_IMPORT_MODE, // 4. load now or later
);
```

Notice: **`src/app` is not written anywhere in this file!** Instead there's a **blank**: `process.env.EXPO_ROUTER_APP_ROOT`.

Think of it like a form: `Folder: ________`. Expo Router's authors couldn't write your folder name — they don't know your project. So they left a blank, and **Expo fills it in for each project.**

**Who fills the blank? (2 steps, real code)**

**Step A — Expo decides the folder** (`node_modules/@expo/cli/.../metro/router.js`):

```js
function getRouterDirectory(projectRoot) {
  // more specific directories first
  if (directoryExistsSync(path.join(projectRoot, 'src', 'app'))) {
    return path.join('src', 'app'); // ✅ our project has src/app → this one
  }
  return 'app';
}
```

That's also why the terminal prints **"Using src/app as the root directory for Expo Router."** when you run `npx expo start`.

**Step B — Babel writes it into the blank** (`node_modules/babel-preset-expo/.../expo-router-plugin.js`):

```js
case 'EXPO_ROUTER_APP_ROOT':
  path.replaceWith(t.stringLiteral(getExpoRouterAppRoot(filename, routerAbsoluteRoot)));
```

= "find the text `process.env.EXPO_ROUTER_APP_ROOT` and **replace it** with the real folder path as a string."
This happens while Metro is transforming the file (remember Metro uses Babel to transform every file — section 0.3).

**Before vs after** (roughly — the path is relative to the `_ctx` file's location):

```js
// BEFORE (what's on disk in node_modules)
require.context(process.env.EXPO_ROUTER_APP_ROOT, true, /.../, process.env.EXPO_ROUTER_IMPORT_MODE);

// AFTER Babel (what actually goes in the bundle)
require.context('../../src/app', true, /.../, 'sync');
```

**Now it's just a normal instruction:** "Metro, collect every `.ts/.tsx/.js/.jsx` file inside `src/app` (and its sub-folders)." → Metro opens `_layout.tsx`, `index.tsx`, `explore.tsx` → and follows **their** imports into `src/components`, `src/hooks`, etc. **That's how we get into `src/`.**

**The 4 arguments in plain words:**

| #   | Argument                              | Plain meaning                                                                                                                 |
| --- | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 1   | `process.env.EXPO_ROUTER_APP_ROOT`    | **Which folder** → becomes `src/app`                                                                                          |
| 2   | `true`                                | **Include sub-folders** → so `src/app/request/[id].tsx` is found too                                                          |
| 3   | the long `/.../` regex                | **Which files** → any `.ts .tsx .js .jsx`, but **skip** special server files (`+api`, `+html`, `+middleware`)                 |
| 4   | `process.env.EXPO_ROUTER_IMPORT_MODE` | **When to load** → also a blank filled by Babel; `'sync'` = load all screens at startup, `'lazy'` = load a screen when opened |

**About that scary regex (arg 3) — you don't need to read it.** It just means:

- ✅ match: `index.tsx`, `_layout.tsx`, `explore.tsx`, `request/[id].tsx`
- ❌ skip: `something+api.ts`, `+html.tsx`, `+middleware.ts` (these are for web servers, not phone screens)
- `(?:\.ios|\.web)?` in the **android** version: allows files like `x.ios.tsx`/`x.web.tsx` to be listed — Expo Router then picks the right platform version.

**One-sentence summary:**

> `_ctx.android.js` has a blank for "which folder". Expo sees our project has `src/app`, and Babel writes `src/app` into that blank while bundling. So the line becomes "collect every file in `src/app`" — and that's how Metro reaches our code.

The rest of this section is the detailed version — read it once you're comfortable with the simple one.

### Part 1 — What is the `main` field?

`main` is a **standard npm field** in `package.json`. It answers one question:

> **"When someone runs this package, which file do they start from?"**

- In a Node.js library, `main` is the file you get when you `require('that-library')`.
- In a React Native app, **Metro reads `main`** to know where to **start bundling**. Every other file gets into the bundle only because something (directly or indirectly) is imported from this starting file.

```
package.json  "main"  ──►  first file Metro reads
                                 │ imports
                                 ▼
                           next files ... → whole app
```

If a file isn't reachable from `main` through imports, it **isn't in the app**.

### Part 2 — The value: `expo-router/entry`

`expo-router/entry` is **not a file in our project**. It's a path to a file **inside the `expo-router` package**:

```
"expo-router/entry"
     │         │
     │         └── file "entry.js" inside that package
     └── package in node_modules/
                 ⇒ node_modules/expo-router/entry.js
```

So we're saying: _"Our app doesn't have its own startup file — use the one Expo Router provides."_

### Part 3 — What that file actually does (real code from node_modules)

It's a short chain of 4 files. Here's the real code (simplified slightly):

**① `node_modules/expo-router/entry.js`**

```js
// This is aliased to another location when server components are enabled.
import 'expo-router/entry-classic';
```

Just a **redirect**. It exists so Expo can swap in a different entry for advanced features (React Server Components) without you changing `package.json`.

**② `node_modules/expo-router/entry-classic.js`**

```js
// `@expo/metro-runtime` MUST be the first import to ensure Fast Refresh works on web.
import '@expo/metro-runtime';
import { App } from 'expo-router/build/qualified-entry';
import { renderRootComponent } from 'expo-router/build/renderRootComponent';

// This file should only import and register the root.
renderRootComponent(App);
```

Three things:

1. Load Metro's runtime helpers (Fast Refresh, etc.).
2. Get the **`App`** component (built in step ③).
3. **Register** it as the root of the app (step ④).

**③ `expo-router/build/qualified-entry.js` — builds the `App` component**

```js
import { ctx } from 'expo-router/_ctx'; // ← list of ALL your route files

function App() {
  return (
    <Head.Provider>
      <ExpoRoot context={ctx} /> // ← the router, given your files
    </Head.Provider>
  );
}
```

`ExpoRoot` is the heart of Expo Router: it takes the list of route files and builds the navigation tree (layouts → screens).

**`expo-router/_ctx.android.js` — how it finds your files**

```js
export const ctx = require.context(
  process.env.EXPO_ROUTER_APP_ROOT, // → our src/app folder
  true, // → include sub-folders
  /^...(?:\.ios|\.web)?\.[tj]sx?$/, // → which files count (.ts/.tsx/.js/.jsx; ignores +api, +html...)
  process.env.EXPO_ROUTER_IMPORT_MODE, // → load eagerly ('sync') or lazily
);
```

- **`require.context`** is a **Metro feature** that means: _"give me every file in this folder that matches this pattern"_. Normally Metro only bundles files you explicitly `import`; `require.context` lets it bundle **a whole folder automatically**. That's the trick behind file-based routing.
- **`EXPO_ROUTER_APP_ROOT`** is filled in at build time by Expo's Babel preset. It points to `src/app` (Expo detects that we use `src/app` instead of `app`).
- There are separate `_ctx.android.js`, `_ctx.ios.js`, `_ctx.web.js` versions — Metro picks the one for the current platform (same platform-extension trick from section 0.14).
- When you **add or delete** a file in `src/app/`, Metro notices (because of `require.context`), rebuilds the list, and the new route appears without restarting.

**④ `expo-router/build/renderRootComponent.js` — registers the app**

```js
function renderRootComponent(Component) {
  try {
    if (process.env.NODE_ENV !== 'production') {
      // set up LogBox (the red/yellow error overlay in development)
    }
    ...
    registerRootComponent(Component);          // from 'expo'
  } catch (e) {
    // hide the splash screen so you can SEE the error
    registerRootComponent(() => <View />);    // render empty screen instead of crashing silently
    ...
  }
}
```

- **`registerRootComponent`** (from the `expo` package) tells React Native: _"this is THE component to show when the app opens."_ Under the hood it calls React Native's `AppRegistry.registerComponent('main', ...)`. The native side (Expo Go / your build) asks for the component named `'main'` and renders it.
- It also sets up dev-only error overlays and handles startup errors gracefully (hides the splash so you aren't stuck on it).

### Part 4 — The whole chain in one picture

```
package.json
  "main": "expo-router/entry"
        │
        ▼
node_modules/expo-router/entry.js           (redirect)
        │
        ▼
node_modules/expo-router/entry-classic.js
   ├─ import '@expo/metro-runtime'           (Fast Refresh helpers)
   ├─ App  ◄── qualified-entry.js
   │            └─ <ExpoRoot context={ctx}>
   │                           ▲
   │                 _ctx.android.js: require.context('src/app', ...)
   │                 = [ _layout.tsx, index.tsx, explore.tsx ]
   │
   └─ renderRootComponent(App)
          └─ registerRootComponent(App)      (expo)
                 └─ AppRegistry.registerComponent('main', () => App)   (react-native)
                          │
                          ▼
                Native app asks for 'main' → renders App
                          │
                          ▼
        ExpoRoot renders src/app/_layout.tsx → <AppTabs/> → src/app/index.tsx
```

### Part 5 — Old way vs Expo Router way

**Before Expo Router** (classic React Native / old Expo projects):

```json
// package.json
"main": "index.js"
```

```js
// index.js  (you wrote this)
import { registerRootComponent } from 'expo';
import App from './App';
registerRootComponent(App);
```

```tsx
// App.tsx  (you wrote this — and ALL navigation by hand)
export default function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen name="Explore" component={ExploreScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

Every new screen = import it + register it manually + configure deep links manually.

**With Expo Router:**

```json
"main": "expo-router/entry"
```

…and that's it. Create `src/app/history.tsx` → route `/history` exists, deep link `pocketapi://history` works.

|                 | Old way                    | Expo Router                                                    |
| --------------- | -------------------------- | -------------------------------------------------------------- |
| Entry file      | You write `index.js`       | Provided by `expo-router/entry`                                |
| Root component  | You write `App.tsx`        | Generated from `src/app/`                                      |
| Adding a screen | Import + register manually | Create a file                                                  |
| Deep links      | Manual linking config      | Automatic from file paths                                      |
| Under the hood  | React Navigation           | **Still React Navigation** (Expo Router is built on top of it) |

### Part 6 — Practical consequences

1. **Don't create `App.tsx` or `index.js` at the project root** — they'd be ignored (not reachable from `main`).
2. **Your real "starting point" is `src/app/_layout.tsx`** — that's where app-wide setup goes (providers, theme, database init, splash hiding). In Phase 3–4 we'll put Zustand/DB setup there.
3. **Every file in `src/app/` becomes a route** (because `require.context` grabs the whole folder). So components/hooks/utils must live **outside** `src/app/`, or they'll turn into broken screens.
4. **If you change `main`**, Expo Router stops working. Only change it if you deliberately want a custom entry (rare — e.g., to register a background task before the app loads; even then you usually create your own entry file that does `import 'expo-router/entry'` at the end).

### Self-check

<details>
<summary><b>Q1. Where does Metro start bundling and why?</b></summary>

From the file in `package.json` → `main`. In this project that's `node_modules/expo-router/entry.js`. Metro follows imports from there to find every file in the app.
</details>

<details>
<summary><b>Q2. How does Expo Router know which screens exist without us importing them?</b></summary>

It uses Metro's `require.context(src/app, recursive, pattern)` to collect every matching file in the app folder into the bundle, then `ExpoRoot` builds the navigation tree from that list.
</details>

<details>
<summary><b>Q3. What does <code>registerRootComponent</code> do?</b></summary>

Registers the given component as the app's root (via `AppRegistry.registerComponent('main', ...)`) so the native side knows what to render at launch.
</details>

<details>
<summary><b>Q4. If you create <code>src/app/utils.ts</code> with helper functions, what goes wrong?</b></summary>

Expo Router treats it as a route (`/utils`). It has no default-exported component, so you get a warning/broken route. Helpers belong in `src/utils/` (outside `src/app/`).
</details>

<details>
<summary><b>Q5. Is Expo Router a replacement for React Navigation?</b></summary>

No — it's built on top of React Navigation. It generates the navigators from files and adds URL/deep-link support automatically.
</details>

---

## 0.21 Resetting the project (blank slate)

The template came with demo screens (Welcome to Expo, Explore). Before building our own app we cleared them with:

```bash
npm run reset-project      # = node ./scripts/reset-project.js
```

It asks: _"Move existing files to /example instead of deleting them? (Y/n)"_ → we answered **Y** (move, so nothing is lost).

### What it did

| Before                                        | After                                                 |
| --------------------------------------------- | ----------------------------------------------------- |
| `src/` (app, components, hooks, constants)    | moved to `example/src/`                               |
| `scripts/reset-project.js`                    | moved to `example/scripts/`                           |
| demo images (`expo-logo.png`, `tabIcons/`, …) | moved to `example/assets/images/`                     |
| —                                             | **new** `src/app/_layout.tsx` and `src/app/index.tsx` |

App icons and splash images (`icon.png`, `splash-icon.png`, `android-icon-*.png`, `favicon.png`) **stayed** in `assets/images/` because `app.json` still uses them.

### The new blank files

**`src/app/_layout.tsx`**

```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return <Stack />;
}
```

- The root layout is now a **Stack** navigator instead of tabs.
- **Stack** = screens pile on top of each other like cards: `router.push()` puts a new card on top, Back removes it. It also shows a **header bar** at the top with the screen name (that's why you'll see "index" as a title).
- We'll switch to **tabs** in Phase 1 (Request / Collections / History / Settings).

**`src/app/index.tsx`**

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function Index() {
  return (
    <View style={styles.container}>
      <Text>Edit src/app/index.tsx to edit this screen.</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1, // fill the whole screen
    alignItems: 'center', // center horizontally (cross axis, since column is default)
    justifyContent: 'center', // center vertically (main axis)
  },
});
```

The smallest possible screen: a `View` that fills the screen and centers one `Text`.

### Clean-up we did after the reset (and why)

1. **`tsconfig.json`** → added `"exclude": ["node_modules", "example"]`.
   _Why:_ `example/src/...` files still import `@/components/...`. But `@/` now points to the **new** `src/`, where those files don't exist → `tsc` would report errors for code we're not using.
2. **`eslint.config.js`** → added `'example/*'` to `ignores` (same reason, for lint).
3. **`.prettierignore`** → added `example` so `npm run format` doesn't touch it.
4. **`package.json`** → removed the `"reset-project"` script, since the script file moved to `example/scripts/` and would fail now.
5. Ran Prettier on the new files (the script writes double quotes `"`; our style is single quotes `'`).
6. ✅ `npm run typecheck` and `npm run lint` both pass.

### Note on `example/`

Metro **won't bundle** `example/` — nothing imports it, and it's not inside `src/app/` (remember 0.20: only files reachable from `main` end up in the app). It's just there for reference: we can copy ideas from `example/src/constants/theme.ts` or `themed-text.tsx` when we build our theme in Phase 5. Delete it when no longer useful.

---

<!-- Next phases will be appended below as we build them. -->
