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

**Instrument:** Law of the Republic of Kazakhstan No. 94-V of 21 May 2013 "On Personal Data and their
Protection", as amended (including Law No. 231-VIII of 17.11.2025, which added Art. 19-1). Read in the
English unofficial translation from adilet.zan.kz.

**Why it applies:** jurisdiction — the fund is imagined as operating in Kazakhstan (Narxoz, KZ names
and universities); subject matter — the system collects and processes personal data (name, age,
grades, income) and makes automated decisions about people; operator — the fund would be the
owner/operator. Because the fund is fictional and the data synthetic, the law does not bind this
prototype; I use it as the standard the fund would have to meet.

| provision | what it requires | my control | required or more than required |
|---|---|---|---|
| Art. 7 para 1; Art. 8 para 1 | processing with the consent of the individual or legal representative | refuse: `no_consent`, `minor` | required |
| Art. 7 para 8 | processing only for specific, predetermined, legitimate goals; not to restrict a person's rights | refuse: `purpose_not_allowed` | required |
| Art. 7 para 9 | no processing of data excessive for the purpose | blind name and university | the law permits minimisation in general; hiding them from the model is my choice, based on Part 1 |
| Art. 19-1 paras 1-3 | automated processing that changes/terminates rights is prohibited without consent; must explain, allow objection, consider it in 3 working days | escalate; refuse `no_consent` | required; escalating low-income rejects before any objection is more than required |
| Art. 22 para 1 subpara 5 | registration and recording of actions | log | required |
| Art. 12 para 2 | storage within the Republic of Kazakhstan | log storage | required (not met by a Colab prototype; noted as a gap) |
| Art. 22 para 1 subparas 1-3 | protection measures, minimising adverse consequences | validate_input | indirect link only |

**Automated decisions specifically:** Art. 19-1 ("Requirements for automated processing of personal
data") is the provision that speaks to automated decisions directly. **No provision** requires input
validation or injection checks as such; I did not find one and do not claim one. The prompt-injection
refusal has no specific legal basis; it is an engineering and safety control.

## 5. The ethical principles

Source: **OECD Recommendation of the Council on Artificial Intelligence (OECD AI Principles),
OECD/LEGAL/0449, 2019, revised 2024.** Status: a recommendation adopted by OECD member and adherent
countries; not binding law.

| principle | which decision in this system it bears on |
|---|---|
| Fairness and non-discrimination (1.2) | not letting the university (+2.5 points) or the name affect the score: blinding |
| Respect for human rights and privacy (1.2) | refusing minors, no consent, and the ethnicity ranking; not storing field values in the log |
| Human oversight / ability to contest (1.2) | escalating 40-49 scores and low-income rejects to a person |
| Robustness, security and safety (1.4) | validating input; refusing injected text |
| Accountability (1.5) | one log record per call, including refusals |

**Conflicts.** *Identical treatment vs fairness given what I measured:* blinding treats every
applicant identically, but it also hides information that could explain a regional student's lower
exam score (fewer resources). I put identical treatment first, because the fund says university must
not matter and the only measured effect worked against regional applicants. *Human oversight vs
privacy:* escalation means a person reads the applicant's data; I accepted this, since the person is
the safeguard Art. 19-1 asks for. *Equal treatment vs protecting the vulnerable:* my
`low_income_reject` rule treats low-income applicants differently (they always get a human). I put
protection of the people who lose most from a wrong rejection first.

**Where ethics demands more than the law:** the law does not require measuring bias with matched
pairs, hiding name or university, or sending low-income rejections to a person. Fairness and
accountability do.

## 6. The trade-offs

| control | cost | harm it prevents |
|---|---|---|
| validate_input | an applicant with a typo must resubmit | decisions on impossible data (the bare pipeline rejected GPA 9.9 / exam −5 instead of refusing) |
| blind | loses any real information in the university; after blinding the pair test can no longer show bias (flips 0 by construction); 166 of 400 scores moved by 14.01 on average | the +2.5 point flagship advantage; any name-based bias |
| refuse | possible wrongful refusals (0 of 400 in my run) | processing minors, without consent, for discriminatory purposes, or with injected instructions |
| log | storage; an extra file to protect | decisions that nobody can explain or audit |
| escalate | reviewer time: 52 of 400 (13.0%) go to a person | machine-only rejections in the uncertain zone and of low-income applicants |
| compute | none extra: 400 model calls in both runs; prompt tokens fell 84,409 → 77,922 | — |

**The escalation band.** I chose 40-49 because it is the only score range where the bare model gave
both rejects and shortlists. On the unblinded scores it would have sent 64 of 400 (16.0%) to a person;
in the harness run it actually sent 52 of 400 (13.0%), because scores moved after blinding. I did
**not** recalibrate the band on the blinded scores (no time for another run); this is a known weakness.
Most of the workload comes from the low-income rule (40 escalations), not the band (12). Cases outside
the band, mainly the 342 shortlists, are never re-checked by anyone; a wrong shortlist costs the fund
money but does not harm the applicant, so I accepted that.

**Why this balance.** I put the cost on reviewers (13% of applicants) rather than on applicants (a
wrong automatic rejection). I would choose differently if reviewer capacity were much smaller, for
example by narrowing the band, or if a larger measurement showed a real gender or name effect, which
would justify the counterfactual check (doubling model calls).

**What the design still fails to protect against:**
- The model itself is not fairer; blinding only hides inputs. Program, age or income could still act
  as proxies. Closing this needs a larger matched-pair study (e.g. 200 pairs per attribute) on the
  blinded harness with proxies varied.
- 40 pairs cannot detect effects under about 15% of pairs. Closing it needs more pairs and more time.
- The band was not recalibrated after blinding.
- Injection through other wording (e.g. in another language) can bypass my keyword list.
- No notice to the refused applicant and no recorded human decision for escalations; the log is not
  tamper-proof and the prototype does not store data in Kazakhstan. These need a real deployment
  (case management, write-once storage, local hosting), which is outside this lab.

## AI-use declaration

I used Gemini AI while completing this assignment. I mainly used it to help me understand the concepts and clarify topics that I found difficult. It helped me gain a better understanding of the subject and improve my knowledge before completing the work. I also used Gemini AI for proofreading to check my writing, correct grammatical mistakes, and improve the clarity and readability of my sentences. However, the AI was used only as a supporting tool, and I reviewed the suggestions to ensure that the final work reflected my own understanding.