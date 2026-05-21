# Parent App User Stories

## Agile User Stories

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