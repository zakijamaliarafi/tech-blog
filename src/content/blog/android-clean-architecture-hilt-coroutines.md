---
heroImage: '/android-clean-architecture-hilt-coroutines.svg'
title: 'Implementing Clean Architecture in Android with Hilt and Coroutines'
description: 'A comprehensive guide to structuring scalable Android apps using Clean Architecture principles, Hilt for dependency injection, and Kotlin Coroutines.'
pubDate: 'May 17 2026'
---

As Android codebases grow, maintaining a highly coupled architecture (like the classic "everything in the Activity") becomes impossible. Clean Architecture provides a layered approach that separates business logic from UI and data frameworks, making your app highly testable and scalable.

## The Three Layers

Clean Architecture is typically divided into three primary layers in Android:

1. **Presentation Layer:** Contains UI elements (Activities, Fragments, Compose) and ViewModels. It formats data for display and handles user input.
2. **Domain Layer:** The core of the application. Contains Use Cases (Interactors) and Domain Models. This layer should have zero dependencies on the Android framework or other layers.
3. **Data Layer:** Handles data retrieval and storage. Contains Repositories, Data Sources (Remote/Local API), and DTOs (Data Transfer Objects).

## Dependency Injection with Hilt

To adhere to the Dependency Inversion principle, high-level modules should not depend on low-level modules; both should depend on abstractions (interfaces). Hilt makes setting up DI in Android effortless.

### Defining Repositories and Use Cases

First, define the abstraction in the Domain layer:

```kotlin
// Domain Layer
interface UserRepository {
    suspend fun getUser(id: String): User
}

class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository
) {
    suspend operator fun invoke(id: String): Result<User> {
        return try {
            Result.success(userRepository.getUser(id))
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Implementing in the Data Layer

Implement the interface in the Data layer and bind it using Hilt:

```kotlin
// Data Layer
class UserRepositoryImpl @Inject constructor(
    private val api: UserApiService,
    private val dao: UserDao
) : UserRepository {
    override suspend fun getUser(id: String): User {
        // Fetch from network or cache
        return api.fetchUser(id).toDomainModel()
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds
    abstract fun bindUserRepository(
        impl: UserRepositoryImpl
    ): UserRepository
}
```

## Tying it together in the Presentation Layer

ViewModels use Hilt to inject Use Cases and expose state to the UI using Kotlin `StateFlow`.

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Loading)
    val uiState = _uiState.asStateFlow()

    fun fetchUser(id: String) {
        viewModelScope.launch {
            val result = getUserUseCase(id)
            result.onSuccess { user ->
                _uiState.value = UserUiState.Success(user)
            }.onFailure {
                _uiState.value = UserUiState.Error("Failed to fetch")
            }
        }
    }
}
```

This strict separation ensures that if you swap your database from Room to SQLDelight, or your UI from XML to Compose, the domain logic remains entirely untouched.
