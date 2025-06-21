# Clay Pigeon Shooting Score Tracker — Full Developer Prompts
*Date: 21 June 2025*

---

## LLM Prompt Set (38 tasks)

Follow each prompt in order. Each one should result in a commit that passes all existing checks. Prompts are provided in `text` code blocks so they can be copied verbatim into a code-generation agent.

### Prompt 1 – init‑expo‑project
```text
You are working in an empty Git repository.

**Goal**  
Scaffold a new Expo React Native project for a single‑user offline score tracker.

**Requirements**
1. Run `npx create-expo-app@latest clay-score-tracker`.
2. Select the **TypeScript** template.
3. Add an MIT `LICENSE` file and a concise `README.md` with install/run instructions.

**Done When**
- `npm start` opens the Expo dev menu with no errors.
- `git status` is clean and the initial commit message is “chore: bootstrap Expo project”.
```

### Prompt 2 – add‑typescript‑infra
```text
The repo already contains an Expo TypeScript project.

**Goal**  
Enable strict TypeScript settings and ensure zero type errors.

**Steps**
1. Update `tsconfig.json`: set `"strict": true` and enable `noUncheckedIndexedAccess`.
2. Replace any Expo default `App.js` references with `App.tsx`.
3. Add `npm run type-check` that runs `tsc --noEmit` in CI.

**Done When**
- `npm run type-check` exits successfully.
- CI badge shows passing for the **type‑check** job.
```

### Prompt 3 – linter‑prettier‑setup
```text
**Goal**  
Add opinionated linting and code formatting to enforce consistent style.

**Requirements**
1. Install and configure:
   - `eslint`, `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser`
   - `eslint-config-airbnb-typescript`, `eslint-plugin-react-hooks`
   - `prettier`, `eslint-config-prettier`, `eslint-plugin-prettier`
2. Create `.eslintrc.js` extending **Airbnb TypeScript** rules and enabling Prettier.
3. Create `.prettierrc` (use 2‑space indent, single quotes, semicolons true).
4. Add npm scripts:  
   - `lint`: `eslint --ext .ts,.tsx src`  
   - `format`: `prettier --write "**/*.{ts,tsx,md, json}"`
5. Set up Husky + lint‑staged so `npm run lint` & `prettier --check` run on pre‑commit.
6. Fix all existing lint and prettier issues.

**Done When**
- `npm run lint` exits 0 with no warnings.
- Committing triggers Husky; the commit is blocked if either lint or prettier fails.
```

### Prompt 4 – jest‑setup
```text
**Goal**  
Add Jest with React Native Testing Library for unit testing.

**Tasks**
1. Install dev dependencies: `jest`, `@testing-library/react-native`, `@testing-library/jest-native`, `babel-jest`, `jest-expo`.
2. Create `jest.config.js` that extends `jest-expo` preset, sets up `@testing-library/jest-native/extend-expect`.
3. Add `setupTests.ts` importing the extend‐expect module.
4. Write a placeholder test `App.test.tsx` that renders `<App />` and asserts `'Learn more'` text appears (Expo default).
5. Add script `test` → `jest --watchAll`.

**Done When**
- `npm test` passes locally.
- CI workflow runs tests and reports success.
```

### Prompt 5 – github‑actions‑ci
```text
**Goal**  
Automate lint, type‑check, and unit tests on every push / PR.

**Steps**
1. Create `.github/workflows/ci.yml`.
2. Use Ubuntu latest, Node 18.  
3. Cache npm packages via `actions/setup-node`.
4. Jobs:
   - `install`: `npm ci`
   - `lint`: `npm run lint`
   - `type-check`: `npm run type-check`
   - `test`: `npm test -- --ci`
5. Add status badge in `README.md`.

**Done When**
- PRs show all checks green.
- Badge renders in README and updates.
```

### Prompt 6 – define‑domain‑models
```text
**Goal**  
Create the TypeScript domain entities.

**Requirements**
1. Add `src/domain/ShootingPattern.ts` exporting enum:
   ```ts
   export enum ShootingPattern {
     ALL_SINGLES = 'ALL_SINGLES',
     MIXED = 'MIXED',
   }
   ```
2. Add `src/domain/ShootingSession.ts` exporting interface:
   ```ts
   export interface ShootingSession {
     id?: number;            // optional before persistence
     date: number;           // epoch millis
     pattern: ShootingPattern;
     stationScores: number[]; // length 4
     totalScore: number;     // pre‑computed sum
     note?: string | null;
   }
   ```
3. Write a helper `computeTotalScore(stationScores: number[]): number`.

**Done When**
- Unit tests verify `computeTotalScore` sums correctly & rejects wrong length input.
```

### Prompt 7 – create‑sqlite‑schema
```text
**Goal**  
Persist sessions locally through SQLite migrations.

**Tasks**
1. Install `expo-sqlite`.
2. Add directory `src/data/migrations`.
3. Add script `001_create_sessions.sql`:
   ```sql
   CREATE TABLE IF NOT EXISTS sessions (
     id INTEGER PRIMARY KEY AUTOINCREMENT,
     date INTEGER NOT NULL,
     pattern TEXT NOT NULL,
     station_scores TEXT NOT NULL, -- JSON string
     total_score INTEGER NOT NULL,
     note TEXT
   );
   ```
4. Create `Database.ts` wrapper that:
   - Opens DB on first import.
   - Runs pending migrations in numeric order.
5. Add migration unit test that opens a fresh in‑memory DB and asserts the table exists.

**Done When**
- Migration test passes.
- App boots without DB errors.
```

### Prompt 8 – repository‑implement‑save‑fetch
```text
**Goal**  
Provide a thin repository for CRUD (read‑only + create).

**Implementation**
1. Create `SessionRepository.ts`:
   - `save(session: Omit<ShootingSession, 'id'>): Promise<void>`
     * Serialises `stationScores` as JSON.
   - `getLast(): Promise<ShootingSession | null>`
   - `getAll(): Promise<ShootingSession[]>`
2. Use parameterised queries to prevent SQL injection.
3. Derive `totalScore` at call‑site; ensure DB value consistent.

**Done When**
- Manual script can save two sessions and fetch last.
- No unhandled promise rejections.
```

### Prompt 9 – repository‑unit‑tests
```text
**Goal**  
Lock behaviour of `SessionRepository`.

**Tests**
1. Saving a session returns fulfilled promise.
2. Fetching last returns the most recent by `date`.
3. Saving malformed (`stationScores.length !== 4`) rejects with error.

Use Jest with a temporary in‑memory DB (override `expo-sqlite` with mock).

**Done When**
- All repository tests green.
```

### Prompt 10 – create‑state‑store
```text
**Goal**  
Expose global state with minimal boilerplate.

**Choice**  
Use **Zustand** for simplicity.

**Tasks**
1. Install `zustand` and `zustand/middleware`.
2. Create `src/store/useSessions.ts` exporting:
   ```ts
   interface SessionsState {
     sessions: ShootingSession[];
     addSession: (s: ShootingSession) => void;
     hydrate: () => Promise<void>;
   }
   ```
3. On `hydrate()`, load all sessions from repository.
4. `addSession` should push into state array **and** persist via repository.

**Done When**
- Integration test hydrates store and adds a session; state length increments by one.
```

### Prompt 11 – state‑store‑tests
```text
**Goal**  
Guarantee reducer / store consistency.

**Tests**
1. After `hydrate`, state contains DB rows (mock with 2 entries).
2. Calling `addSession` with valid object writes to repository and updates state.
3. Invalid session throws error and does **not** mutate state.

Use Jest mocks for repository.

**Done When**
- All store tests green.
```

### Prompt 12 – nav‑shell‑setup
```text
**Goal**  
Provide screen routing infrastructure.

**Steps**
1. Install `@react-navigation/native` + required deps.
2. Set up `NavigationContainer` in `App.tsx`.
3. Create stack navigator with screens:
   - `HomeScreen`
   - `NewSessionNavigator` (nested stack for multi‑step flow)
   - `HistoryScreen`
4. Add placeholder components for each.

**Done When**
- `npm start` launches app, navigation works (can tap through placeholders).
```

### Prompt 13 – home‑screen‑static
```text
**Goal**  
Render Home screen with basic UI.

**UI**
1. Title: “Clay Score Tracker”.
2. If `sessions.length === 0` show “No sessions yet”.
3. Otherwise show card with:
   - Date (formatted)
   - Station Scores e.g. “8 / 7 / 9 / 10”
   - Total Score
4. Button “Start New Session” navigates to NewSession flow.
5. Text link “View All Sessions” navigates to History.

Use React Native Paper or plain RN components.

**Done When**
- Snapshot test passes for empty and non‑empty states.
```

### Prompt 14 – home‑screen‑tests
```text
**Goal**  
Lock Home screen rendering.

**Tests**
1. With empty store, render shows “No sessions yet”.
2. With mock session, render shows total score.
3. Pressing “Start New Session” triggers `navigate('PatternSelector')`.

Use React‑Native‑Testing‑Library.

**Done When**
- Tests green in CI.
```

### Prompt 15 – pattern‑selector‑view
```text
**Goal**  
First step in the New Session flow: choose shooting pattern.

**UI**
1. Radio buttons: “All Singles” (default) and “Mixed”.
2. “Next” button enabled once a selection exists.
3. Store choice in component state, pass via param to next screen.

**Done When**
- Unit test renders radio buttons and `onPress` updates selection.
```

### Prompt 16 – score‑entry‑view
```text
**Goal**  
Collect 4 station scores.

**UI**
1. Four numeric TextInput fields labelled Station 1‑4.
2. Use controlled inputs; allow blank initially.
3. “Next” button disabled until all 4 fields contain a number.
4. Validate value is integer 0‑10; if outside, show red helper text but still allow.

**Done When**
- Unit test: entering 4 numbers enables button.
- Values passed forward correctly.
```

### Prompt 17 – confirm‑and‑save‑view
```text
**Goal**  
Show summary and collect optional note.

**UI**
1. Summary card:
   - Pattern name
   - Station scores list
   - Total computed score
2. Multiline TextInput placeholder “Optional note…”.
3. “Save Session” button commits:
   - Build `ShootingSession` object.
   - Calls `addSession`.
   - Navigates back to Home.

**Done When**
- Integration test confirms new session appears on Home.
```

### Prompt 18 – new‑session‑unit‑tests
```text
**Goal**  
Ensure New Session flow rules.

**Tests**
1. Cannot go to Score Entry without picking pattern.
2. Cannot save until all 4 station scores present.
3. Saving triggers `addSession` exactly once.

Mock navigation and store.

**Done When**
- Tests green.
```

### Prompt 19 – connect‑repo‑to‑ui
```text
**Goal**  
Wire repository/store into screens.

**Tasks**
1. Call `hydrate()` on store inside `App.tsx` (use `useEffect`).
2. Home subscribes to store via hook.
3. New Session screens use store actions.

**Done When**
- Manual test: add session, home card updates without reload.
```

### Prompt 20 – integration‑tests‑save‑flow
```text
**Goal**  
Simulate complete session save in tests.

**Steps**
1. Render app with mocked navigation container.
2. Tap through pattern -> scores -> save.
3. Assert card on Home shows added session.

**Done When**
- Test passes in CI.
```

### Prompt 21 – history‑screen‑view
```text
**Goal**  
List all past sessions.

**UI**
1. FlatList sorted `date` desc.
2. Each item: Date, scores, total, note (ellipsis after 40 chars).
3. Scrollable; divider between items.

**Done When**
- Manual run shows list; empty state text when none.
```

### Prompt 22 – history‑screen‑tests
```text
**Goal**  
Ensure ordering and rendering.

**Tests**
1. Provide store with 3 sessions dated **today‑2**, **today‑1**, **today**.  
   Assert FlatList order is descending (today first).
2. Empty store shows empty message.

**Done When**
- Tests pass.
```

### Prompt 23 – unsaved‑exit‑guard
```text
**Goal**  
Prevent accidental loss of in‑progress session.

**Implementation**
1. In Score Entry & Confirm screens, track `isDirty`.
2. Use `useFocusEffect` and `navigation.addListener('beforeRemove', …)` to intercept back gestures.
3. Show `Alert.alert` dialog: “Exit session? All entered scores will be lost.”.

**Done When**
- Manual: back button shows prompt; choosing “Cancel” stays on screen.
```

### Prompt 24 – guard‑unit‑tests
```text
**Goal**  
Test back‑guard behaviour.

**Tests**
1. Render Score Entry, fill one score → back press triggers Alert.
2. Accepting “Exit” calls `navigation.dispatch`.

Mock `Alert.alert`.

**Done When**
- Tests green.
```

### Prompt 25 – orientation‑lock
```text
**Goal**  
Lock portrait orientation.

**Steps**
1. Install `expo-screen-orientation`.
2. In `App.tsx`, call `ScreenOrientation.lockAsync(ScreenOrientation.OrientationLock.PORTRAIT_UP)` on mount.

**Done When**
- Running on device/emulator: rotating does not change layout.
```

### Prompt 26 – error‑handling‑wrapper
```text
**Goal**  
Gracefully handle uncaught UI errors.

**Implementation**
1. Create `components/ErrorBoundary.tsx` using React `componentDidCatch`.
2. Render fallback UI: “Something went wrong. Please restart the app.”
3. Wrap `NavigationContainer` inside `ErrorBoundary`.

**Done When**
- Throwing error in child shows fallback screen instead of crash.
```

### Prompt 27 – presentation‑polish
```text
**Goal**  
Finalize branding & visuals.

**Checklist**
1. Create simple icon (orange clay target) via Figma, export `icon.png`.
2. Add `splash.png` minimalist logo.
3. Update `app.json` with name, icon, splash background colour.
4. Replace placeholder text (“Open up App.js”) etc.

**Done When**
- Expo build shows new icon & splash.
```

### Prompt 28 – detox‑init
```text
**Goal**  
Add end‑to‑end testing with Detox.

**Steps**
1. Install `detox`, `detox-expo-helpers`, `@testing-library/react-native`.
2. Configure `detox.config.js` for Android emulator (API 33).
3. Add npm script `e2e:build` & `e2e:test`.
4. Write first test that opens app and expects Home screen header.

**Done When**
- `npm run e2e:test` builds APK and passes test locally.
```

### Prompt 29 – detox‑test‑save‑flow
```text
**Goal**  
E2E happy‑path session creation.

**Test Steps**
1. Launch app.
2. Tap “Start New Session”.
3. Select pattern “All Singles”.
4. Enter scores 8,9,7,10.
5. Leave note “Friday practice”.
6. Save session.
7. Expect Home shows total 34.

**Done When**
- Detox test passes.
```

### Prompt 30 – detox‑test‑discard‑flow
```text
**Goal**  
E2E exit guard path.

**Scenario**
1. Start new session.
2. Enter one score.
3. Press back → confirm “Exit”.
4. Return Home.
5. Assert “No sessions yet” still shown.

**Done When**
- Test passes.
```

### Prompt 31 – detox‑test‑history‑order
```text
**Goal**  
Verify chronological order.

**Scenario**
1. Programmatically insert 3 sessions via repository mock (use Detox device.setURLBlacklist).
2. Open History screen.
3. Expect list order newest‑first (check first item date equals today).

**Done When**
- Test passes.
```

### Prompt 32 – ci‑detox‑pipeline
```text
**Goal**  
Run Detox tests in GitHub Actions.

**Steps**
1. Extend `ci.yml` with job `e2e`.
2. Use `react-native-community/cli` action to start Android AVD.
3. Cache build output.
4. Run `npm run e2e:test -- --headless`.

**Done When**
- PRs include E2E status check.
```

### Prompt 33 – release‑config
```text
**Goal**  
Prepare for production build.

**Tasks**
1. Install Expo Application Services CLI (`eas-cli`) globally.
2. Create `eas.json` with `preview` and `production` profiles.
3. Configure Android package name `com.example.clayscore`.
4. Add build badge to README.

**Done When**
- Running `eas build --profile preview --platform android` succeeds locally.
```

### Prompt 34 – bump‑version‑script
```text
**Goal**  
Automate version bump & changelog.

**Steps**
1. Install `standard-version`.
2. Add npm script `release`: `standard-version`.
3. Configure commit message convention (`feat`, `fix`, etc.).
4. Add Husky hook to run tests before release.

**Done When**
- Running `npm run release` bumps version, updates `CHANGELOG.md`, creates git tag.
```

### Prompt 35 – readme‑badges‑qa
```text
**Goal**  
Make README production‑ready.

**Add**
1. CI status badge.
2. Coverage badge (use `jest --coverage` + Coveralls).
3. Expo Go QR preview badge (`expo.dev/@<username>/clay-score-tracker`).
4. Manual QA checklist table.

**Done When**
- README renders correctly.
```

### Prompt 36 – cleanup‑unused‑deps
```text
**Goal**  
Reduce bundle size.

**Actions**
1. Run `npm prune --production` and review `package.json`.
2. Remove unused packages (e.g., moment if not used).
3. Add ESLint rule `no-extraneous-dependencies` to catch future bloat.

**Done When**
- Build size reduced; `npm prune` lists 0 removed.
```

### Prompt 37 – tag‑v1‑alpha
```text
**Goal**  
Mark first public alpha.

**Steps**
1. Ensure branch is `main` and CI green.
2. Run `npm run release -- --release-as 1.0.0-alpha`.
3. Push tag `v1.0.0-alpha` to origin.

**Done When**
- GitHub shows release tag and generated changelog section.
```

### Prompt 38 – post‑mortem‑doc
```text
**Goal**  
Capture project retrospective.

**Create** `docs/post-mortem.md` answering:
- **What went well** (tooling, CI, TDD)
- **What went poorly** (time sinks)
- **Action items** (future refactors, features)

**Done When**
- Document committed and referenced in README “Further Reading”.
```

