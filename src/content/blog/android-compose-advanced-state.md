---
heroImage: '/android-compose-advanced-state.svg'
title: 'Jetpack Compose: Advanced State Management and Recomposition'
description: 'Master Jetpack Compose state management. Learn how to minimize recompositions, use derived state, and handle side effects properly.'
pubDate: 'May 17 2026'
---

Jetpack Compose has revolutionized Android UI development. Instead of mutating views, we declare what the UI should look like based on current state. However, inefficient state management can lead to excessive recompositions, dropping frames and draining battery.

## The Recomposition Loop

When state changes, Compose re-executes the composable functions reading that state. This is called *recomposition*. Compose is smart—it only recomposes the parts of the tree that read the changed state. But if you're not careful, you can trigger unnecessary recompositions.

### Stabilizing State

Compose can only skip recomposition if it knows the inputs haven't changed. Primitive types and data classes with `val` properties are "Stable". If you pass an unstable object (like an interface or a class with `var` properties from another module) to a composable, it will always recompose when its parent does.

You can force Compose to treat a class as stable using the `@Stable` annotation:

```kotlin
@Stable
class UnstableClassFromNetwork(var mutableData: String)
```

## Derived State: Avoid Wasted Work

Sometimes, your UI depends on a specific condition calculated from a rapidly changing state, like a scroll offset.

```kotlin
// BAD
val listState = rememberLazyListState()
// Recomposes on EVERY pixel scrolled!
val showButton = listState.firstVisibleItemIndex > 0

if (showButton) {
    ScrollToTopButton()
}
```

Using `derivedStateOf` tells Compose to only trigger a recomposition when the *result* of the calculation changes.

```kotlin
// GOOD
val listState = rememberLazyListState()
val showButton by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 0 }
}
// Now only recomposes when the index changes from 0 to 1, or 1 to 0.
```

## Managing Side Effects

Composables should be side-effect free. But sometimes you need to trigger a one-off event, like showing a Snackbar or starting a network request when a screen opens.

- **`LaunchedEffect`:** Runs a suspend function in the scope of the composable. It cancels the coroutine if the composable leaves the tree. Use it for initial loads.
- **`rememberCoroutineScope`:** Use this when you need to launch a coroutine from a callback (like an `onClick` event), rather than from the composable itself.

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel) {
    // Runs once when ProfileScreen enters the composition
    LaunchedEffect(Unit) {
        viewModel.fetchUserData()
    }

    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()

    Button(onClick = {
        scope.launch {
            snackbarHostState.showSnackbar("Saved successfully!")
        }
    }) {
        Text("Save")
    }
}
```

By understanding these advanced concepts, you can build buttery-smooth Compose applications.
