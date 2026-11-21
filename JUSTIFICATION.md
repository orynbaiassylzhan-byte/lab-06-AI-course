# Write the design justification alongside the audit artifacts.
JUSTIFICATION = """# Justification for the Tulpar Scholarship Fund harness

Orynbai Asylzhan · Narxoz University

## 1. What the system may do

Tulpar is a fictional Kazakhstan scholarship fund used in this lab scenario. The
prototype processes synthetic applications to assist initial screening. It may produce
a provisional `shortlist` or `reject` recommendation only for a valid, consented
adult application submitted for `scholarship_screening`. Any uncertain, unparseable,
or boundary case is routed to a human reviewer (`escalate`), and final award
decisions remain outside this automated pipeline.

For the assumed deployment jurisdiction in Kazakhstan, I use the Republic of Kazakhstan
Law On Artificial Intelligence, No. 230-VIII of 17 November 2025, as the primary legal
benchmark, alongside the UNESCO Recommendation on the Ethics of Artificial Intelligence
(2021), Section III.2, for ethical principles. [Primary text on Adilet](https://adilet.zan.kz/rus/docs/Z2500000230)

| Control | Ethical principle | Legal provision and implementation |
| --- | --- | --- |
| Validate input | Safety and security | RK AI Law No. 230-VIII, Art. 11(1) (safety and reliability of AI systems). `validate_profile` blocks out-of-range numbers and wrong types (`invalid_input`) before inference. |
| Blind | Fairness and non-discrimination | RK AI Law No. 230-VIII, Art. 6 (fairness and equality; prevention of discrimination) and Art. 10(1) (personal data protection and confidentiality). `BLIND` strips `name`, `university`, `age`, and `consent_to_automated_processing` from the prompt rendered to the LLM. |
| Refuse | Do no harm and preserve human autonomy | RK AI Law No. 230-VIII, Arts. 6, 9, and 11(1). Built-in gates (`purpose_not_allowed`, `minor`, `no_consent`) and `my_extra_refusal` halt unauthorized or malformed requests prior to any model call. |
| Log | Responsibility and accountability | RK AI Law No. 230-VIII, Art. 8(1) (ongoing control/monitoring) and Art. 15(2)(3) (documentation duties of owners/holders). `decisions.jsonl` records every outcome, control fired, input hash, and raw model output. |
| Escalate | Human oversight and determination | RK AI Law No. 230-VIII, Arts. 7(2), 8(1), and 9 (right to information/objection, ongoing control, and human autonomy). Unparseable JSON, missing rationales, and scores inside `BORDERLINE` route the case to human review. |

## 2. What it must refuse

The harness enforces hard pre-inference refusal boundaries so the language model never
scores out-of-scope or unlawful requests:
- **Purpose limitation (`purpose_not_allowed`):** Any request whose `purpose` is not
  `scholarship_screening` (such as `rank_by_ethnicity`) is rejected immediately.
- **Age threshold (`minor`):** Applicants under 18 are refused automated screening
  and must be handled through a separate manual intake process.
- **Consent requirement (`no_consent`):** Applications where
  `consent_to_automated_processing` is not `True` are refused without calling the model.
- **Schema and range integrity (`invalid_input`):** Impossible values (e.g., GPA 9.9,
  negative exam scores) or non-boolean consent flags are blocked at validation.
- **Vocabulary and injection boundary (`*_outside_lab_catalog`):** `my_extra_refusal`
  checks `name`, `university`, and `program` against the synthetic reference catalog,
  blocking prompt-injection strings placed in the name field while producing zero false
  refusals across all 400 clean synthetic profiles. In a real deployment, closed name
  lists would unfairly exclude valid applicants and would be replaced by structured
  character/length validation and prompt-injection delimiters.

## 3. What record it keeps

Every call to `screen` appends one JSON object to `decisions.jsonl` containing `record_id`,
`pid`, `input_sha256`, `fields_shown`, `fields_withheld`, `model_id`, `prompt_version`,
`harness` settings, `raw_output`, parsed `decision` and `score`, `controls_fired`,
`outcome`, `reason_code`, `escalated_to_human`, and UTC `timestamp`.

Storing field names (`fields_shown` / `fields_withheld`) and SHA-256 digests rather than
full applicant profiles prevents the audit log from becoming a duplicate personal data
store while still allowing an auditor with the original record to verify exact prompt
inputs and reconstruct why a control fired. Refusals log a profile fingerprint and
`[no model call]`, proving the model was never invoked. Escalated records populate
`REVIEW_INBOX` so a human reviewer can retrieve the application by `pid` and record an
independent final decision.

## 4. Trade-offs

- **Reviewer workload vs. error capture:** `BORDERLINE` is calibrated around the
  observed baseline overlap between `reject` and `shortlist` scores, targeting up to
  20% of scored baseline profiles. Widening the band catches more uncertain calls but
  increases human review queue latency; tightening it saves reviewer hours at the cost
  of higher unreviewed error risk.
- **Threshold shift under blinding:** Removing `name`, `university`, `age`, and
  `consent_to_automated_processing` changes prompt length and structure, shifting the
  score distribution compared to the unblinded baseline. Consequently, the actual share
  escalated in Part 3 differs from the baseline estimate and must be monitored after
  any prompt or blinding change.
- **Blinding vs. counterfactual checking:** Blinding eliminates direct leakage of
  protected cues at zero extra inference cost (1 model call per profile), whereas
  `counterfactual_check` doubles model calls and latency. Blinding cannot prevent
  indirect proxy correlations through remaining fields (`program`, `gpa`, `income`),
  which requires ongoing paired audits.

## AI-use declaration

I used OpenAI ChatGPT (Codex) and Google Gemini during this assignment to analyze the
repository structure (`lab06_data.py` and `lab06_core.py`), debug the harness hooks
(`my_extra_refusal`, `my_extra_escalation`, and `BORDERLINE` calibration), structure
the automated report and justification builders, and verify formatting checks against
`check_register`.

All screening measurements and decision logs (`decisions.jsonl`) were generated
deterministically inside Google Colab using the course-supplied `Qwen/Qwen2.5-3B-Instruct`
model with greedy decoding (`do_sample=False`), without synthetic or hand-edited log
entries. In accordance with the course policy on legal citations, I personally
consulted the primary text of the Republic of Kazakhstan Law On Artificial Intelligence
(No. 230-VIII of 17 November 2025) on Adilet (`adilet.zan.kz/rus/docs/Z2500000230`) and
the UNESCO Recommendation on the Ethics of Artificial Intelligence (2021) to verify
every article, section number, and substantive claim mapped in `CONTROL_REGISTER` and
this document.
"""
Path("JUSTIFICATION.md").write_text(JUSTIFICATION, encoding="utf-8")
print("Wrote JUSTIFICATION.md")
