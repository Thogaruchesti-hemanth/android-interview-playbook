# 📱 Android Interview MCQ #001
## Topic: Jetpack Compose

## ❓ Question

In a Jetpack Compose screen, you collect a `StateFlow` from a ViewModel to show UI data. What is the recommended way to collect it so the UI updates correctly and respects the lifecycle?

A) Use `viewModel.stateFlow.value` directly inside Composable
B) Use `collectAsState()` inside the Composable
C) Launch a coroutine in `LaunchedEffect` and manually update a variable
D) Convert `StateFlow` to LiveData and observe it 


## ✅ Correct Answer
B) Use `collectAsState()` inside the Composable


## 💡 Explanation

`collectAsState()` collects the `StateFlow` in a lifecycle-aware way and automatically triggers recomposition when data changes. It is the recommended and clean approach in Compose for observing Flow-based state from a ViewModel.

Using .value will not automatically recompose the UI.

## 🚀 Sample App: StateFlow + collectAsState()

### 🧠 Step 1: Create ViewModel

```kotlin
import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class CounterViewModel : ViewModel() {

    private val _counter = MutableStateFlow(0)
    val counter: StateFlow<Int> = _counter.asStateFlow()

    fun increment() {
        _counter.value += 1
    }
}
```

#### ✅ What’s happening?

* `_counter` holds the mutable state.
* `counter` exposes immutable `StateFlow`.
* Whenever value changes → UI should recompose.

### 🎨 Step 2: Composable Screen

```kotlin
import androidx.compose.runtime.*
import androidx.compose.material3.*
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun CounterScreen(
    viewModel: CounterViewModel = viewModel()
) {
    val count by viewModel.counter.collectAsState()

    Column {
        Text(text = "Count: $count")

        Button(onClick = {
            viewModel.increment()
        }) {
            Text("Increment")
        }
    }
}
```

## 🧩 What `collectAsState()` Does

```kotlin
val count by viewModel.counter.collectAsState()
```

This line:

1. Collects the `StateFlow`
2. Converts it into Compose `State`
3. Automatically triggers recomposition when value changes
4. Is lifecycle-aware (when using lifecycle-runtime-compose dependency)

👉 So when `increment()` updates `_counter`
→ `StateFlow` emits new value
→ `collectAsState()` receives it
→ UI recomposes
→ Text updates automatically


## ❌ What Happens If We Do This Instead?

```kotlin
val count = viewModel.counter.value
```

Problem:

* UI will NOT recompose automatically.
* Compose does not track changes on `.value`
* You lose reactive UI behaviour.


## 🎯 How To Make It Lifecycle-Aware (Best Practice)

Add dependency:

```gradle
implementation("androidx.lifecycle:lifecycle-runtime-compose:2.6.2")
```

Then use:

```kotlin
val count by viewModel.counter.collectAsStateWithLifecycle()
```

This ensures:

* Collection stops when screen is not visible
* No unnecessary work
* Recommended in production


## 🧠 Interview Angle

If the interviewer asks:

> Why not use LaunchedEffect?

Answer:

* `collectAsState()` is simpler
* Automatically handles recomposition
* Cleaner and more idiomatic Compose way


## 🔥 What You Learned

* How to expose `StateFlow` from ViewModel
* How Compose observes Flow
* Why `collectAsState()` is correct answer
* Why `.value` is wrong in Composable
* Why lifecycle-aware collection matters


