# Rawha — Project Feature Summary

---

## 1. Project Overview

**Rawha** is a lightweight, offline-first wellness web app designed specifically for working moms. It guides users through a structured daily exercise routine with beginner-friendly movements, calming ambient sounds, gentle voice coaching, and a warm wellness aesthetic, all delivered as a single HTML file with no frameworks, no backend, and no internet connection required.

**Tech stack:** HTML5 · CSS3 · Vanilla JavaScript · Web Audio API · localStorage  
**File architecture:** Everything lives in one self-contained `.html` file — no build step, no dependencies, no server needed. Open in any browser and it works.

Link: https://rawha.netlify.app/ 
---

## 2. Core Features Implemented

### Exercise Routine System
- **13 beginner-friendly exercises** in a fixed daily sequence:
  Lymph Hops · Body Waves · Arm Swings · Trunk Twist · Dead Arms · Golf Swing · Marches · Ballerina Squats · Bear Hugs · Horse Stance Lunges · Upper Slaps · Lower Slaps · Face Taps
- Each exercise is displayed one at a time with a large, readable name and a subtle hint: *"Move gently at your own pace"*
- Exercise number badge (e.g. `3 / 13`) shown at a glance

### Countdown Timer — Dual Duration
- Users choose **30 seconds or 60 seconds** per exercise before starting via a pill-style toggle on the home screen
- An animated **SVG ring timer** drains clockwise as seconds count down, with the number displayed inside
- The selected duration label appears on the exercise screen (e.g. *"30 sec per exercise"*)
- Timer **pulses gently** when 10 or fewer seconds remain, drawing attention without alarm

### Exercise Controls
| Button | Behaviour |
|---|---|
| ⏸ Pause / ▶ Resume | Freezes and resumes the countdown |
| 🔁 Repeat | Queues the same exercise to replay after the current timer expires (does not interrupt the active countdown) |
| Next ➜ | Immediately marks the exercise complete and advances |
| Finish Session | Skips to the summary screen at any time |

### Repeat Exercise Logic
Tapping **Repeat** queues the replay — the button changes to "🔁 Queued!" and disables to prevent double-tapping. When the timer hits zero, instead of advancing, the same exercise restarts with a fresh full timer. The "Next Up" preview stays unchanged. The repeat count increments immediately and is tracked in the session total.

---

## 3. User Flow

```
Home Screen
  └─ Choose Calm Sound (optional)
  └─ Choose Duration: 30s or 60s
  └─ Start Workout
        │
        ▼
Exercise Screen  ──(every 3 exercises)──▶  Break Screen (20s)
   │  Timer counts down                        │  Lavender ring countdown
   │  Repeat / Pause / Next                    │  Floating bubble animation
   │  Next Up preview shown                    │  Next exercise name shown
   │                                           │  Skip Break button
   └──────────────────────────────────────────┘
        │
        ▼ (after all 13 exercises)
Summary Screen
   └─ Exercises completed · Repeats · Total time
   └─ Chips showing each completed exercise
   └─ Encouraging message + celebration animation
   └─ Restart or Back to Home
```

---

## 4. Timer & Break Logic

### Exercise Timer
- `setInterval` ticks every 1 second, decrementing `timeLeft`
- SVG arc updates via `stroke-dashoffset` for smooth visual drain
- On reaching zero, `onExerciseDone()` checks for a pending repeat first; if none, it increments the completed count and routes to the next exercise or break

### Break Screen — Triggered Every 3 Exercises
- Fires automatically when `nextIdx % 3 === 0`
- Displays a **20-second lavender ring countdown** (separate SVG arc from the exercise timer)
- Ambient sound volume **automatically lowers to ~45%** during the break, then eases back up when exercise resumes
- A calming break-specific voice prompt appears after 3 seconds (e.g. *"Feel your feet on the ground"*)
- Five **floating pastel bubbles** drift upward as a visual relaxation cue
- A pulsing 🌿 circle animates with a slow "breathe in / breathe out" rhythm
- User can skip the break at any time

### Automatic Flow
- After the break timer expires, the app transitions automatically to the next exercise — no user action needed
- `onExerciseDone()` handles all routing: repeat → same exercise, break trigger → break screen, else → next exercise, last exercise → summary

---

## 5. Progress Tracking Features

### In-Session
- **"Next Up" strip** at the top of the exercise card always shows the upcoming exercise name in a soft sage-green pill; on the final exercise it reads *"Routine complete — amazing work!"*
- **Animated shimmer progress bar** (`0 / 13` → `13 / 13`) updates after each completed exercise
- **Repeat badge** appears below the timer ring showing how many times the current exercise has been repeated

### localStorage — Daily Persistence
- Progress is saved to `localStorage` under a date-keyed entry (`rawha_YYYY-MM-DD`) after every completed exercise and every repeat tap
- Saved fields: `completed` count, `repeats` count, `names` array of completed exercises
- The home screen reads this on load and displays today's stats in a sage-green "Today" banner
- Data resets automatically the next day (new date key = fresh start)

### Summary Screen Stats
At the end of each session, the summary card shows three stat tiles:

| Stat | Source |
|---|---|
| Exercises completed | Count of exercises fully timed out or skipped forward |
| Repeats added | Total repeat button taps across the session |
| Total workout time | `Date.now()` delta from session start, formatted as `m:ss` |

Completed exercises are also shown as individual sage-green chips (e.g. `Lymph Hops`, `Body Waves` …).

---

## 6. Design & UI Features

### Wellness Color Palette
All colors are defined as CSS custom properties and used consistently throughout:

| Token | Hex | Used for |
|---|---|---|
| `--cream` | `#FDF8F2` | Page background |
| `--peach` | `#F4A87C` | Primary actions, timer arc, progress |
| `--blush` | `#E8A0A0` | Accent, gradient end |
| `--lavender` | `#B8A4D4` | Break screen, sound bar, repeat badge |
| `--sage` | `#8BAF8B` | Next Up strip, progress chips, today banner |

### Screen Layouts
- **Home:** App logo · today's progress banner · sound picker · duration toggle · Start button
- **Exercise:** App name header · Next Up strip · exercise number badge · exercise name · duration label · animated progress bar · SVG ring timer · Pause / Repeat / Next / Finish buttons · sound bar
- **Break:** Pulsing 🌿 circle · break title · lavender ring countdown · Up Next card · floating bubbles · Skip button
- **Summary:** Celebration emoji burst · title · 3-column stat grid · exercise chips · Restart / Home buttons

### Responsive Design
- Max content width: `440px`, centred on all screen sizes
- Mobile media query (`max-width: 480px`) reduces font sizes and card padding
- All buttons are full-width with `16px` padding — large touch targets for mobile use
- Viewport meta tag ensures correct scaling on phones

### Ambient Sound Picker — "Choose Your Calm"
A 3×2 grid of sound chips on the home screen lets users choose from 6 options before starting:

| Option | Synthesis technique |
|---|---|
| 🌧️ Rain | Pink noise through dual bandpass filters (3 kHz + 6 kHz) + brown noise rumble + 0.07 Hz swell LFO |
| 🌊 Ocean | Brown noise shaped by a 0.12 Hz wave-gain LFO + slower 0.07 Hz swell — produces the rolling surge rhythm |
| 🌲 Forest | Pink noise wind + cricket bandpass (5.5 kHz, Q=8) + 19 precisely timed FM bird chirps with ASR envelopes |
| 🎹 Soft Piano | Karplus-Strong plucked string synthesis in a C major pentatonic arpeggio with short convolution reverb |
| 🎧 Lo-fi | Kick / snare / hi-hat drum pattern at 90 BPM + vinyl crackle + warm Fmaj7 chord pad |
| 🐦 Birdsong | Three "species" — warbler (ascending trill), robin (descending slur), wren (staccato pair) — with near-silence between calls |

All sounds are 100% offline — generated in real-time via the Web Audio API with no audio files needed. A compact **sound bar** on the exercise screen shows the active sound name and a Play/Pause toggle. Volume drops during breaks and restores on resume. Selecting a different sound mid-workout switches immediately.

### Soft Voice Coaching
A floating pill toast slides up from the bottom of the screen with gentle motivational messages:
- **During exercise:** 10 prompts drawn randomly every ~14 seconds (e.g. *"Relax your shoulders"*, *"You are enough, just as you are"*)
- **During breaks:** 5 calming prompts after a 3-second delay (e.g. *"Breathe in slowly… and out"*)
- **On session start:** Welcome message (*"Let's go, mama — you've got this! 🌸"*)
- **On summary:** Celebration message (*"Great job! You showed up for yourself today 🌟"*)

---

## 7. Technical Implementation

| Aspect | Implementation |
|---|---|
| **Architecture** | Single `.html` file — inline CSS + inline JS, no build step |
| **Timer engine** | `setInterval` at 1 s intervals; `clearInterval` on pause/advance/stop |
| **SVG ring timers** | `stroke-dashoffset` animation on `<circle>` elements; `stroke-dasharray = 2πr` |
| **Sound engine** | Web Audio API — `AudioContext`, `GainNode`, `BiquadFilterNode`, `OscillatorNode`, `BufferSourceNode` |
| **Noise generation** | Pink noise via Paul Kellet's IIR filter approximation; brown noise via random walk integration |
| **Scheduled audio** | `ctx.currentTime`-based scheduling for chirps/beats; `setInterval` re-schedules every 16–30 s |
| **Screen transitions** | CSS opacity fade + `translateY` slide via `.fading` class toggled with `setTimeout` |
| **localStorage** | `JSON.stringify/parse` on a date-keyed object; auto-expires by key mismatch next day |
| **Responsive layout** | CSS Flexbox + Grid; `max-width: 440px`; `@media (max-width: 480px)` breakpoint |
| **Animations** | `@keyframes` for: `fadeIn`, `timerPulse`, `shimmer`, `breathePulse`, `floatUp`, `popIn`, `celebBurst` |
| **No dependencies** | Zero external libraries, fonts, or CDN calls — fully offline after first load |

---

## 8. Future Improvement Ideas

**Workout Customisation**
- Let users reorder or remove exercises from the routine
- Add custom exercise creation with name and duration
- Support multiple preset routines (Morning / Desk Break / Evening)

**Audio**
- Allow volume control slider in the sound bar
- Add more ambient sound options (fireplace, rain on window, café)
- True text-to-speech voice coaching using the Web Speech API (`SpeechSynthesisUtterance`)

**Tracking & Streaks**
- Weekly streak counter and calendar heatmap view
- Total cumulative session history across days
- Personal best tracking (fastest completion, most repeats)

**Fitness Guidance**
- Short GIF or illustrated description for each exercise
- Difficulty levels (gentle / standard / energising)
- Breathing cue overlays during exercises

**UX Enhancements**
- Haptic feedback on mobile (Vibration API) at timer end
- Onboarding walkthrough for first-time users
- Dark mode / high-contrast accessibility mode
- Export / share session summary as an image

**Platform**
- Progressive Web App (PWA) manifest + service worker for true home screen install and offline caching
- Push notification reminders ("Time for your Rawha! 🌸")
