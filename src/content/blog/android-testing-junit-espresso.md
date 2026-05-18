---
heroImage: '/android-testing-junit-espresso.svg'
title: 'Testing Android Apps: From JUnit to Espresso and UI Automator'
description: 'A comprehensive strategy for testing Android applications. Discover how to effectively utilize Unit Tests, Integration Tests, and UI Tests.'
pubDate: 'May 17 2026'
---

Testing is the foundation of a stable Android application. Without automated tests, every new feature or refactor risks breaking existing functionality. Android testing is generally divided into local unit tests and instrumented UI tests.

## The Testing Pyramid

1. **Unit Tests (70%):** Fast, isolated, run on the local JVM.
2. **Integration Tests (20%):** Test how components work together.
3. **UI/End-to-End Tests (10%):** Slow, run on an emulator/device, test the entire user flow.

## 1. Local Unit Tests with JUnit and MockK

Unit tests run locally on your computer's JVM in the `src/test/` directory. They are blazing fast because they don't require an Android emulator.

The goal is to test pure Kotlin/Java logic (like ViewModels and Use Cases) without touching the Android Framework (`Context`, `Activity`, etc.).

```kotlin
// Example ViewModel Test
class UserViewModelTest {

    // Mock dependencies
    private val userRepository = mockk<UserRepository>()
    private lateinit var viewModel: UserViewModel

    @Before
    fun setup() {
        viewModel = UserViewModel(userRepository)
    }

    @Test
    fun `when fetchUser succeeds, state is Success`() = runTest {
        // Arrange
        val fakeUser = User("1", "Alice")
        coEvery { userRepository.getUser("1") } returns Result.success(fakeUser)

        // Act
        viewModel.fetchUser("1")

        // Assert
        assertEquals(UserUiState.Success(fakeUser), viewModel.uiState.value)
    }
}
```

## 2. Instrumented UI Tests with Espresso

Instrumented tests run on a real device or emulator and live in the `src/androidTest/` directory. Espresso is the standard framework for testing Android Views.

Espresso uses a specific formula: `ViewMatchers` (find), `ViewActions` (perform), and `ViewAssertions` (check).

```kotlin
@RunWith(AndroidJUnit4::class)
class LoginActivityTest {

    @get:Rule
    val activityRule = ActivityScenarioRule(LoginActivity::class.java)

    @Test
    fun testFailedLoginShowsError() {
        // Find email field and type text
        onView(withId(R.id.email_input))
            .perform(typeText("wrong@email.com"), closeSoftKeyboard())

        // Find password field and type text
        onView(withId(R.id.password_input))
            .perform(typeText("badpass"), closeSoftKeyboard())

        // Click login button
        onView(withId(R.id.login_button))
            .perform(click())

        // Assert error message is displayed
        onView(withText("Invalid credentials"))
            .check(matches(isDisplayed()))
    }
}
```

*(Note: If you use Jetpack Compose, use `ComposeTestRule` instead of Espresso).*

## 3. Cross-App Testing with UI Automator

Espresso is limited to testing views within your own application's sandbox. If your test requires interacting with system dialogs (like runtime permission prompts) or opening another app, you need **UI Automator**.

UI Automator can inspect the entire screen hierarchy regardless of the active application.

```kotlin
val device = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())

// Wait for a system permission dialog
val allowButton = device.findObject(
    UiSelector().textMatches("(?i)allow|while using the app")
)

if (allowButton.exists()) {
    allowButton.click()
}
```

A robust testing suite utilizes all three levels: JUnit for fast logic verification, Espresso for specific UI flows, and UI Automator for system interactions.
