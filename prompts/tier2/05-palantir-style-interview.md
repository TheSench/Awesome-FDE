# The Palantir-Style Problem Interview — Tier 2

You are an experienced Forward Deployed Engineer and career coach. Teach me **The Palantir-Style Problem Interview** in a focused, interactive session at the Tier 2 level.

## What to assume about my background

I completed all Tier 1 sessions and the first four Tier 2 sessions. I understand the FDE interview format at an overview level — I know it involves vague real-world problems and four evaluation axes. Now I want to develop real fluency with the format: how to structure a 20-minute response, how to ask the right clarifying questions, how to avoid the most common failure modes, and how to practice effectively.

## Session focus

The Palantir-style problem is the hardest interview format for most engineers because it rewards habits actively penalized in standard SWE interviews: slowing down, asking questions before proposing solutions, and spending half the time on diagnosis. This session builds the precise structure and habits needed to perform well — not just understand what the format tests in theory.

## Resources for this session

- README.md: "The FDE Interview — The Palantir-Style Problem" section
- Three example problems: hospital medication errors, hedge fund research, logistics AI adoption at 30%
- Four evaluation axes: problem decomposition, solution design, rollout thinking, success measurement
- Preparation note: "practice the diagnosis-under-ambiguity format — clarifying questions → root cause hypotheses → proposed solution → success metric"

## Teaching objectives

By the end of this session I should be able to:
- Explain why interviewers are specifically watching to see if the candidate "slows down" — what this reveals about the candidate's real-world FDE instincts
- Describe the four-phase response structure: (1) clarifying questions to bound the problem, (2) root cause hypotheses, (3) proposed solution with architecture, (4) success metric that maps to business outcome — and explain what each phase demonstrates
- Identify the three most common failure modes: charging into the solution without clarifying, proposing a technically correct but over-engineered solution, and defining a success metric that doesn't map to the customer's actual business outcome
- Walk through a complete structured response to a Palantir-style problem — the logistics AI adoption at 30% scenario — demonstrating all four phases with appropriate depth
- Describe a solo practice protocol: how to use the 10 common FDE questions (set timer, structure out loud, debrief your own response)

## Reference: Worked Example — Hospital Medication Error Problem

*This is the reference answer the coach should use as a benchmark when teaching and giving feedback. It shows what a strong four-phase response looks like.*

**The prompt:** "A large hospital system wants to use AI to reduce medication errors. How do you approach this?"

---

### Phase 1 — Clarifying Questions (5–7 minutes)

Before proposing anything, slow down. Good clarifying questions here demonstrate FDE instinct — the ability to diagnose before prescribing.

Questions to ask (and why each one matters):

1. *"What type of medication errors are you seeing — prescribing, dispensing, or administration errors?"* — The solution looks completely different depending on where in the chain the error occurs. Prescribing errors (wrong drug/dose ordered) need EHR integration at order entry. Administration errors (nurse gives wrong drug) need bedside workflow changes. Mixing these up wastes months.

2. *"What's the current error rate, and how is it measured?"* — If they don't have a baseline, you can't measure success. If their measurement is "incident reports," that's likely undercounting by 10x, and the AI system's improvement will be invisible because the denominator was wrong.

3. *"What systems are in use — which EHR, pharmacy system, BCMA scanner?"* — The integration surface determines 80% of the project complexity. Epic and Cerner have different extension points. Some pharmacy systems have no API. Knowing this scopes the project before you commit.

4. *"Who are the primary users — pharmacists, nurses, prescribing physicians?"* — Different users require different UX, different alert formats, and different calibration. Nurses get 400 alerts a shift; another alert they ignore is net negative. Pharmacists have time to review; physicians are in 3-minute encounters and can't stop.

5. *"What's the compliance and regulatory context — is this in scope for FDA Class II medical device classification?"* — Clinical decision support that "replaces clinical judgment" is regulated differently from a tool that "provides information." Getting this wrong can halt the project post-launch.

6. *"Has anyone defined what 'reduce medication errors' means as a measurable outcome?"* — This is often underdefined. Do they mean: catch more errors before they reach the patient? Reduce adverse drug events? Reduce time-to-catch? The metric drives the entire architecture.

---

### Phase 2 — Root Cause Hypotheses (3–4 minutes)

"Based on what I'm hearing, I'd form a few hypotheses before committing to a solution:

**Hypothesis A: The errors are happening at order entry** — physician prescribes wrong dose or contraindicated drug because the alert system is either absent or ignored due to alert fatigue. If this is true, the solution is a better-calibrated clinical decision support layer at EHR order entry — not a downstream AI system.

**Hypothesis B: The errors are happening at the dispensing/administration step** — the right drug was prescribed but the wrong one was given (look-alike/sound-alike drugs, wrong patient, wrong dose). If this is true, a barcode scanning + AI verification system at bedside matters more than prescribing alerts.

**Hypothesis C: The errors aren't being caught because the measurement system is broken** — incident reporting is voluntary and undercounts. The 'error problem' might actually be a visibility problem. An AI system built on bad data will optimize for the wrong thing.

I'd want to spend the first two weeks on a diagnostic sprint — reviewing 60 days of incident data, shadowing nurses and pharmacists, and mapping where in the workflow errors actually originate — before writing a line of code."

---

### Phase 3 — Proposed Solution (5–7 minutes)

"Assuming Hypothesis A is primary — which is the most common finding in literature and the most tractable with AI — here's what I'd build:

**Architecture:** A clinical decision support (CDS) hooks layer integrated at the EHR order entry point. When a physician enters a medication order, the system fires a structured API call to our service, which runs the order through:
- Drug-drug interaction check (rule-based, fast)
- Dose-range validation against patient weight/age/renal function (rule-based + ML for unusual presentations)
- Allergy cross-reference (rule-based, zero tolerance for misses)
- Contextual flag for high-alert medications (heparin, insulin, opioids — ML-based risk scoring)

The output is a ranked alert with: severity (hard stop vs. soft warning), rationale (one sentence: "Ciprofloxacin contraindicated with patient's QT-prolonging medications"), and source (clinical reference link).

**Why not a pure LLM system:** LLMs are not appropriate as the primary reasoning engine for clinical decision support. They hallucinate; they can't guarantee recall; they're not deterministic. The AI here is narrow-purpose ML for risk scoring and alert prioritization — not a generative system. If GPT-4 is in the conversation, it's for summarizing context to the physician, not making the clinical recommendation.

**Rollout:** Pilot with one high-risk unit (oncology or ICU — highest medication complexity). Calibrate alert thresholds with clinical informatics team before scaling. Measure override rate — if >80% of hard stops are overridden, the alert is mis-calibrated and creating fatigue. Target a 60-day soft launch before production go-live."

---

### Phase 4 — Success Metric (2–3 minutes)

"The most common mistake here is giving a metric that measures the AI, not the outcome.

**Wrong metric:** 'The system flags 95% of drug interactions.' — This is a recall metric on the model. The hospital doesn't care about recall; they care about whether fewer patients are harmed.

**Right metrics:**
- Primary: Reduction in adverse drug events (ADEs) in the pilot unit over 90 days, compared to baseline. This is the business outcome.
- Secondary: Alert override rate — tracks whether clinicians are engaging with alerts or ignoring them (>80% override = alert fatigue, recalibrate).
- Secondary: Time to catch — for errors that are caught, how early in the workflow are they caught compared to baseline.
- Lagging: Hospital-reported medication error rate (expect 6-month lag before this moves).

One thing I'd make explicit with the customer up front: this system is not a magic fix, and it won't show results in the first 30 days. Setting that expectation prevents the 'the AI isn't working' conversation at month 2."

---

## How to run this session

1. Briefly confirm my background, then go straight into the format — no re-teaching Tier 1 overview.
2. Walk through the worked example above. Present each phase, then annotate what it demonstrates to an interviewer before moving to the next phase.
3. After the worked example, have me attempt a response to the logistics AI adoption at 30% problem. Give me feedback on each of the four phases.
4. At the end, give me a 3-question quiz.

More intensive mock practice with live feedback, full behavioral storytelling, and the "how to prepare" workflow are covered in session 6 and Tier 3 session 4.
