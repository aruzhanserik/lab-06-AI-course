# REPORT — what the measurement showed, who it affects, and whose responsibility it is
## 1. Measurement read-out

The agent screened 200 matched pairs (400 profiles, 400 model calls): 40 pairs for each of three
attributes that must not matter (language of the name, gender coding of the name, university tier)
and 40 pairs for each of two null floors (`null_name`: another name of the same language and gender
coding; `null_university`: another university of the same tier). Inside a pair every merit field is
identical.

General behaviour of the bare agent: 0 of 400 replies were unparseable (I predicted 5-10), and the
agent is very lenient: 356 of 400 profiles (89%) were shortlisted. Rejected profiles scored 20-45 and
shortlisted profiles 45-85; both outcomes occur only in the 40-49 band.

### Flips (decision changed inside a pair), read against the floors

| attribute | flips | flip rate [95% CI] | floor | floor flips | overlap |
|---|---|---|---|---|---|
| name_language | 2/40 | 5.0% [1.4, 16.5] | null_name | 0/40 [0.0, 8.8] | yes |
| gender_coded_name | 4/40 | 10.0% [4.0, 23.1] | null_name | 0/40 [0.0, 8.8] | yes |
| university | 1/40 | 2.5% [0.4, 12.9] | null_university | 1/40 [0.4, 12.9] | yes |

Direction: both name-language flips favoured the Kazakh-language name; all 4 gender flips favoured
the **female**-coded name (McNemar exact p = 0.125), which is the opposite of what I predicted; the
one university flip favoured the flagship university.

### Score differences

| attribute | pairs with different scores | mean \|b − a\| [95% CI] | floor: pairs / mean \|b − a\| |
|---|---|---|---|
| name_language | 11/40 | 3.12 [1.38, 5.25] | 7/40 / 2.00 [0.62, 3.62] |
| gender_coded_name | 6/40 | 1.12 [0.25, 2.13] | 7/40 / 2.00 [0.62, 3.62] |
| university | 10/40 | 2.50 [1.12, 4.12] | 9/40 / 2.12 [0.88, 3.62] |

Signed score gaps (first group minus the other): Kazakh name +0.7 [−1.8, +3.8]; Russian name +0.2
[−1.7, +2.1]; English name −1.0 [−4.2, +1.9]; male vs female −0.4 [−1.2, +0.6];
**flagship vs regional +2.5 [+1.1, +4.1], sign test p = 0.002**.

### What I conclude

- **No attribute moved the decision beyond its floor.** Every flip rate overlaps its floor's interval.
  With 40 pairs, an exact McNemar test needs at least 6 one-sided flips (15% of pairs) to be
  significant; the largest count I saw was 4 (gender). So a real effect smaller than about 15% of
  pairs could not be detected by this run. Absence of a measured effect is not proof of absence.
- **One score effect is clear:** applicants from a regional university were scored on average 2.5
  points lower than an otherwise identical applicant from a flagship university. The interval
  excludes zero and the sign test gives p = 0.002, which survives even a strict correction for the
  five comparisons (0.05 / 5 = 0.01). The mean absolute difference (2.50) is still inside the floor's
  range (2.12), so the size is small; it changed a decision in only 1 of 40 pairs.
- The 4 female-favouring gender flips are a pattern to watch, not a finding.

### What the floors do and do not cover

The floors measure how much the output moves when an irrelevant detail of the same kind changes
(another name of the same group, another university of the same tier). Because decoding is greedy,
there is no run-to-run noise. The floors are **not** a bound on all noise: a different prompt wording,
line order or another GPU could move the outputs more. I did not run the optional order probe. After
the harness, the model's scores moved by 14.01 points on average for 166 of 400 profiles just because
two lines were removed from the prompt, which shows how sensitive this model is to the form of its
input.

### After the harness

With name and university hidden, all flip rates and score differences fell to 0/40 — **by
construction**, because both members of a pair were shown identical text. This does not prove the
model became fairer. At profile level: outcomes went from 356 shortlist / 44 reject to 342 shortlist /
6 reject / 52 escalated / 0 refused; 57 outcomes changed (33 reject → escalate, 19 shortlist →
escalate, 5 reject → shortlist). 52 of 400 (13.0%) went to a person against the 16.0% planned. The
harness used the same 400 model calls (no extra cost in compute). In Part 4 the harness refused 5 of
6 bad requests that the bare pipeline screened, and wrongly refused 0 of 400 clean applicants.

## 2. Who is affected

- **Applicants from regional universities.** They are scored about 2.5 points lower for the same
  merit. Near the decision line (40-49) that can be the difference between shortlist and reject. They
  are often students from outside the large cities, who may already have fewer opportunities, so a
  wrong rejection hits the people a scholarship is meant to help.
- **Applicants whose name signals a gender or an ethnic/language group.** In this run the effect did
  not exceed the floor, but 4 of 40 gender pairs flipped, all in the same direction. With only 40
  pairs I cannot rule out a real bias of up to roughly 15% of pairs. If such a bias exists, a group
  would be shortlisted less often for reasons unrelated to merit.
- **Low-income applicants.** For them a wrong rejection means losing support they need most; they
  also have the least ability to challenge the decision. That is why a low-income rejection goes to a
  person in my harness.
- **Applicants near the decision line.** The model is unstable there: removing two lines from the
  prompt moved scores by 14 points on average for 166 profiles. Whoever lands in 40-49 is essentially
  decided by chance if the machine decides alone.
- **Minors, applicants without consent and applicants with wrong data.** The bare pipeline gave all of
  them a decision. They would be processed without a legal basis or judged on impossible values.
- **Human reviewers and the fund.** The harness sends 13% of applicants to a person. That is real
  workload, and reviewers who see too many cases start approving without reading.

## 3. Professional responsibility

The model is supplied, but the system that uses it is mine. I am responsible for:

- **Measuring before deploying.** I cannot claim the agent is fair because it "seems fine". I measured
  it with matched pairs and floors and I report what n = 40 could and could not detect.
- **Not over-claiming.** The "after" flips are zero by construction; I must not present that as proof
  that the harness fixed the model. The only measured effect (university, +2.5 points) is small.
- **Choosing what the model sees.** I decided to hide name and university, because neither is merit
  and the university was shown to shift scores.
- **Deciding what the system must refuse and who must see a case.** A pipeline that screens a minor or
  an ethnicity-ranking request without complaint is my failure, not the model's.
- **Keeping a record** so that every decision, refusal and escalation can be audited later, without
  storing a second copy of applicants' data.
- **Telling the fund the limits:** the band was calibrated on unblinded scores, the escalation share
  will drift if the prompt changes, and the model should be re-measured with more pairs (for example
  200 per attribute) before any real use.
