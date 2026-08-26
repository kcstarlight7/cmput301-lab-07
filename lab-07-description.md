## CMPUT 301 - Lab 7: Navigation and UI Testing

## 1. Walkthrough
1. <ins>Create a new project named `lab-7` on Android Studio (File > New > New Project > Select "Empty Activity").</ins>
> [!WARNING]
> Make sure that the project language is **Kotlin**, not Java!

2. <ins>Add dependencies</ins>
- In the `build.gradle.kts (:app)` file, add `implementation("androidx.navigation:navigation-compose:2.9.8")` on a newline in the `dependencies` section
- Make sure you **sync** your project after applying the change

> [!NOTE]
> The most stable and latest version of the Jetpack Compose wrapper for the Android Navigation component may change from 2.9.8

3. <ins>Create the following files: `FormulaZeroApp.kt`, `HomeScreen.kt`, and `UpcomingRacesScreen.kt`, then add the following skeleton code:</ins>
- In `FormulaZeroApp.kt`:
```kotlin
@Composable
fun FormulaZeroApp() { }
```
- In `HomeScreen.kt`:
```kotlin
@Composable
fun HomeScreen(navController: NavController) { }
```
- In `UpcomingRacesScreen.kt`:
```kotlin
@Composable
fun UpcomingRacesScreen(navController: NavController) { }
```

- In the relevant files, make sure to import the classes `Composable` and `NavController`

> Each file's purpose is as follows:
> - `MainActivity` will be the app's entry point
> - `FormulaZeroApp.kt` will define how we navigate between screens
> - `HomeScreen.kt` will be the starting screen of the app and will allow us to navigate to other screens
> - `UpcomingRacesScreen.kt` is a screen that will allow us to add and display upcoming race locations as a list
  
4. <ins>Update `MainActivity.kt`</ins>
- Remove the functions `Greeting` and `GreetingPreview()`from `MainActivity.kt`
- Inside the scope of `Lab7Theme` in the `onCreate()` method, replace the code there with `FormulaZeroApp()`

5. <ins>Add navigation in `FormulaZeroApp.kt`</ins>
- We want to navigate from the "FormulaZero Home" screen to the "Upcoming Races" screen, so we will use a navigation controller and host to help us with that:
```kotlin
@Composable
fun FormulaZeroApp() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "home") {
        composable("home") { HomeScreen(navController) }
        composable("races") { UpcomingRacesScreen(navController) }
    }
}
```

- Make sure to import the functions `rememberNavController` and `NavHost`, and the extension function `NavGraphBuilder.composable`

6. <ins>Finish fleshing out `HomeScreen()` in `HomeScreen.kt`</ins>
- Inside a column layout component, we will have the title text, "FormulaZero Home", and a button that, when clicked, will take us to the "Upcoming Races" screen
```kotlin
@Composable
fun HomeScreen(navController: NavController) {
    Column(
        modifier = Modifier.fillMaxSize().padding(16.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "FormulaZero Home",
            fontSize = 24.sp
        )

        Spacer(modifier = Modifier.height(16.dp))

        Button(
            onClick = {
                navController.navigate("races")
            },
            modifier = Modifier.testTag("raceButton") // for testing
        ) {
            Text("Upcoming Races")
        }
    }
}
```

- Make sure to import the functions `Column`, `Text`, `Spacer`, and `Button`, the classes `Modifier` and `Alignment`, the extension functions `Modifier.fillMaxSize`, `Modifier.padding`, `Modifier.height`, and `Modifier.testTag`, the extension properties `Int.dp` and `Int.sp`, and the object `Arrangement`

7.  <ins>Finish fleshing out `UpcomingRacesScreen()` in `UpcomingRacesScreen.kt`</ins>
- To keep changes on our UI, we need to use the `remember` function for the current location value inputted in our text field, and for the list of race locations we want to display
  - Inside the `UpcomingRacesScreen()` function, add:
<br></br>
  ```kotlin
  var location by remember {
      mutableStateOf("")
  }

  var races = remember {
      mutableStateListOf<String>()
  }
  ```

  - Make sure to import the function `remember`, `mutableStateOf`, and `mutableStateListOf`, and operators `State.getValue` and `MutableState.setValue`
  
- Inside a column layout component, we will have a back button to return to the "Home" screen, the title text "Upcoming Races", a text field plus a button to add race locations, and a list (`LazyColumn`) to display upcoming race locations
   - Inside the `UpcomingRacesScreen()` function, additionally add:
<br></br>
  ```kotlin
  Column(
      modifier = Modifier.fillMaxSize().padding(16.dp)
  ) {
      Spacer(modifier = Modifier.height(16.dp))
  
      // back button
      Button(
          onClick = {
              navController.popBackStack()
          },
          modifier = Modifier.testTag("backButton") // for testing
      ) {
          Text("Back")
      }
  
      Spacer(modifier = Modifier.height(16.dp))
  
      // title text
      Text(
          text = "Upcoming Races",
          fontSize = 30.sp
      )
  
      Spacer(modifier = Modifier.height(16.dp))
  
      // text field and button to add race locations
      Row (
          modifier = Modifier.fillMaxWidth(),
          horizontalArrangement = Arrangement.SpaceBetween
      ) {
          TextField(
              value = location,
              onValueChange = {
                  location = it // update location to new value
              },
              label = {
                  Text("Race Location")
              },
              modifier = Modifier.testTag("locationInput") // for testing
          )
  
          Spacer(modifier = Modifier.height(8.dp))
  
          Button(
              onClick = {
                  if (location.isNotBlank()) {
                      races.add(location)
                      location = ""
                  }
              },
              modifier = Modifier.testTag("addRaceButton") // for testing
          ) {
              Text("Add")
          }
      }
  
      // list to display races
      LazyColumn {
          items(races) { race ->
              Spacer(modifier = Modifier.height(12.dp))
  
              Text(
                  text = race,
                  fontSize = 24.sp
              )
          }
      }
  }
  ```

- Make sure to import the functions `Column`, `Text`, `Spacer`, `Button`, `Row`, `TextField`, and `LazyColumn`, the class `Modifier`, the extension functions `Modifier.fillMaxSize`, `Modifier.padding`, `Modifier.height`, `Modifier.testTag`, `Modifier.fillMaxWidth`, and `LazyListScope.items`, the extension properties `Int.dp` and `Int.sp`, and the object `Arrangement`
  
- At this point, we have all the code we need before we write our UI test for this walkthrough. You can run the app and see if it works as intended.
  
10. <ins>Create and run the UI test</ins>
- Create a new class file called `FormulaZeroTest.kt` under the folder `com.example.lab_7 (androidTest)`
- First, we add the following code for set up:
  - `@RunWith(...)` tells JUnit to use the Android testing system
  - `@get:Rule` automatically sets up Compose UI testing
  - `composeTestRule` is the object used to test UI
<br></br>
   ```kotlin
  @RunWith(AndroidJUnit4::class)
  class FormulaZeroTest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>() // access MainActivity
  }
  ```
   
  - Make sure to import the classes `RunWith`, `AndroidJUnit4`, and `Rule`, and the function `createAndroidComposeRule`
    
- Then, inside the FormulaZeroTest class, we will create a test method that will navigate to the "Upcoming Races" screen, add a race, and return to the "Home" screen
```kotlin
@Test
fun navigate_to_races_add_race_and_return_home() {
    // navigate to race screen
    composeTestRule
        .onNodeWithTag("raceButton")
        .performClick()

    // verify race screen
    composeTestRule
        .onNodeWithText("Upcoming Races")
        .assertExists()

    // enter race location
    composeTestRule
        .onNodeWithTag("locationInput")
        .performTextInput("Canada")

    // add race location
    composeTestRule
        .onNodeWithTag("addRaceButton")
        .performClick()

    // verify race appears
    composeTestRule
        .onNodeWithText("Canada")
        .assertExists()

    // return to home screen
    composeTestRule
        .onNodeWithTag("backButton")
        .performClick()

    // verify return to home screen
    composeTestRule
        .onNodeWithText("FormulaZero Home")
        .assertExists()
}
```

- Make sure to import the class `Test` and the extension functions `SemanticsNodeInteractionsProvider.onNodeWithTag`, `SemanticsNodeInteraction.performClick`, `SemanticsNodeInteractionsProvider.onNodeWithText`, and `SemanticsNodeInteraction.performTextInput`

- Finally, right-click the `FormulaZeroTest` file and click `Run FormulaZeroTest` to run the test.
  - You should see that 1/1 tests passed!

> [!NOTE]
> - The `walkthrough.md` contains the complete code for `MainActivity.kt`, `FormulaZeroApp.kt`, `HomeScreen.kt`, `UpcomingRacesScreen.kt`, `FormulaZeroTest.kt` and `build.gradle.kts (:app)` for double-checking purposes
> - While you could simply copy and paste the code, you're stripping yourself of a learning opportunity :)

## 2. Lab 7 Participation Exercise
1. Create another file called `DriversScreen.kt`
  - You will need to update `HomeScreen.kt` and `FormulaZeroApp.kt` so you can navigate from the  "FormulaZero Home" screen to the "Drivers" screen
  - In `DriversScreen.kt`:
    - There should be a button to return to the `HomeScreen`
    - There should be text indicating that it's the `DriversScreen` (i.e., "FormulaZero Drivers")
    - There should be a text field and a button to add drivers, where drivers added will be displayed in a list format
    - There should be a button to clear all drivers displayed

2. Create a test that verifies the following:
  - Verify that you can properly go from the `HomeScreen` to the `DriversScreen`
  - Verify that you can properly add at least one driver, and if that driver is properly displayed
  - Verify that you can properly clear the driver(s) from the list
  - Verify that the back button takes you back to the `HomeScreen`

> [!NOTE]
> Data persistence is optional

## 3. Submission Specifications
1. Fork and then clone this repository
    - Make sure your forked repository is **public**
    - Hint: Use `git clone`
3. Add your Android Studio Project to your forked repository
    - Hint: Use `git add`, `git commit`, and `git push`
4. Update the `README.md` file with your details and references/collaborators
5. Update the `LICENSE.md` file with your full name
6. Submit the link to your GitHub repository on Canvas

> [!IMPORTANT]
> - This lab is graded on a complete/incomplete basis. You will receive a “complete” if you finish the walkthrough, complete the participation exercise, and follow ALL submission requirements. You will receive an “incomplete” if any of these requirements are not met, such as an inaccessible (non-public) repository, missing participation exercise, or an incorrect submission.
> - **There will be no exceptions, partial marks, or late submissions allowed.**
