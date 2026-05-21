# Parent App Screens


## Part 1: Standardized Screen Directory (10 Screens)

To ensure consistency, the application uses a local-first, always-ready design system. Parents do not have time to re-authenticate during crisis moments, so the app remains permanently active on device unless manually locked.

### Screen 1: Permanent-Ready Lock Screen (No-Auth Quick Entry)

* **Purpose:** Allows instant logging from a locked state without security bypass friction.
* **Key UI Elements:**
  * A black status bar showing "Emergency Entry Active"
  * A single giant microphone button occupying the center of the screen
  * Three large quick-action buttons below the mic: Meltdown, Meds Taken, Food Refused
  * A secure "Unlock Full App" button at the very bottom
* **Behavior:** Tapping any quick action logs the event immediately with a local timestamp. Entering data does not reveal historical logs until the parent authenticates.

### Screen 2: Quick Capture Home (The Command Center)

* **Purpose:** The primary starting point of the unlocked app, built for speedy logging.
* **Key UI Elements:**
  * Top child profile selector with active child avatar
  * Giant "Hold to Speak" button with voice assistant prompts
  * Categorized grid of 1-touch hot buttons (Behavior, Diet, Medication, Emotion)
  * Persistent global floating action button (bright red) for the Stress Screen
  * Bottom navigation bar (Home, Timeline, Insights, Settings)

### Screen 3: Stress Screen (Deep Incident Log)

* **Purpose:** Deep, structured behavioral entry optimized for one-handed thumb navigation during crisis times.
* **Key UI Elements:**
  * Top status bar labeled "Stress Logging Active"
  * Large, chunky selection pills for behaviors (Meltdown, Self-Harm, Eloping)
  * Horizontal severity slider (Mild, Moderate, Severe) with clear color boundaries
  * Large trigger selection pills (Noise, Change in Routine, Sensory Overload)
  * Quick voice memo note recorder
  * Two dominant bottom action buttons: "Save" and "Save and Remind"

### Screen 4: Voice Transcriber and Verification Screen

* **Purpose:** Shows real-time audio transcription and category mapping.
* **Key UI Elements:**
  * Oscillating wave graphic representing live voice input
  * Live-updating text container showing raw transcription
  * Automated categorization feedback cards showing how the system parsed the words (like "Identified: Medication Taken")
  * Manual edit text box
  * "Confirm Log" and "Discard" bottom buttons

### Screen 5: Daily Timeline and Log Feed

* **Purpose:** Chronological record of the day's events, providing parents with validation.
* **Key UI Elements:**
  * Header displaying current date and child summary
  * Stack of timeline cards, color-coded by category (red for stress, green for success, blue for clinical data)
  * Swipe-to-delete and swipe-to-edit gestures on each log card
  * "Undo" button overlay for accidental deletions
  * Manual search and filter bar at the top

### Screen 6: Pediatrician Visit Planner

* **Purpose:** Prepares parents for medical appointments by turning raw logs into clinical evidence.
* **Key UI Elements:**
  * Date range selector (Last 7 Days, Last 30 Days, Custom)
  * Checkbox list of metrics to include (Meltdowns, Medication adherence, Sleep, Appetite changes)
  * Direct preview window showing a structured PDF layout
  * "Generate Shareable Link" and "Export PDF" primary actions

### Screen 7: Habit Trends and Insights Dashboard

* **Purpose:** Demonstrates correlation value back to the parent.
* **Key UI Elements:**
  * Visual correlation cards (such as "Aarav had 3 meltdowns on days with under 6 hours of sleep")
  * Simple bar charts showing behavior frequency by hour of the day
  * Trigger frequency distribution pie charts
  * Positive reinforcement highlights (such as "3 days of calm recorded after early med times")

### Screen 8: Child Profile and Settings Manager

* **Purpose:** Configures custom details unique to each child's neurodivergence.
* **Key UI Elements:**
  * Child avatar and basic bio fields
  * Custom triggers checklist (enables or disables buttons on the Quick Capture screen)
  * Custom behavioral label creator (such as adding "Hand-flapping" as a standard stim tag)
  * Safe medical dosage recorder (for medication reminders)

### Screen 9: Notification and Reminder Setup

* **Purpose:** Configures the proactive check-in schedule to keep logs consistent.
* **Key UI Elements:**
  * Proactive check-in scheduler (such as "Ask me how transition from school went at 3:30 PM")
  * Quiet hours toggle (to silence notifications during expected sleep times)
  * Notification style configuration (silent flash versus critical audio alert)

### Screen 10: Local-First and Sync Settings

* **Purpose:** Manages offline database capabilities and optional cloud sharing.
* **Key UI Elements:**
  * Database size indicator
  * "Export raw CSV database" button (for complete data ownership)
  * Cloud sync toggle (optional connection to secure remote storage)
  * Co-parent invite system (generates secure join tokens for partners)



### Epic 1: Low-Friction Entry Mechanisms

#### Story 1.1: Always-Ready Locking Bypass

* **As a** busy, stressed parent holding a crying child
* **I want to** access the app's core logging features from my phone's lock screen without typing a PIN
* **So that** I do not lose critical timestamp accuracy during high-tension behaviors
* **Acceptance Criteria:**
  * App must expose a secure lock screen widget or quick notification card.
  * Tapping the quick access card must launch Screen 1 directly.
  * The user must be blocked from browsing historical timeline data until standard device authentication (PIN or biometrics) is completed.

#### Story 1.2: One-Touch Hot Tag Commit

* **As a** multitasking parent
* **I want to** tap a single predefined behavior card and have it logged instantly
* **So that** I do not have to fill out confirmation popups or multiple form screens
* **Acceptance Criteria:**
  * Tapping any Quick Tag on Screen 2 must save an event directly to the local database.
  * The saved event must auto-populate the correct category, name, active child, and current localized timestamp.
  * The screen must show a temporary visual confirmation toast that disappears automatically after 2 seconds.

#### Story 1.3: Natural Voice Capture Parsing

* **As a** parent who prefers dictation over typing on tiny keyboards
* **I want to** speak a natural sentence into the app and have it automatically categorized
* **So that** my logs contain descriptive context without requiring text editing
* **Acceptance Criteria:**
  * Holding the microphone button must record clear audio and output a real-time text transcription.
  * The system must run keywords through a local parser to identify target categories (for example, if a sentence contains "Ritalin", assign to Medication).
  * The final log must store both the raw transcript and the calculated category tags.

### Epic 2: Data Value and Clinic Readiness

#### Story 2.1: Local Trend Correlation Calculations

* **As a** parent seeking clarity about behavioral issues
* **I want to** see simple connections between my child's sleep, eating, and meltdowns
* **So that** I can identify patterns and make proactive daily changes
* **Acceptance Criteria:**
  * The system must calculate correlations using local data (like comparing days with meltdowns against total sleep hours recorded).
  * The insights generated must use simple, clear language devoid of clinical jargon or confusing medical terms.
  * Insights must update dynamically whenever a new behavioral incident is saved.

#### Story 2.2: Structured Clinical PDF Generation

* **As a** parent presenting tracking evidence to doctors
* **I want to** export my child's behavior and medication records into a clean clinical PDF report
* **So that** developmental pediatricians can quickly review medical history without reading raw chat logs
* **Acceptance Criteria:**
  * The user must be able to filter exported logs by date and category type.
  * The generated PDF must be formatted as a single-page tabular timeline.
  * The PDF generation code must run locally on the user's device to protect patient data privacy.

### Epic 3: Security, Safety, and Offline Resilience

#### Story 3.1: Zero-Network Offline Logging

* **As a** parent tracking data in rural school zones or low-connectivity environments
* **I want to** log incidents offline and have them save immediately
* **So that** I do not lose behavioral data due to cellular network dropouts
* **Acceptance Criteria:**
  * The application must prioritize saving data to local browser storage or local device database first.
  * All UI screens must remain fully interactive and editable without an active internet connection.
  * When a network connection is restored, offline data must gracefully sync with cloud backups without creating duplicate entries.