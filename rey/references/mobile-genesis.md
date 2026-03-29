# REY — Mobile Genesis Protocol (Expo + React Native)

> "From zero to production-ready mobile app. iOS + Android. One codebase."

---

## 📱 MOBILE STACK DECISION

**REY Default Mobile Stack:**
```
Framework:      Expo (SDK 52+) + Expo Router v4
Language:       TypeScript strict
Styling:        NativeWind v4 (Tailwind for RN)
Navigation:     Expo Router (file-based, like Next.js)
State:          Zustand + TanStack Query
Auth:           Clerk (React Native SDK)
DB/Backend:     Supabase (same as web)
Storage:        MMKV (sync) + Expo SecureStore (sensitive)
Animation:      React Native Reanimated v3 + Moti
Notifications:  Expo Notifications
OTA Updates:    Expo Updates + EAS Update
Build:          EAS Build (Expo Application Services)
Distribution:   EAS Submit → App Store + Play Store
```

---

## 🗓️ DAY 0 — FOUNDATION

### Step 0.1: Create Expo Project
```bash
# Create with Expo Router template (REY default)
npx create-expo-app@latest MyApp --template tabs
cd MyApp

# Or blank TypeScript template
npx create-expo-app@latest MyApp --template blank-typescript
cd MyApp

# Install Expo Router (if blank template)
npx expo install expo-router expo-constants expo-linking expo-status-bar
```

### Step 0.2: TypeScript Config
```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".expo/types/**/*.d.ts",
    "expo-env.d.ts"
  ]
}
```

### Step 0.3: Directory Structure
```
MyApp/
├── app/                      # Expo Router pages (file-based routing)
│   ├── (auth)/               # Auth group
│   │   ├── sign-in.tsx
│   │   └── sign-up.tsx
│   ├── (tabs)/               # Bottom tab group
│   │   ├── _layout.tsx       # Tab bar config
│   │   ├── index.tsx         # Home tab
│   │   ├── explore.tsx       # Explore tab
│   │   └── profile.tsx       # Profile tab
│   ├── _layout.tsx           # Root layout (providers)
│   └── +not-found.tsx        # 404 screen
├── src/
│   ├── components/
│   │   ├── ui/               # Base UI components
│   │   ├── common/           # Shared components
│   │   └── features/         # Feature components
│   ├── lib/
│   │   ├── supabase.ts       # Supabase client
│   │   ├── auth.ts           # Auth helpers
│   │   └── utils.ts          # Utilities
│   ├── hooks/                # Custom hooks
│   ├── stores/               # Zustand stores
│   └── types/                # TypeScript types
├── assets/
│   ├── fonts/
│   ├── images/
│   └── icons/
├── app.json                  # Expo config
└── eas.json                  # EAS Build config
```

### Step 0.4: Core Dependencies
```bash
# Navigation & Layout
npx expo install expo-router expo-constants expo-linking expo-status-bar
npx expo install react-native-safe-area-context react-native-screens

# Styling
npx expo install nativewind
pnpm add -D tailwindcss

# Animation (essential pair)
npx expo install react-native-reanimated react-native-gesture-handler
pnpm add moti  # Moti wraps Reanimated with cleaner API

# Storage
npx expo install react-native-mmkv expo-secure-store

# Image
npx expo install expo-image  # Better than RN's Image

# Haptics
npx expo install expo-haptics

# Icons
pnpm add @expo/vector-icons  # Usually pre-installed

# State & Data
pnpm add zustand @tanstack/react-query

# Auth
pnpm add @clerk/clerk-expo
npx expo install expo-web-browser expo-auth-session

# Dev tools
pnpm add -D @tanstack/react-query-devtools
```

### Step 0.5: app.json Config
```json
{
  "expo": {
    "name": "MyApp",
    "slug": "myapp",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/icon.png",
    "scheme": "myapp",
    "userInterfaceStyle": "automatic",
    "newArchEnabled": true,
    "splash": {
      "image": "./assets/images/splash-icon.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myapp",
      "infoPlist": {
        "NSCameraUsageDescription": "Allow $(PRODUCT_NAME) to access your camera",
        "NSPhotoLibraryUsageDescription": "Allow $(PRODUCT_NAME) to access your photos"
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.yourcompany.myapp",
      "permissions": ["CAMERA", "READ_EXTERNAL_STORAGE"]
    },
    "web": {
      "bundler": "metro",
      "output": "static",
      "favicon": "./assets/images/favicon.png"
    },
    "plugins": [
      "expo-router",
      "expo-secure-store",
      [
        "expo-notifications",
        {
          "icon": "./assets/images/notification-icon.png",
          "color": "#ffffff"
        }
      ]
    ],
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

### Step 0.6: NativeWind Setup
```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./app/**/*.{js,jsx,ts,tsx}", "./src/**/*.{js,jsx,ts,tsx}"],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {
      colors: {
        brand: {
          50: "#eff6ff",
          500: "#3b82f6",
          600: "#2563eb",
          900: "#1e3a8a",
        },
      },
      fontFamily: {
        sans: ["Inter_400Regular"],
        "sans-medium": ["Inter_500Medium"],
        "sans-bold": ["Inter_700Bold"],
      },
    },
  },
  plugins: [],
}
```

```javascript
// metro.config.js
const { getDefaultConfig } = require("expo/metro-config")
const { withNativeWind } = require("nativewind/metro")

const config = getDefaultConfig(__dirname)

module.exports = withNativeWind(config, { input: "./global.css" })
```

```css
/* global.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Step 0.7: EAS Config
```json
// eas.json
{
  "cli": {
    "version": ">= 12.0.0",
    "promptToConfigurePushNotifications": false
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": { "resourceClass": "m-medium" }
    },
    "preview": {
      "distribution": "internal",
      "ios": { "resourceClass": "m-medium" }
    },
    "production": {
      "autoIncrement": true,
      "ios": { "resourceClass": "m-medium" },
      "env": {
        "NODE_ENV": "production"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your@email.com",
        "ascAppId": "YOUR_APP_STORE_ID",
        "appleTeamId": "YOUR_TEAM_ID"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "internal"
      }
    }
  }
}
```

---

## 🗓️ DAY 1 — CORE INFRASTRUCTURE

### Step 1.1: Root Layout + Providers
```tsx
// app/_layout.tsx
import { useEffect } from "react"
import { Stack } from "expo-router"
import { StatusBar } from "expo-status-bar"
import { GestureHandlerRootView } from "react-native-gesture-handler"
import { ClerkProvider, useAuth } from "@clerk/clerk-expo"
import { QueryClient, QueryClientProvider } from "@tanstack/react-query"
import * as SplashScreen from "expo-splash-screen"
import * as Font from "expo-font"
import {
  Inter_400Regular,
  Inter_500Medium,
  Inter_700Bold,
} from "@expo-google-fonts/inter"
import "../global.css"

SplashScreen.preventAutoHideAsync()

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
      staleTime: 1000 * 60 * 5, // 5 minutes
    },
  },
})

function RootLayoutNav() {
  const { isLoaded, isSignedIn } = useAuth()

  if (!isLoaded) return null

  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="(tabs)" />
      <Stack.Screen name="(auth)" />
      <Stack.Screen name="+not-found" />
    </Stack>
  )
}

export default function RootLayout() {
  const [fontsLoaded] = Font.useFonts({
    Inter_400Regular,
    Inter_500Medium,
    Inter_700Bold,
  })

  useEffect(() => {
    if (fontsLoaded) SplashScreen.hideAsync()
  }, [fontsLoaded])

  if (!fontsLoaded) return null

  return (
    <ClerkProvider publishableKey={process.env.EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY!}>
      <QueryClientProvider client={queryClient}>
        <GestureHandlerRootView style={{ flex: 1 }}>
          <StatusBar style="auto" />
          <RootLayoutNav />
        </GestureHandlerRootView>
      </QueryClientProvider>
    </ClerkProvider>
  )
}
```

### Step 1.2: Auth Flow (Clerk)
```tsx
// src/lib/auth.ts
import * as SecureStore from "expo-secure-store"
import { ClerkProvider } from "@clerk/clerk-expo"

// Token cache for Clerk (persist across app restarts)
export const tokenCache = {
  async getToken(key: string) {
    try {
      return await SecureStore.getItemAsync(key)
    } catch {
      return null
    }
  },
  async saveToken(key: string, value: string) {
    try {
      await SecureStore.setItemAsync(key, value)
    } catch {
      // Handle error silently
    }
  },
  async clearToken(key: string) {
    try {
      await SecureStore.deleteItemAsync(key)
    } catch {}
  },
}
```

```tsx
// app/(auth)/sign-in.tsx
import { useSignIn } from "@clerk/clerk-expo"
import { Link, useRouter } from "expo-router"
import { Text, TextInput, TouchableOpacity, View, Alert } from "react-native"
import { useState } from "react"
import { SafeAreaView } from "react-native-safe-area-context"

export default function SignInScreen() {
  const { signIn, setActive, isLoaded } = useSignIn()
  const router = useRouter()
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [loading, setLoading] = useState(false)

  const handleSignIn = async () => {
    if (!isLoaded) return
    setLoading(true)
    try {
      const result = await signIn.create({ identifier: email, password })
      if (result.status === "complete") {
        await setActive({ session: result.createdSessionId })
        router.replace("/(tabs)")
      }
    } catch (err: unknown) {
      Alert.alert("Error", "Invalid email or password")
    } finally {
      setLoading(false)
    }
  }

  return (
    <SafeAreaView className="flex-1 bg-white px-6 justify-center">
      <Text className="text-3xl font-sans-bold text-gray-900 mb-2">Welcome back</Text>
      <Text className="text-base text-gray-500 mb-8">Sign in to your account</Text>

      <TextInput
        className="border border-gray-200 rounded-xl px-4 py-3.5 mb-4 text-base text-gray-900"
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
        autoComplete="email"
      />
      <TextInput
        className="border border-gray-200 rounded-xl px-4 py-3.5 mb-6 text-base text-gray-900"
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        autoComplete="current-password"
      />

      <TouchableOpacity
        className={`bg-brand-600 rounded-xl py-4 items-center ${loading ? "opacity-50" : ""}`}
        onPress={handleSignIn}
        disabled={loading}
      >
        <Text className="text-white font-sans-bold text-base">
          {loading ? "Signing in..." : "Sign In"}
        </Text>
      </TouchableOpacity>

      <Link href="/(auth)/sign-up" className="mt-6 text-center text-brand-600 font-sans-medium">
        Don't have an account? Sign up
      </Link>
    </SafeAreaView>
  )
}
```

### Step 1.3: Supabase Client
```typescript
// src/lib/supabase.ts
import { createClient } from "@supabase/supabase-js"
import { AppState } from "react-native"
import * as SecureStore from "expo-secure-store"

// Custom storage for Supabase auth tokens
const ExpoSecureStoreAdapter = {
  getItem: (key: string) => SecureStore.getItemAsync(key),
  setItem: (key: string, value: string) => SecureStore.setItemAsync(key, value),
  removeItem: (key: string) => SecureStore.deleteItemAsync(key),
}

export const supabase = createClient(
  process.env.EXPO_PUBLIC_SUPABASE_URL!,
  process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!,
  {
    auth: {
      storage: ExpoSecureStoreAdapter,
      autoRefreshToken: true,
      persistSession: true,
      detectSessionInUrl: false,
    },
  }
)

// Auto-pause when app goes to background (save battery/requests)
AppState.addEventListener("change", (state) => {
  if (state === "active") {
    supabase.auth.startAutoRefresh()
  } else {
    supabase.auth.stopAutoRefresh()
  }
})
```

### Step 1.4: MMKV Store (Fast Local Storage)
```typescript
// src/lib/storage.ts
import { MMKV } from "react-native-mmkv"

export const storage = new MMKV({
  id: "myapp-storage",
  encryptionKey: "your-encryption-key", // Optional encryption
})

// Type-safe wrappers
export const Storage = {
  getString: (key: string) => storage.getString(key),
  setString: (key: string, value: string) => storage.set(key, value),
  getBoolean: (key: string) => storage.getBoolean(key),
  setBoolean: (key: string, value: boolean) => storage.set(key, value),
  getNumber: (key: string) => storage.getNumber(key),
  setNumber: (key: string, value: number) => storage.set(key, value),
  delete: (key: string) => storage.delete(key),
  getObject: <T>(key: string): T | null => {
    const value = storage.getString(key)
    return value ? JSON.parse(value) : null
  },
  setObject: <T>(key: string, value: T) =>
    storage.set(key, JSON.stringify(value)),
}

// Zustand persist with MMKV
import { StateStorage } from "zustand/middleware"
export const zustandMMKVStorage: StateStorage = {
  getItem: (key) => storage.getString(key) ?? null,
  setItem: (key, value) => storage.set(key, value),
  removeItem: (key) => storage.delete(key),
}
```

### Step 1.5: Zustand Store with Persistence
```typescript
// src/stores/app-store.ts
import { create } from "zustand"
import { persist, createJSONStorage } from "zustand/middleware"
import { zustandMMKVStorage } from "@/lib/storage"

interface AppState {
  theme: "light" | "dark" | "system"
  hasOnboarded: boolean
  notificationsEnabled: boolean
  setTheme: (theme: AppState["theme"]) => void
  setHasOnboarded: (value: boolean) => void
  setNotificationsEnabled: (value: boolean) => void
  reset: () => void
}

const initialState = {
  theme: "system" as const,
  hasOnboarded: false,
  notificationsEnabled: false,
}

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({
      ...initialState,
      setTheme: (theme) => set({ theme }),
      setHasOnboarded: (value) => set({ hasOnboarded: value }),
      setNotificationsEnabled: (value) => set({ notificationsEnabled: value }),
      reset: () => set(initialState),
    }),
    {
      name: "app-store",
      storage: createJSONStorage(() => zustandMMKVStorage),
    }
  )
)
```

### Step 1.6: Environment Variables
```bash
# .env.local (gitignored)
EXPO_PUBLIC_APP_NAME=MyApp
EXPO_PUBLIC_APP_URL=https://myapp.com

# Clerk
EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...

# Supabase
EXPO_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# API
EXPO_PUBLIC_API_URL=https://api.myapp.com
```

```typescript
// src/config/env.ts — validated env
function requireEnv(key: string): string {
  const value = process.env[key]
  if (!value) throw new Error(`Missing required env var: ${key}`)
  return value
}

export const Config = {
  appName: process.env.EXPO_PUBLIC_APP_NAME ?? "MyApp",
  appUrl: requireEnv("EXPO_PUBLIC_APP_URL"),
  clerk: {
    publishableKey: requireEnv("EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY"),
  },
  supabase: {
    url: requireEnv("EXPO_PUBLIC_SUPABASE_URL"),
    anonKey: requireEnv("EXPO_PUBLIC_SUPABASE_ANON_KEY"),
  },
  api: {
    url: requireEnv("EXPO_PUBLIC_API_URL"),
  },
} as const
```

---

## 🗓️ DAY 2 — UI FOUNDATION

### Step 2.1: Design Tokens
```typescript
// src/lib/theme.ts
export const colors = {
  brand: {
    50: "#eff6ff",
    100: "#dbeafe",
    500: "#3b82f6",
    600: "#2563eb",
    700: "#1d4ed8",
    900: "#1e3a8a",
  },
  gray: {
    50: "#f9fafb",
    100: "#f3f4f6",
    200: "#e5e7eb",
    400: "#9ca3af",
    600: "#4b5563",
    900: "#111827",
  },
  success: "#22c55e",
  warning: "#f59e0b",
  error: "#ef4444",
}

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  "2xl": 48,
} as const

export const typography = {
  sizes: { xs: 12, sm: 14, base: 16, lg: 18, xl: 20, "2xl": 24, "3xl": 30 },
  weights: { regular: "400" as const, medium: "500" as const, bold: "700" as const },
  lineHeights: { tight: 1.25, normal: 1.5, relaxed: 1.75 },
}

export const radius = { sm: 8, md: 12, lg: 16, xl: 20, full: 9999 }

export const shadows = {
  sm: {
    shadowColor: "#000",
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
    elevation: 2,
  },
  md: {
    shadowColor: "#000",
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.08,
    shadowRadius: 8,
    elevation: 4,
  },
  lg: {
    shadowColor: "#000",
    shadowOffset: { width: 0, height: 8 },
    shadowOpacity: 0.12,
    shadowRadius: 16,
    elevation: 8,
  },
}
```

### Step 2.2: Base UI Components
```tsx
// src/components/ui/Button.tsx
import { ActivityIndicator, Text, TouchableOpacity, TouchableOpacityProps } from "react-native"
import Animated, { useAnimatedStyle, useSharedValue, withSpring } from "react-native-reanimated"
import * as Haptics from "expo-haptics"

const AnimatedTouchable = Animated.createAnimatedComponent(TouchableOpacity)

interface ButtonProps extends TouchableOpacityProps {
  variant?: "primary" | "secondary" | "outline" | "ghost" | "destructive"
  size?: "sm" | "md" | "lg"
  loading?: boolean
  children: string
  haptic?: boolean
}

const variantStyles: Record<string, string> = {
  primary: "bg-brand-600 border-transparent",
  secondary: "bg-gray-100 border-transparent",
  outline: "bg-transparent border border-gray-200",
  ghost: "bg-transparent border-transparent",
  destructive: "bg-red-600 border-transparent",
}

const variantTextStyles: Record<string, string> = {
  primary: "text-white",
  secondary: "text-gray-900",
  outline: "text-gray-900",
  ghost: "text-gray-700",
  destructive: "text-white",
}

const sizeStyles: Record<string, string> = {
  sm: "py-2 px-3 rounded-lg",
  md: "py-3.5 px-4 rounded-xl",
  lg: "py-4 px-6 rounded-xl",
}

const sizeTextStyles: Record<string, string> = {
  sm: "text-sm",
  md: "text-base",
  lg: "text-lg",
}

export function Button({
  variant = "primary",
  size = "md",
  loading = false,
  haptic = true,
  children,
  onPress,
  disabled,
  ...props
}: ButtonProps) {
  const scale = useSharedValue(1)

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }))

  const handlePressIn = () => {
    scale.value = withSpring(0.96, { damping: 15, stiffness: 300 })
  }

  const handlePressOut = () => {
    scale.value = withSpring(1, { damping: 15, stiffness: 300 })
  }

  const handlePress = (e: any) => {
    if (haptic) Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)
    onPress?.(e)
  }

  return (
    <AnimatedTouchable
      style={animatedStyle}
      className={`
        flex-row items-center justify-center
        ${variantStyles[variant]}
        ${sizeStyles[size]}
        ${disabled || loading ? "opacity-50" : ""}
      `}
      onPressIn={handlePressIn}
      onPressOut={handlePressOut}
      onPress={handlePress}
      disabled={disabled || loading}
      activeOpacity={1}
      {...props}
    >
      {loading && (
        <ActivityIndicator
          size="small"
          color={variant === "primary" || variant === "destructive" ? "#fff" : "#374151"}
          className="mr-2"
        />
      )}
      <Text className={`font-sans-bold ${variantTextStyles[variant]} ${sizeTextStyles[size]}`}>
        {children}
      </Text>
    </AnimatedTouchable>
  )
}
```

```tsx
// src/components/ui/Card.tsx
import { View, ViewProps } from "react-native"
import { shadows } from "@/lib/theme"

interface CardProps extends ViewProps {
  variant?: "default" | "elevated" | "outlined"
  padding?: "sm" | "md" | "lg"
}

export function Card({ variant = "default", padding = "md", style, children, ...props }: CardProps) {
  const paddingMap = { sm: 12, md: 16, lg: 24 }

  return (
    <View
      style={[
        {
          backgroundColor: "#fff",
          borderRadius: 16,
          padding: paddingMap[padding],
          ...(variant === "elevated" ? shadows.md : {}),
          ...(variant === "outlined" ? { borderWidth: 1, borderColor: "#e5e7eb" } : {}),
        },
        style,
      ]}
      {...props}
    >
      {children}
    </View>
  )
}
```

```tsx
// src/components/ui/Input.tsx
import { TextInput, TextInputProps, View, Text } from "react-native"
import { useState } from "react"

interface InputProps extends TextInputProps {
  label?: string
  error?: string
  hint?: string
}

export function Input({ label, error, hint, ...props }: InputProps) {
  const [focused, setFocused] = useState(false)

  return (
    <View className="mb-4">
      {label && (
        <Text className="text-sm font-sans-medium text-gray-700 mb-1.5">{label}</Text>
      )}
      <TextInput
        className={`
          border rounded-xl px-4 py-3.5 text-base text-gray-900 bg-white
          ${focused ? "border-brand-500" : error ? "border-red-400" : "border-gray-200"}
        `}
        placeholderTextColor="#9ca3af"
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        {...props}
      />
      {error && <Text className="text-xs text-red-500 mt-1">{error}</Text>}
      {hint && !error && <Text className="text-xs text-gray-400 mt-1">{hint}</Text>}
    </View>
  )
}
```

### Step 2.3: Bottom Tab Navigator
```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from "expo-router"
import { Ionicons } from "@expo/vector-icons"
import { Platform, View } from "react-native"
import { BlurView } from "expo-blur"

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarActiveTintColor: "#2563eb",
        tabBarInactiveTintColor: "#9ca3af",
        tabBarStyle: {
          position: "absolute",
          borderTopWidth: 0,
          elevation: 0,
          backgroundColor: Platform.OS === "ios" ? "transparent" : "#ffffff",
          height: Platform.OS === "ios" ? 84 : 64,
          paddingBottom: Platform.OS === "ios" ? 28 : 8,
        },
        tabBarBackground: () =>
          Platform.OS === "ios" ? (
            <BlurView
              tint="light"
              intensity={80}
              style={{ position: "absolute", inset: 0 }}
            />
          ) : (
            <View style={{ flex: 1, backgroundColor: "#fff" }} />
          ),
        tabBarLabelStyle: { fontSize: 11, fontWeight: "600" },
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: "Home",
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" color={color} size={size} />
          ),
        }}
      />
      <Tabs.Screen
        name="explore"
        options={{
          title: "Explore",
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="compass" color={color} size={size} />
          ),
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: "Profile",
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" color={color} size={size} />
          ),
        }}
      />
    </Tabs>
  )
}
```

### Step 2.4: Moti Animations
```tsx
// Common animation patterns with Moti
import { MotiView, MotiText } from "moti"
import { Skeleton } from "moti/skeleton"

// Fade in card
export function FadeInCard({ children, delay = 0 }: { children: React.ReactNode; delay?: number }) {
  return (
    <MotiView
      from={{ opacity: 0, translateY: 20 }}
      animate={{ opacity: 1, translateY: 0 }}
      transition={{ type: "timing", duration: 400, delay }}
    >
      {children}
    </MotiView>
  )
}

// Skeleton loader
export function CardSkeleton() {
  return (
    <View className="p-4 gap-3">
      <Skeleton colorMode="light" width="60%" height={20} radius={8} />
      <Skeleton colorMode="light" width="100%" height={14} radius={6} />
      <Skeleton colorMode="light" width="80%" height={14} radius={6} />
    </View>
  )
}

// Staggered list animation
export function StaggeredList({ items }: { items: React.ReactNode[] }) {
  return (
    <>
      {items.map((item, index) => (
        <MotiView
          key={index}
          from={{ opacity: 0, translateX: -20 }}
          animate={{ opacity: 1, translateX: 0 }}
          transition={{ type: "spring", delay: index * 80 }}
        >
          {item}
        </MotiView>
      ))}
    </>
  )
}
```

---

## 🗓️ DAY 3 — PRODUCTION FEATURES

### Push Notifications
```typescript
// src/lib/notifications.ts
import * as Notifications from "expo-notifications"
import * as Device from "expo-device"
import Constants from "expo-constants"
import { Platform } from "react-native"

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
})

export async function registerForPushNotifications(): Promise<string | null> {
  if (!Device.isDevice) {
    console.warn("Push notifications only work on physical devices")
    return null
  }

  const { status: existingStatus } = await Notifications.getPermissionsAsync()
  let finalStatus = existingStatus

  if (existingStatus !== "granted") {
    const { status } = await Notifications.requestPermissionsAsync()
    finalStatus = status
  }

  if (finalStatus !== "granted") return null

  const projectId = Constants.expoConfig?.extra?.eas?.projectId
  const token = await Notifications.getExpoPushTokenAsync({ projectId })

  if (Platform.OS === "android") {
    await Notifications.setNotificationChannelAsync("default", {
      name: "default",
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: "#2563eb",
    })
  }

  return token.data
}
```

### OTA Updates
```typescript
// src/hooks/useOTAUpdate.ts
import { useEffect } from "react"
import * as Updates from "expo-updates"
import { Alert } from "react-native"

export function useOTAUpdate() {
  useEffect(() => {
    if (__DEV__) return

    async function checkForUpdates() {
      try {
        const update = await Updates.checkForUpdateAsync()
        if (update.isAvailable) {
          await Updates.fetchUpdateAsync()
          Alert.alert(
            "Update Available",
            "A new version is ready. Restart to apply?",
            [
              { text: "Later", style: "cancel" },
              { text: "Restart", onPress: () => Updates.reloadAsync() },
            ]
          )
        }
      } catch {
        // Silently fail — updates are not critical
      }
    }

    checkForUpdates()
  }, [])
}
```

### Deep Linking
```typescript
// app.json — add scheme
// "scheme": "myapp"

// Usage: myapp://profile/123
// Expo Router handles this automatically via file-based routing
// app/profile/[id].tsx → myapp://profile/123

// Handle incoming links
import { useURL } from "expo-linking"

export function useDynamicLink() {
  const url = useURL()
  // url will be the deep link URL
  // Expo Router auto-handles routing
}
```

---

## 🚀 BUILD & RELEASE

### EAS Build Commands
```bash
# Development build (for testing with dev client)
eas build --profile development --platform ios
eas build --profile development --platform android

# Preview build (internal distribution)
eas build --profile preview --platform all

# Production build
eas build --profile production --platform all

# Submit to stores
eas submit --platform ios
eas submit --platform android

# OTA update (no store review needed)
eas update --branch production --message "Bug fix: resolved crash on login"
```

### GitHub Actions for Mobile
```yaml
# .github/workflows/mobile-ci.yml
name: Mobile CI

on:
  push:
    branches: [main]
    paths: ['apps/mobile/**']

jobs:
  build-preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - uses: expo/expo-github-action@v8
        with:
          expo-version: latest
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}
      - run: pnpm install
      - run: eas build --profile preview --platform all --non-interactive
```

---

## 📋 MOBILE PRE-LAUNCH CHECKLIST

```
Performance:
□ FlatList instead of ScrollView + map for long lists
□ Images using expo-image (WebP support, blurhash placeholders)
□ Reanimated for 60fps animations (no JS thread animations)
□ useMemo / useCallback where actually beneficial
□ Bundle size checked (npx expo export --dump-assetmap)

UX:
□ Keyboard avoiding behavior on all forms
□ Haptic feedback on important interactions
□ Loading and error states on all async operations
□ Empty states with helpful CTAs
□ Offline detection + user feedback

Platform:
□ Tested on iOS 15+ physical device
□ Tested on Android 10+ physical device
□ Notch/Dynamic Island handling (SafeAreaView)
□ Bottom navigation safe area handled
□ Keyboard behavior correct on both platforms
□ Dark mode works correctly

Store:
□ App icon (1024x1024 for iOS, 512x512 for Android)
□ Splash screen configured
□ Privacy policy URL
□ App Store screenshots (multiple sizes)
□ App description and keywords
□ In-app review prompt planned
```
