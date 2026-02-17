# 📱 Android Interview MCQ #001
## Topic: Jetpack Compose

## ❓ Question

In a Jetpack Compose screen, you collect a StateFlow from a ViewModel.
What is the recommended way?

A) Use stateFlow.value  
B) Use collectAsState()  
C) Launch coroutine in LaunchedEffect  
D) Convert to LiveData  


## ✅ Correct Answer
B) Use collectAsState()


## 💡 Explanation

collectAsState():

- Collects Flow safely
- Triggers recomposition
- Is lifecycle-aware
- Is idiomatic Compose

Using .value will not recompose the UI automatically.
