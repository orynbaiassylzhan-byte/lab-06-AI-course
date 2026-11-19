# Scholarship screening audit

Orynbai Asylzhan · Narxoz University · Lab 06

## 1. Measurement read-out

I ran Qwen2.5-3B-Instruct on a Colab T4. With seed 2026, the experiment generated 200 matched pairs, 40 per comparison. Both profiles in each pair had the same scholarship qualifications. All 400 baseline replies parsed successfully. The agent shortlisted 356 profiles and rejected 44.

| Attribute changed | Decision flips / pairs | Flip rate (95% CI) | Pairs with different scores | Mean absolute score gap, points (95% CI) |
| --- | ---: | --- | ---: | --- |
| Name language | 2/40 | 5.0% (1.4–16.5%) | 11/40 | 3.12 (1.38–5.25) |
| Gender coding of name | 4/40 | 10.0% (4.0–23.1%) | 6/40 | 1.12 (0.25–2.13) |
| Name floor: another name with the same language and gender coding | 0/40 | 0.0% (0.0–8.8%) | 7/40 | 2.00 (0.62–3.62) |
| University tier | 1/40 | 2.5% (0.4–12.9%) | 10/40 | 2.50 (1.12–4.12) |
| University floor: another university of the same tier | 1/40 | 2.5% (0.4–12.9%) | 9/40 | 2.12 (0.88–3.62) |

None of the three flip rates clearly exceeded its corresponding floor under the notebook’s descriptive interval comparison. The intervals for absolute score gaps also overlapped their floors. The floors measure sensitivity to these particular substitutions under greedy decoding. Other changes, such as reordering prompt lines, require a separate test. With 40 pairs and no flips in the opposite direction, the two-sided exact McNemar test needs at least six flips in one direction to reach the 0.05 level.

**Direction matters for scores.** The table below orients pairs so that a positive gap means the first group received the higher score. The language comparisons use only pairs involving the named language, so their counts overlap.

| First group versus comparison group | Pairs | Only first shortlisted | Only comparison shortlisted | Exact McNemar p | Mean signed score gap (95% CI), points |
| --- | ---: | ---: | ---: | ---: | --- |
| Kazakh-coded name vs other name languages | 28 | 2 | 0 | 0.500 | +0.7 (−1.8 to +3.8) |
| Russian-coded name vs other name languages | 26 | 0 | 1 | 1.000 | +0.2 (−1.7 to +2.1) |
| English-coded name vs other name languages | 26 | 0 | 1 | 1.000 | −1.0 (−4.2 to +1.9) |
| Male-coded vs female-coded name | 40 | 0 | 4 | 0.125 | −0.4 (−1.2 to +0.6) |
| Flagship vs regional university | 40 | 1 | 0 | 1.000 | +2.5 (+1.1 to +4.1) |

My prediction about university tier was partly borne out. The observed average absolute score gap was larger across tiers than within a tier (2.50 versus 2.12 points), although the intervals overlapped. The direction of the cross-tier differences was more consistent: flagship profiles averaged 2.5 points higher, with a paired score sign-test p-value of 0.002. This is evidence about score direction in these pairs. The shortlist flipped once in each university comparison.

**After applying the controls**, the model saw identical text for the two members of every pair. All five groups had 0/40 flips (95% CI 0.0–8.8%) and zero within-pair score differences. This equality follows from blinding. The comparison of individual applicants across runs shows the cost of changing the input.

| Outcome across 400 measured profiles | Bare agent | With controls |
| --- | ---: | ---: |
| Shortlisted | 356 | 276 |
| Rejected | 44 | 124 |
| Sent for human review | 0 | 0 |
| Processing refused | 0 | 0 |
| Model calls | 400 | 400 |
| Prompt / generated tokens | 84,409 / 11,122 | 72,322 / 11,256 |
| Model time, seconds | 465.4 | 468.6 |

Of 82 changed outcomes, 81 moved from shortlist to rejection and one moved in the opposite direction. Scores changed for 175 of 400 profiles, with a mean absolute shift of 14.54 points among those 175. Removing fields also changed the wording and layout of the prompt, so this comparison cannot isolate which removed field caused a given change. The review band of exactly 45 points covered 34/400 baseline scores (8.5%). After blinding, zero profiles met any escalation condition.

In the separate six-request demonstration, the bare agent screened all six. The controlled agent shortlisted the ordinary request and refused five before calling the model: an ethnicity-ranking request, a minor, missing consent, impossible input values and an instruction in the name field. The refusal rules blocked 0/400 clean profiles. The log contains 406 events in total: 277 shortlists, 124 rejections and five refusals. The extra shortlist comes from the ordinary demonstration request.

## 2. Who is affected

Applicants bear the immediate cost of a changed recommendation. In four gender-coded pairs, only the female-coded profile reached the shortlist. The observed shortlist rates were 90% for female-coded names and 80% for male-coded names. These synthetic name cues do not identify anyone’s actual gender. The small sample and the name-floor interval limit how confidently this difference can be generalized. In the university pairs, only one regional applicant lost a shortlist relative to the matched flagship applicant, although the score difference could matter more if a fund later ranked applicants by score.

The largest change affected applicants across configurations: 81 previously shortlisted profiles received a rejection once fields were hidden. A real applicant could lose consideration for funding before a reviewer saw their application. The fund would also face a different shortlist and would need to explain how its process changed. Reviewers received zero cases despite a baseline projection of 34. That empty queue gives the fund little opportunity to notice mistaken rejections. The experiment includes no independently verified award decisions, so it cannot establish which recommendations were correct.

## 3. Professional responsibility

As the developer, I am responsible for what the screening pipeline does when the model gives a score, fails to explain itself or receives a request outside its purpose. In this run the log made the five refusals distinguishable from merit-based recommendations. It also exposed a weakness that a check of within-pair equality alone would miss: the shortlist shrank substantially while every rejection bypassed human review.

Before using this design with real applicants, I would ask scholarship officers to assess a separate sample using published criteria, including cases rejected outside the review band. I would use those assessments to test the threshold on blinded inputs and measure missed candidates alongside reviewer workload. The fund would need to assign a reviewer, provide an applicant correction route and record the eventual human decision. Until those checks exist, this run supports evaluation on synthetic applications only.
