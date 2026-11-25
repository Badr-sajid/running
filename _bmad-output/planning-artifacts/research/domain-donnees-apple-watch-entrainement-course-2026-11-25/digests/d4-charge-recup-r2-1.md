# Digest d4: Training load and recovery, round 2 (follow-up 1)

Accessed 2026-11-25. Budget used: 20 tool calls, 11 distinct sources. PMC (pmc.ncbi.nlm.nih.gov) was blocked by a bot check, so I used Europe PMC's REST API (abstracts and metadata) or the publisher's HTML instead.
"Read depth" says how much of each source I saw: **full read** (whole text or the relevant section), **abstract** (Europe PMC metadata and abstract), **partial** (part of the page), **snippet** (search-engine summary only).

## (a) Claims

### Q1. Apple Watch SDNN validity and sampling

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 1 | O'Grady, Lambe, Baldwin, Acheson, Doherty: Apple Watch Series 9 and Ultra 2 against a Polar H10 chest strap processed in Kubios. 39 adults, 316 measurements over 14 days. The watch **underestimated HRV (SDNN) by a mean of 8.31 ms**, which fell outside the pre-set equivalence margin of ±10 ms. Resting HR agreed closely (mean difference −0.08 bpm). The authors advise caution when using the HRV for clinical or performance monitoring. Protocol was serial resting measurements, not overnight. | https://www.mdpi.com/1424-8220/24/19/6220 (DOI 10.3390/s24196220) | Sensors, MDPI | 2024-09-26 | abstract | High for the abstract figures; LoA and ICC not retrieved | accuracy-study |
| 2 | **Altini's November 2024 critique is based on claim 1.** The post "Trusting the Right Tool for the Job" links to that Sensors paper and says it "shows poor accuracy for the Apple Watch when used to compute HRV". It also says re-exporting the same data gave very different HRV values ("50ms to 100ms"). It calls sleep-time measurement "suboptimal to track training-related responses" and recommends a morning measurement instead. | https://marcoaltini.substack.com/p/trusting-the-right-tool-for-the-job | Altini Substack (**practitioner**) | 2024-11-22 | full read | Medium. The re-export problem is anecdotal. | accuracy-study (practitioner) |
| 3 | **The "SDNN is the most accurate wearable metric" study** is Dewig, Cohen, Renaghan, Leary, Leary, Au, Tenan, "Are Wearable Photoplethysmogram-Based Heart Rate Variability Measures Equivalent to Electrocardiogram? A Simulation Study", Sports Med 2024;54(11):2927–2934. It is a **Bayesian simulation**: real ECG from 1,084 PhysioNet subjects plus pulse-arrival-time dispersion taken from the literature, not a test of any device. At the lowest dispersion, 88.4% of SDNN values and 21.4% of RMSSD values fell inside the equivalence region. The authors conclude that PPG-derived HRV is not a substitute for ECG-derived HRV, and that SDNN has better measurement properties than RMSSD. Preprint: medRxiv 10.1101/2023.08.24.23294449. | https://doi.org/10.1007/s40279-024-02066-5 | Sports Medicine, Springer | epub 2024-06-27 | abstract | High that the paper says this. It does **not** validate the Apple Watch. | accuracy-study (simulation) |
| 4 | Apple's Vitals app (support page dated 2026-09-14) needs **7 nights of wear to set a typical range**. It sends a notification "if multiple metrics are outside of your typical range". The number that counts as "multiple" and the outlier rule are not published. Overnight metrics listed: heart rate, respiratory rate, blood oxygen, wrist temperature, sleep duration. On "Series 12 / Ultra 4" (model names as the page states them), **HRV replaces sleep duration**, and a daytime view adds resting HR and HRV. | https://support.apple.com/120142 | Apple | 2026-09-14 | full read | High for what Apple does. Apple's own design is a concordance rule ("multiple metrics"), which supports the brief's approach. | tool-behaviour |
| 5 | How often Apple samples HRV: about **once every 2 h during sleep on average**, at irregular, "quite random" times, with one SDNN value per opportunistic window or Breathe session. Only third-party sources say this; I found no Apple documentation. | tryterra.co/research/how-hrv-actually-works (from search results) | Terra (vendor blog) | n/d | snippet | Low (unverified) | tool-behaviour |
| 6 | Earlier lab validations, outside the freshness window and not overnight: in 2018, Apple Watch RR intervals agreed with a Polar H7 at above 0.9, with about 5 gaps per recording (PMC6111985). In 2025, Series 6 against a 3-lead Biopac ECG in 78 adults showed agreement highest at rest (Sensors 25(8):2380). | https://www.mdpi.com/1424-8220/25/8/2380 | MDPI | 2025 | snippet | Low-medium | accuracy-study |

### Q2. Baseline and smallest worthwhile change (SWC) method

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 7 | Buchheit 2014 (primary source for the SWC rule). The paper recommends the **individual coefficient of variation (CV)** as the reference for the SWC, and notes that authors have used "SWC = 0.5 (Le Meur 2013b) or 1 (Plews 2012, 2013b) × CV". It also says "there is currently no evidence that changes greater than any fraction of the CV would actually be meaningful in practice." Day-to-day **CV of Ln rMSSD is 10–20%** and of resting HR about 10%. SWC for positive adaptation is about +3% for vagal HRV and about −2% for resting HR. Correlations with running performance appeared only when **at least 3–4 days were averaged**, and the paper recommends measuring at least 3–4 times a week. It does not discuss 7-day rolling means or compare SDNN with rMSSD. For field use it favours rMSSD or SD1. | https://www.frontiersin.org/journals/physiology/articles/10.3389/fphys.2014.00073/full | Frontiers in Physiology | 2014-02-27 | partial (first 100k of 218k characters) | High | threshold |
| 8 | A rolling 2–4 day average gave a more precise mean than single days in overload microcycles, but its link to performance was unclear (Schneider 2019, Ln RMSSD, n=37). | https://doi.org/10.3389/fphys.2019.00582 | Frontiers in Physiology | 2019-05-22 | abstract | Medium | empirical |

### Q3. Resting HR flag threshold

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 9 | Buchheit 2014 gives **no bpm threshold**. Thresholds are expressed as a percentage or CV (resting HR CV about 10%, SWC about 2%). It warns that "an increased HR should not be used as a clear marker of fatigue". A rise in HR together with a fall in rMSSD suggests sympathetic activity or accumulated fatigue. A large fall in both HR and HRV that persists through a taper may point to an overtraining state. The only percentage cut-off is a **>4% rise in exercise HR** as a possible predictor of sickness. | same as claim 7 | Frontiers | 2014 | partial | High | threshold |
| 10 | Apple Watch resting HR is accurate against a chest strap (bias −0.08 bpm), so resting HR is a better-measured input than SDNN. | same as claim 1 | Sensors | 2024 | abstract | High | accuracy-study |

### Q4. Sleep and injury

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 11 | Milewski 2014: adolescent athletes aged 12–18 (n=112, retrospective survey). Those sleeping **less than 8 h had 1.7× the odds of injury** (95% CI 1.0–3.0, p=0.04). Level III evidence. The population is not adult runners, and the confidence interval touches 1.0. | https://doi.org/10.1097/bpo.0000000000000151 | J Pediatr Orthop | 2014-03 | abstract | Medium (weak design) | threshold |

### Q5. Monotony and strain (Foster 1998)

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 12 | Foster 1998, MSSE 30(7):1164–1168, n=25. Load = session RPE × duration. Monotony = daily mean / SD. Strain = load × monotony. Most illnesses occurred when athletes passed **individual** thresholds, mainly for strain. **The abstract gives no "monotony > 2" figure**, so that cut-off is not supported by the abstract, and the full text is unchecked. | https://doi.org/10.1097/00005768-199807000-00023 | MSSE | 1998-07 | abstract | High that the abstract has no "> 2" | threshold |

### Q6. intervals.icu Form zones

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 13 | The developer (David Tinker) said the Form zones "come from Joe Friel and are for absolute TSB". He applied the same thresholds to Form as a percentage of fitness because that worked for him at about 70 TSS/day, and called the zones "more of a 'rule of thumb' than science". The original question in the thread says the chart uses relative Form %. | https://forum.intervals.icu/t/zones-of-form-in-fitness-chart/3623 | intervals.icu forum (developer post) | 2021-04-12 | full read (thread) | Medium. Older than the 18-month freshness window. | tool-behaviour |
| 14 | Users say the high-risk zone starts at **−30% of relative Form**. The in-app help text, as quoted by a user, says the percentage option scales the zones to current fitness and "is probably the better choice". Neither point comes from official docs. | trainerroad.com/forum/t/overtraining-per-intervals-icu/82946 and forum.intervals.icu/t/high-risk-zone-always/69368 | user forums | n/d | snippet | Low-medium | tool-behaviour |

### Q7. HRV-guided training meta-analyses

| # | Claim | Source | Publisher | Pub date | Read depth | Confidence | Class |
|---|---|---|---|---|---|---|---|
| 15 | A **third meta-analysis** joins the two found in round 1: Manresa-Rocamora et al. 2021 (IJERPH, PMC8507742). It found SMDs that were consistently small and non-significant (p>0.05) in favour of HRV-guided training for VO2max and endurance performance, and concluded that "if superior... only by a small margin". Granero-Gallegos 2020 found ES 0.402 for VO2max, with larger effects in amateurs and women. Düking 2021 found small, non-significant effects but fewer non-responders. **No 2024–2026 meta-analysis was found.** | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8507742/ ; https://mdpi.com/1660-4601/17/21/7999 | IJERPH, MDPI | 2021 / 2020 | snippet | Medium | empirical |

## (b) Leads (contradictions first)

1. **Contradiction: is SDNN the "better" PPG metric?** Dewig 2024 (simulation) says SDNN is more robust to PPG error than RMSSD. O'Grady 2024 (real Apple devices) found that Apple's SDNN misses the ±10 ms equivalence margin. Both can be true: SDNN may be the better metric, while Apple's implementation is still biased low. A fixed bias may not matter for change detected within one person against their own baseline, but Altini's re-export instability would, if it is real. **Next step:** get the O'Grady full text for limits of agreement and within-subject correlation, since a consistent bias is acceptable for baseline-relative flags and a random error is not.
2. **Contradiction: sleep-time vs morning HRV.** Apple measures at irregular times during sleep. Altini (practitioner) says sleep-time HRV is suboptimal and recommends a morning reading. Buchheit's averaging logic (at least 3–4 days) suggests a multi-day mean could absorb sparse, irregular samples. No study tested Apple's sleep-time SDNN baselines for training readiness.
3. **"Monotony > 2" is not in the Foster 1998 abstract.** Check the full text, or Foster 2001 (J Strength Cond Res, session-RPE method) or later papers, before using it as a threshold.
4. The SWC multiplier is a **convention, with no evidence for any value** (Buchheit's words). Use 0.5 × CV (Le Meur) or 1 × CV (Plews) and label it as a convention.
5. **Apple Vitals already uses a concordance rule** ("multiple metrics" out of a 7-night range). The coach could follow the same logic and state that it does.
6. Not yet fetched: Altini's "What's your normal range for heart rate variability?" (https://marcoaltini.substack.com/p/whats-your-normal-range-for-heart), the practitioner text on the normal range (presumably a 60-day mean ± SD with a 7-day rolling mean; this is unverified).
7. Bellenger 2021 (Front Physiol, PMC7820717): post-exercise RMSSD rose under functional overreaching in runners, so HRV can go up during overreaching. Worth reading for interpreting a high HRV value.

## (c) Not found

- An overnight validation of Apple Watch SDNN against ECG from 2023–2026.
- Any study that used Apple SDNN baselines for training readiness.
- Official Apple documentation on how often HRV is sampled during sleep. The only figure (about every 2 h) comes from a vendor blog.
- A second independent source for a resting HR flag threshold. No source gave a bpm or SD-based cut-off. Buchheit gives only CV and percentage figures, so the two-source rule is **not met**.
- Walsh 2021 "Sleep and the athlete" consensus: not returned by Europe PMC. Fullagar 2015 review: not searched (budget).
- Ramp rate (CTL points per week): not searched (budget). Still open.
- intervals.icu numeric zone boundaries and the current default display (absolute vs %) in official docs from 2025–2026. Only a 2021 developer post and user reports were found.
- Any 2024–2026 meta-analysis of HRV-guided endurance training.
