# D2 Glossary, round 2.1: endurance running physiology and training terms

Accessed: 2026-11-25 for all entries. Budget used: about 20 tool calls and 12 distinct sources.
Confidence scale: **high** means the full text or an official abstract was read this run. **medium** means abstract-level or secondary only. **low/unverified** means background knowledge only; these entries must be verified before use.
Class: definition | formula. Labels: [PR] peer-reviewed, [BOOK], [AUTHOR-SITE], [SECONDARY], [BG] unverified background knowledge.

PMC (pmc.ncbi.nlm.nih.gov) served a reCAPTCHA wall for one article (PMC10695480). Springer full-text PDFs redirect to an auth page.

---

## (a) Entries

### Cluster 1: Aerobic capacity and speed

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| VO2max | Maximal rate of O2 uptake (mL·kg⁻¹·min⁻¹). Measured in the lab with a graded test and gas analysis, plateau criterion. Wearables (Apple Watch/Garmin) give an *estimate* from HR and speed. Typical values: sedentary adult about 35–45; elite male distance runner about 70–85 [BG]. Not directly sourced this run. Jones & Vanhatalo 2017 uses it as the reference frame: CP/CS occurs at about 70–90 % VO2max depending on training status. | https://pmc.ncbi.nlm.nih.gov/articles/PMC5371646 | Springer, Sports Med [PR] | 2017-03-22 | medium (frame) / unverified (values) | definition |
| vVO2max (Billat) | "Minimum velocity which elicits VO2max". Term introduced in 1984. It **combines VO2max and running economy into one factor**. Time to exhaustion at vVO2max (Tlim) is reproducible within an individual but varies widely between individuals (typically about 4–8 min [BG]). Tlim is inversely related to VO2max and positively related to vLT expressed as a % of vVO2max. Billat & Koralsztein, Sports Med 22(2):90–108. | https://pubmed.ncbi.nlm.nih.gov/8857705/ | Springer, Sports Med [PR] | 1996 | medium (abstract) | definition |
| VMA / MAS | French "vitesse maximale aérobie" = maximal aerobic speed. In practice it is treated as approximately vVO2max. Field estimate: Léger-Boucher/UMTT incremental track test, VAMEVAL, or 5–6 min max run. [BG], not sourced this run. Caveat: protocol-dependent, so VMA from a field test is not strictly equal to lab vVO2max. | — | — | — | unverified | definition |
| Running economy (RE) | "Complex, multifactorial concept that reflects metabolic, cardiorespiratory, biomechanical and neuromuscular characteristics during **submaximal** running". Strong predictor of performance. Determining factors are trainable, but responses are individual. Units [BG]: VO2 at a given speed (mL·kg⁻¹·min⁻¹), O2 cost (mL·kg⁻¹·km⁻¹, about 180–230 typical), or energy cost (kcal·kg⁻¹·km⁻¹). Measured at steady state below LT on a treadmill. The review gives norms tables, but the full text was not read this run. | https://link.springer.com/article/10.1186/s40798-015-0007-y (Barnes & Kilding) | Springer, Sports Med Open [PR, OA] | 2015 | medium | definition |

### Cluster 2: Thresholds and intensity domains

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| Intensity domains | Moderate domain lies below the 1st threshold (LT1/GET/VT1). Heavy domain lies between the 1st and 2nd. Severe domain lies above the 2nd (CP/CS). Jamnick et al. found "some evidence" that LT1, GET and VT delineate moderate from heavy. They found "little evidence" for most methods delineating heavy from severe, **except CP and CS**. | https://pubmed.ncbi.nlm.nih.gov/32729096/ | Springer, Sports Med 50(10):1729–1756 [PR] | 2020 | medium (abstract + secondary) | definition |
| LT1 / VT1 (aerobic threshold) | LT1 is the first sustained rise in blood lactate above baseline. VT1 (≈ GET) is the first ventilatory breakpoint (VE/VO2 rises without a rise in VE/VCO2) [BG detail]. It bounds the moderate and heavy domains (Jamnick 2020). Lucia 2003 uses VT1 as the top of HR zone 1. Lab-measured (blood lactate or gas exchange). Wearables can only estimate it. DFA-α1 is a lead (see section b). | Jamnick 2020 (above); Lucia 2003 via https://core.ac.uk/download/229776672.pdf | [PR] | 2020 / 2003 | medium | definition |
| LT2 / VT2 / RCP (anaerobic threshold) | The second threshold: VT2 = respiratory compensation point (RCP), top of Lucia zone 2. LT2 operationalisations vary (fixed 4 mmol·L⁻¹ OBLA, Dmax, MLSS) [BG]. Jamnick 2020 shows these methods are **not interchangeable** and most poorly mark the heavy/severe boundary. MLSS = highest constant intensity where lactate rises ≤1 mmol·L⁻¹ over the final 20 min of a 30-min bout [BG, standard Heck/Beneke criterion, not fetched]. | Jamnick 2020; Lucia 2003 | [PR] | 2020 | medium (concept) / unverified (MLSS criterion) | definition |
| Critical speed (CS) | Running analogue of CP: "the highest power output [speed] that can be sustained without progressively drawing on W′ [D′]". Unit m·s⁻¹. CS separates the heavy domain (VO2/lactate steady state possible) from the severe domain (no steady state). It sits above LT, is "akin to" MLSS, and occurs at about 70–90 % VO2max. Elite marathoners race at about 96 ± 2 % CS. Field estimate: ≥3–4 maximal efforts or race PBs, about 2–15 min each (1500 m–15 km PBs used), or a 3-min all-out test. | https://pmc.ncbi.nlm.nih.gov/articles/PMC5371646 (Jones & Vanhatalo) | Springer, Sports Med 47(Suppl 1):65–78 [PR, OA] | 2017-03-22 | high | definition |
| D′ (D-prime) | Finite distance that can be covered above CS (m), analogue of W′. Elite male marathoners (n=12): D′ 224–616 m, mean 328 ± 104 m. CS 5.84 ± 0.14 m·s⁻¹. These are **elite** values and do not represent recreational runners. | same | same | 2017 | high | definition |
| CS/D′ model | **Formula:** distance d = CS · t + D′ (linear distance–time; R² > 0.999 in runners). Equivalent hyperbola: t = D′ / (v − CS) for v > CS. Fit by OLS on ≥3 (d, t) pairs; CS is the slope and D′ the intercept. The source states the linearisation explicitly for work–time (CP/W′). The distance form is implied by the same linearisation for running. | same | same | 2017 | high | formula |
| Maximal metabolic steady state | Jones, Burnley, Black, Poole & Vanhatalo argue that CP/CS, not MLSS, is the best index of the maximal metabolic steady state. Physiol Rep 7(10):e14098. | doi:10.14814/phy2.14098 (found by search, not fetched) | Wiley/APS, Physiol Rep [PR] | 2019-05-23 | medium (title/abstract via search) | definition |

### Cluster 3: Heart-rate-based intensity

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| Heart-rate reserve (HRR) | **Formula:** HRR fraction = (HRex − HRrest) / (HRmax − HRrest). It is used inside Banister TRIMP. | https://pmc.ncbi.nlm.nih.gov/articles/PMC7240108/ (Shaw et al.) | Frontiers in Physiology 11:480 [PR] | 2020-05-14 | high | formula |
| Karvonen method | **Formula:** target HR = HRrest + %intensity × (HRmax − HRrest) (Karvonen 1957) [BG]. The inverse of the HRR fraction above. ACSM uses %HRR as an intensity metric. The primary Karvonen/ACSM page was not fetched this run. | — | — | 1957 | formula sourced indirectly (HRR) / attribution unverified | formula |
| Resting HR (HRrest) | Lowest HR at rest, best measured on waking or (on a wearable) as a nightly or rest-period minimum. It is an input to HRR, Karvonen and Banister TRIMP. A downward trend reflects aerobic adaptation; an acute rise suggests fatigue, illness or heat [BG]. No dedicated source this run. | — | — | — | unverified | definition |
| Friel LTHR run zones | Friel's own page (revised Sep 2025) describes a field test: a **20-min solo time trial; average HR − 5 % = FTHR (≈LTHR)**. The zone % table on that page is an image and could not be extracted. **Conflict with background knowledge:** older Friel material (Triathlete's Training Bible) uses a 30-min TT with the average HR of the last 20 min, and run zones of Z1 <85 %, Z2 85–89 %, Z3 90–94 %, Z4 95–99 %, Z5a 100–102 %, Z5b 103–106 %, Z5c >106 % LTHR [BG, unverified]. Do not merge the two. Use the current page's protocol and verify the zone table from the book. | https://joefrieltraining.com/a-quick-guide-to-setting-zones/ | Joe Friel [AUTHOR-SITE] | 2020-04-01, rev. 2025-09 | protocol high / zones unverified | formula |
| Cardiac (cardiovascular) drift | During prolonged constant-intensity exercise, HR rises progressively and stroke volume declines after about 10–20 min. Coyle & González-Alonso attribute the SV decline mainly to the rising HR (reduced filling time), not to skin blood flow. Drift is graded in proportion to dehydration (Heaps et al. 1994, at 21 °C). Implication for the app: an HR-defined zone overstates intensity late in long runs. Pace:HR decoupling is the derived metric (see section b). | Coyle & González-Alonso, Exerc Sport Sci Rev 29(2):88–92, via https://scilit.com/publications/b2c41c04695672f811b81cfb09bf4f36 | Lippincott/ACSM [PR] | 2001 | medium (abstract) | definition |

### Cluster 4: Training load

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| Banister TRIMP (exponential) | **Formula:** TRIMP = D × HRR × A × e^(B × HRR). D is duration in minutes. Men: A = 0.64, B = 1.92. Women: A = 0.86, B = 1.67. HRR is the fraction defined above, using the session's mean HR. Another secondary source (a patent) gives a conflicting female A = 0.64. The peer-reviewed source (Shaw 2020) gives 0.86, so prefer 0.86. | https://pmc.ncbi.nlm.nih.gov/articles/PMC7240108/ | Frontiers in Physiology [PR] | 2020 | high | formula |
| Edwards TRIMP (summated HR zones) | **Formula:** Σ (minutes in zone × weight). Zones by %HRmax: 50–60 → 1, 60–70 → 2, 70–80 → 3, 80–90 → 4, 90–100 → 5 (Edwards 1993). | same | same | 2020 | high | formula |
| Lucia TRIMP | Three HR zones anchored to ventilatory thresholds: Z1 is below VT1, Z2 is VT1–VT2 (RCP), Z3 is above VT2 up to HRmax. **Formula:** Σ (minutes in zone × weight). Weights of 1/2/3 are standard [BG]; this run found only an inference from a secondary worked example, not an explicit statement. Origin: Lucia et al. 2003, MSSE 35(5):872–878. It needs a lab or estimated VT1/VT2. | https://core.ac.uk/download/229776672.pdf (search snippet); BikeRadar [SECONDARY] | [PR] citing paper | 2003 (orig.) | zones medium / weights unverified | formula |
| Session RPE (Foster) | **Formula:** sRPE load = CR-10 RPE × session duration (min) (Foster et al. 2001). RPE is collected about 15–30 min post-session. Edwards and Banister TRIMPs correlate with sRPE at r ≈ 0.46–0.96 within individuals. | https://pmc.ncbi.nlm.nih.gov/articles/PMC7240108/ | Frontiers in Physiology [PR] | 2020 | high | formula |
| Borg CR10 RPE | Category-ratio 0–10 scale of perceived exertion (0 = nothing, 10 = maximal). Foster's sRPE uses the **modified** CR-10 (Foster 2001). This run confirmed only that the modified CR-10 underlies sRPE. Original Borg 1982 anchors not fetched: [BG] 0 rest, 1 very easy, 2 easy, 3 moderate, 5 hard, 7 very hard, 10 maximal. Not to be confused with the Borg 6–20 RPE scale. | same | same | 2020 | medium | definition |

### Cluster 5: HRV

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| SDNN | SD of normal-to-normal (NN) inter-beat intervals in ms, with ectopic beats removed. **Both sympathetic and parasympathetic activity contribute.** In short resting recordings it mainly reflects parasympathetic RSA. In 24 h recordings LF power and sympathetic influence contribute. **Recording:** 24 h is the clinical gold standard (Task Force 1996), 5 min is the conventional short-term standard, and ultra-short proposals are 60–240 s. **Norms:** 24 h <50 ms unhealthy, 50–100 compromised, >100 healthy. Short-term mean 50 ± 16 ms (range 32–93, Nunan). Age 25–41, 24 h: men 160 ± 40, women 147 ± 36 ms. | https://www.frontiersin.org/articles/10.3389/fpubh.2017.00258/full (Shaffer & Ginsberg) | Frontiers in Public Health 5:258 [PR, OA] | 2017-09-28 | high | definition / formula |
| RMSSD | Root mean square of successive NN differences, in ms. **Formula:** √(mean((NNᵢ₊₁ − NNᵢ)²)). The **primary time-domain index of vagal (parasympathetic) activity**, more parasympathetically weighted than SDNN. **Recording:** 5 min conventional; ultra-short 10 s, 30 s and 60 s proposed. It is the index used for morning readiness or recovery tracking. Short-term norm: mean 42 ± 15 ms (range 19–75, Nunan). | same | same | 2017 | high | definition / formula |
| Task Force 1996 | Shaffer & Ginsberg rely on it for the 24 h and 5 min standards. The original (Circulation 93:1043–1065; Eur Heart J 17:354–381) was not fetched this run. | — | ESC/NASPE | 1996 | medium (via Shaffer) | definition |

Note for Apple Watch: HealthKit's HRV value is SDNN computed from short (about 60 s) PPG windows [BG, not sourced here; see other digests]. That is ultra-short SDNN, which Shaffer & Ginsberg say mostly reflects vagal RSA at rest. It is **not comparable** to the 24 h SDNN norms above.

### Cluster 6: Pace systems

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| VDOT (Daniels & Gilbert 1979) | A pseudo-VO2max derived from race performance (book *Oxygen Power*). It combines a speed→O2-cost curve and a duration→sustainable %VO2max curve. **Formula [SECONDARY, coefficients not verified against primary]:** VO2(v) = −4.60 + 0.182258·v + 0.000104·v² (v in **m/min**). %max(t) = 0.8 + 0.1894393·e^(−0.012778·t) + 0.2989558·e^(−0.1932605·t) (t in min) [second equation BG]. VDOT = VO2(v) / %max(t). Secondary summary: %max > 100 % for races under about 11 min, 94.0 % at 25 min, 88.8 % at 60 min, plateaus just above 80 %. **Units caveat:** one calculator mixes m/s and m/min. Use m/min. | getfitcraft.com/tools/vdot-calculator & search summary [SECONDARY]; PMC10695480 (blocked) | — | 1979 (orig.) | medium (structure) / low (exact coefficients) | formula |
| E/M/T/I/R paces | Daniels' training intensities: **E**asy (about 59–74 % VO2max/VDOT, ≈65–79 % HRmax), **M**arathon, **T**hreshold (about 83–88 % VDOT, "comfortably hard", about 20–60 min sustainable), **I**nterval (about 95–100 % VO2max, 3–5 min reps), and **R**epetition (faster than I, short reps for economy and speed). All values are [BG] from *Daniels' Running Formula*; no primary source was fetched this run. | — | Human Kinetics [BOOK] | 3rd ed. 2014 | unverified | definition |

### Cluster 7: Planning, adaptation and periodization

| Term | Definition / claim | Source | Publisher | Pub date | Conf. | Class |
|---|---|---|---|---|---|---|
| Taper | Bosquet, Montpetit, Arvisais & Mujika meta-analysis (27 of 182 studies). Optimal taper: **2 weeks** (ES 0.59 ± 0.33), with **training volume reduced exponentially by 41–60 %** (ES 0.72 ± 0.36) and intensity and frequency unchanged. Confirmed for endurance athletes by Wang et al. 2023 (PLOS One): ≤21 days, volume −41–60 %, intensity and frequency kept. | https://pubmed.ncbi.nlm.nih.gov/17762369/ ; https://pmc.ncbi.nlm.nih.gov/articles/PMC10171681/ | MSSE 39(8):1358–1365 [PR]; PLOS One [PR] | 2007; 2023 | high (abstract) | definition / formula (rule) |
| Periodization (traditional vs block) | **Traditional** (linear/Matveyev) develops many abilities at once, in macro/meso/microcycles. **Block** (Issurin) develops chosen target abilities **one after another** in concentrated mesocycles (accumulation, transmutation, realization). Kiely 2010 argues the evidence base is weak and calls it a "new variation". A 2016 review finds multi-target block programs superior in several sports. Other types [BG]: reverse, undulating (DUP), polarized/pyramidal **intensity distribution** (Seiler). The last is a TID model, not a periodization model. | https://pubmed.ncbi.nlm.nih.gov/20199119/ (Issurin); https://clok.uclan.ac.uk/5139 (Kiely) | Sports Med 40(3):189–206 [PR] | 2010 | medium | definition |
| Supercompensation | Model: training stress causes fatigue and a performance dip, recovery follows, and performance rebounds above baseline if the next load comes at the right time (Yakovlev). Formal alternative: Banister fitness–fatigue impulse-response model, performance = p0 + k1·fitness − k2·fatigue, with exponential decay constants τ1 ≈ 42 d and τ2 ≈ 7 d (this is the basis of CTL/ATL/TSB) [BG]. Not sourced this run. Note: supercompensation as a single-session mechanism is a simplification. The impulse-response model is the computable form. | — | — | — | unverified | definition / formula |

### Cluster 8: Session types

All entries below are [BG] and unverified this run. The source to verify against is Daniels' Running Formula [BOOK] or Seiler's TID literature.

| Term | Working definition | Conf. |
|---|---|---|
| Easy / recovery | Conversational pace, below LT1/VT1 (moderate domain), Daniels E. | unverified |
| Long run | The week's longest run, mostly at E pace. It often makes up about 20–30 % of weekly volume. | unverified |
| Tempo | Sustained continuous run near LT2 (Daniels T, "comfortably hard"), about 20 min. Usage of "tempo" varies across coaches (some mean marathon to half-marathon pace). | unverified |
| Threshold intervals / cruise intervals | Repeats at T pace with short recoveries (e.g. 5 × 1 mile with 1 min rest). | unverified |
| Intervals (VO2max) | Repeats of 3–5 min at I pace (≈vVO2max), with work:rest about 1:1. Billat's Tlim at vVO2max (sourced above) is why reps are kept shorter than Tlim. | unverified (Tlim link medium) |
| Repetitions (R) | Short fast reps (200–400 m) with full recovery, aimed at economy and speed. | unverified |
| Strides | About 15–30 s accelerations to near-sprint with full recovery, usually after an easy run. | unverified |
| Fartlek | "Speed play": unstructured, continuous alternation of paces. | unverified |
| Progression run | Pace increases through the run, often finishing at M or T pace. | unverified |

---

## (b) Leads

- Jamnick et al. 2020 full text (Sports Med 50:1729) holds threshold-method tables (LT1/LT2 variants, %VO2max). Try europepmc.org/article/MED/32729096 or vuir.vu.edu.au/41901/.
- Barnes & Kilding 2015 full text: https://link.springer.com/article/10.1186/s40798-015-0007-y (OA). It has the RE norms tables by calibre.
- Jones et al. 2019, Physiol Rep e14098: CS as the "gold standard" maximal metabolic steady state (OA).
- Field critical speed tests for runners: a 2024 systematic review appeared in search (Springer s00421-021-04780-8 / PMC11933073). Relevant for estimating CS from Apple Watch race and segment data.
- Daniels' %VO2max equation: verify both equations against *Oxygen Power* (1979) or *Daniels' Running Formula* appendix. PMC10695480 may quote them (blocked by captcha; try europepmc).
- Lucia TRIMP weights: Lucia et al. 2003 MSSE 35(5):872 original, or Borresen & Lambert 2009 Sports Med "The quantification of training load".
- Friel run zone % table: *The Triathlete's Training Bible* / *Total Heart Rate Training*. The website table is an image.
- Poole, Rossiter, Jones et al. 2021 "The anaerobic threshold: 50+ years of controversy" (J Physiol) is the canonical review on thresholds (not fetched).
- DFA-α1 (Rogers & Gronwald) as a wearable VT1 estimate: relevant for Apple Watch RR data.
- Banister fitness–fatigue model: Banister 1975 / Busso; Clarke & Skiba 2013 Adv Physiol Educ "Rationale and resources for teaching the mathematical modeling of athletic training and performance" (OA).
- Pace:HR decoupling (aerobic decoupling, Friel/TrainingPeaks) is the operational metric for cardiac drift [AUTHOR-SITE].

## (c) Not found / unverified this run

- VO2max typical population values and the lab plateau criterion: no source fetched.
- VMA/MAS: no Léger/Cazorla source fetched.
- Exact Karvonen 1957 / ACSM %HRR attribution. Only the HRR formula is sourced.
- MLSS ≤1 mmol·L⁻¹ criterion and the 4 mmol OBLA: background knowledge only.
- Daniels' %VO2max-duration equation coefficients and E/M/T/I/R % ranges: unverified.
- Lucia TRIMP weights 1/2/3: inferred, not stated by a source.
- Friel LTHR zone percentages: not extractable (the page shows them as an image). The protocol on the current page conflicts with the older 30-min/last-20 version.
- Borg CR10 original verbal anchors (Borg 1982): not fetched.
- Resting HR as a standalone definition, supercompensation, and session-type definitions: background knowledge only.
- Task Force 1996 original: cited only via Shaffer & Ginsberg.
- Apple Watch HRV = SDNN from short windows: not sourced in this digest.
