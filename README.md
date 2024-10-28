### 1. Project Setup

**Step 1:** Create a new Android project using Android Studio:

- **Project Name:** `spike-compose`
- **Template:** Choose `Empty Activity`

**Step 2:** Update dependencies in `build.gradle` (Module: `app`):

Ensure that the following dependencies are added:

```gradle
dependencies {
    // ...

    // Navigation Compose dependency
    implementation(libs.androidx.navigation.compose)

    // Lifecycle ViewModel Compose dependency
    implementation(libs.androidx.lifecycle.viewmodel.compose)

    // ...
}
```

### 2. Create the `GameViewModel`

Create a file named `GameViewModel.kt`:

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

### 3. Define the App's Navigation

Create a file named `Navigation.kt`:

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

### 4. Implement the Landing Screen

Create a file named `LandingScreen.kt`:

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontStyle
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
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

@Preview(showBackground = true)
@Composable
fun LandingScreenPreview() {
  val mockNavController = rememberNavController()
  val mockViewModel = GameViewModel()
  LandingScreen(viewModel = mockViewModel, navController = mockNavController)
}
```

### 5. Implement the Game Screen

Create a file named `GameScreen.kt`:

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

@Preview(showBackground = true)
@Composable
fun GameScreenPreview() {
  val mockNavController = rememberNavController()
  val mockViewModel = GameViewModel()
  GameScreen(viewModel = mockViewModel, navController = mockNavController)
}
```

### 6. Update the Main Activity

Modify the `MainActivity.kt` file:

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
