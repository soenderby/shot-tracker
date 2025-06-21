# Clay Pigeon Shooting Score Tracker — Developer Specification

## Overview

This is a minimalist, single-user React Native application using Expo designed to track scores for clay pigeon shooting. The app operates fully offline, storing all data locally on the device. It supports two shooting patterns and records session-based scores for personal use.

## Functional Requirements

### Home Screen
- Displays the most recent session result:
  - Date of session
  - Scores per station
  - Total score
  - Any optional note
- "Start New Session" button
- Link to view a list of all previous sessions

### New Session Flow
1. Select Shooting Pattern
   - All Singles
   - Mixed Pattern: Single, Single, Double, Single, Single, Double, Single, Single
   - Pattern choice resets for every new session (no persistence)

2. Score Entry
   - For each of 4 stations:
     - A number input (0–10, but no range enforcement)
   - All stations use the same selected pattern
   - Users can navigate back and forth between station inputs before confirmation

3. Add Optional Note
   - Free-text input field

4. Confirm & Save
   - On confirmation:
     - Save session with timestamp, pattern, station scores, total, and note
     - Show confirmation message ("Session saved successfully")
     - Return to home screen
   - If user attempts to exit mid-session, show confirmation dialog:  
     "Exit session? All entered scores will be lost."

### Session History
- List ordered by most recent first
- Each item displays:
  - Date
  - Station scores (4 values)
  - Total score
  - Optional note
- Sessions are read-only:
  - No editing or deletion
  - Tapping does not open additional details

## Data Storage

### Storage Type
- Local device storage only  
- Use Room database (preferred) or SharedPreferences + JSON (if extremely minimal)

### Session Data Model
```kotlin
data class ShootingSession(
    val id: Long, // Auto-increment primary key
    val date: Long, // Epoch millis
    val pattern: ShootingPattern, // Enum
    val stationScores: List<Int>, // Size = 4
    val totalScore: Int, // Computed sum
    val note: String? // Optional
)
```

```kotlin
enum class ShootingPattern {
    ALL_SINGLES,
    MIXED
}
```

## UI & UX Design

- Portrait-only orientation
- Minimal look and feel using default Material components
- No animations, vibration, or other feedback
- No settings screen
- No user onboarding

## Architecture Choices

- Language: JavaScript or TypeScript
- Architecture: Component-based + Hooks
- UI Framework: React Native components (via Expo)
- Data Storage: Expo SQLite or MMKV (for structured local storage)
- State Management: React Context or Zustand (optional but recommended)

## Error Handling

- Score Input Validation: No enforcement; allow any number
- Unsaved Session Exit: Prompt with confirmation dialog
- Database Failures: Log errors and show a generic message ("Something went wrong. Please try again.")
- App Crashes: Prevent crashes by using try-catch and default fallbacks where applicable

## Testing Plan

### Unit Tests
- ViewModel logic for session creation
- Validation that 4 station scores are entered before allowing save
- Computation of total score
- Mapping of UI input to domain model

### Instrumentation Tests (UI)
- Complete session creation flow
- Confirm session appears in home screen and session list
- Back navigation and session discard confirmation
- Rotation lock behavior (should stay in portrait)

### Manual QA Scenarios
- Start and discard a session
- Enter session with and without note
- Try invalid inputs (e.g., empty score field, non-numeric input if not handled)
- Fill partial session and exit
- View session history and check correct order

## Additional Notes

- All data is private to the app (no external file access or sharing)
- No need for dark mode or accessibility enhancements
- App state should be preserved during screen off/backgrounding if possible (optional)

## Summary of Key Non-Functional Requirements

| Feature               | Decision     |
|-----------------------|--------------|
| Multi-user support    | No           |
| Internet access       | None         |
| Authentication        | None         |
| Cloud sync / export   | None         |
| Edits or deletes      | Not allowed  |
| Portrait-only         | Yes          |
| Offline support       | Full         |
| Local storage         | Room (preferred) |
| Minimal design        | Yes          |
| No onboarding         | Yes          |
