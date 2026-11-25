# D4 — Training load, fatigue & recovery monitoring — Round 1, digest 1

Accessed: 2026-11-25 for all sources. Budget used: about 20 tool calls, 11 distinct sources read (most of them as abstracts or search snippets, not full text). Every source below is peer-reviewed unless it is labelled otherwise.

**Caveat on depth:** several claims rest on abstracts or search-engine summaries, not on full text I read. Confidence is set to match. Fetching the full text failed for the AIS HRV guide (404), Imbach 2022 (Springer auth wall) and Foster 1998.

## (a) Claims

### Q1 — Banister / PMC / intervals.icu

| # | Claim | Source URL | Publisher | Pub date | Confidence | Class |
|---|---|---|---|---|---|---|
| 1.1 | intervals.icu can show Form as a **percentage of Fitness**. This is the **default for new athletes** and can be changed in Options on the fitness page or in /settings. | https://forum.intervals.icu/t/form-as-a-percentage-of-fitness/869 | intervals.icu forum, Announcements (author: David, the developer) | 2020-05-08 | High (primary, developer post), but over 18 months old, so check against the current app | tool-behaviour |
| 1.2 | The developer's rationale for the risk line: TSBr = ATL×100/CTL. A TSBr above 130 is "risky", which he equates to **Form -30%**. He says he got this ratio from reading elsewhere and does not cite an empirical source. | same | same | 2020-05-08 | Medium: the tool behaviour is clear, but the threshold has **no empirical basis cited** | threshold / tool-behaviour |
| 1.3 | Community threads describe the zones as green/"optimal" at about -5 to -30 and red/"high risk" below -30. One user attributes them to **Joe Friel**, defined for *absolute* TSB. In %-mode the same numbers scale with CTL, which explains users' confusion about "always in high risk". | https://forum.intervals.icu/t/zones-of-form-in-fitness-chart/3623 ; https://forum.intervals.icu/t/high-risk-zone-always/69368 ; https://www.trainerroad.com/forum/t/overtraining-per-intervals-icu/82946 | Forums (user posts, not official) | various, not verified | Low–Medium: secondary, read via search snippet only | tool-behaviour |
| 1.4 | Per a forum reply, Fitness (CTL) is an exponentially weighted moving average over **42 days** and Fatigue (ATL) over **7 days**. The search summary described Form as "fatigue minus fitness". The standard PMC definition is TSB = CTL − ATL, and the %-form post above (ATL/CTL ratio, negative = fatigued) is consistent with CTL − ATL. **The sign convention has not been verified in the official docs.** | forum snippet (search result) | intervals.icu forum | n/a | Medium for 42/7. Sign convention unverified. | model-description |
| 1.5 | Hellard et al. 2006 studied nine elite swimmers over one season and assessed the Banister model's limitations: accuracy, **ill-conditioning** and parameter **stability**. | https://www.thieme-connect.de/products/ejournals/pdf/10.1055/s-2006-923791.pdf (search hit; J Sports Sci 24:509–520, doi 10.1080/02640410500244697) | Journal of Sports Sciences | 2006 | Medium: only the abstract-level content was seen | empirical |
| 1.6 | Stephens Hemingway (thesis) found that "despite over 45 years" of use, the validity of fitness-fatigue models is inadequately studied. In simulation, measurement noise with **CV > ~4%** makes training-response prediction unsatisfactory. | https://rgu-repository.worktribe.com/output/682069 | Robert Gordon University repository (PhD thesis, **not peer-reviewed as a whole**) | 2021 | Medium | empirical |
| 1.7 | A 2022 paper, "The use of Fitness-Fatigue models for sport performance modelling: conceptual issues and contributions from machine-learning" (Sports Medicine – Open, doi 10.1186/s40798-022-00426-x), exists. It is presumed to be Imbach et al. **Content not retrieved** (auth wall). | doi above | Springer / Sports Med Open | 2022 | Existence: Medium. Content: unverified | model-description |
| 1.8 | Impellizzeri et al. say the **only reference model** behind ACWR-type metrics is Banister's. They argue the Banister model was oversimplified and that an additive model was turned into a ratio without justification. | https://pmc.ncbi.nlm.nih.gov/articles/PMC7534938 | J Athl Train 55(9):893–901 | 2020 (correction 2021) | High | model-description |

### Q2 — ACWR

| # | Claim | Source URL | Publisher | Pub date | Confidence | Class |
|---|---|---|---|---|---|---|
| 2.1 | **Lolli et al. 2019:** ACWR "failed to normalize the numerator for the denominator even after controlling for the mathematical coupling". | https://pmc.ncbi.nlm.nih.gov/articles/PMC7534938 (citing Lolli 2019) | J Athl Train | 2020 | High that the citation is accurate. Lolli's primary paper was not read. | empirical |
| 2.2 | The **U-shaped "sweet spot" (0.8–1.3)** is weakly supported. Hulin's original studies showed no U shape, and the curve appeared only after unpublished AFL data were pooled. The authors recommend dismissing the model. | same | J Athl Train | 2020 | High (critique). The pro side (Gabbett 2016) was not read this round. | threshold |
| 2.3 | In a simulation (Bornn et al.) where injury was **independent of ACWR by design**, ACWR outside the sweet spot still appeared "associated" with injury. The effect vanished once the training schedule was modelled. "ACWR predicts the upcoming TL", not injury. | same | J Athl Train | 2020 | High | empirical |
| 2.4 | The Australian Institute of Sport advised that "the ACWR not be used as an indicator of injury risk". The authors conclude that ACWR should not guide injury-prevention decisions. | same | J Athl Train | 2020 | High that the statement is cited accurately | threshold |
| 2.5 | Impellizzeri et al., IJSPP 2020 (15:907–913): manipulating ACWR to change injury rates "remains conjecture". The ratio adds noise and statistical artefacts. | https://opus.lib.uts.edu.au/handle/10453/141898 | IJSPP | 2020 | High (abstract) | empirical |
| 2.6 | Impellizzeri et al., Sports Med 2021 ("What role do chronic workloads play…"): substituting contrived chronic loads "disproves the etiological theory" behind ACWR. | https://iris.univr.it/retrieve/e34cfb98-e922-4c2f-aab5-18583ab7e31b/Impellizzeri_What%20Role%20Do%20Chronic%20Workloads%20Play_SportMed_2021.pdf | Sports Medicine | 2021 | Medium–High (summary only) | empirical |
| 2.7 | **Counter-position:** a 2021 Frontiers editorial records that the literature is contested and cites a "level A evidence" claim for sRPE-based ACWR in elite athletes. | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8138569/ | Frontiers (editorial) | 2021 | Medium | empirical (contested) |

### Q3 — HRV-guided training and Apple Watch SDNN

| # | Claim | Source URL | Publisher | Pub date | Confidence | Class |
|---|---|---|---|---|---|---|
| 3.1 | Granero-Gallegos et al. 2020 (meta-analysis): HRV-guided training had a **small positive effect on VO2max (ES 0.402)**. The effect was larger and significant in amateurs and women. | https://pmc.ncbi.nlm.nih.gov/articles/PMC7663087 | IJERPH (MDPI) | 2020 | Medium–High | empirical |
| 3.2 | Düking et al. 2021 (meta-analysis): a **medium** effect on submaximal physiological parameters, and only a **small, non-significant** effect on performance and VO2peak. There were **fewer non-responders** with HRV guidance. | https://pubmed.ncbi.nlm.nih.gov/34489178/ | (PubMed, 2021) | 2021 | Medium–High | empirical |
| 3.3 | Methodological meta-analysis 2021 (IJERPH 18:10299): endurance performance SMD +0.20 (95% CI −0.09 to 0.48), non-significant. "If superior… only by a small margin." | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8507742/ | IJERPH (MDPI) | 2021 | Medium–High | empirical |
| 3.4 | Another 2020 pooled analysis: HRV-guided vs predefined training showed **no significant VO2max difference** (MD 0.96, CI −1.11 to 3.03). | https://www.pubmed.gov/34639599/ (via search) | (PubMed) | 2020/21 | Medium | empirical |
| 3.5 | Javaloyes et al. 2019 (IJSPP 14:23–32; n=17 cyclists): a 4-week baseline, then 8 weeks of HRV-guided vs traditional training, with HRV taken each morning as a **7-day moving average**. **No significant between-group differences.** Magnitude-based inference favoured the HRV group on some outcomes. A 2020 JSCR follow-up (n=20) also found similar results between groups. | https://pubmed.ncbi.nlm.nih.gov/29809080/ ; https://pubmed.ncbi.nlm.nih.gov/31490431/ | IJSPP / JSCR | 2019 / 2020 | High (abstract) | empirical |
| 3.6 | Decision rule as described secondhand: the smallest worthwhile change (SWC) is **0.5 × SD** of lnRMSSD (attributed to Plews 2012). Training load is reduced when the 7-day average drops below that SWC band. | https://www.trainingpeaks.com/blog/new-study-widens-hrv-evidence-for-more-athletes/ (blog, **secondary**) ; UMH thesis https://dspace.umh.es/bitstream/11000/5478/1/Tesis%20Javaloyes%20Torres%2C%20Alejandro.pdf | TrainingPeaks blog / PhD thesis | n/a | Medium: two non-independent secondary descriptions; primary methods not read | threshold |
| 3.7 | Plews et al.: a **weekly (7-day) averaged lnRMSSD** tracked adaptation and maladaptation better than single days. The correlation with training phase rose as the window grew from 1 to 7 days. In a 77-day case study of 2 elite triathletes, the athlete who became non-functionally overreached (NFOR) showed a declining weekly lnRMSSD trend. | https://pubmed.ncbi.nlm.nih.gov/22367011/ ; https://openrepository.aut.ac.nz/handle/10292/7122 (thesis) | EJAP 2012 / AUT thesis | 2012–2013 | Medium–High (case study n=2) | empirical |
| 3.8 | The AIS HRV best-practice guide recommends **lnRMSSD**, measured **on waking**, judged as a 7-day rolling average against the SWC. Only the search snippet was seen; the PDF returned 404. | https://www.ais.gov.au/__data/assets/pdf_file/0013/1130260/HRV-BPG_FINAL.pdf | Australian Institute of Sport (grey literature) | n/a | Medium | threshold / method |
| 3.9 | Buchheit 2014: the most useful HR-based monitoring tools are **resting HRV from roughly daily 5-minute recordings** and submaximal exercise HR. Changes should be judged against **measurement error and the smallest important change**, and in training context. HR cannot capture all aspects of fatigue, so it should be combined with logs, questionnaires and simple performance tests. | https://pubmed.ncbi.nlm.nih.gov/24578692/ ; https://www.frontiersin.org/journals/physiology/articles/10.3389/fphys.2014.00073/full | Frontiers in Physiology | 2014 | High (abstract) | model-description |
| 3.10 | Apple Watch (2018, n=20, vs Polar H7): RR data showed **very good agreement (>0.9) at rest**, with some gaps from missing RR intervals. RMSSD detected mild mental stress. This was a **daytime/short-recording** validation, not overnight SDNN. | https://pmc.ncbi.nlm.nih.gov/articles/PMC6111985 | (peer-reviewed, PMC) | 2018 | Medium | empirical |
| 3.11 | Apple Watch Series 6 vs ECG (2025, n=78): agreement was best at rest, N-N agreement was only moderate, and the authors note a **shortage of independent validation**. | https://www.mdpi.com/1424-8220/25/8/2380 | Sensors (MDPI) | 2025 | Medium | empirical |
| 3.12 | Dial et al. 2025 compared nocturnal HRV/RHR against ECG (n=13, 536 nights) for Oura Gen 3/4, WHOOP 4.0, Garmin Fenix 6 and Polar. Oura had the highest agreement (CCC 0.97–0.99). **Apple Watch was not tested.** | https://pmc.ncbi.nlm.nih.gov/articles/PMC12367097 | Physiological Reports | 2025-08-20 | High (study exists; Apple not covered) | empirical |
| 3.13 | Marco Altini says Apple Health exposes only **SDNN** at **irregular, opaque times** (the Breathe app forces a reading). Third-party apps cannot get the RR data needed for reliable HRV analysis. He considers rMSSD more physiologically meaningful, but says SDNN and rMSSD give "broadly the same types of results" for short resting readings. His Nov 2024 newsletter says an Apple Watch HRV validation "failed"; the paper was not identified. | https://medium.com/@marco_alt/how-to-make-sense-of-your-apple-watch-heart-rate-variability-hrv-data-89bf4a510438 ; https://marcoaltini.substack.com/p/recent-articles-and-updates-november | Practitioner blog (HRV4Training founder, **commercial interest**) | 2023–2024 | Medium | tool-behaviour |
| 3.14 | Oura overnight validation (other device): **SDNN** accuracy was acceptable only when averaged per night, not per 5 minutes. This suggests the aggregation level matters for SDNN. | https://pmc.ncbi.nlm.nih.gov/articles/PMC8808342 (JMIR 2022) | JMIR | 2022 | Medium | empirical |

### Q4 — Subjective vs objective, RHR, sleep

| # | Claim | Source URL | Publisher | Pub date | Confidence | Class |
|---|---|---|---|---|---|---|
| 4.1 | Saw, Main & Gastin 2016 (systematic review of 56 studies): subjective and objective measures **generally did not correlate**. Subjective well-being measures tracked acute and chronic load with **better sensitivity and consistency** than objective measures: impaired by acute load increases, improved by decreases. Subjective measures may stand alone or be part of mixed methods. | https://bjsm.bmj.com/content/50/5/281.short ; PMC4789708 | BJSM 50(5):281–291 | 2016 (online 2015-09) | High | empirical |
| 4.2 | No evidence-based RHR threshold (for example "+5 bpm") was found in this round. Buchheit 2014 frames all changes relative to each individual's measurement error and smallest important change, not fixed bpm cut-offs. | see 3.9 | — | — | n/a | threshold (absent) |

### Q5 — Overreaching / OTS

| # | Claim | Source URL | Publisher | Pub date | Confidence | Class |
|---|---|---|---|---|---|---|
| 5.1 | Meeusen et al. 2013 (ECSS/ACSM joint consensus, MSSE 45(1):186–205 and EJSS 13(1):1–24) defines a continuum: functional overreaching (FOR, a short performance dip, then recovery), non-functional overreaching (NFOR, stagnation for weeks to months) and overtraining syndrome (OTS). Its causes are multifactorial, and training is not necessarily the only cause. | https://pubmed.ncbi.nlm.nih.gov/23247672/ | MSSE / EJSS | 2013 | High | model-description |
| 5.2 | Telling NFOR from OTS is "very difficult" and depends on clinical outcome and exclusion diagnosis. Hormonal, performance, psychological, biochemical and immune markers are used, but **"none of them meets all criteria"** for general acceptance. Diagnosis requires ruling out organic disease or infection. | same | same | 2013 | High | empirical / threshold (negative) |
| 5.3 | A secondary summary mentions a protocol of two maximal tests 4 hours apart. Not verified from the primary text. | https://journal.aspetar.com/en/journals/volume-3-issue-1/overtraining-syndrome | Aspetar journal (secondary) | n/a | Low | model-description |

### Q6 — Monotony / strain / ramp

| # | Claim | Source URL | Publisher | Pub date | Confidence | Class |
|---|---|---|---|---|---|---|
| 6.1 | Foster 1998 (MSSE 30:1164–1168; n=25): load = session RPE × minutes; **monotony = weekly mean daily load / SD**; **strain = weekly load × monotony**. Most banal illnesses occurred when athletes exceeded their **own individual** thresholds, mostly strain-related. | https://lida.sport-iat.de/ta/Record/4001707?lng=en (abstract record) | MSSE | 1998 | High for the definitions; the thresholds are individual | model-description / empirical |
| 6.2 | The commonly quoted "**monotony > 2.0**" cut-off could **not** be traced to Foster 1998 in this round. | — | — | — | Unverified | threshold (unverified) |

## (b) Leads worth chasing (contradictions first)

1. **CONTRADICTION: HRV-guided benefit.** Granero-Gallegos 2020 reports a significant small positive effect on VO2max. Düking 2021, the 2021 methodological meta-analysis and another 2020 pooled analysis report non-significant or marginal differences against predefined training. Both sides should be reported. The one consistent signal is **fewer non-responders**. A post-2023 meta-analysis is still needed (none found).
2. **CONTRADICTION: ACWR.** The Impellizzeri/Lolli/Bornn critiques and the AIS guidance say not to use ACWR for injury risk. The Frontiers 2021 editorial cites a "level A" claim. Next: read Gabbett 2016 (BJSM) and Wang et al. 2020 directly, and look for any 2024–2026 position statement (for example an IOC load update).
3. **CONTRADICTION/GAP: intervals.icu zones.** The -30 (absolute, Friel) and -30% (developer's ATL/CTL ≥ 1.3) "high risk" lines mean different things depending on the display mode, and neither cites empirical evidence. Next: read the current intervals.icu docs or guide pages (2025–2026) for the zone definitions and the present default. Also check whether the 42/7 constants can be configured.
4. **Apple Watch SDNN for readiness:** there is no overnight validation of Apple's SDNN against ECG, and Apple was not in the Dial 2025 study. Next: find the paper behind Altini's Nov 2024 "failed validation". Search for 2024–2026 studies on Apple Watch nocturnal HRV or using SDNN baselines for readiness. Also check whether Apple's sleep-time SDNN samples are dense enough for a 7-day rolling baseline.
5. **Primary text of the Plews/AIS SWC rules** (0.5 SD vs a normal range like mean ± 1 SD over a 28–60 day baseline). Vesterinen 2016 and Kiviniemi 2007 were not reached in this round.
6. Read **Imbach 2022** in full: its conceptual issues and its machine-learning alternative to fitness-fatigue models.
7. **Foster 1998 full text**, to confirm or reject the "monotony > 2" cut-off.
8. The "SDNN most accurate wearable metric" claim (Sports Medicine study, via medicalxpress 2024) has only been seen secondhand. Locate the primary paper.

## (c) Looked for, not found

- An official intervals.icu documentation page (not the forum) stating the form zone boundaries, and any documentation from the last 18 months.
- Any 2024–2026 meta-analysis of HRV-guided endurance training.
- Any study validating Apple Watch **overnight SDNN** against ECG, or using Apple SDNN for training readiness or guided training.
- An evidence-based resting-HR flag threshold (for example +5 bpm, or N days above baseline) with two independent sources.
- Evidence-based thresholds for sleep duration or quality as a recovery flag (not searched this round, so still open).
- A primary source for the "monotony > 2.0" cut-off.
- A validated ramp-rate guideline (for example CTL points per week). Not searched this round; still open.
- Full text of Imbach 2022, the AIS HRV guide (404) and Lolli 2019.
