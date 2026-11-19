# Justification for the Tulpar Scholarship Fund harness

Orynbai Asylzhan · Narxoz University

## 1. What the system may do

The fictional Tulpar Scholarship Fund uses this prototype to screen synthetic applications. For an adult who consents to automated screening, it may issue a shortlist or rejection recommendation. A scholarship officer would make the award decision. The architecture separates eligibility checks from scoring. The harness reads age and consent before the model is called. The model then sees only programme, GPA, entrance exam score, household income bracket and achievements. A valid application outside the review band receives the model’s recommendation automatically unless another escalation rule fires.

## 2. What it must refuse

A request outside scholarship screening receives `purpose_not_allowed`.
A valid applicant could experience delay if a client sends the wrong purpose.
An age below 18 receives `minor`. This lab gate means that a qualified
17-year-old needs a separate intake route. Absent permission receives
`no_consent`. A missing or mistaken consent flag can interrupt a valid
application. The age threshold and Boolean consent check are prototype rules.
Invalid types and ranges receive `invalid_input`, which calls for correction of
the source record. A name, university or programme outside the synthetic
catalogue receives a field-specific code. A typo or newly eligible programme
could also trigger this gate. Applying a closed name catalogue to real people
would exclude legitimate names.

Part 4 checks the outcome, reason code and zero model calls for all five
blocked cases. It also checks an ordinary request and counts wrong refusals
among the generated clean applications. A refusal is logged and returned to
the caller. A real fund would need a documented correction route and an
independently staffed manual intake for minors, objections and applicants who
decline automation. The notebook has no applicant notification service.

In Part 4 the bare agent screened all six requests. The harness allowed the ordinary application and blocked the other five with zero model calls on each. Of 400 clean profiles, zero triggered a refusal. That result tests the catalogue against the same synthetic population used to define it. It gives little assurance about a real applicant whose name is missing from a database.

## 3. What record it keeps

**Audit question.** Why did the same application ID receive a shortlist recommendation in one interaction and a refusal in another?

| Evidence from the actual log | What a reviewer can establish |
| --- | --- |
| Record `Re35821b5b3cb`, application `P0124a` | Qwen2.5-3B-Instruct received five permitted fields under prompt v1 and returned a shortlist with score 60. The raw reply mentions GPA, exam score and household income. The controls recorded were blinding and logging. |
| Record `R79bb4662b485`, application `P0124a` | The requested purpose was `rank_by_ethnicity`. The harness returned `purpose_not_allowed` before inference. The shown-fields list was empty and both decision and score were null. |

Each row preserves the event ID, application ID, UTC time, model and prompt version, harness configuration, fields shown and withheld, input fingerprint, raw reply or refusal note, parsed result, final outcome, reason code and human-review flag. A reviewer can reconstruct the routing from those fields. Checking the input hash requires a separately held copy of the input. A model-generated reason records what the model said and can itself contain errors or repeat applicant data.

The submitted log contains 406 events. The 400 measured profiles produced 276 shortlists and 124 rejections. The six demonstration requests added one shortlist and five refusals. This explains why the full log contains 277 shortlists. Every event was logged. A new measured run starts a fresh log. The notebook imposes no retention deadline or role-based access. Anyone with write access to the file can edit it, and an input hash cannot reveal an edit to the recorded outcome. A real fund would need restricted reviewer access, a retention period tied to its appeal process and a separate, tamper-evident history of reviewer actions.

## 4. The legal constraints

I assume that a Kazakhstan scholarship fund owns and operates the proposed screening system. I use the Republic of Kazakhstan’s *Law On Artificial Intelligence*, No. 230-VIII of 17 November 2025, as a design benchmark for that scenario. Article 7(2) specifically addresses people affected by decisions using AI, including information about automated processing, its consequences and the possibility of objection. Articles 8(1) and 15(2) address oversight and the responsibilities of system owners or holders. All applicants in the experiment are fictional. [Official text, Articles 7, 8 and 15][law]

| Implemented control | Provision in the Kazakhstan AI Law | Design decision |
| --- | --- | --- |
| Input validation | [Art. 11(1)][law] addresses safety and reliability. | `validate_profile` checks types and ranges. A GPA of 9.9 stops processing before the model can interpret it. |
| Blinding | [Arts. 6 and 10(1)][law] address equality, discrimination and data protection. | `BLIND` removes name and university from the prompt and keeps age and consent at intake. |
| Refusal | [Arts. 6, 9 and 11(1)][law] address fairness, autonomy and protection against misuse. | `refusal_reason` blocks prohibited purposes, minors and lack of consent. `my_extra_refusal` enforces the lab catalogue. The age threshold and catalogue are lab policies. |
| Decision log | [Arts. 8(1) and 15(2)(3)][law] concern oversight and documentation appropriate to impact. | `screen` writes an audit event for every controlled outcome. JSONL and its individual fields are implementation choices. |
| Human review | [Arts. 7(2), 8(1) and 9][law] support informed objection, human oversight and autonomy. | Unparseable output, a missing explanation or a borderline score routes the case to the review inbox. A working objection channel would also require an applicant-facing service. |

The hardest design choice to justify from a legal provision is the numerical review threshold. Article 8(1) supports ongoing oversight. The fund must still justify a specific threshold using evidence from its own screening process. [Art. 8(1)][law]

## 5. The ethical principles

UNESCO’s 2021 *Recommendation on the Ethics of Artificial Intelligence* supplies ethical guidance. Safety and security support validating inputs. Proportionality and prevention of harm support refusing a request to rank people by ethnicity. Fairness and non-discrimination support hiding the name and university. Responsibility and accountability support recording each route. Human oversight and determination support sending uncertain cases to a reviewer who can exercise independent judgment. [UNESCO, section III.2, principles][ethics]

Privacy and auditability pull in different directions. Keeping the model’s reply lets an auditor investigate a questionable explanation but can retain details of the applicant’s circumstances. I accepted that exposure for these synthetic records and omitted a separate full profile from the log. For access to a real scholarship, fairness would also require testing who loses an opportunity when the prompt changes. Equal treatment within a matched pair alone cannot answer that question.

## 6. The trade-offs

**Blinding changed who reached the shortlist.** In the baseline, changing the name language flipped 2/40 decisions, changing gender coding flipped 4/40, and changing university tier flipped 1/40. The corresponding floors were 0/40 for names and 1/40 for universities. The 95% flip-rate intervals overlapped their respective floors. Flagship universities nevertheless showed a directional score advantage of 2.5 points (95% interval +1.1 to +4.1). A score difference can matter even when both applicants receive the same shortlist decision.

After blinding, both members of each pair received identical prompts. All five groups had zero flips. Across the two runs, however, 81 profiles moved from shortlist to reject and one moved from reject to shortlist. Scores changed on 175 of 400 profiles, by an average of 14.54 points among those that changed. Removing fields changed the form and content of the prompt together. With no independently assessed merit labels, the experiment cannot say which of the 82 changed recommendations was more accurate.

**The review threshold did not transfer.** A score of 45 appeared among both baseline shortlists and rejections. The selected band of 45–45 covered 34/400 baseline profiles (8.5%). After blinding, zero of 400 reached the human-review inbox. The band was calibrated once on baseline scores and held fixed for the comparison. Sending zero cases saved reviewer time, but left all 124 controlled rejections without an independent check. Before a real pilot, I would recalibrate on separate blinded cases with human assessments and test a sample of rejections outside the band. Until then I would use the system only for evaluation.

**Compute cost.** Each configuration used 400 model calls. Prompt tokens fell from 84,409 to 72,322. Generation took 465.4 seconds before controls and 468.6 seconds after them. The catalogue and range checks prevented five invalid uses in the demonstration at the cost of requiring correction if a legitimate entry falls outside the accepted values. The log makes such events visible. A staffed correction and appeal process remains necessary to act on them.

[law]: https://adilet.zan.kz/rus/docs/Z2500000230
[ethics]: https://www.unesco.org/en/legal-affairs/recommendation-ethics-artificial-intelligence

## AI-use declaration

I used ChatGPT (Codex) to help me work with the supplied notebook. It drafted the prediction text, proposed the fields to hide, helped implement the refusal and escalation rules, and prepared code that writes the control register and builds the report from the run. It also helped locate the Kazakhstan AI law and the UNESCO Recommendation, suggested links between their provisions and the controls, and drafted and revised this justification. Its earlier implementation checks used scripted replies to test control logic.

I ran the notebook in Google Colab on a T4 GPU and supplied the outputs and sample audit records used here. The course model, Qwen2.5-3B-Instruct, generated the screening responses for the synthetic profiles. The before-and-after figures and refusal counts in this document come from that Colab run. The prediction cells contain AI-assisted hypotheses, and the observations describe measured outcomes.

I also used ChatGPT to interpret why the planned and actual review workloads differed and why removing fields could change individual recommendations. The legal and ethical references point to the published instruments themselves. I am responsible for checking their wording and applicability, explaining the limits of this experiment and accurately disclosing the help I used.
