## High-Level Blueprint

| Phase | Goal | Key Artifacts |
|-------|------|---------------|
| 0. Scaffolding & Tooling | Create a clean Android/Kotlin project with CI, linting, and test harness | Git repo, GitHub Actions (unit + UI tests), `build.gradle.kts` with Compose & Room deps |
| 1. Domain & Storage | A **tested** data layer that persists `ShootingSession` objects locally with Room | Kotlin `enum`, `@Entity`, `@Dao`, `RoomDatabase`, JUnit tests + in-memory DB |
| 2. Repository & Use-Cases | Thin repository + use-case layer isolating ViewModels from Room | `SessionRepository`, `CreateSessionUseCase`… unit-tested |
| 3. ViewModels | Business logic (state, validation, navigation events) | `HomeViewModel`, `SessionEntryViewModel`, tests with Turbine |
| 4. UI – Vertical Slices | Compose screens wired to ViewModels (no real nav yet) | `HomeScreen`, `PatternPickerScreen`, `ScoreEntryScreen`, `NoteScreen`, preview + screenshot tests |
| 5. Navigation & Back-stack | App graph & exit dialogs | `NavHost`, `rememberNavController`, unit tests with `TestNavHostController` |
| 6. Feature Completion | History list, discard-session dialog, totals, pattern reset | Final ViewModel/UI glue |
| 7. Polish & Hardening | Rotation lock, error states, logging, strings | QA check list |
| 8. Instrumentation | End-to-end Espresso tests for main flows | CI green |
| 9. Release Prep | Versioning, README, store listing assets (optional) | Signed APK/AAB |

---

## Chunking Into Iterations and Micro-Tasks

Below, each **Iteration** is small enough for a single focused PR with green CI in under an hour.  
Within each iteration, **Micro-tasks** map one-to-one to individual commits.

### Iteration 0 Scaffolding
1. Init repo — `git init`, `.gitignore (Android)`.
2. Create project — Android Studio wizard → empty Activity.
3. Add CI workflow — GitHub Actions + `./gradlew test` + `./gradlew lint`.
4. Add dependencies — Compose BOM, Room, Hilt, JUnit 5, Turbine.

### Iteration 1 Domain Model
1. Add `ShootingPattern` enum.
2. Add `ShootingSession` `@Entity` with `TypeConverter` for `List<Int>`.
3. Write a unit test that creates an in-memory DB and inserts + reads a session.

### Iteration 2 DAO & Repository
1. Add `SessionDao` with `insert()` & `getAllOrderedByDateDesc()`.
2. Write DAO test verifying order by date.
3. Create `SessionRepository` (wraps DAO).
4. Mock DAO and test repository logic (especially total-score pre-calc).

### Iteration 3 Use-Cases
1. `CreateSessionUseCase` – validates 4 scores, computes total, calls repository.
2. `GetLatestSessionUseCase` – repository flow.
3. Unit tests with fake repository implementations.

### Iteration 4 ViewModels
1. `HomeViewModel` exposes `latestSessionUiState`.
2. `SessionEntryViewModel` holds pattern, scores, note, validation, save event.
3. ViewModel tests with Turbine.

### Iteration 5 Home UI
1. Compose `HomeScreen` with card for latest session and FAB “Start”.
2. Preview in Android Studio.
3. Screenshot test verifying default placeholder text.

### Iteration 6 Session Flow UI
1. Compose `PatternPickerScreen` (two toggle buttons).
2. Compose `ScoreEntryScreen` (4 `OutlinedTextField`s, header showing pattern).
3. Compose `NoteEntryScreen` (optional).
4. Wire these screens to ViewModel functions (no navigation yet).
5. Compose test: entering scores enables “Next”.

### Iteration 7 Navigation
1. Add `NavHost` & routes.
2. Add `BackHandler` in entry flow → discard dialog.
3. Test navigation with `TestNavHostController`.

### Iteration 8 History Screen
1. `HistoryViewModel` + `HistoryScreen` (`LazyColumn`).
2. Compose test: list shows newest first.

### Iteration 9 Instrumentation
1. Espresso test: complete session, return to Home, verify card shows data.
2. Espresso test: discard mid-session shows dialog.

*(If any micro-task feels larger than about 100 LOC or 20 minutes, split again.)*

---

## Right-Sized Code-Generation LLM Prompt Series

Each block can be pasted directly into a code-generation LLM to create code and tests.  
Prompts assume the repo already contains the output of previous prompts.

> Usage hint: run prompts sequentially; after each code-gen, commit, run tests, then continue.

```text
# Prompt 1 – Init Project & Tooling
You are ChatGPT-DevOps.  
Create an Android project named "ClayScoreTracker" (minimum SDK = 24, compile 34) using Kotlin & Jetpack Compose.  
Add Gradle Kotlin DSL files that include:
- Compose BOM
- Room (runtime, compiler)
- Hilt (android, compiler)
- JUnit 5, Turbine, Espresso
Add GitHub Actions workflow `.github/workflows/android.yml` running unit & lint on pull-request.  
Produce only the Gradle files and workflow (nothing else). Include shell commands to init Git repo.
Return runnable code.
```

```text
# Prompt 2 – Domain Entities
Add to `:app` module:
1. `enum class ShootingPattern { ALL_SINGLES, MIXED }`
2. `data class ShootingSession` annotated with `@Entity`:
   - `id` `Long` primaryKey autoGenerate
   - `date` `Long`
   - `pattern` `ShootingPattern`
   - `stationScores` `List<Int>` (use `TypeConverter` with JSON)
   - `totalScore` `Int`
   - `note` `String?`
Provide accompanying `SessionTypeConverters`.
Add in-memory Room DB unit test ensuring an inserted session is returned unchanged.
```

```text
# Prompt 3 – DAO & Repository
1. Create `SessionDao` with:
   - `@Insert` suspend `insert(session: ShootingSession): Long`
   - `@Query("SELECT * FROM ShootingSession ORDER BY date DESC")`
     `fun getAll(): Flow<List<ShootingSession>>`
2. Write `ClayScoreDatabase` abstract class with singleton builder.
3. Implement `SessionRepository` exposing:
   - `suspend fun add(session: ShootingSession)`
   - `fun latest(): Flow<ShootingSession?>`
4. Write JUnit test using in-memory DB to verify `latest()` emits newest by `date`.
Only output Kotlin code and tests.
```

```text
# Prompt 4 – Use-Cases
Add domain layer:
- `CreateSessionUseCase(repository)`  
  validates: exactly 4 station scores; computes total; saves; returns ID.
- `GetLatestSessionUseCase(repository)`  
Provide unit tests:
- Invalid score list (≠4) throws `IllegalArgumentException`.
- Saving valid session emits via `latest` flow.
```

```text
# Prompt 5 – Home & SessionEntry ViewModels
Create `HomeViewModel`:
- Depends on `GetLatestSessionUseCase`.
- Exposes `StateFlow<SessionUiState>` (Loading / Empty / Success).

Create `SessionEntryViewModel`:
- Holds pattern, scores: `MutableList<String>`, note.
- `fun setPattern(p)`, `fun updateScore(index, value)`.
- `val canSave` true when 4 scores provided.
- `fun save()` uses `CreateSessionUseCase` and emits `UiEvent.Saved`.

Write Turbine tests covering:
- `canSave` logic.
- `save` emits event when valid.
```

```text
# Prompt 6 – HomeScreen Composable
Using Compose Material 3:
- `HomeScreen(state: SessionUiState, onStartSession: () -> Unit)`
  *If Loading*: CircularProgress.
  *If Empty*: Text “No sessions yet”.
  *If Success*: Card with date, per-station scores, total, note.
- FloatingActionButton “START”.

Include `@Preview` for empty and success states (fake data).  
No navigation yet. Provide only UI code and previews.
```

```text
# Prompt 7 – Session Flow Composables
Create three screens:

1. PatternPickerScreen  
   - Two `FilterChip`s for `ALL_SINGLES` / `MIXED`.

2. ScoreEntryScreen  
   - Four `OutlinedTextField`s (indices 0-3).  
   - “Next” enabled when all non-blank.

3. NoteScreen  
   - `OutlinedTextField` multiline.  
   - “Save” button.

All screens accept ViewModel callbacks; no navigation yet.  
Write Compose test: entering scores enables Next.
```

```text
# Prompt 8 – Navigation & Dialogs
Add `NavHost` with routes:
- home, patternPicker, scoreEntry, noteEntry, history  
Start destination = home.  
Wire FAB → patternPicker.  
Add `BackHandler` in score-entry and note screens that shows dialog:
“Exit session? All entered scores will be lost.”

Dialog calls `navController.popBackStack(home, inclusive = true)` on confirm.

Provide `NavGraph.kt` and update `MainActivity` to host it.  
Add unit test with `TestNavHostController` verifying backstack clears.
```

```text
# Prompt 9 – HistoryScreen
Create `HistoryViewModel` → repository `getAll()` as StateFlow.  
`HistoryScreen` uses `LazyColumn`; each item shows date, 4 scores, total, note.  
No click action.  
Compose test verifies newest-first ordering.
```

```text
# Prompt 10 – Instrumentation Tests
Using Espresso & Barista:
1. Complete full session (pattern → scores 10,9,8,7 → note “Test”)  
   Verify Home shows total 34.
2. Start session, press back, dialog appears, confirm → Home.
3. Rotate device; ensure Home UI state persists.

Provide test code and update CI workflow (`connectedCheck`).
```

```text
# Prompt 11 – Polish & Release
1. Force portrait in Manifest.
2. Catch `Exception` in repository and emit error UiState; show Snackbar “Something went wrong”.
3. Remove all TODOs, add KDoc.  
4. Produce README with build, run, and test instructions.

Generate only changed files plus README.
```

---

### Review Checklist (run after each prompt)

- Unit and UI tests pass: `./gradlew test connectedCheck`
- No TODO or unused code
- CI stays green
- No feature added without UI and test coverage
- Source files stay under roughly 120 lines; split otherwise
