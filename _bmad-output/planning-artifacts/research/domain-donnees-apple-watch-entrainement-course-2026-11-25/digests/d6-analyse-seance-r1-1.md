# D6 — Post-workout analysis and threshold estimation (round 1, digest 1)

Accessed: 2026-11-25 for every source. Budget used: 19 tool calls, 13 distinct sources consulted. Some sources were seen only as search abstracts or snippets, marked **[abstract only]**. The PMC and PDF full texts of Smyth & Muniz-Pumares 2020 could not be read: PMC returned a captcha and the PDF text was compressed.

Confidence scale: **H** = primary source read directly. **M** = primary source seen through an abstract or snippet, or a secondary source consistent with the primary. **L** = forum anecdote, third party, or unverified.

---

## (a) Claims

### 1. Workout compliance (planned vs completed)

| # | Claim | Source URL | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| 1.1 | intervals.icu activity compliance = `training load × 100 / planned load`. If there is no load, it uses moving (or elapsed) time / planned duration. If there is no planned load or duration, compliance = 100%. | https://forum.intervals.icu/t/compliance-activity-field/47805 | intervals.icu forum (dev "david", announcement) | 2023-08-30 | H | tool-behaviour |
| 1.2 | intervals.icu uses a one-to-one mapping between an activity and a planned workout, computed server-side. The older calendar compliance was per day per sport. The official text does not describe any per-interval compliance. | same | intervals.icu forum | 2023-08-30 | H | tool-behaviour |
| 1.3 | intervals.icu colours: green 80–120%, orange 50–150%, red otherwise. An unpaired activity in a week with a plan = 0%, unless marked "Ignore compliance". In the API, `activity.compliance` is negative when ignored and null when not computed. | same | intervals.icu forum | 2023-08-30 | H | tool-behaviour |
| 1.4 | intervals.icu does not auto-pair an activity whose load is < 50% of the planned load. This stops a commute or warmup from pairing. Manual pairing is done by drag and drop. | https://forum.intervals.icu/t/workout-planning-and-matching-activities/1809 (search snippet) | intervals.icu forum | n/d | M | tool-behaviour |
| 1.5 | TrainingPeaks computes compliance from TSS, then duration, then distance (default priority, configurable). Orange = completed value > 50% above or below plan. Grey = unplanned. Red = not completed. Another TP help page describes red differently ("50% below (most likely) or above"), so the two TP sources disagree. I found no published green/yellow bands for endurance; the ±20% green band appears only for strength sets. | https://www.trainingpeaks.com/learn/trainingpeaks-athlete-user-guide/ ; https://help.trainingpeaks.com/hc/en-us/articles/204861204 | TrainingPeaks | n/d (help center, current) | M | tool-behaviour |
| 1.6 | Neither of the two market tools surfaced here exposes a standard per-block (per-interval) hit-rate score. Both work at the whole-session level, using load or duration. An app that compares block by block has to define its own metric. | inference from 1.1–1.5 | — | — | M | guidance (inference) |

### 2. intervals.icu specifics

| # | Claim | Source URL | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| 2.1 | intervals.icu computes load from power, HR and/or pace. A "Training load priority" setting per activity type picks which one wins (e.g. power over pace). Power load uses NP, which weights hard efforts more heavily. A user reports HR load on easy runs much lower than pace or power load, which is an anecdote. | https://forum.intervals.icu/t/running-load-threshold-intervals-vs-easy-runs/100899 | intervals.icu forum (users) | n/d | L–M | tool-behaviour |
| 2.2 | intervals.icu HRRc is computed only when HR reaches at least LTHR. It equals the largest HR drop over about 60 s after an above-threshold effort, for a single interval (the one with the max drop). It is not computed when there is no data during recoveries. Only the LTHR condition is confirmed by the developer; the 60 s window comes from users. | https://forum.intervals.icu/t/heart-rate-recovery-hrrc/387 | intervals.icu forum (david, 2021-12-31; users) | 2020–2021 | M | tool-behaviour |
| 2.3 | intervals.icu offers a "Seiler-inspired" decoupling based on 60 s moving averages, with power and HR expressed as % of reserve. The official page says lower decoupling means better aerobic fitness. | https://www.intervals.icu/features/seiler-decoupling/ ; https://www.intervals.icu/features/decoupling/ | intervals.icu | n/d | M [abstract only] | tool-behaviour |
| 2.4 | TrainingPeaks updates its HR threshold suggestion when the higher of Peak 60 min HR and 95% of Peak 20 min HR exceeds the current threshold. It never suggests a decrease. | https://www.trainingpeaks.com/learn/articles/thresholds-411/ (snippet) | TrainingPeaks | n/d | M | tool-behaviour |

### 3. Post-run indicators: evidence

| # | Claim | Source URL | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| 3.1 | Efficiency Factor (Friel / TrainingPeaks) = NP or Normalized Graded Pace / average HR, over the session or a segment. | https://trainingpeaks.com/blog/efficiency-factor-and-decoupling | TrainingPeaks blog | n/d | M | formula |
| 3.2 | The "< 5% decoupling = good aerobic endurance, > 10% = fatigue or bad pacing" cutoffs come from secondary sources and Friel's practice. I found **no validation study for the 5% cutoff**. | search snippets (third-party guides) | — | — | L | guidance (unvalidated) |
| 3.3 | Strava data from 82,303 marathon runners: the HR / grade-adjusted speed ratio, indexed to km 5–10, decouples on average at 25.2 ± 9.9 km. The low-decoupling third ran at a higher % of CS (88 ± 6%) and finished faster. Adding decoupling to a CS model drops the marathon prediction error from 6.45% to 5.16%. Males decoupled more and earlier. | https://link.springer.com/article/10.1007/s40279-022-01680-5 | Sports Medicine (peer-reviewed, Smyth et al.) | 2022 | M [abstract only] | empirical |
| 3.4 | Daanen et al. 2012, systematic review of HRR as a marker of training status (IJSPP 7:251–260). 3 of 5 cross-sectional studies show faster HRR in trained subjects, 2 show no difference. Most longitudinal studies show HRR improving alongside performance. Results are confounded by age, temperature, and the intensity and duration of the preceding exercise. Conclusion: potentially useful, but needs standardisation. | https://research.vu.nl/ws/files/3163042/292221.pdf (search snippet) | IJSPP (peer-reviewed) | 2012 | M [abstract only] | empirical |
| 3.5 | Buchheit 2014: a faster HRR is **not always positive**. It is faster with high training volumes and slower after acute high loads. A persistent rise not reversed by a taper may signal overtraining. HRR day-to-day CV is about 25% (HRR60) to 35% (HRRτ), with SWC about +7%. Submaximal exercise HR (HRex) has CV about 3% and SWC about −1%, the best signal-to-noise ratio (1.6). HRex is measured over the last 30–60 s after ≥ 3–4 min at a steady intensity. "An increased HR should not be used as a clear marker of fatigue." | https://www.frontiersin.org/journals/physiology/articles/10.3389/fphys.2014.00073/full | Frontiers in Physiology (peer-reviewed, technology report) | 2014 | H | empirical / guidance |
| 3.6 | Implication: HRRc from a single session (intervals.icu) is too noisy (CV about 25%) to judge fatigue alone. HR at a standardised sub-max pace, tracked as a trend, is more robust. GAP-indexed and temperature-controlled values are preferred. | inference from 3.5 + 2.2 | — | — | M | guidance (inference) |
| 3.7 | Buchheit reports that changes in HRex/HRV during a camp were **dissociated** from changes in perceived exertion. RPE and HR do not necessarily move together, so an RPE/HR mismatch cannot be read as fatigue without context. | same as 3.5 | Frontiers | 2014 | M | empirical |

### 4. Threshold estimation from field data

| # | Claim | Source URL | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| 4.1 | CP/CS concept (Jones & Vanhatalo 2017, Sports Med 47 Suppl 1:65–78): CS (m/s) and D′ (m) are the running equivalents of CP and W′. They are determined from trials to exhaustion lasting about 2 to 15 min. The "2–4 trials" figure comes from Poole et al. 2016 and was not confirmed in J&V. | https://pubmed.ncbi.nlm.nih.gov/28332113/ ; https://pmc.ncbi.nlm.nih.gov/articles/PMC5371646 | Sports Medicine (peer-reviewed review) | 2017 | M [abstract only] | formula / guidance |
| 4.2 | Smyth & Muniz-Pumares 2020 (MSSE 52(12):2637–45): CS computed from **raw Strava training data** (about 31,000 runners, 16 weeks before the race) as the slope of the distance–time regression over at least 3 target distances. The best model used 400 m, 800 m and 5000 m (R² = 0.695, error 7.67%). It predicts marathon performance. A first half run at > 94% of CS predicts a > 25% slowdown in the second half. | https://pmc.ncbi.nlm.nih.gov/articles/PMC7664951 ; https://uhra.herts.ac.uk/handle/2299/22878 | MSSE (peer-reviewed) | 2020 | M [abstract only] | empirical / formula |
| 4.3 | Friel LTHR test: a solo 30 min effort (not a race), with LTHR = average HR over the last 20 min. Starting too fast biases the result, and repeating the test improves accuracy. A forum variant uses 20 min − 5%, which is unsourced. | https://www.trainingpeaks.com/learn/articles/joe-friel-s-quick-guide-to-setting-zones/ | TrainingPeaks (Friel) | n/d | M | guidance |
| 4.4 | Validation: in competitive runners and triathletes, HR from the 30 min time trial does not differ significantly from the lab criterion. SEE is about 8 bpm, which is not negligible for zone edges. | https://pubmed.ncbi.nlm.nih.gov/16095403/ | JSCR? (2005, "A comparison of methods for estimating the lactate threshold"; peer-reviewed; journal not verified) | 2005 | M [abstract only] | empirical |
| 4.5 | Riegel is built on world records and **underestimates** recreational marathon time: it predicts at least 10 min too fast for half of runners (N = 2,303). It holds up better on short distances. Adding weekly mileage plus a prior race time works better (Vickers & Vertosick models). | https://link.springer.com/article/10.1186/s13102-016-0052-y | BMC Sports Sci Med Rehabil (peer-reviewed) | 2016 | M | empirical |
| 4.6 | Re-estimation frequency: TrainingPeaks adjusts **upward only**, automatically, when it detects a peak (see 2.4). I found no study defining an optimal re-estimation interval. | — | — | — | L | tool-behaviour / gap |

### 5. Feedback, adherence, motivation

| # | Claim | Source URL | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| 5.1 | An SDT systematic review (Soltani et al., ECIS 2024) classifies fitness-tech experiences by satisfaction or frustration of autonomy, competence and relatedness. Persistence after adoption depends on these needs. Details not read. | https://jyx.jyu.fi/bitstream/handle/123456789/95180/1/Going%20beyond%20Tracking_Understanding%20the%20Fitness%20Technology%20Use%20Persistence.pdf | ECIS 2024 proceedings (peer-reviewed conference) | 2024 | L [snippet] | empirical (review) |
| 5.2 | Krukowski et al. 2024, systematic review and meta-analysis (IJBNPA) on how feedback **generation and presentation** affect self-monitoring and physical activity. Not specific to wearables; results not read. | https://ijbnpa.biomedcentral.com/counter/pdf/10.1186/s12966-023-01555-6.pdf | IJBNPA (peer-reviewed) | 2024 | L [snippet] | empirical |
| 5.3 | A 2025 meta-analysis on older adults: long-term adherence often needs extra support, such as personalised goals and social interaction. The population is far from the target user. | https://jmir.org/2025/1/e59507/PDF | JMIR (peer-reviewed) | 2025 | L [snippet] | empirical |

---

## (b) Leads worth chasing

1. **Krukowski 2024 (IJBNPA)**: read the full text for which feedback formats (personalised, explanatory, timing) improve self-monitoring. This bears directly on the format of the "3–5 indicators" feedback.
2. **Smyth & Muniz-Pumares 2020, full text**: how they extract the 400/800/5000 m best efforts (window, filters, GPS), and the CS-to-marathon-pace relation by level. This is the direct algorithm for an automatic CS from Apple Watch / intervals.icu data. Try the Europe PMC or uhra HTML version.
3. **intervals.icu official docs**: the threshold pace / LTHR auto-detection settings, the API `pace_curve`/`eFTP` fields, "Polarization index", the precise "efficiency" field definition, and the GAP model used. Search forum.intervals.icu for "threshold pace estimate", "polarization index", "GAP".
4. **Maunder et al. / durability literature 2021–2025** (Sports Med, EJAP): validity of within-session decoupling as an individual marker, and the influence of heat.
5. **Heat and cardiac drift**: quantifying the HR rise per °C (e.g. studies by Périard, Racinais) and Strava's GAP model (public Strava engineering blog, 2017) for correcting hills.
6. **Tanaka 2001 (HRmax = 208 − 0.7 × age)** vs 220 − age, and field estimation of HRmax from training data. Not covered this round.
7. **Training "prescription vs execution" studies** (e.g. Foster et al. on coach intended vs athlete perceived RPE, 2001, and successors). Not covered.
8. **Daniels VDOT**: the original formula (Daniels & Gilbert 1979) and its validity against Riegel and CS.

## (c) Looked for, not found

- **TrainingPeaks green/yellow bands** for endurance compliance (the help center page returned 403). Only the orange and red rules and the priority order were confirmed, and the TP sources disagree on red.
- **A per-interval compliance metric** in intervals.icu or TrainingPeaks: none documented. Both work at session level.
- **Scientific validation of the 5% decoupling cutoff** (Friel): not found. It is practitioner guidance only.
- **Developer-confirmed HRRc definition** (window, interval choice): only the "HR ≥ LTHR" condition is confirmed.
- **Official intervals.icu doc** on automatic LTHR / threshold-pace estimation and the per-sport load priority logic: not reached (forum anecdotes only).
- **Evidence-based re-estimation frequency** for thresholds: not found.
- **Studies on RPE vs HR mismatch as a fatigue sign** in recreational runners: not found beyond Buchheit's dissociation remark.
- **Heat-adjusted pace / quantified HR correction for heat**: not searched (budget).
