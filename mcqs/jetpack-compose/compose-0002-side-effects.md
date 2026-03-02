# 📱 Android Interview MCQ #002

## Topic: Jetpack Compose Side Effects

## ❓ Question

You have a Composable that calls a suspend function to load data from network. Where should this call be made to avoid running it again on every recomposition?

A) Directly inside the Composable body  
B) Inside `remember {}` block  
C) Inside `LaunchedEffect(Unit)`  
D) Inside a `SideEffect` block  

## ✅ Correct Answer

C) Inside `LaunchedEffect(Unit)`

## 💡 Explanation

Composables can recompose many times (e.g., due to state changes). If you call a suspend function directly inside the composable body, it will be invoked **on every recomposition**, leading to wasted resources, potential race conditions, and even infinite loops.  

`LaunchedEffect` runs the given suspend block in a coroutine **only when its key changes**. By passing `Unit` (which never changes), the block executes **exactly once** when the composable first enters the composition. It is automatically cancelled when the composable leaves the composition, preventing memory leaks and making it lifecycle‑aware.

## 🚀 Sample App: Loading Data with `LaunchedEffect`

We’ll build a simple screen that loads a user profile from a fake network API when it first appears.

### 🧠 Step 1: ViewModel (Data Layer)

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class ProfileViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(ProfileUiState())
    val uiState: StateFlow<ProfileUiState> = _uiState.asStateFlow()

    fun loadProfile() {
        viewModelScope.launch {
            _uiState.value = _uiState.value.copy(isLoading = true)
            try {
                val profile = fetchProfileFromNetwork() // suspend function
                _uiState.value = _uiState.value.copy(
                    profile = profile,
                    isLoading = false
                )
            } catch (e: Exception) {
                _uiState.value = _uiState.value.copy(
                    error = e.message,
                    isLoading = false
                )
            }
        }
    }

    private suspend fun fetchProfileFromNetwork(): ProfileData {
        delay(1500) // Simulate network delay
        return ProfileData(name = "Alice", email = "alice@example.com")
    }
}

data class ProfileUiState(
    val isLoading: Boolean = false,
    val profile: ProfileData? = null,
    val error: String? = null
)

data class ProfileData(val name: String, val email: String)
```

### 🎨 Step 2: Composable Screen

```kotlin
import androidx.compose.runtime.*
import androidx.compose.material3.*
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun ProfileScreen(
    viewModel: ProfileViewModel = viewModel()
) {
    val uiState by viewModel.uiState.collectAsState()

    // ✅ Correct: trigger load exactly once when the screen appears
    LaunchedEffect(Unit) {
        viewModel.loadProfile()
    }

    // Display UI based on uiState
    when {
        uiState.isLoading -> CircularProgressIndicator()
        uiState.error != null -> Text("Error: ${uiState.error}")
        uiState.profile != null -> {
            Column {
                Text("Name: ${uiState.profile.name}")
                Text("Email: ${uiState.profile.email}")
            }
        }
    }
}
```

#### ✅ Why this works

- `LaunchedEffect(Unit)` runs the suspend `loadProfile()` function when `ProfileScreen` first enters composition.
- The call is **not repeated** on recompositions because the key `Unit` never changes.
- The coroutine is **cancelled** automatically if the composable leaves the composition (e.g., user navigates away).

## ❌ What Happens If You Use the Wrong Approach?

### ❌ Option A – Directly inside the composable body

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel = viewModel()) {
    viewModel.loadProfile() // ❌ called on every recomposition!
    // ... UI
}
```

**Problems:**
- `loadProfile()` runs again on every recomposition, flooding the network with requests.
- May cause an infinite recomposition loop if the function modifies state that triggers recomposition.

### ❌ Option B – Inside `remember {}`

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel = viewModel()) {
    remember { viewModel.loadProfile() } // ❌ loadProfile() is not suspending?
    // ...
}
```

**Problems:**
- `remember` only **stores** a value across recompositions; it does **not** execute code after the first composition. The lambda is executed during composition, but the result is stored. If `loadProfile()` is a suspend function, it cannot be called here because `remember`’s lambda is not a suspend block. Even if it were non‑suspending, it would run only once, but you cannot call suspend functions inside `remember`.

### ❌ Option D – Inside `SideEffect`

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel = viewModel()) {
    SideEffect {
        viewModel.loadProfile() // ❌ SideEffect cannot call suspend functions
    }
    // ...
}
```

**Problems:**
- `SideEffect`’s block is **not a suspend lambda** – you cannot call suspend functions inside it.
- It runs after every successful recomposition, so it would still be called many times if you made it a blocking call.

## 🎯 Best Practice: Let ViewModel Handle Loading, Composable Only Observe

The pattern above – triggering a one‑time load from the Composable – is valid, but even better is to let the ViewModel start loading automatically when it is created. Then the Composable simply observes the state.

```kotlin
class ProfileViewModel : ViewModel() {
    init {
        loadProfile() // start loading as soon as ViewModel is created
    }
    // ... rest same as before
}
```

Then the Composable no longer needs `LaunchedEffect` for loading:

```kotlin
@Composable
fun ProfileScreen(viewModel: ProfileViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    // just display uiState, loading already triggered
}
```

This decouples the side effect from the UI layer even further.

## 🧠 Interview Angle

If the interviewer asks:  
> “What if you need to call a suspend function in response to a button click?”

**Answer:**  
Use `rememberCoroutineScope()` to get a coroutine scope tied to the composable’s lifecycle, and launch the suspend function from the click handler.

```kotlin
val scope = rememberCoroutineScope()
Button(onClick = {
    scope.launch {
        viewModel.loadProfile() // suspend call
    }
}) { Text("Load") }
```

> “Why not just use `LaunchedEffect` with a changing key?”

You can use a changing key to re‑trigger the effect (e.g., `LaunchedEffect(shouldRefresh)`), but for one‑time loading on screen entry, `LaunchedEffect(Unit)` is the simplest.

## 🔥 What You Learned

- How to safely call suspend functions from a Composable without triggering them on every recomposition.
- The role of `LaunchedEffect` for one‑time side effects.
- Why the other options (direct call, `remember`, `SideEffect`) are incorrect for this scenario.
- Best practices: separating data loading into ViewModel and using `LaunchedEffect` only when necessary.
- How to handle user‑triggered suspend calls with `rememberCoroutineScope`.
