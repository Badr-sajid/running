# Digest D5 round 2, part 1: injury-load rules, detraining, return after a break, plan structure

Accessed: 2026-11-25 for all sources. Read depth: **full** means the full text, or the full abstract or table, was retrieved this run. **snippet** means search-result text or a secondary summary only.

## (a) Claims

### Q1. Frandsen et al. 2025, Garmin-RUNSAFE single-session spikes

| # | Claim | Source | Publisher / date | Read | Confidence | Class |
|---|---|---|---|---|---|---|
| 1.1 | Exposure = the session's distance divided by the **longest session in the preceding 30 days**. Categories: reference = regression or up to 10% progression; small spike >10–30%; moderate >30–100%; large >100%; NP = not possible to calculate (denominator 0, i.e. no session in the 30-day window). | https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12421110/fullTextXML (Schuster Brandt Frandsen J et al., "How much running is too much? Identifying high-risk running sessions in a 5200-person cohort study") | BJSM 2025;59(17):1203–1210 | full (first 100k of 118k chars) | high | empirical |
| 1.2 | Cohort: 5,205 runners followed for 18 months; 1,820 (35%) injured over 588,071 sessions. Runs less than 15 min apart were merged into one session. Sessions under 500 m or over 100 km were excluded. | same + https://blogs.bmj.com/bjsm/?p=11751 | BJSM 2025 / BJSM blog | full | high | empirical |
| 1.3 | Adjusted HRR for overuse injury: small 1.64 (1.31–2.05); moderate 1.52 (1.16–2.00); large 2.28 (1.50–3.48); NP 0.80 (0.40–1.61, NS). The dose-response is not monotonic (small ≈ moderate). | same | BJSM 2025 | full | high | empirical |
| 1.4 | Progressions of 1–10% also showed a non-significant trend toward higher risk (HRR 1.19, roughly −10% to +57%). The authors suggest a session-specific cut-off **below 10%** and say "increases in the longest run may be vastly less than 10%". | same | BJSM 2025 | full | medium (non-significant trend) | empirical → rule (authors' suggestion) |
| 1.5 | Week-to-week ratio: no significant association (HRR 0.97 / 0.88 / 0.91). ACWR: inverse association (large spike 0.75, 0.59–0.96), which the authors treat as unreliable rather than protective. | same | BJSM 2025 | full | high | empirical |
| 1.6 | **No stratification** by novice/experienced, sex or pace/intensity. The authors say effect modifiers were not analysed because of low injury numbers. Intensity spikes were **not** studied; interval sessions were included in distance only. | same | BJSM 2025 | full | high | empirical (absence) |
| 1.7 | Limitations: injuries were self-reported, load was distance only, the design is observational. The authors call for RCTs or target-trial emulation, and say multi-session progression can still be excessive, so the single-session rule should not be used on its own. | same | BJSM 2025 | full | high | empirical |
| 1.8 | How the 30-day window behaves after a break: the paper does not discuss this. By construction, after more than 30 days off the runner falls into NP (no denominator). After a shorter break the reference becomes the longest run done inside the window, which may be short. This is **my inference** from the definition, not a finding. | inferred from 1.1 | n/a | n/a | inference (unverified) | n/a |
| 1.9 | Replication or critique 2025–2026: none found in this run. | n/a | n/a | n/a | n/a | n/a |

### Q2. Why weekly ratios and the "10% rule" fail; risk factors

| # | Claim | Source | Publisher / date | Read | Confidence | Class |
|---|---|---|---|---|---|---|
| 2.1 | Nielsen et al. 2014: 874 novice runners with GPS watches. More than 30% weekly progression (over 2 weeks) vs less than 10% gave HR 1.59 (0.96–2.66), **P=0.07** for distance-related injuries, which is not significant. The authors suggest novices keep progression under 30% per week. | https://vbn.aau.dk/da/publications/excessive-progression-in-weekly-running-distance-and-risk-of-runn | JOSPT 2014;44(10):739–747 | snippet | medium | empirical |
| 2.2 | Damsted et al. 2019 (ProjectRun21, half-marathon preparation): runners who increased weekly distance by 20–60% were more often injured by day 21 than those who increased it by less than 20%. | https://pmc.ncbi.nlm.nih.gov/articles/PMC6253751 (via search snippet) | 2019 | snippet only; not verified | low-medium | empirical |
| 2.3 | ACWR mathematical coupling: the acute load is part of the chronic load, which creates spurious correlation (Lolli et al., BJSM). | https://bjsm.bmj.com/lookup/doi/10.1136/bjsports-2017-098884 | BJSM ~2017–2019 | snippet (403 on full text) | medium | empirical / methodological |
| 2.4 | Hulme et al. 2017 systematic review (middle- and long-distance runners): **previous injury** is the most consistent risk factor. Menstrual irregularity raises stress-fracture risk; oral contraceptives are associated with lower risk. Van Mechelen-style reviews (2014 PubMed 24809248) agree: previous injury in the last 12 months appeared in 5 of 8 studies. | https://www.doi.org/10.1007/s40279-016-0636-4 ; https://pubmed.ncbi.nlm.nih.gov/24809248/ | Sports Med 2017;47(5):869–886; 2014 review | snippet | medium-high (two sources agree) | empirical |
| 2.5 | Biomechanical and musculoskeletal measures are generally not supported as risk factors in non-elite runners (2022 meta-analysis). | https://pubmed.ncbi.nlm.nih.gov/35254562/ (snippet) | 2022 | snippet | medium | empirical |

### Q3. Detraining

| # | Claim | Source | Publisher / date | Read | Confidence | Class |
|---|---|---|---|---|---|---|
| 3.1 | Coyle et al. 1984, 7 highly trained endurance subjects: VO2max **−7% by day 21**, stable at **−16% by day 56**. At day 84 it was still above sedentary controls (50.8 vs 43.3 ml/kg/min). Exercise stroke volume fell early. Oxidative enzymes had a half-time of about 12 days and stabilised 50% above controls. Capillarisation did not decline. | Europe PMC abstract, PMID 6511559 | J Appl Physiol 1984;57(6):1857–1864 | full (abstract) | high (note n=7, highly trained) | empirical |
| 3.2 | Note: a secondary source (Rundell 1994, coachsci.sdsu.edu) gives −7/−13/−15% at 12/56/84 days, which conflicts with the abstract. **Use the abstract figures (3.1).** | https://coachsci.sdsu.edu/csa/vol11/rundell.htm | 1994 | snippet | n/a | n/a |
| 3.3 | Mujika & Padilla 2000 Part I: short-term detraining (<4 weeks) in highly trained athletes brings a rapid fall in VO2max and blood volume and a lower lactate threshold. Changes are **milder in recently trained people**. | Europe PMC abstract, PMID 10966148 | Sports Med 2000;30(2):79–87 | full (abstract; no numbers in the abstract) | high (qualitative) | empirical (review) |
| 3.4 | Mujika & Padilla 2000 Part II (>4 weeks): VO2max declines markedly but stays above control values in trained athletes; **recently gained VO2max is fully lost**; endurance performance is impaired. | Europe PMC abstract, PMID 10999420 | Sports Med 2000;30(3):145–154 | full (abstract) | high (qualitative) | empirical (review) |
| 3.5 | 2023 systematic review: short-term (0–4 wk) VO2max change ranges from −25.5% to +2.4%; long-term change ranges from −16% to −11.3%. Heterogeneity is wide. | https://www.frontiersin.org/articles/10.3389/fphys.2023.1334766/full | Front Physiol 2023 | snippet | medium | empirical |
| 3.6 | Reduced training: cutting frequency or duration by one-third to two-thirds does not significantly alter VO2max **if intensity is maintained** (Neufer 1989 review). | https://pubmed.ncbi.nlm.nih.gov/2692122/ (snippet) | Sports Med 1989 | snippet | medium | empirical |
| 3.7 | Recreational marathoners (Feely, Smyth et al., Frontiers Sports Act Living 2022; 43,933 Strava runners averaging about 40 years old and about 25 mi/wk): a training gap in the 12-week build of 7–13 days was associated with about **4.25%** slower marathons, 2 weeks with about 6%, and 3 weeks with about 7.5%. Gaps 3–7 weeks out mattered more than gaps 8–12 weeks out. Men, younger runners and sub-4h runners were affected more. Observational. | https://www.outsideonline.com/health/training-performance/missed-marathon-training-study-2023 (Sweat Science, 2023-01-26) | Outside / study 2022 | full (secondary article); primary not fetched | medium | empirical |
| 3.8 | How fast fitness returns: **not found** with primary data this run. | n/a | n/a | n/a | n/a | n/a |

### Q4. Daniels: return after a break

| # | Claim | Source | Publisher / date | Read | Confidence | Class |
|---|---|---|---|---|---|---|
| 4.1 | Daniels' FVDOT table. Multiply the pre-break VDOT by FVDOT-1 (no cross-training) or FVDOT-2 (cross-trained). Values (FVDOT-1 / FVDOT-2): ≤5 d 1.000/1.000; 6 d .997/.998; 7 d .994/.997; 10 d .985/.992; 14 d .973/.986; 21 d .952/.976; 28 d .931/.965; 35 d .910/.955; 42 d .889/[.994 printed, probably a typo for ~.944]; 49 d .868/.934; 56 d .847/.923; 63 d .826/.913; 70 d .805/.902; ≥72 d .800/.900. | https://news.vdoto2.com/?p=1285 "VDOT Adjustments For Time Off" (official Daniels/VDOT O2 site) | 2018-02-14, updated 2022-03-23 | full | high that this is the official convention; not validated | coaching-convention (book) |
| 4.2 | Weight adjustment: adjusted VDOT = old VDOT × old kg / new kg, then apply FVDOT. Re-establish VDOT from a race once racing resumes. | same | same | full | high (convention) | coaching-convention |
| 4.3 | The page gives **no** volume or mileage return rule; FVDOT adjusts paces only. | same | same | full | high | n/a |
| 4.4 | Popular guidance: "run easy for as many days as you missed" (Outside). Source quality is low. | https://www.outsideonline.com/health/running/training-advice/marathon/how-to-adjust-your-training-schedule-for-conflicts-and-injury | Outside | snippet | low | coaching-convention |

Cross-check: the FVDOT pace losses (about −2.7% at 14 d, −6.9% at 28 d) are the same order of magnitude as the observational marathon slowdowns in 3.7 (about 6% at 2 wk, 7.5% at 3 wk). The marathon slowdowns are larger, which plausibly reflects lost volume and specific preparation, not just VO2max. This comparison is my inference.

### Q5. Plan structure

| # | Claim | Source | Publisher / date | Read | Confidence | Class |
|---|---|---|---|---|---|---|
| 5.1 | Long run ≈ **25–30% of weekly mileage**. The rule is attributed to Daniels, with a time cap of about 2.5–3 h. Hansons adopts it: the 16-mile peak follows from that ratio (16–17 mi ≈ 33% of 50 mi/wk) and from time on feet. Hansons cites diminishing returns after about 3 h and rejects plans where the long run is 50% or more of weekly volume. | https://lukehumphreyrunning.com/hansons-marathon-method-the-16-miler/ (Luke Humphrey, Hansons co-author, 2011-07-06); https://runningmagazine.ca/?p=53689 (snippet) | coach sites | full + snippet (two sources) | high as a convention; no RCT evidence | coaching-convention |
| 5.2 | Pfitzinger: plans of 18 or 12 weeks (6–12-week plans for runners racing marathons close together); long runs peak at about 22 mi in the 55–70 mpw plans; a 5-week post-race recovery plan. | https://us.humankinetics.com/products/advanced-marathoning-3rd-edition ; https://runningwithrock.com/pfitz-marathon-training-explained/ | Human Kinetics / blog | snippet | medium | coaching-convention |
| 5.3 | Recovery or step-back weeks every 3–4 weeks: **no direct trial** in recreational runners was found. The closest evidence is one study in 88 men with undulating load and a volume drop after 4 weeks, which showed fitness gains and lower muscle-damage markers. The 3:1 rule otherwise rests on coaching blogs (RunnersConnect). | https://pmc.ncbi.nlm.nih.gov/articles/PMC9473708 (snippet); https://runnersconnect.net/marathon-periodization/ | 2022 / blog | snippet | low | coaching-convention (weak empirical) |
| 5.4 | Individualised load (HRV, perceived recovery, HR-speed index; adjusted twice a week) has been tested in recreational runners as an alternative to fixed recovery weeks. | https://pubmed.ncbi.nlm.nih.gov/35975912/ | MSSE 2022 | snippet | medium | empirical |

## (b) Leads
- Read the Feely/Smyth 2022 primary paper in Frontiers in Sports and Active Living for exact effect sizes by gap length and any rebound-time analysis.
- Damsted 2019 (PMC6253751): verify the 20–60% result and whether it is a cohort or RCT.
- Check whether the BJSM blog post (blogs.bmj.com/bjsm/?p=11751) or 2026 letters raise critiques of Frandsen 2025, e.g. reverse causation or the NP handling.
- The remaining 18k characters of the Frandsen full text (offset 100000) may contain the supplement or sensitivity analyses.
- The Daniels book, 4th ed. (2021), may have return-to-volume guidance that is not on the vdoto2 page.
- Mujika & Padilla full text: the numeric tables are not in the abstracts.

## (c) Not found
- Any Nielsen 2019/2020 BJSM commentary specifically on ratio failure. The search did not confirm one exists; the closest are Lolli (coupling) and the ratio-index critique at bjsm 54/15/934, which returned 403.
- van Gent 2007: not searched (budget).
- Primary data on how fast VO2max or performance returns after detraining.
- Frandsen effect by novice vs experienced, or for pace/intensity spikes: not analysed in the paper.
- Any 2025–2026 replication of Frandsen.
- Evidence-based plan length by distance (5k/10k/half/marathon) or number of quality sessions per week: only coaching conventions surfaced, and not for 5k/10k specifically.
- Any controlled evidence for 3:1 loading in recreational runners.
