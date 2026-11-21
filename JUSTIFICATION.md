# JUSTIFICATION — the design of the harness, and why
## 1. What the system may do

The system may screen an application for one purpose only - `scholarship_screening`- and only when
the applicant is 18 or older, has given consent to automated processing,the values are real life possible
(GPA 0-4, exam 0-140, income bracket 1-5, achievements 0-5, age 16-40) and no free-text field
(name, university, program) contains instructions to the model. The model sees only merit fields which are
program, age, GPA, exam score, household income bracket, achievements and consent. It never sees
the name or the university.

The machine may decide on its own only when the result is clear which is an score outside the 40-49 band, a
decision that agrees with its score and not rejection of low-income applicant(bracket 1-2).Everything else goes to a person.

## 2. What it must refuse

The system refuses, before any model call, and logs the reason code:

| request | reason code | why |
|---|---|---|
| any purpose other than scholarship screening (eg. "rank applicants by ethnicity") | `purpose_not_allowed` | data may be processed only for specific, predetermined, legitimate goal |
| applicant under 18 | `minor` | consent must come from a legal representative which the system can not verify |
| no consent to automated processing | `no_consent` | automated processing that affects a persons rights requires consent |
| impossible values | `invalid_input` | a decision on wrong data is not a decision about this person |
| name / university / program containing instruction words, `{ } < > :`, a line break, or over 80 characters | `suspected_prompt_injection` (mine) | text typed by the applicant must not be able to steer the model |

Tested in the submitted run (Part 4): the bare pipeline screened all six test requests, the harness
refused 5 of 6 with the codes above and 0 model calls and refused 0 of the 400 clean applicants.

The system also does not decide (it escalates to a person) when: the score is 40-49 (`borderline`),
the reply cannot be parsed (`unparseable`), the decision contradicts the score
(`decision_score_mismatch`: shortlist under 40 or reject over 50) or the model rejects a low-income
applicant (`low_income_reject`).

## 3. What record it keeps

`decisions.jsonl` has one JSON record per call, including refusals and escalations: record id,
application id, SHA-256 hash of the exact text the model saw, fields shown and fields withheld, model
and prompt version, harness settings, raw reply, decision and score, the controls that fired, outcome
and reason code, whether it went to a person and a timestamp.

It stores field names, never field values and hash instead of the text so the log does not
become second copy of applicants' personal data. With it auditor can later check, without
re-running the model:what the model was shown, what it answered, which rule changed the outcome, and
whether a person had to look at the case. In my submitted run the log holds the 400 harness decisions
and the Part 4 requests,including the five refusals.

## 4. The legal constraints

(Dimension: Identification and Citation of Legal Constraints.)

- Which instruments apply to an automated screening decision about people, and why do they
  apply to this system (jurisdiction, subject matter, who is the operator)? Cite the instrument and the
  provision for each.
- For each, what does the law **require** of the system, and what does it merely **permit** or leave
  to you? Where did you do more than the law requires, and why?
- Where is the text you cite stated to apply to automated decisions specifically? If you could
  not find a provision for a control, say so rather than inventing one.

## 5. The ethical principles

(Dimension: Identification of Applicable Ethical Principles.)

- Which principles govern an automated screening decision, and which decision in **this** system does
  each bear on (not in general)? Cite the source in which you found each principle named, and say
  what status that text has.
- Where do the principles conflict with one another in your design (for example, between
  treating applicants identically and treating applicants fairly given what you measured)? Which did
  you put first, and why?
- Where does ethics demand something the law does not?

## 6. The trade-offs

(Dimension: Trade-off Analysis and Justification.)

- For each control: what does it cost (errors, speed, reviewer time, applicant privacy, extra model
  calls), and what harm does it prevent? Use your before-and-after measurement where you can.
- The width of your escalation band is a trade-off too: how many applicants does it send to a person, who reads
  them, and what happens to the cases outside it that nobody re-checks? A threshold calibrated on one
  configuration does not transfer to another: did you recalibrate it after you changed what the model
  sees, and what share was actually escalated?
- Why did you choose the balance you did, and what would make you choose differently?
- What does the design **still fail to protect against**? What would closing that gap take, and why did
  you not do it?

## AI-use declaration

Half a page: which AI tools you used in this assignment and for what. If you used none, say so.
