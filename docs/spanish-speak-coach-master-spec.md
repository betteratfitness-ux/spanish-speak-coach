# Spanish Speak Coach — Master Product Spec

> **Source of truth for all improvements to Spanish Speak Coach.**
> Last updated: 2026-07-04

---

## 1. Product Promise

> "Learn Spanish you can actually speak and understand in real life — one focused daily practice at a time."

The app exists to close the gap between knowing Spanish and being able to use it. Every feature must serve that promise. If a feature does not help users speak, understand, or recall Spanish in real situations, it does not belong.

---

## 2. Target User

**Primary:** English-speaking adults (30–55) who have tried to learn Spanish before and stopped. They are motivated but busy. They do not need another app that makes them feel behind.

**Secondary:** Adult learners in community, church, or workplace settings who need Spanish for real daily interactions — not travel, not tests.

**Secondary:** Tutors, community teachers, and program coordinators who want a simple tool to assign and track student progress without a full LMS.

**What they share:**
- Motivated by a real reason (neighbors, coworkers, family, travel)
- Limited time — 10–15 minutes per day at best
- Easily discouraged by missed days, long queues, or complicated interfaces
- Not looking for grammar drills — looking for phrases they can actually say

**What they do not want:**
- Shame for missing a day
- Streaks that feel like a job
- Vocabulary they will never use
- To feel like a student — they want to feel like a speaker

---

## 3. Current Strengths

- **30-day curriculum** with a clear daily lesson structure
- **4-phase lesson flow:** Learn It → Practice It → Say It Out Loud → Mini Conversation
- **Single daily CTA:** "Start Today's Spanish" — removes decision fatigue
- **Audio pronunciation** on every phrase
- **5 practice modes:** Listen→Repeat, EN→ES, ES→EN, Speak From Memory, Speed Round
- **Spaced repetition review** with nextDue timestamps
- **Streak + comeback tracking** already tracked in state (`stats.comebacks`)
- **Confidence rating (1–5)** after speaking — self-assessed
- **Daily stars system** (up to 5 categories per day)
- **XP + 18 badges** for milestone recognition
- **20 reading comprehension stories** across 4 difficulty levels
- **8 grammar mini-lessons**
- **35 native word chunks** (fillers, reactions, transitions)
- **Emergency SOS sheet** — one tap to survival phrases
- **Teacher progress report** — screenshot-ready summary with 30-day lesson map
- **Export / import** — full data backup via JSON
- **No account required** — all state in localStorage, zero friction to start
- **Works on any device** — mobile-first, browser-based, no install

---

## 4. Weaknesses From the Audit

### Learning integrity
- Self-rated confidence (1–5) tracks emotional confidence, not objective recall. It cannot be the primary signal for learning progress.
- There is no objective speaking proof — only self-reports. Users who tap "5" every time look fluent but may not be.
- The current "Fluency Score" label overpromises. It does not measure fluency. It measures activity.
- Review queue has no cap — returning users who missed days can face an overwhelming backlog. This is anti-comeback.

### Gamification risks
- Streaks, XP, badges, and stars can create anxiety if framed as things to maintain rather than proof of what has been done.
- The system currently tracks comebacks but does not celebrate them explicitly or reduce queue pressure for returning users.
- Badges are all-or-nothing unlocks — there is no partial progress visibility before earning.

### Technical risks
- **localStorage is fragile.** A browser clear, device switch, or accidental reset erases all learning history permanently. For a spaced repetition system built on timestamps, this is a serious long-term risk.
- The app has no backup reminder — users do not know they are one browser clear away from losing months of progress.
- Export/import exists but is buried in Settings and never surfaced proactively.

### Curriculum gaps
- No audio comprehension (listening) exercises — users hear audio only during Learn It phase.
- No true recall-from-nothing exercises — Practice It shows prompts, not blank recall.
- Mini conversations are multiple-choice only — they test recognition, not production.

### Progress reporting
- Teacher report shows activity metrics (days active, XP, streak) but limited learning proof.
- A teacher cannot currently tell from the report whether a student can actually recall or produce Spanish independently.

---

## 5. Required Learning Principles

The app must embody these six principles in every feature decision.

**1. The learning loop is non-negotiable:**
> Hear it → understand it → say it → recall it → use it in context → review it later.

Every lesson phase maps to this loop. Features that skip steps weaken retention.

**2. Production beats recognition.**
Choosing from options (multiple choice) is recognition. Recalling a phrase without a prompt is production. The app must build toward production — that is what speaking requires.

**3. Real-life Spanish first.**
Phrases are chosen because they come up in real situations — not because they are grammatically convenient to teach. Grammar serves communication; it is never the goal.

**4. Spaced repetition is the retention engine.**
What a user learns today must come back at scientifically timed intervals. A phrase seen once is not learned. A phrase recalled correctly after 1 day, 3 days, 7 days, and 21 days is approaching mastery.

**5. Speaking is not optional.**
An app that does not require speaking trains passive Spanish — recognition without production. Every lesson must include a moment where the user says the phrase out loud.

**6. Context accelerates retention.**
Phrases learned in a story, a mini conversation, or a real-life scenario are retained better than phrases on flashcards. Every phrase should appear in a sentence and ideally in a realistic exchange.

---

## 6. Behavior-Change Principles

**The app must be an anti-Duolingo experience.**

Duolingo's model: daily pressure + streak anxiety + gamification as the primary hook.
Spanish Speak Coach's model: meaningful progress + no shame + comeback always welcome.

**Six rules for behavior change in this app:**

**1. Never punish a missed day.**
A user who comes back after a week away is doing something hard. They deserve a warm welcome and a manageable path back in — not a broken streak and a queue of 47 overdue phrases.

**2. Comebacks matter more than perfect streaks.**
Returning after a gap is a bigger commitment signal than maintaining a streak. The app must recognize and celebrate that explicitly.

**3. One CTA per day.**
When a user opens the app, there is one primary action: start today's lesson. All other features are secondary. Decision fatigue kills daily habits.

**4. Make progress visible and specific.**
Vague progress ("You're doing great!") does not motivate. Specific proof does ("You've recalled 12 phrases without a prompt this week").

**5. Gamification is proof, not pressure.**
XP, badges, stars, and streaks are receipts — evidence of what was done. They must never be framed as things that need to be maintained, protected, or recovered.

**6. Low friction at every re-entry point.**
The hardest moment is opening the app after a gap. The experience at that moment must be warm, specific, and immediately actionable in under 60 seconds.

---

## 7. Spaced Repetition Requirements

### Core algorithm
- Items are tracked with `nextDue` timestamps in localStorage state.
- After a correct recall: next review in 1 day → 3 days → 7 days → 14 days → 30 days.
- After a missed or incorrect recall: next review in 1 day (reset the interval).
- Confidence rating (1–5) adjusts the interval: low confidence = shorter interval, high = longer.

### Queue management
- **Hard cap on review queue after missed days:** A returning user must never see more than 10 items in a review session, regardless of how many are overdue. Prioritize oldest-overdue items. Remaining items re-queue for tomorrow and the day after automatically.
- **No guilt messaging around overdue items.** The queue is not "falling behind." It is "catching up."
- Overdue items are surfaced as an opportunity, not a debt.

### Item states
Each vocabulary item must track:
- `correct` — count of correct recalls
- `missed` — count of missed/incorrect recalls
- `speakReps` — number of times spoken (self-reported)
- `lastReviewed` — timestamp
- `nextDue` — timestamp
- `confidence` — last self-rated confidence (1–5)
- `memoryState` — one of: `unseen` | `learning` | `reviewing` | `mastered`

Memory states:
- `unseen` — not yet encountered in any lesson
- `learning` — seen in a lesson but not yet correctly recalled independently
- `reviewing` — recalled correctly at least once; in spaced repetition cycle
- `mastered` — recalled correctly 5+ times with consistently high confidence and long intervals

### localStorage risk mitigation
- localStorage is acceptable for Phase 1 MVP.
- The app must proactively remind users to export their data after every 7 days of active use, and again when state exceeds 30 days of entries.
- Export prompt must be visible and framed as protecting their learning — not as a technical warning.

---

## 8. Comeback Review Engine Requirements

### What it is
A system that detects when a user returns after a gap and gives them a warm, manageable re-entry path — not a punishing backlog.

### Trigger conditions
- User has not opened the app in 2+ days (compare `streak.lastDate` to `today()`).
- The review queue has 5+ overdue items.

### Behavior on return
1. **Detect the gap.** Calculate days missed.
2. **Show a comeback message.** Warm, not guilt-inducing. Examples:
   - "You're back. That takes courage. Let's pick up where you left off."
   - "Life gets busy. You came back — that's what matters."
3. **Cap the review queue.** Show maximum 7 overdue phrases. Others silently re-queue.
4. **Offer a fast-win path.** Let the user do a 5-minute "Comeback Warm-Up" before today's full lesson — phrases from the last completed lesson only.
5. **Award a Comeback badge** (already tracked in `stats.comebacks`) and surface it visibly.

### What it must not do
- Must not show the full backlog count ("You have 34 overdue items").
- Must not reset the streak as an opening message.
- Must not require users to clear the backlog before starting today's lesson.

---

## 9. Speaking and Listening Requirements

### Speaking requirements
**Self-rated confidence (1–5) tracks emotional readiness — not objective fluency.**
It is useful for adjusting spaced repetition intervals and for user self-awareness. It must not be the primary signal for progress toward fluency.

**Objective speaking proof must include:**
- Count of phrases spoken from memory (Speak From Memory mode completions)
- Count of speaking reps across all modes (`stats.totalSpeakReps`)
- Mini conversation completion rate (how often the user chose the correct response)
- Say-it-out-loud confirmations in lesson Learn phase (self-reported but counted)

**Say-it-from-memory tracking:**
- When a user completes a Speak From Memory card, that must be recorded separately as a higher-quality signal than other speaking reps.
- A phrase with 3+ speak-from-memory reps is a stronger retention signal than 10 listen-and-repeat reps.

### Listening requirements (Phase 2)
- True listening comprehension is not currently tested. The audio button plays pronunciation — it does not test whether the user understood what they heard.
- Phase 2 must add: hear a phrase played audio-only, then select the English meaning from options.
- Phase 2 must add: hear a phrase played audio-only, then attempt to repeat it and rate your own accuracy.

---

## 10. Progress Score Requirements

### The current "Fluency Score" problem
The current score is labeled "Fluency Score" but it measures activity, not fluency. Calling it a fluency score overpromises and misleads users and teachers.

### Renamed score: "Speaking Progress Score" or "Practice Score"
The score must be named honestly — it measures practice engagement and recall performance, not fluency.

### Score components (proposed formula)
The score should weight:
- **Phrase recall accuracy (40%)** — correct recalls / total attempts, across all review sessions
- **Speaking reps (25%)** — total speaking reps, weighted higher for speak-from-memory completions
- **Lesson completion rate (20%)** — lessons completed / total lessons available
- **Review consistency (15%)** — percentage of due reviews completed on time

### Score display rules
- Never call it a fluency score unless the app can objectively verify comprehension, production, and listening.
- Frame it as: "Here is what your practice data shows."
- Display it alongside specific sub-metrics so users understand what drives it.
- Teachers must see the component breakdown, not just the final number.

---

## 11. Gamification Safety Rules

These rules govern every gamification element — XP, badges, stars, streaks, and scores.

**Rule 1: Frame everything as proof, never pressure.**
Every gamification element answers the question "what have you done?" — never "what are you at risk of losing?"

**Rule 2: Streaks are history, not obligations.**
A streak shows consistency that happened. It must never be presented as something that needs to be maintained. The streak display should say "You've practiced X days in a row" — not "Don't break your streak."

**Rule 3: Comebacks must be celebrated, not penalized.**
When a streak breaks and the user returns, the comeback event must be surfaced prominently. "You came back after X days. That's a comeback." The prior streak still shows as a personal best.

**Rule 4: No red states, no urgent warnings.**
The app must never show red indicators, warning icons, or urgent language around missed practice. Neutral or warm framing only.

**Rule 5: Badges are recognition, not requirements.**
Badges celebrate what happened. The locked/unlocked state is fine, but locked badges must not be framed as things the user is failing to achieve. Show them as "what's ahead" not "what you're missing."

**Rule 6: XP is a receipt.**
XP is logged as evidence of effort. It must not be used to rank users, to unlock features, or to create pay-to-win dynamics.

**Rule 7: Stars are effort markers, not grades.**
5 stars on a day means the user did five different types of practice. 2 stars means they did two. Both are good. The display must not imply that fewer than 5 stars is a failure.

---

## 12. Teacher and Reporting Requirements

### Who uses the teacher report
- Private Spanish tutors tracking student homework
- Community or church program coordinators
- Workplace program supervisors
- The user themselves, as a self-accountability tool

### What a teacher actually needs to know
A teacher cannot assess a student from XP or a streak count. They need learning proof:

**Minimum required data in teacher report:**
1. Student name + date generated
2. Current lesson day (out of 30)
3. Lessons completed (count + visual map)
4. Phrases practiced (total)
5. Phrases recalled without prompt (speak-from-memory reps)
6. Mini conversations completed (count + average correct response rate)
7. Review consistency (what % of due reviews were completed)
8. Current streak + best streak + total comeback count
9. Active days in last 30 days
10. Stories completed

**Future (Phase 2):** Listening comprehension score, objective production accuracy.

### Report format requirements
- Screenshot-ready on a phone screen (single-column, readable at 375px)
- Generated date visible
- No technical jargon — teachers are not developers
- Framed positively — shows what the student has done, not what they haven't
- Export to text/clipboard so it can be pasted into a message or email

---

## 13. Phase 1 Implementation Priorities

These are the items to build next. They correct the most important current gaps without adding new complexity.

**P1.1 — Comeback Review Engine**
Detect return after a gap. Show a warm welcome message. Cap overdue queue at 7 items. Surface the comeback count and badge explicitly.

**P1.2 — Phrase memory states**
Add `memoryState` field to item progress (`unseen` / `learning` / `reviewing` / `mastered`). Update state on every practice interaction. Display memory state in the Learn tab domain view.

**P1.3 — Capped review queue**
Never show more than 10 overdue items in a single review session regardless of total overdue count. Silently re-queue the rest across the next 2 days.

**P1.4 — Say-it-from-memory tracking**
Count Speak From Memory completions separately from other speaking reps. Store as `stats.speakFromMemoryReps`. Use this as a higher-weight signal in the progress score.

**P1.5 — Rename and rebuild progress score**
Remove the "Fluency Score" label. Replace with "Practice Score" or "Speaking Progress Score." Rebuild formula using: recall accuracy (40%), speaking reps (25%), lesson completion (20%), review consistency (15%). Show sub-metric breakdown.

**P1.6 — No-shame gamification copy audit**
Audit all streak, badge, XP, and star copy. Replace any pressure framing with proof framing. Ensure no element uses urgency, warning, or failure language.

**P1.7 — Backup/export reminder**
Surface a non-intrusive export reminder after 7 consecutive days of use and again at 30 days. Frame it as protecting learning history. Link directly to the export function.

**P1.8 — Improved teacher report**
Add to the teacher report: speak-from-memory reps, mini conversation score breakdown, review consistency percentage, total comeback count. Remove or reframe any metric that is not a learning proof.

---

## 14. Phase 2 Implementation Priorities

Build these after Phase 1 is stable and tested in daily use.

**P2.1 — Listening comprehension exercises**
Audio-only prompts: the phrase plays without the Spanish text visible. User selects the English meaning from options. Tracked separately from speaking reps.

**P2.2 — Blind recall mode**
Show the English prompt only. No Spanish shown. User says the Spanish out loud, then taps to reveal and self-grades. Higher retention signal than prompted recall.

**P2.3 — Expanded mini conversations**
Move mini conversations from 3-turn multiple-choice to 5-turn open-ended. User types or selects responses. App evaluates whether the response is contextually appropriate (not just one correct answer).

**P2.4 — Cloud backup option**
Offer optional account-based backup (email + password) so progress is preserved across devices and browser clears. localStorage remains the primary store; cloud is the redundant copy.

**P2.5 — More lesson packs**
Days 31–60: Intermediate real-life Spanish. New domains: expressing opinions, describing past events, making requests, talking about the future. Follows the same 4-phase lesson structure.

**P2.6 — Teacher dashboard (multi-student)**
A web view where a tutor or coordinator enters a list of student names and access codes. Each student's progress report is viewable without the student needing to share a screenshot.

**P2.7 — Paywall gating**
Days 1–5 free. Days 6–30 require a subscription or one-time purchase. Gate is soft (reminder, not lockout) until account system is in place.

---

## 15. Phase 3 Implementation Priorities

Long-term capabilities that require backend infrastructure or AI integration.

**P3.1 — Voice recording and pronunciation feedback**
User records themselves saying a phrase. App compares to a reference pronunciation and gives basic feedback ("good," "try again," "your accent sounds natural"). Requires speech recognition API.

**P3.2 — AI conversation partner**
Open-ended conversation practice with a virtual Spanish speaker. User types or speaks. AI responds in natural Spanish. App offers in-line hints and post-conversation vocabulary review.

**P3.3 — Adaptive curriculum**
App detects which domains a user consistently struggles with and adjusts lesson order. If a user's transportation phrases have low recall accuracy, they get an extra day on transportation before moving to emotions.

**P3.4 — Institution licensing**
School, church, or corporate programs can purchase a cohort license. Program coordinator gets a dashboard view of all enrolled students. Students get the app at no individual cost.

**P3.5 — Push notifications**
Daily practice reminder via push (requires PWA or native app wrapper). Opt-in. Personalized — cites specific phrases the user is about to forget based on spaced repetition schedule.

**P3.6 — Offline-first PWA**
Convert to a Progressive Web App with service worker. Users can install it to their home screen, use it offline, and sync when connected. Removes the friction of navigating to a URL daily.

---

*End of spec. This document governs all product decisions for Spanish Speak Coach. Feature requests, improvements, and new ideas should be evaluated against the product promise, learning principles, and behavior-change rules before implementation.*
