---
heroImage: '/android-performance-profiling-ui-cpu.svg'
title: 'Optimizing Android App Performance: Profiling UI and CPU'
description: 'Identify and resolve performance bottlenecks in your Android app using Android Studio Profilers, Systrace, and strict mode.'
pubDate: 'May 17 2026'
---

Users expect fluid 60fps (or even 120fps) animations and instantaneous responsiveness. Dropped frames and "Application Not Responding" (ANR) dialogs are a sure way to get uninstalled. Optimizing performance requires moving beyond guessing and utilizing rigorous profiling tools.

## The 16ms Rule

To achieve 60 frames per second, the system must draw a frame every ~16 milliseconds. If your layout calculations, measurements, or Main Thread logic take longer than 16ms, you drop frames—commonly known as "jank".

## Finding Bottlenecks with Android Studio Profilers

Android Studio includes powerful built-in profilers for CPU, Memory, Network, and Energy.

### The CPU Profiler

The CPU profiler tracks thread activity and records method traces. If your app stutters on scroll, start a recording, scroll the UI, and stop the recording. 

Examine the **Flame Chart**:
- The x-axis represents time.
- The y-axis represents the call stack.
- Look for wide, flat blocks on the Main Thread. These are methods taking a long time to execute. Typical culprits are complex JSON parsing on the main thread, slow database reads, or extremely nested UI hierarchies.

### Hierarchy Viewer and Layout Inspector

Deeply nested XML layouts require multiple measurement passes, increasing rendering time exponentially.

Use the **Layout Inspector** to view your app's view hierarchy in real-time. If you see layouts deeply nested 7 or 8 levels (e.g., `LinearLayout` inside `RelativeLayout` inside `ScrollView`), consider flattening them using `ConstraintLayout`.

*(Note: Jetpack Compose behaves differently and does not suffer from deep nesting penalties in the same way XML views do, but excessive recomposition can cause similar jank).*

## StrictMode: Catching Errors Early

`StrictMode` is an invaluable tool for catching accidental disk or network access on the main thread during development.

Add this to your `Application` class or main `Activity`:

```kotlin
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(StrictMode.ThreadPolicy.Builder()
        .detectDiskReads()
        .detectDiskWrites()
        .detectNetwork()   // or .detectAll()
        .penaltyLog()      // Log to Logcat
        .penaltyDialog()   // Show a dialog to annoy developers into fixing it
        .build())
}
```

If you accidentally trigger a Room database query without a coroutine dispatcher, StrictMode will immediately flag it.

## Micro-optimizations

- **Avoid object allocation in loops or `onDraw()`:** Allocating memory triggers the Garbage Collector. If you allocate objects inside a custom view's `onDraw()` (which is called 60 times a second), the GC will constantly run, causing micro-stutters.
- **Use ViewStubs:** For heavy UI elements that are rarely shown, use a `ViewStub` to inflate them only when needed, reducing initial layout inflation time.

Through consistent profiling, you can ensure your application remains blazing fast across all device tiers.
