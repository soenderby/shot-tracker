# Clay Pigeon Shooting Score Tracker — Project Checklist
*(Use the ☐/☑ boxes to track progress)*

---

## 1. Bootstrap & Tooling
- [ ] **Init Expo project** – `npx create-expo-app` with TypeScript template  
- [ ] **Add MIT License & README**
- [ ] **Enable strict TypeScript** (`strict`, `noUncheckedIndexedAccess`)
- [ ] **Set up ESLint + Prettier** (Airbnb rules, Husky pre‑commit)
- [ ] **Add Jest + RTL** with sample test & `setupTests.ts`
- [ ] **Create GitHub Actions CI** (install, lint, type‑check, unit tests)

## 2. Data Layer
- [ ] **Define domain models** (`ShootingPattern`, `ShootingSession`, helper `computeTotalScore`)
- [ ] **SQLite schema migration** (`001_create_sessions.sql`)
- [ ] **SessionRepository implementation** (`save`, `getLast`, `getAll`)
- [ ] **Repository unit tests** (happy path, malformed input)

## 3. State Management
- [ ] **Create Zustand store** (`sessions`, `addSession`, `hydrate`)
- [ ] **Store unit tests** (hydrate & addSession behaviour)

## 4. Navigation Shell
- [ ] **Install & configure React Navigation**
- [ ] **NavigationContainer** in `App.tsx`
- [ ] **Stacks & placeholders** (Home, NewSession flow, History)

## 5. Home Screen
- [ ] **Static Home UI** (title, “Start New Session”, “View All Sessions”)
- [ ] **Render latest session card**
- [ ] **Home screen tests** (empty + populated)

## 6. New Session Flow
- [ ] **Pattern Selector view** (radio buttons, Next)
- [ ] **Score Entry view** (4 inputs, validation)
- [ ] **Confirm & Save view** (summary, optional note, Save button)
- [ ] **Unsaved exit guard** (`beforeRemove` alert)
- [ ] **Flow unit tests** (selection, validation, save once)

## 7. Wiring & Integration
- [ ] **Hydrate store on app start**
- [ ] **Wire New Session save into store & DB**
- [ ] **Integration tests** (simulate full save, assert Home updates)

## 8. Session History Screen
- [ ] **FlatList of sessions** (newest first, card layout)
- [ ] **History screen tests** (order, empty state)

## 9. UX Polish & Safety
- [ ] **Orientation lock** to portrait (`expo-screen-orientation`)
- [ ] **Global ErrorBoundary**
- [ ] **Branding assets** (icon, splash, copy updates)

## 10. End‑to‑End Testing (Detox)
- [ ] **Detox init & config**
- [ ] **E2E happy‑path save flow**
- [ ] **E2E discard flow**
- [ ] **E2E history order**
- [ ] **Detox running headless in CI**

## 11. Release Preparation
- [ ] **EAS build profiles (`eas.json`)**
- [ ] **Standard‑version release script**
- [ ] **README badges & QA checklist**
- [ ] **Prune unused dependencies**
- [ ] **Tag `v1.0.0‑alpha`**

## 12. Documentation
- [ ] **Post‑mortem / retrospective** (`docs/post-mortem.md`)

---

### Tips
* Work task‑by‑task; each should end with green CI.
* Check off sub‑tasks only after writing **tests** or **manual verification**.
