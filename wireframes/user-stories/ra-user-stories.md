## Epic 1: Universal Onboarding & Dashboard (Home)

**Epic Goal:** Establish a low-cognitive-load launchpad that handles basic configuration (multiple profiles) and provides immediate, friction-free access to data entry.

### Story 1.1: Multi-Profile Child Setup

* **As a** caregiver with one or more neurodivergent children,
**I want to** create distinct, privacy-focused profiles for my children using just a display name and birth month/year,
**So that** I can track their individual journeys without exposing sensitive personal identifiers.
* **Acceptance Criteria:**
* Caregiver can add a child with fields: `display_name` and `birth_year_month` (e.g., "Max", "2018-04").
* No full legal names or exact dates of birth are required.
* Profiles can be edited or deleted later from the settings.



### Story 1.2: Dynamic Dashboard & Child Switching

* **As a** busy parent,
**I want to** view a quick activity snapshot for the selected child on the Home screen and switch between children with a single tap,
**So that** I don't accidentally log data or read patterns for the wrong child.
* **Acceptance Criteria:**
* The Home screen features a prominent, easy-to-tap child selector profile icon.
* Switching the child instantly updates the entire dashboard view (recent activity, quick shortcuts).
* The app remembers the last selected child profile on subsequent launches.



---

## Epic 2: Voice-First Journaling (The Flagship UX)

**Epic Goal:** Create an ultra-low friction, one-handed recording interface that serves as a "calm memory extension" during active or stressful moments.

### Story 2.1: Instant Global Voice Capture

* **As an** overwhelmed caregiver holding a dysregulated child,
**I want to** tap a persistent, large microphone button from the main navigation to start recording a raw verbal observation,
**So that** I can capture the moment completely hands-free or with one hand.
* **Acceptance Criteria:**
* A large floating microphone button is globally accessible on the bottom navigation layout.
* Tapping it opens an immersive audio recording state immediately (under 200ms latency).
* Provides clear visual feedback that recording is active (e.g., soundwave animation) without being visually loud or bright.
* Supports an interruption-tolerant workflow (e.g., if a phone call comes in or the screen locks, the audio clip safely auto-saves up to that point).



### Story 2.2: Live Transcript & AI Event Review

* **As a** caregiver who just finished a voice log,
**I want to** see a live text transcript alongside the AI-extracted structured events (triggers, interventions) so I can quickly edit or confirm them,
**So that** my data is structured accurately in under 45 seconds before I return to my child.
* **Acceptance Criteria:**
* Upon stopping the recording, the app displays the raw text transcript.
* The backend/local NLP engine parses the text and displays extracted entities as interactive chips (e.g., `Category: Trigger`, `Value: Loud Noise`).
* The caregiver can tap an extracted chip to change its category, edit the text, or delete it entirely if the AI misread the context.
* Saving writes to both the `NarrativeLogs` table (raw text) and `AIStructuredEvents` table.



---

## Epic 3: Quick Tap Logging

**Epic Goal:** Provide an alternative data entry method for structured, highly repetitive daily events that requires zero typing and takes less than 10 seconds.

### Story 3.1: Routine Quick-Tap Presets

* **As a** caregiver tracking baseline routines,
**I want to** tap pre-configured quick log buttons for categories like Sleep, Food, Medication, and School Transitions,
**So that** I don't have to repeatedly type or speak the exact same routine data every day.
* **Acceptance Criteria:**
* The Quick Tap Log screen provides a grid of primary categories: Sleep, Food, Medication, Sensory Load, Transitions, and Successes.
* Selecting a category opens a simple, minimalist choice modal (e.g., Medication -> [Select Supp/Med] -> [Select Dosage/Time] -> Save).
* The flow must be executable in 3 taps or fewer, requiring zero keyboard typing.



### Story 3.2: Logging "Success Moments"

* **As a** caregiver looking for balanced, emotionally safe tracking,
**I want to** quickly log a "Success Moment" or a helpful intervention,
**So that** the app tracks what brings peace to my child, rather than feeling like a rigid behavioral compliance chart.
* **Acceptance Criteria:**
* The Quick Log grid features a dedicated, positively styled button for "Success Moments" or "Helped".
* Uses empowering, non-clinical language (avoids terms like "compliant" or "good behavior").



---

## Epic 4: The Holistic Timeline ("Life Stream")

**Epic Goal:** Compile all fragmented inputs, voice notes, and quick logs into a continuous, chronological timeline that behaves as the primary review surface.

### Story 4.1: Chronological Lived-Experience Stream

* **As a** caregiver reviewing our week,
**I want to** view a chronological, unified feed of all voice transcripts, quick logs, meals, and environmental contexts,
**So that** I can see how different events across the day interlock.
* **Acceptance Criteria:**
* Displays a continuous scrolling timeline grouped by day.
* Visual markers differentiate entry types (e.g., an icon for a Voice Log vs. a Quick Tap icon for Medication).
* Tapping a timeline entry expands it to show the full raw text narrative alongside its AI-extracted categories.



### Story 4.2: Interruption-Tolerant Partial Data Display

* **As a** frantic caregiver who could only log half-finished fragments,
**I want** the timeline to gracefully display partial or incomplete data entries without throwing errors or locking me out,
**So that** I don't feel penalized or stressed by rigid input validation rules.
* **Acceptance Criteria:**
* The database handles missing fields (e.g., a voice entry with a timestamp but an empty text block because the caregiver had to drop the phone).
* The UI renders these incomplete entries gracefully (e.g., "Incomplete entry - Tap to finish adding details").

