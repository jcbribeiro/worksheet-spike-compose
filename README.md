## Jetpack Compose Practical Worksheet: Building a Spike Game App

This worksheet is a practical spike to explore the fundamentals of **Jetpack Compose** on Android. The app tracks game statistics (played and won counts) across a simple two-screen flow using a shared `ViewModel` and navigation.

### 0. Project Setup

1. Open **Android Studio** → **File > New > New Project…**
2. Template: **Empty Activity**
3. Name: **spike-compose**
4. Language: **Kotlin**
5. Build configuration language: **Kotlin DSL (`build.gradle.kts`)**

Android Studio will configure the Compose BOM and core libraries. Update your Version Catalog and dependencies to include Navigation and Lifecycle ViewModel Compose.

In `gradle/libs.versions.toml`:

```toml
[versions]
navigationCompose = "2.7.7"
lifecycleViewmodelCompose = "2.8.3"

[libraries]
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigationCompose" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleViewmodelCompose" }

```

In `app/build.gradle.kts`:

```kotlin
dependencies {
    // Navigation Compose dependency
    implementation(libs.androidx.navigation.compose)

    // Lifecycle ViewModel Compose dependency
    implementation(libs.androidx.lifecycle.viewmodel.compose)
}

```

> #### Imperative XML vs Declarative Compose
> 
> 
> * **View System / XML**: you define layouts in XML trees and mutate them imperatively in code (`findViewById`, `setText()`).
> * **Jetpack Compose**: you **describe** the UI in Kotlin functions annotated with `@Composable`; the UI is a *function of state*.
> * Key idea: **when state changes, Compose automatically recomposes** affected composables.
> 
> 
> Docs:
> * Jetpack Compose Tutorial — [https://developer.android.com/develop/ui/compose/tutorial]
> * Compose State and Jetpack Compose Lifecycle — [https://developer.android.com/develop/ui/compose/state]
> 
> 

---

## 1. ViewModel & State

Create a new file named `GameViewModel.kt`:

```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel

class GameViewModel : ViewModel() {
  var playedGames by mutableStateOf(0)
    private set

  var wonGames by mutableStateOf(0)
    private set

  fun incrementPlayedGames() {
    playedGames += 1
  }

  fun incrementWonGames() {
    wonGames += 1
  }

  fun resetGame() {
    playedGames = 0
    wonGames = 0
  }
}

```

> #### `ViewModel`, `mutableStateOf`, and Encapsulation
> 
> 
> * `ViewModel`: survives configuration changes (such as screen rotations) and holds business state for the UI lifecycle.
> * `mutableStateOf(x)`: creates an observable snapshot state holder. When read inside a composable, Compose subscribes to changes; when written to, it triggers **recomposition**.
> * `private set`: exposes read-only values to the UI while ensuring mutations only occur through explicit methods (`incrementPlayedGames()`), establishing **Unidirectional Data Flow (UDF)**.
> 
> 
> 
> 
> Docs:
> * [https://developer.android.com/topic/libraries/architecture/viewmodel](https://developer.android.com/topic/libraries/architecture/viewmodel)
> * [https://developer.android.com/develop/ui/compose/state#state-in-viewmodels](https://www.google.com/search?q=https://developer.android.com/develop/ui/compose/state%2523state-in-viewmodels)
> 
> 

---

## 2. Navigation Host

Create a new file named `Navigation.kt`:

```kotlin
import androidx.compose.runtime.Composable
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable

@Composable
fun AppNavHost(navController: NavHostController, viewModel: GameViewModel) {
  NavHost(navController = navController, startDestination = "landing") {
    composable("landing") {
      LandingScreen(
        viewModel = viewModel,
        navController = navController
      )
    }
    composable("game") {
      GameScreen(
        viewModel = viewModel,
        navController = navController
      )
    }
  }
}

```

> #### Navigation in Jetpack Compose
> 
> 
> * `NavHostController`: the central coordinator that tracks the backstack and performs transitions between destinations.
> * `NavHost`: links the controller with a navigation graph that maps route strings (e.g. `"landing"`, `"game"`) to composable destinations.
> 
> 
> Docs:
> * [https://developer.android.com/develop/ui/compose/navigation](https://developer.android.com/develop/ui/compose/navigation)
> 
> 

---

## 3. Landing Screen (Main Screen)

Create a new file named `LandingScreen.kt`:

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.navigation.NavHostController
import androidx.navigation.compose.rememberNavController

@Composable
fun LandingScreen(
  viewModel: GameViewModel,
  navController: NavHostController
) {
  Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
  ) {
    Text(text = "Played games: ${viewModel.playedGames} | Won games: ${viewModel.wonGames}")
    Spacer(modifier = Modifier.height(16.dp))
    Button(onClick = {
      viewModel.incrementPlayedGames()
      navController.navigate("game")
    }) {
      Text(text = "Play")
    }
  }
}

// MARK: - Preview
@Preview(showBackground = true)
@Composable
fun LandingScreenPreview() {
  val mockNavController = rememberNavController()
  val mockViewModel = GameViewModel()
  LandingScreen(viewModel = mockViewModel, navController = mockNavController)
}

```

> #### Layout Basics: Column, Row, and Modifiers
> 
> 
> * `Column`: positions items vertically, equivalent to a vertical `LinearLayout`.
> * `Modifier`: the standard mechanism in Compose to configure element size, padding, layout behavior, and styling (e.g., `Modifier.fillMaxSize()`).
> 
> 

---

## 4. Game Screen & Reusable Component

Create a new file named `GameScreen.kt`:

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.navigation.NavHostController
import androidx.navigation.compose.rememberNavController

@Composable
fun GameScreen(
  viewModel: GameViewModel,
  navController: NavHostController
) {
  var toggleState1 by remember { mutableStateOf(false) }
  var toggleState2 by remember { mutableStateOf(false) }

  Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
  ) {
    ToggleButton(toggleState1) { isChecked ->
      toggleState1 = isChecked
      if (toggleState1 && toggleState2) {
        viewModel.incrementWonGames()
        navController.popBackStack()
      }
    }
    Spacer(modifier = Modifier.height(16.dp))
    ToggleButton(toggleState2) { isChecked ->
      toggleState2 = isChecked
      if (toggleState1 && toggleState2) {
        viewModel.incrementWonGames()
        navController.popBackStack()
      }
    }
  }
}

@Composable
fun ToggleButton(isChecked: Boolean, onCheckedChange: (Boolean) -> Unit) {
  Button(onClick = { onCheckedChange(!isChecked) }) {
    Text(text = if (isChecked) "ON" else "OFF")
  }
}

// MARK: - Preview
@Preview(showBackground = true)
@Composable
fun GameScreenPreview() {
  val mockNavController = rememberNavController()
  val mockViewModel = GameViewModel()
  GameScreen(viewModel = mockViewModel, navController = mockNavController)
}

```

> #### Local State (`remember`) and State Hoisting
> 
> 
> * `remember { mutableStateOf(...) }`: allocates and stores state in the composition memory. It survives recompositions during the lifetime of the screen, but is reset once the screen leaves the composition (or backstack).
> 
> 
> * State Hoisting (`ToggleButton`): `ToggleButton` does not own its state. It receives a value (`isChecked`) and emits an event (`onCheckedChange`). This makes the component stateless, testable, and reusable.
> 
> 
> * `navController.popBackStack()`: pops the current destination off the backstack and returns to the previous screen.
> 
> 
> 
> 
> Docs:
> * [https://developer.android.com/develop/ui/compose/state#state-hoisting](https://developer.android.com/develop/ui/compose/state#state-hoisting)
> 
> 

---

## 5. App Entry Point

Edit `MainActivity.kt` to bind the ViewModel lifecycle to the activity and initialize the Compose UI tree:

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.viewModels
import androidx.navigation.compose.rememberNavController

class MainActivity : ComponentActivity() {
  private val gameViewModel: GameViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      val navController = rememberNavController()
      AppNavHost(navController = navController, viewModel = gameViewModel)
    }
  }
}

```

---

## 6. To Learn More

* Jetpack Compose Tutorial: [https://developer.android.com/develop/ui/compose/tutorial](https://developer.android.com/develop/ui/compose/tutorial)

* Kotlin Crash-Course: [https://developer.android.com/kotlin/learn](https://developer.android.com/kotlin/learn)

* Android Basics with Compose (Guided Path): [https://developer.android.com/courses/android-basics-compose/course](https://developer.android.com/courses/android-basics-compose/course)
