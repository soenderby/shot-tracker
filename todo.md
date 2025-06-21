# Clay Pigeon Shooting Score Tracker — TODO Checklist

Use this file to track progress. Mark each box with **x** when the task or sub‑task is complete.

---

## Phase 0  Scaffolding & Tooling
- [ ] Initialise Git repository
- [ ] Add `.gitignore` (Android / IntelliJ / macOS / Windows)
- [ ] Create Android Studio project (Empty Activity)
  - [ ] Minimum SDK 24
  - [ ] Compile / Target SDK 34
  - [ ] Language: Kotlin
  - [ ] App name: **ClayScoreTracker**
- [ ] Convert Gradle scripts to Kotlin DSL
  - [ ] Enable Jetpack Compose BOM
  - [ ] Add Room (runtime, compiler kapt/ksp)
  - [ ] Add Hilt (android, compiler)
  - [ ] Add JUnit 5, Turbine, Espresso, Barista
- [ ] Set up GitHub Actions workflow `.github/workflows/android.yml`
  - [ ] Cache Gradle deps
  - [ ] Job: Unit tests (`./gradlew testDebugUnitTest`)
  - [ ] Job: Lint (`./gradlew lint`)
  - [ ] Job: Build (`./gradlew assembleDebug`)
- [ ] Verify local build succeeds
- [ ] Push initial commit / open PR

---

## Phase 1  Domain & Storage (layer 1)
### Iteration 1  Domain Model
- [ ] Add `enum class ShootingPattern { ALL_SINGLES, MIXED }`
- [ ] Add `data class ShootingSession` with Room `@Entity`
  - [ ] `id: Long` – primary key, autoGenerate
  - [ ] `date: Long` – epoch millis
  - [ ] `pattern: ShootingPattern`
  - [ ] `stationScores: List<Int>`
  - [ ] `totalScore: Int`
  - [ ] `note: String?`
- [ ] Implement `TypeConverter` for `List<Int>` (JSON) and `ShootingPattern`
- [ ] Write JUnit test: in‑memory Room DB insert & read round‑trip

---

## Phase 2  DAO & Repository (layer 2)
### Iteration 2
- [ ] Create `SessionDao`
  - [ ] `@Insert suspend fun insert(session)`
  - [ ] `@Query("SELECT * FROM ShootingSession ORDER BY date DESC") fun getAll(): Flow<List<ShootingSession>>`
- [ ] Build `ClayScoreDatabase` (singleton)
- [ ] Implement `SessionRepository`
  - [ ] `suspend fun add(session)`
  - [ ] `fun latest(): Flow<ShootingSession?>`
- [ ] Unit tests
  - [ ] DAO ordering by date
  - [ ] Repository emits latest

---

## Phase 3  Use‑Cases (layer 3)
### Iteration 3
- [ ] Add `CreateSessionUseCase`
  - [ ] Validate list size == 4
  - [ ] Compute `totalScore`
  - [ ] Persist via repository
- [ ] Add `GetLatestSessionUseCase`
- [ ] Unit tests
  - [ ] Invalid size throws `IllegalArgumentException`
  - [ ] Saving valid session propagates via `latest` flow

---

## Phase 4  ViewModels (layer 4)
### Iteration 4
- [ ] `HomeViewModel`
  - [ ] Inject `GetLatestSessionUseCase`
  - [ ] `StateFlow<SessionUiState>` (Loading / Empty / Success)
- [ ] `SessionEntryViewModel`
  - [ ] Holds `pattern`, `scores: MutableList<String>`, `note`
  - [ ] `canSave` derived state
  - [ ] `save()` triggers use‑case and emits `UiEvent.Saved`
- [ ] ViewModel tests with Turbine
  - [ ] `canSave` toggles appropriately
  - [ ] Save event emitted

---

## Phase 5  UI – Home Screen slice
### Iteration 5
- [ ] Compose `HomeScreen`
  - [ ] Loading → CircularProgress
  - [ ] Empty → “No sessions yet”
  - [ ] Success → Card with date, 4 scores, total, note
  - [ ] FloatingActionButton “START”
- [ ] Previews (empty + success)
- [ ] Screenshot test placeholder text

---

## Phase 6  UI – Session Flow slices
### Iteration 6
- [ ] `PatternPickerScreen` – two `FilterChip`s
- [ ] `ScoreEntryScreen`
  - [ ] Four `OutlinedTextField`s
  - [ ] Header shows current pattern
  - [ ] “Next” enabled when 4 scores entered
- [ ] `NoteScreen`
  - [ ] Multiline text field
  - [ ] “Save” button
- [ ] Compose tests
  - [ ] Entering scores enables Next

---

## Phase 7  Navigation & Dialogs
### Iteration 7
- [ ] Add `NavHost` in `MainActivity`
  - [ ] Destinations: home, patternPicker, scoreEntry, noteEntry, history
  - [ ] Start destination: home
- [ ] Wire FAB → patternPicker
- [ ] Add discard confirmation dialog on back press within session flow
- [ ] Unit test with `TestNavHostController`

---

## Phase 8  History Screen
### Iteration 8
- [ ] `HistoryViewModel` collects all sessions
- [ ] `HistoryScreen` with `LazyColumn`
  - [ ] Row shows date, 4 scores, total, note
- [ ] Compose test: newest first order

---

## Phase 9  Instrumentation Tests
### Iteration 9
- [ ] Espresso / Barista test: complete session, verify Home total
- [ ] Test: discard mid‑session confirmation
- [ ] Test: rotation preserves state
- [ ] Add `connectedCheck` to CI

---

## Phase 10  Polish & Release
- [ ] Lock orientation to portrait in Manifest
- [ ] Repository catches exceptions, emits error UiState
- [ ] Show Snackbar on generic error
- [ ] Remove all TODOs and dead code
- [ ] Generate Javadoc/KDoc
- [ ] Draft README
  - [ ] Build instructions
  - [ ] Running tests
  - [ ] CI status badge
- [ ] Version code & name for release
- [ ] Generate signed APK/AAB (optional)

---

## Continuous Review Checklist  (verify for every pull request)
- [ ] All unit tests pass (`./gradlew testDebugUnitTest`)
- [ ] All instrumentation tests pass (`./gradlew connectedCheck`)
- [ ] Lint passes (`./gradlew lint`)
- [ ] PR linked to GitHub issue / ticket
- [ ] No unchecked TODO/FIXME markers
- [ ] Source files ≤ 120 lines (guideline)
- [ ] Public classes / functions documented
- [ ] Code reviewed by at least one teammate
