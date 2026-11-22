# BaseLine — User Stories & MVP Scope (v0.1)

## 1. Context
- **Problem:** Building this app to bring together certain aspects like UI, functionality, and tracking from apps like bevel, google health, and lift card.
- **User:** This app will be used by me just to make sure I can have a funtional app that I can use to track my health and lift sessions
- **Success looks like:** I will be using BaseLine as my only app and not open the other apps I usually use.

## 2. User stories

### Format
As a <role>, I want <capability> so that <benefit>.
Acceptance criteria: 3–5 testable bullets.

### Stories

### Area A: Data ingestion

#### A1. Automatic Fitbit sync
As someone who wears a Fitbit Air every day, I want my sleep and heart data to arrive in BaseLine automatically so that my sleep and recovery numbers are ready when I wake up, without exporting anything by hand.

Acceptance criteria:
- Within 30 minutes of last night's sleep session appearing in Google Health, it is stored in BaseLine: sleep session (with stages), heart rate samples, HRV, resting HR, and respiratory rate
- Steps and workouts sync at least every hour during the day
- Re-fetching the same time range never creates duplicate records (a record is identified by `source + metric_type + timestamp`)
- A re-fetched record with the same key and a different value **updates** the stored value, unless I edited that record by hand (see A2)
- If my server was offline, the next sync backfills everything since the last successful sync, up to 7 days back
- Every record is tagged with its source (`fitbit`) so Apple Health can be added later without mixing data
- I can see the time of the last successful sync and the last error, if any
- If my Google authorization expires, I see a "Reconnect" message instead of the sync silently failing
- I get a push alert if no new data has arrived for 6 hours (must not fire every night just because I'm asleep) via ntfy and an email
#### A2. Manual corrections
As someone whose watch sometimes records bad data, I want to correct or hide a wrong value so that one bad reading doesn't skew my scores and baselines.

Acceptance criteria:
- I can edit a daily value (resting HR, HRV, sleep duration, bedtime/wake time) or mark it "exclude from baselines"
- An edited record is flagged `manually_edited` and **sync never overwrites it**
- The original synced value is kept, and I can revert to it with one action
- Derived metrics for that day (recovery, strain, sleep performance) recompute within 1 minute of the edit
- Edited values show a small "edited" marker wherever they appear
---

### Area B: Sleep

#### B1. Morning sleep summary
As someone checking my phone after waking up, I want to see how last night's sleep went so that I know whether I slept enough before planning my day.

Acceptance criteria:
- Shows total sleep, time in bed, efficiency (asleep ÷ in bed), and bedtime/wake time
- Shows sleep stages, labeled "low confidence" (wrist-based staging is approximate)
- Shows a 7-day bar chart of sleep duration for comparison
- A night of sleep belongs to the date I fell asleep
- If no sleep session synced for last night, shows "No sleep data yet. Last sync: <time>" instead of zeros
---

### Area C: Lifting log

#### C1. Log a set quickly
As a lifter, I want to log a set (exercise, weight, reps, RPE) in under 10 seconds so that logging doesn't interrupt my workout.

Acceptance criteria:
- Weight and reps prefill from my last set of this exercise
- Works on my phone browser
- Saved sets survive a page refresh
- I can mark a set as a warmup (warmups are excluded from e1RM and hard-set counts)
- I can edit or delete a set I just logged
#### C2. Start a workout and see last time
As a lifter, I want to see what I did last time for each exercise when I start a session so that I know what weight and reps to beat today.

Acceptance criteria:
- I can start a workout, add exercises from a list, and finish it
- Each exercise shows my sets from the most recent previous session (e.g. "Last: 185 × 8, 185 × 7, 185 × 6")
- I can add a custom exercise with a name and primary muscle group
- When I finish, I'm asked for a session RPE (0–10), which is stored with the workout
- An unfinished workout is still there if I close the browser and come back
---

### Area D: Recovery & Strain

#### D1. Morning recovery score
As someone deciding how hard to train today, I want a recovery score each morning so that I can push on good days and back off on bad ones.

Acceptance criteria:
- Shows recovery 0–100% with a Green (≥ 67) / Yellow (34–66) / Red (≤ 33) band, calculated per metrics spec §2.3
- For the first 7 days, shows "Calibrating (day X of 7)" instead of a score
- Between days 7 and 30, the score shows a "provisional" label
- Tapping the score shows each input (HRV, resting HR, resp. rate, sleep) with today's value, my baseline, and how much it moved the score
- Shows the 7-day HRV trend next to today's value
- If an input is missing, the score shows a "low confidence" flag and names the missing input
#### D2. Daily strain
As someone who trains and moves throughout the day, I want to see my strain so far today so that I understand how much load I've taken on.

Acceptance criteria:
- Shows strain 0–100%, calculated per metrics spec §2.1 (TRIMP from HR, with the sRPE correction for strength sessions)
- Updates after each sync during the day
- Tapping the score shows the load from each workout plus "all-day activity"
- A 60-minute session at ~80% HRR produces roughly 76% strain with default settings (spec sanity check)
---

### Area E: Progress & history

#### E1. Strength trend per lift
As a lifter, I want to see my estimated 1RM trend for a lift over time so that I know whether my strength is actually going up.

Acceptance criteria:
- Picking an exercise shows a chart of my best e1RM per week
- I can switch the range between 4 weeks, 12 weeks, 1 year, and all time
- Only non-warmup sets with reps + RIR ≤ 10 count toward e1RM (per metrics spec §3.1)
- Tapping a week shows the set that produced it (e.g. 185 × 6 @ RPE 8)
- With fewer than 2 weeks of data, shows "Not enough data yet" instead of an empty chart
---

### Area G: Ops / self-hosting

#### G1. Backups
As the person running BaseLine on my homelab, I want automatic database backups so that a dead drive doesn't erase my fitness history.

Acceptance criteria:
- The database is backed up automatically every night
- Backups are stored on a different disk or machine than the database
- At least the last 14 nightly backups are kept
- A restore has been tested at least once and the steps are documented in the README
- A failed backup triggers the same alert channel as A1
#### G2. Secure remote access
As someone who checks BaseLine at the gym, I want to reach it from my phone away from home so that I can log workouts anywhere without exposing my server to the internet.

Acceptance criteria:
- The app is reachable from my phone over Tailscale, both on and off my home network
- No ports are open to the public internet
- The app loads on my phone over cellular in under 3 seconds
---

## 3. MVP scope

| IN (MVP) | OUT (later) | Why it's out |
|---|---|---|
| A1 Fitbit sync (+ sync alert) | Apple Health ingestion | One data source first; source tagging keeps the door open |
| A2 Manual corrections | Running metrics (EF, Zone 2 pace, decoupling, Riegel) | Needs reliable run/GPS data; lifting and recovery come first |
| B1 Sleep summary | Sleep need + bedtime recommendation | Depends on strain and sleep-debt history; add once B1 and D2 are stable |
| C1 Log a set | Strain target range + ACWR | Needs 28 days of load data before it means anything |
| C2 Workout session + last time | CTL / ATL / TSB fitness chart | Needs 6+ weeks of data; a "journey" feature, not a daily one |
| D1 Recovery score | Coaching rules + LLM wording | Rules depend on every metric being trustworthy first |
| D2 Daily strain | PR celebrations, stall detection, next-set suggestions | Built on top of C1/E1; low effort once the log has data |
| E1 Strength trend | Hard sets per muscle / tonnage dashboards | Easy add-on to the lifting log after v1 |
| G1 Backups | Bodyweight / smart-scale integration | Not needed for the core sleep / recovery / lifting loop |
| G2 Tailscale access | Native iPhone app (Swift) | Web app first; native app once the API is stable |
|  | Multi-user accounts / auth beyond Tailscale | Single user; the Tailscale network is the security boundary for v1 |

## 4. Open questions

1. **Sync dependency:** The Fitbit has to sync to Google Health before BaseLine can fetch anything, and BaseLine can't control that delay. How long does it usually take? (Measure it during calibration.)
2. **Google API (spike):** What does Google's API currently expose for the Fitbit Air (HRV, respiratory rate, sleep stages, intraday HR), and how do authorization and rate limits work? This could change A1.
3. **HRV type:** Does the Fitbit report HRV as RMSSD or SDNN? Store it as `kind` per spec §2.3.
4. **Sleep date:** Assign a night of sleep to the wake-up date or the fall-asleep date? (B1) currently will go on the fall-asleep date
5. **Alerts:** Threshold and channel for the no-data alert. Will start with 6 hours but will tweak based on results
6. **Time zones:** Store everything in UTC and display in local time? What happens to sleep data when I travel?
7. **Exercise library:** Seed it with a starter list of common lifts, or start empty and add as I go?
8. **Units:** lb only for v1, or support kg from the start?
