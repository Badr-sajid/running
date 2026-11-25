# D3 Methods: round 2, follow-up 1 (TID, LLM plans, feedback format)

Accessed: 2026-11-25 for every source. Tool budget: 20/20 calls used.
"Full read" means the text was retrieved and checked this run. "Snippet" means only the abstract, a search summary or a partial page was seen.

## (a) Claims

### Q1: Training intensity distribution (TID) for recreational runners

| # | Claim | Source | Publisher / date | Status / read | Confidence | Class |
|---|---|---|---|---|---|---|
| 1.1 | The IPD network meta-analysis covered 13 studies and n=348 athletes (296 M, 52 F; about 150 recreational, the rest competitive). Baseline VO2max ranged 46.6–68.3 mL/kg/min. Using HR time-in-zone, it found **no overall POL vs PYR difference**: VO2max SMD 0.06, p=0.68; TT SMD 0.05, p=0.34. POL showed no significant difference against any other TID either (THR, HIGH, LOW). | Rosenblat et al., doi:10.1007/s40279-024-02149-3 (text taken from a PDF copy: https://www.fisiologiadelejercicio.com/wp-content/uploads/2025/02/Which-Training-Intensity-Distribution-Intervention-will.pdf) | Sports Med 55(3):655–673; epub 2025-01-31 | peer-reviewed; full read (text extracted from the PDF; minus signs lost in extraction) | high | empirical |
| 1.2 | **Subgroup by performance level.** The difference *between* subgroups was significant (SMD 0.63, p<0.05; about 2.7 mL/kg/min). **Within** each subgroup the difference was NOT significant. Recreational athletes favoured PYR (SMD 0.32, 95% CI −0.14 to 0.79, p=0.18). Competitive athletes favoured POL (SMD 0.31, 95% CI −0.70 to 0.09, p=0.13). The authors call the recreational result "not statistically significant but clinically meaningful". Baseline VO2peak also differed between groups (rec 53.0 vs comp 67.0), so fitness level is confounded with the subgroup. The effect appears for VO2max only; TT performance showed no such interaction. | same | same | full read | medium (a subgroup interaction with wide CIs, from a single collaborative group) | empirical |
| 1.3 | Definition: "competitive" means tier 3 or above (university/state/national/international/pro). Everyone else is "recreational". | same | same | full read | high | empirical |
| 1.4 | **Studies behind the recreational subgroup:** Festa (running, n≈35, VO2 53.3, PET vs FOC/THR, 2 km TT), Muñoz 2014 (running, 35, 61.6, PET vs BThET/PYR, 10 km TT), Pérez (running, 20, 56.8, POL vs THR/PYR), Sellés-Pérez (triathlon, NRCT, 15, 55.7, POL vs PYR), Zinner (running, NRCT, 45, 46.6, POL vs HIIT/HVT). **Competitive:** Filipas ×2 (running, 30 each, VO2 about 68), Festa/Treff-type running (15, 67.0), Neal (cycling, RCT-X), Schneeweiss (cycling), Stöggl (combined, 41), Talsnes (XC-skiing juniors), Treff (rowing). The recreational evidence therefore rests on about 5 small studies, 2 of them non-randomised. Some author/row attributions came out garbled in text extraction; check them against Table 1. | same, Table 1 | same | full read (table text garbled) | medium | empirical |
| 1.5 | Weekly training duration was entered as a study-level covariate. No covariate reached significance besides performance level (age p=0.71 was quoted). **No separate low-volume (<5 h/wk) result was reported.** Sample size: the authors estimate 81 per group would be needed to detect SMD 0.44, and the subgroups were smaller than that. | same | same | full read (the covariate p-value table could not be extracted) | medium | empirical |
| 1.6 | HR drift: participants prescribed LOW (or HIGH) TID tended to drift into a de facto PYR distribution when measured by HR. The authors recommend monitoring *completed* internal load, and external load (pace) where it is available. | same | same | full read | medium | guidance |
| 1.7 | Rosenblat critiques Oliveira et al. 2024 (which found POL superior for VO2peak). The criticisms are double-counting of participants, use of change-from-baseline, pooling of sprint and long TTs, and combining all interventions (including strength) into one pairwise analysis. | same; Oliveira 2024 PMC11329428 (snippet) | Sports Med 2024 | full read (critique); Oliveira: snippet | medium | empirical |
| 1.8 | Filipas et al.: 60 well-trained male runners, 16 weeks, load held constant across groups. **The PYR→POL sequence (8 + 8 weeks) gave the largest gains (about 1.5% on a 5 km TT).** All four groups improved. This population was *competitive* (VO2 about 68), so it is not a direct test for recreational runners. | https://pubmed.ncbi.nlm.nih.gov/34792817/ | Scand J Med Sci Sports 32(3):498–511, 2022 | peer-reviewed; snippet | medium (single lab, about 15 per group) | empirical |

### Q2: LLM-generated training plans

| # | Claim | Source | Publisher / date | Status / read | Confidence | Class |
|---|---|---|---|---|---|---|
| 2.1 | ChatGPT (reported as "Version 3.0.1", i.e. GPT-3.5 era, May 2023) generated 6-week running plans at 3 levels of input detail. 10 experts rated them on 22 criteria. **Median overall ratings: 2 / 3 / 4.** Plan 1 rated <3 on 19 criteria, Plan 2 on 11, Plan 3 on 1. More input led to better plans. | https://jssm.org/jssm-23-56.xml-Fulltext | J Sports Sci Med 23:56–72, 2024 | peer-reviewed; full read | high for that model; low transfer to 2026 models | ai-evaluation |
| 2.2 | Inputs that helped (they are confounded with follow-up check-backs, so their separate effects are unknown): age/sex, current frequency/volume/duration, running history, HR range, other sports, health status, % performance goal, available devices. | same | same | full read | medium | ai-evaluation |
| 2.3 | **Typical errors.** Even the best plan **progressed training variables too fast**, raising injury risk, and volume progression was rated below neutral. Several items were only neutral: health screening, testing, monitoring of contextual factors, frequency/progression, psychological and skill training. The model asked no coach-like feedback questions. It offered blog-style advice that lacks evidence (cool-downs for recovery). Raters were not blinded. | same | same | full read | medium | ai-evaluation |
| 2.4 | The authors recommend expert review before use, citations/evidence ranking in outputs, and integration of wearable data. | same | same | full read | — | guidance |
| 2.5 | Single-subject half-marathon preparation (Jul–Sep 2025) with ChatGPT (version not named). The runner finished (about 6'30"/km) with **70–80% session adherence**. **Failure modes:** no in-run sensing; dependence on user-reported screenshots and context; no persistent athlete state; guardrails (load caps, back-to-back intensity) applied only conversationally and not enforced; passive, user-initiated motivation. **Recommendations:** persistent athlete profile, *enforced* weekly load caps and back-to-back intensity flags, **pair each instruction with a brief rationale and the expected sensations**, one cue at a time, proactive check-ins, replacement workouts after a missed session. | https://arxiv.org/html/2509.26593v1 | arXiv, 2025-09 | **preprint**, n=1; full read (via fetch summary) | low (n=1, auto-ethnographic) | ai-evaluation |
| 2.6 | GPT-3.5 answers to general exercise FAQs scored higher than personal trainers' answers on correctness, comprehensibility and actionability (blinded raters, 6/9 questions). The answers were longer (241 vs 140 words). This study did not test plans or safety. | https://www.jssm.org/jssm-25-235.xml-Fulltext | J Sports Sci Med 25:235–261, 2026-02-01 | peer-reviewed; full read (fetch summary) | medium (exploratory, 9 questions) | ai-evaluation |
| 2.7 | Same lab (Düking/Wirth/Sperlich group), outside running: GPT-4 produced better hypertrophy plans than Gemini, and quality improved with input detail. The pattern repeats within the same lab. | termedia.pl Biology of Sport (snippet only) | Biol Sport 2025 | peer-reviewed; snippet | low–medium | ai-evaluation |

### Q3: Feedback format, explanation, trust

| # | Claim | Source | Publisher / date | Status / read | Confidence | Class |
|---|---|---|---|---|---|---|
| 3.1 | Feedback added to self-monitoring PA interventions: **d = 0.29 (95% CI 0.16–0.43)**, corrected from the originally published 0.73 (SE/SD error). Pooled over 19 studies (n=3,261) in the review, but the PA effect came from only about half the studies. **Evidence on which feedback generation/presentation format is best was mixed.** Many studies had high risk of bias and short duration. | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10765525/ ; correction PMC10885474 | IJBNPA 21:3, 2024-01-04 (+ correction 2024) | peer-reviewed; snippet (via search summary) | medium | empirical |
| 3.2 | Bloom: 4-week randomized field study, N=54, GPT-4o coach vs a no-LLM app. **Both arms increased PA** (the share meeting 150 min/wk doubled), with **no clear LLM advantage on wearable-measured PA**. The LLM arm showed better mindsets/enjoyment/self-compassion and >5× more time in the app. Harm taxonomy: overexertion/bodily harm, body image/eating disorders, mental health, negative feedback, inaccurate/out-of-scope advice. Every response passed through a safety filter, tested on a 600-example benchmark. | https://arxiv.org/html/2510.05449v2 | arXiv v2 2026-03-03; accepted to CHI 2026 | preprint/conference; partial read (first 100k chars) | medium-low (small, exploratory) | ai-evaluation |
| 3.3 | Explanation format and trust: a hybrid explanation format raised trust/transparency/satisfaction compared with single-format explanations in a chronic-condition app, but users high in need for cognition preferred single-format. Separately, trust in AI PA recommendations varied with the recommendation and with the user's decision style (van Arum, CHI 2025). **No RCT found linking explanation type to PA adherence.** | ACM Trans Comput Healthc 2025, doi:10.1145/3772071 ; TU Delft repository (van Arum CHI 2025) | 2025 | peer-reviewed; snippet only | low | empirical |

## (b) Leads

- Rosenblat 2025 Supplementary Appendix: the per-covariate p-values (weekly training duration, polarization index) were not extractable. The PDF is at the fisiologiadelejercicio URL above; it needs a proper PDF text extractor.
- Rosenblat et al. 2019 (earlier POL vs THR meta-analysis, ref [31]) and Rosenblat's call for large-scale wearable/crowdsourced TID datasets. This is directly relevant to an Apple Watch app.
- Oliveira et al. 2024 Sports Med (PMC11329428): not read in full this round.
- Biology of Sport 2025 (Düking group): GPT-4 vs Gemini hypertrophy plans, plus "professional assessment of strength plans by GenAI". These are the closest multi-LLM comparisons found.
- Bloom (arXiv 2510.05449): the rest of the paper (results stats, safety-filter metrics) is unread past offset 100000.
- Press: Bloomberg 2026-04-10 "training for a marathon with an AI coach"; Daily Beast "chatbot almost ruined the race". These are anecdotes only and could seed failure-mode hypotheses.
- Said, arXiv 2508.07980 (PA recommender trust/explanation framework); Naliyatthaliyazchayil HealthRecSys'24 (explainable exercise app).

## (c) Not found

- **Petreça 2025 (IJSSC):** no match found. It may be a mis-citation; treat it as unverified.
- **Any 2025–2026 peer-reviewed head-to-head comparison of GPT-4o/5, Claude and Gemini for running/marathon plans:** none found. No peer-reviewed evaluation of Runna or other AI running coach apps was found either.
- **A newer (2025–2026) TID meta-analysis or a published critique of Rosenblat 2025:** not found in this budget.
- **TID evidence specific to low volume (<5 h/week):** no subgroup result exists in Rosenblat. Weekly duration was a covariate but was not reported as significant (extraction incomplete).
- Stöggl & Sperlich 2014 and Seiler 2010 were not re-fetched (canon, not re-verified this run). Stöggl appears in Rosenblat's dataset as a competitive "combined" study.
- SDT applied to wearables: no source retrieved this round.

## Implication notes for the decision (inference, not evidence)

- A defensible default rule for recreational runners is **pyramidal**: the subgroup trend points that way, and both POL and PYR beat nothing in particular. The one-line explanation should state that POL and PYR perform about the same on average. A PYR→POL switch for the 8 weeks before a race is supported only in well-trained runners (Filipas).
- Hard guardrails should be enforced in code, not left to the LLM: progression caps, back-to-back intensity checks, and health screening. These map directly to the documented errors (2.3, 2.5, 3.2).
