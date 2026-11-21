# REPORT — what the measurement showed, who it affects, and whose responsibility it is

All numbers below were printed by my run of `lab06_harness_says_no.ipynb` (Colab, T4 GPU, `Qwen/Qwen2.5-3B-Instruct`, greedy decoding, `N_PAIRS = 40`, `SEED = 2026`). The bare run is 200 pairs = 400 profiles = 400 model calls.

## 1. Measurement read-out

**Basic facts of the bare run.** 0 of 400 replies were unparseable. 356 of 400 profiles were shortlisted (89%). Rejected profiles scored 20–45, shortlisted profiles 45–85.

**Flips, read against the floors** (40 pairs each, flip rate with 95% interval):

| attribute | flips | rate [95% CI] | its floor | floor flips | intervals overlap? |
|---|---|---|---|---|---|
| name_language | 2/40 | 5.0% [1.4%, 16.5%] | null_name | 0/40 (0.0% [0.0%, 8.8%]) | yes |
| gender_coded_name | 4/40 | 10.0% [4.0%, 23.1%] | null_name | 0/40 | yes |
| university | 1/40 | 2.5% [0.4%, 12.9%] | null_university | 1/40 (2.5% [0.4%, 12.9%]) | yes |

**No attribute changed the decision clearly beyond its own floor.** So I cannot say that any of them moved the decision in this run.

- *Gender-coded name* is the closest to a signal: all 4 flips went the same way (the female-coded twin was shortlisted, the male-coded one rejected). Shortlist rates were 90.0% (female, n = 40) vs 80.0% (male, n = 40). But the exact McNemar p = 0.125, the interval overlaps its floor, and the mean absolute score difference (1.12 [0.25, 2.13]) is actually *below* the null_name floor (2.00 [0.62, 3.62]). This is something to re-test with more pairs, not a finding.
- *Name language*: 2/40 flips, 11/40 pairs with different scores, mean |diff| 3.12 [1.38, 5.25] vs floor 2.00 [0.62, 3.62] — overlapping. Signed gaps: Kazakh vs others +0.7 [-1.8, +3.8], Russian +0.2 [-1.7, +2.1], English -1.0 [-4.2, +1.9]; all include zero.
- *University*: only 1/40 flips, the same as its floor. **But the scores move in one direction:** flagship minus regional = **+2.5 points [+1.1, +4.1], sign test p = 0.002**, 10/40 pairs with different scores. Mean |diff| 2.50 [1.12, 4.12] vs its floor 2.12 [0.88, 3.62]. So the model gives regional-university applicants slightly lower scores. The shift is small and turned into a different decision only once in 40 pairs, but it is systematic, and it matters near the cut-off (see section 2).

**What the floors test and what they don't.** `null_name` changes which name of the *same* language group and gender coding is shown; `null_university` swaps a university for another of the same tier. Greedy decoding has no run-to-run noise, so a floor measures how much the output moves when *these* irrelevant details change. They do not test other wordings of the prompt, other field orders, other models, or real applicants. I also ran the optional order probe (40 pairs of *identical* applicants, the second shown with its lines in another order): **10/40 flips = 25.0% [14.2%, 40.2%], all 10 in the same direction (shortlisted in the default order, not in the other), exact McNemar p = 0.002, mean score -6.6 [-9.9, -3.5]**. So changing only the layout of the prompt moved more decisions than any protected attribute did. This is the most important limit of my floors: the agent is fragile to formatting, and a different prompt layout could produce bigger (or different) attribute effects than the ones I measured.

**What n = 40 can detect.** The notebook printed that an exact two-sided McNemar test at 0.05 needs at least 6 one-sided flips out of 40 (15%) to call a result significant. A real effect smaller than that (for example the 4/40 gender flips) cannot be told from chance here. For scores, the half-widths were about ±0.9 (gender) to ±3.1 (English-name) points, so shifts smaller than that cannot be told from zero. **"No measured effect" is not "no effect"**: it means that if there is an effect, it moves fewer than ~15% of decisions in this setup.

**Predictions against results.** *P0:* predicted 0–5 unparseable replies, got 0 of 400 — matched. *P1:* predicted floors of 2–4/40 and a university effect of 5–8 flips; got floors 0/40 and 1/40 and university 1/40 flips — wrong on flips; the university effect showed up as a score shift (+2.5) instead, and the gender row, which I expected to be flat, had the most flips (4/40). *Prediction 2* (which controls would reduce the divergence) **was not written down before I built the harness**, so I do not claim one. *P3:* predicted 15–25% escalated; the band calibrated on the bare run caught only 8 of 400 harness scores, because blinding moved the scores; predicted 5–15% of outcomes changed; the submitted run prints 67 of 400 changed outcomes, most of them rejects that rule 4 turned into escalations, so this is not comparable with my prediction, which assumed no such rule; but 166 of 400 scores changed (mean |delta| 14.01), far more than I expected. The order probe explains why: the model reacts to the form of the prompt (−6.6 points from reordering lines), and removing two lines changes the form. The addendum to P3 (written before the final run, after seeing earlier runs) expected 62 of 400 escalated with rule 4; the run gave 62. *P4:* all six outcomes as predicted.

**Does this generalise?** No claim beyond this run. With another prompt, another model, another GPU (the README says counts may differ slightly) or real applicants with messier data, the same attribute could behave differently. A stronger estimate would need several hundred pairs per attribute, several prompt wordings, and a repeated run on another machine.

## 2. Who is affected

- **Applicants from regional universities.** They score about 2.5 points lower than an identical flagship applicant. Because the cut-off is around 45 (rejects ≤ 45, shortlists ≥ 45), a 2.5-point drop can push a borderline regional applicant from shortlist to reject. In my run this happened in 1 pair of 40. The person who bears it loses a scholarship they qualified for: money for tuition or living costs, maybe the ability to study at all, and the time spent applying. Regional applicants are *possibly* also more often from lower-income families (my pairs keep income identical, so my data cannot show this), in which case the cost would fall on people who need the money most.
- **Male-coded applicants (possible, not shown).** 4/40 flips all went against the male-coded twin. Not significant, but if real, men with otherwise identical profiles would be rejected more often.
- **Wrongly shortlisted applicants and who they displace.** Places are limited. If one twin is shortlisted because of a flagship name or a name coding, somebody else with the same merit is pushed out. The wrongly shortlisted person gains, but the cost moves to a displaced applicant who never learns why. A wrong rejection costs the applicant much more than a wrong shortlist (a shortlist still goes to later review).
- **People not in the brief:** students who do not apply because they expect a regional university to count against them; the fund staff who must read escalated cases (in the submitted run 62 of 400 applicants, 15.5%, were sent to a person); the fund itself (reputation, legal risk); applicants from universities the model has never heard of (in real data the shift could be bigger than with fictional names).
- **People affected by the harness itself.** Blinding changed outcomes for real profiles: 67 of 400 outcomes changed versus the bare run (reject → escalate 39, shortlist → escalate 23, reject → shortlist 5). The 5 reject → shortlist profiles gained a place they did not get with the name and university visible; the 23 shortlist → escalate profiles now wait for a human instead of passing automatically. With rule 4, every rejection (including the 24 profiles the model scored 20) now reaches a person instead of the applicant directly; before rule 4, those rejections went out with nobody reading them. The reviewers carry this: 62 of 400 cases (15.5%).
- **If there is an effect my 40 pairs could not see** (anything below ~6/40 one-sided flips), the same groups would bear it silently, because nobody would look.
- **Compounding.** Used year after year, a 2.5-point shift every year adds up to fewer regional winners, which can then be read as "regional students are weaker" and fed into the next stage (later reviews, other funds, a published list of winners by university).

## 3. Professional responsibility

**My responsibilities as the developer of the harness:**

1. *Check before it runs.* Re-run the matched-pair measurement whenever the model, the prompt or the blinded fields change. In this lab I did it once before and once after the harness. The band calibrated on the bare run did not transfer: (45, 50) would catch only 8 of 400 harness scores, so I recalibrated to (40, 50), and the submitted run escalated 62 of 400 (15.5%) including rule 4. I also found and fixed a bug in my own escalation rule (substring matching: the first version would fire on 188 of 400 replies of this run).
2. *Tell the people who deploy it what I measured and its limits:* no attribute cleared its floor, a small consistent university score shift exists, 40 pairs cannot detect effects under ~15%, the floors do not cover prompt wording.
3. *Not claim* that the system is "fair" or "unbiased". Blinding makes the after-harness flips zero by construction for the blinded fields; that is not proof the model became fair. Also not claim that the harness stopped injection in general: it stopped the one injection I tested.
4. *Make it auditable:* every decision, escalation and refusal goes into `decisions.jsonl` with the input hash and the fields shown/withheld.

What is left undone: a bigger sample, a test across prompt wordings, a check of the escalation workload with real reviewers, and a test of my refusal rule on real application data.

**Responsibilities of the institution (the fund):** setting the purpose (only scholarship screening), deciding who reviews escalated cases and how fast, informing applicants that AI is used and how to object, handling appeals, and deciding whether the system is allowed to make final rejections at all.

**Where we could conflict:** if the fund wants to remove escalation because reviewers cost money, I can show that the band around the reject/shortlist boundary (40–50 after blinding) is exactly where a few points, the size of the 2.5-point university shift, decide the outcome, and that it costs 62 reviews per 400 applicants. If they still remove it, I should document the risk in writing and not sign off the system as safe. If they want to show the university again "because it predicts success", that has to be an explicit, documented rule decided by people, not something left to the model.
