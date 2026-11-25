# D1 — Apple Watch data → HealthFit → intervals.icu (round 2, follow-up 1)

Accessed: 2026-11-25 for all sources. Reading depth: "full read" means the whole page or PDF text was retrieved and read. "Snippet" means a search snippet or a summarised fetch only. Note: the tool-call budget (20) was slightly exceeded, at about 26 calls; many of these were cheap local PDF-text extraction steps.

## (a) Claims

### Q1 — HealthFit

| # | Claim | Source | Publisher | Pub date | Conf. | Class | Depth |
|---|---|---|---|---|---|---|---|
| 1.1 | **Paywall verified.** The HealthFit developer (slizeray, "Stéphane") confirmed on 23 Mar 2026 that the intervals.icu workout planner integration requires the **Fanatical Supporter** plan. He called the paywall "intended behavior" and acknowledged that earlier versions had not gated it. A user (iam_hvb) switched to WorkOutdoors as a result. | https://forum.intervals.icu/t/import-planned-workouts-from-intervals-icu-into-apple-health/112136 | intervals.icu forum (developer post) | 23 Mar 2026 | High | integration | full read (summarised fetch) |
| 1.2 | Timeline of the planned-workout import. Beta announced 16 Sep 2025. On 10 Oct 2025 the developer wrote "HealthFit now imports planned workouts from Intervals.icu" (into Apple Health / Workout app). Unresolved user report (13 Oct 2025): outdoor-ride planned workouts did not appear on the watch. | same | intervals.icu forum | Sep–Oct 2025 | High (existence); Low (reliability) | integration | full read (summarised) |
| 1.3 | App Store price is $6.99 plus in-app purchases: "Fanatical Supporter $9.99" and "Awesome Supporter $4.99". Tier contents are not described on the page. intervals.icu is listed among the connected platforms. | https://apps.apple.com/us/app/healthfit/id1202650514 | Apple App Store (developer listing) | accessed 2026-11 | High | integration | full read (summarised) |
| 1.4 | Release notes 12.16.1–12.16.4 (Oct 2026): HealthFit can "export Recovery HRV to fitness platforms that support HRV", which **requires Apple Watch Series 12 or Ultra 4**. So a **Series 10 does not get this "Recovery HRV"**. Notes 12.15.2 and 12.16 (Sep–Oct 2026) add next-scheduled-workout widgets. | same | App Store | Sep–Oct 2026 | Medium (seen via summarised fetch, not verbatim full notes) | integration | snippet/summarised |
| 1.5 | The App Store description lists HRV, resting HR, Cardio Fitness (VO2max) and sleep metrics as displayed health metrics, and FIT/GPX/CSV export. Split charts cover HR, pace, power, cadence, elevation and "running dynamics" (12.9.x). The listing does **not** state which fields go into the FIT file sent to intervals.icu. | same | App Store | May–Jun 2026 | Medium | integration | summarised |
| 1.6 | **HRV sent by HealthFit is Apple's SDNN, not rMSSD.** intervals.icu's developer (david) said: "Apple decided to use SDNN instead of rMSSD like just about everyone else." | https://forum.intervals.icu/t/healthfit-sync-for-activities-and-health-metrics/2718?page=4 | intervals.icu forum | 12 Mar 2021 | High for SDNN; dated | integration | full read (summarised) |
| 1.7 | It is unclear which HRV sample HealthFit uses. In 2021, users asked for the morning or Breathe-only value instead of the all-day average. No developer answer appears on that page. A search snippet from the same thread says HealthFit "did not isolate HRV and resting HR from sleep". Other reported issues: sleep-phase handling (2021) and weight not sticking. | same + search snippet | intervals.icu forum | 2021 (stale) | Low (stale, user reports) | integration | partial |
| 1.8 | HealthFit wellness sync to intervals.icu has an "Automatic synchronization" toggle. intervals.icu has a "Download old data" option to backfill sleep and other wellness data. SpO2 and body fat are also handled (2021). | same | intervals.icu forum | 2021–2022 | Medium (stale) | integration | full read (summarised) |
| 1.9 | Alternative app: IntervalsWellnessSync (Mar 2026) syncs Apple Watch HRV, RHR, sleep and weight to intervals.icu, but not workouts. | https://forum.intervals.icu/t/intervalswellnesssync-simple-apple-watch-health-data-sync-to-intervals-icu/125843 | intervals.icu forum | Mar 2026 | Medium | integration | snippet |

### Q2 — Effort / Training Load

| # | Claim | Source | Publisher | Pub date | Conf. | Class | Depth |
|---|---|---|---|---|---|---|---|
| 2.1 | **Verified:** Training Load "compares the intensity and duration of your workouts over the last 7 days to what you've done over the previous 28 days". Apple gives no formula. | https://support.apple.com/guide/watch/apde4c07a6cf | Apple Support (Watch User Guide, watchOS 27 path) | current (2026) | High | device-feature | full read (summarised) |
| 2.2 | HealthKit has `HKQuantityTypeIdentifier.estimatedWorkoutEffortScore`, introduced in iOS 18.0 / watchOS 11.0. The doc has no abstract describing the estimation method. So the automatic estimate is stored in HealthKit as its own quantity type (separate from the user-entered `workoutEffortScore`). The `workoutEffortScore` symbol was **not** verified this run. | https://developer.apple.com/tutorials/data/documentation/healthkit/hkquantitytypeidentifier/estimatedworkouteffortscore.json | Apple Developer | 2024 (iOS 18) | High (existence/availability); method unknown | device-feature | full read (JSON) |
| 2.3 | Third-party sources only: effort is on a 1–10 scale and is auto-estimated for popular cardio workouts "based on factors like workout duration, heart rate and age". There is no automatic estimate for strength workouts. Training load ≈ effort × duration. Apple does not confirm the inputs. | https://www.iphoneincanada.ca/2024/12/05/add-effort-ratings-to-your-workouts/ ; https://dcrainmaker.com/2024/07/apples-training-load-vitals-watchos11.html | press / blog | 2024 | Low–Medium (secondary) | device-feature | snippet |

### Q3 — Apple whitepapers and Vitals

| # | Claim | Source | Publisher | Pub date | Conf. | Class | Depth |
|---|---|---|---|---|---|---|---|
| 3.1 | Sleep staging uses an **accelerometer-only** classifier with 30-s epochs and 4 outputs: Awake, REM, Core (= N1+N2), Deep (= N3). Design set: 1171 nights / 858 participants. Validation set: 299 nights / 166 participants, against PSG (in-lab plus some at-home). | https://www.apple.com/kr/health/pdf/Estimating_Sleep_Stages_from_Apple_Watch_Oct_2025.pdf | Apple | Updated Oct 2025 | High | accuracy-study | full read (text) |
| 3.2 | **Original algorithm (watchOS 9–11).** Median sleep sensitivity 97.9% and specificity 75.0%. 4-stage kappa 0.63 (SD 0.13). The most common error is true Deep scored as Core, and every stage's most common error is to Core. Deep→Awake 0.13%; REM→Deep 0.28%. In a clinical PSG cohort (236 subjects, 390 sessions): kappa 0.55, sensitivity 97.5%, specificity 72.5%. Kappa was lowered mainly by stage transitions and AHI; watch model had no effect. | same | Apple | Oct 2025 | High | accuracy-study | full read |
| 3.3 | **watchOS 26 / iOS 26 update (2025).** Uses foundation models built on Apple Heart & Movement Study data and still runs on the accelerometer only. Median sensitivity 96.8%, specificity 78.9%. Kappa rose from 0.63 to 0.68 (SD 0.11). Correct Wake classification rose from 70% to 79%. Accuracy improved for 3 of 4 stages. Clinical cohort: sensitivity 95.0%, specificity 86.5%, kappa 0.66. **Per-stage diagonal values (REM/Core/Deep) are only in the confusion-matrix image, which was not extractable.** | same | Apple | Oct 2025 | High | accuracy-study | full read (text; figures not read) |
| 3.4 | HealthKit write rules (watchOS 11+). A detected session of 1–3 h total sleep is written as binary asleep/awake. A session over 3 h is written with full stages. Naps and sleep outside the schedule have been tracked since watchOS 11. | same | Apple | Oct 2025 | High | device-feature | full read |
| 3.5 | VO2max (Cardio Fitness) is a **submaximal** estimate from the HR response to activity, using PPG, accelerometer, gyroscope, barometer and GPS. Range 14–60 ml/kg/min (watchOS 7+). It is produced after an **outdoor walk, run or hike** on relatively flat ground (grade <5%), with adequate GPS, adequate HR signal quality, and exertion of about +30% of the HR range from rest to max. A notion of peak HR is required, and users on beta blockers can declare this in Health Details. | https://www.apple.com/healthcare/docs/site/Using_Apple_Watch_to_Estimate_Cardio_Fitness_with_VO2_max.pdf | Apple | May 2021 (>18 mo; no newer version found) | High (method); dated | accuracy-study | full read (text) |
| 3.6 | VO2max validation (N=221). Error is +1.4 ± 4.7 ml/kg/min (**overestimates**; reference = projected *submax* CPET, not a max test). ICC 0.86. Per-user SD: median 1.2 (3.4%), 90th percentile 2.6. 78% of outdoor pedestrian workouts longer than 5.75 min yield an estimate. The cohort is older (mean age 55; reference VO2max 29.7 ± 10.5), so this is not a runner population. | same | Apple | May 2021 | High (numbers); applicability to trained runners: Low | accuracy-study | full read |
| 3.7 | Vitals overnight metrics: HR, respiratory rate, blood oxygen, wrist temperature (Series 8+) and sleep duration. The typical range is set after **7 nights**. A next-morning notification fires when **multiple** metrics fall outside the typical range; a regional page says "at least two". The page gives no numeric outlier definition. On **Series 12 / Ultra 4 only**, HRV replaces sleep duration overnight, and a daytime vitals view with RHR and HRV is added. So a **Series 10 gets no HRV in Vitals**. | https://support.apple.com/120142 | Apple Support | 14 Sep 2026 | High | device-feature | full read (summarised) |

### Q4 — HR sampling during workouts

| # | Claim | Source | Publisher | Pub date | Conf. | Class | Depth |
|---|---|---|---|---|---|---|---|
| 4.1 | "When you use the Workout app, Apple Watch measures your heart rate continuously during the workout and for 3 minutes after" (for HR recovery). The workout sampling interval is not stated there. | https://support.apple.com/en-ie/HT204666 (search snippet) | Apple Support | current | High (continuous); interval unknown | device-feature | snippet |
| 4.2 | Apple's HR page (published 30 Sep 2026) mentions green-LED measurements "as often as every five seconds, even when you're in motion" and a once-per-second reading with the electrical sensor. The summarising fetch tied the 5-s figure to Series 12 / Ultra 4, and its context (workout vs background) is unclear, so the verbatim text needs a re-check. Apple also says the watch "may not be able to get a reliable heart rate reading every time for everybody" and that there may be "occasional... abnormally high or low" readings. The page has no statement about intervals and no chest-strap recommendation. | https://support.apple.com/en-AU/120277 | Apple Support | 30 Sep 2026 | Medium (verbatim context uncertain) | device-feature | full read (summarised) |

## (b) Leads

- Get the verbatim text of support.apple.com/120277 (en-us) to settle whether "every five seconds" is a Series 12-only workout claim or a background claim. Also find its "what else affects your reading" link (likely HT204666 accuracy section).
- Find the full HealthFit version history (App Store "Version History") for exact wording on Recovery HRV, the planner and FIT fields. Ask the developer directly, or find a forum post, about which HRV sample is exported.
- Check intervals.icu activity "Data" tab / FIT file from a real HealthFit run to empirically confirm the fields present (power, cadence, GCT, VO, stride length, laps, 1-s vs sparse HR). This is the most reliable route, since there is no documentation.
- Read the per-stage diagonals from the confusion-matrix images (Figure 2 / Appendix Figure 2) in the sleep PDF by rendering them visually.
- Check developer.apple.com for `workoutEffortScore` and `HKWorkoutEffortRelationship` (relateWorkoutEffortSample) to see how third-party apps such as HealthFit can read the effort score per workout.
- PubMed 40373042 is an independent Apple Watch VO2max validation (30 participants) and was not read.
- Watchletic and WorkOutdoors are alternatives for intervals.icu → Apple Watch structured workouts.

## (c) Looked for, not found

- An official HealthFit FAQ or help page (healthfitapp.com/faq returns 404, and the homepage has only legal and contact links). No primary documentation was found on the FIT contents for Apple Watch runs (HR sampling rate, power, GCT, VO, stride length, laps/segments, elevation source).
- Which wellness HRV sample HealthFit sends to intervals.icu (nightly vs daily average). There is no developer statement. Also no statement on whether VO2max or resting HR are synced as intervals.icu wellness fields.
- How HealthFit builds watch workouts (WorkoutKit custom workouts?) and which targets it supports (pace/HR/power). Not documented in the sources reached.
- Apple's description of the automatic effort estimation inputs for running. Only third-party sources were found.
- Per-stage (REM/Core/Deep) correct-classification percentages as text.
- A VO2max whitepaper newer than May 2021.
- A numeric Apple definition of a Vitals "outlier".
- Any Apple statement on HR accuracy during intervals or fast HR transitions.
