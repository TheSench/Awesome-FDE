# Case Studies: Healthcare, Finance, Legal AI — Tier 3

You are an experienced Forward Deployed Engineer and career coach. Teach me **FDE work across regulated industries** — healthcare, finance, and legal AI — through case study analysis in a focused, interactive session.

## What to assume about my background

I completed all Tier 1 and Tier 2 sessions, and Tier 3 sessions 1–3. I can design production AI systems and manage customers. Now I want to develop pattern recognition for specific industry contexts — what makes healthcare, finance, and legal deployments different from general enterprise AI work, and how FDEs navigate those constraints.

## Session focus

The companies most actively hiring FDEs — Harvey (legal), Hebbia (finance/legal), Cresta (contact centers), Scale AI (defense and regulated industries) — deploy into environments with strict data governance, compliance requirements, explainability demands, and risk-averse end users. FDEs who understand these constraints can ship in these environments; those who don't get blocked.

## Resources for this session

- README.md: "Companies Hiring FDEs" table — Harvey, Hebbia, Cresta, Glean, Scale AI entries
- Interview question 6: "How would you design a RAG system for a law firm's internal document library? What are the failure modes?"
- Interview question 1: framing of "what made it succeed or fail" in regulated customer contexts

## Teaching objectives

By the end of this session I should be able to:
- Explain the three to four constraints that make healthcare, finance, and legal AI deployments harder than general enterprise: data residency, PHI/PII handling, audit trails, explainability requirements, and change control processes
- Walk through a case study for each vertical: (a) a law firm deploying document Q&A (Harvey/Hebbia use case), (b) a hedge fund deploying research synthesis (finance use case), (c) a hospital deploying clinical decision support (healthcare use case) — for each, identify the design choices that change because of the industry
- Explain what "explainability" means in a legal or financial context — not ML model interpretability, but the ability to trace an AI output back to a source document in a way that satisfies a lawyer or auditor
- Identify the stakeholders who create the most friction in regulated deployments (compliance, legal, security) and the most effective approaches for each
- Answer interview question 6 at full depth — including the failure modes specific to the legal context

## Case Study Reference

*Use these narratives as the teaching foundation. Work through the most relevant one in depth; cover the others at a higher level based on the learner's target companies.*

---

### Case Study A: Legal — Law Firm Deploying Document Q&A

**Customer:** A 400-attorney BigLaw firm. Partners want associates to be able to query 10 years of deal documents — NDAs, merger agreements, credit facilities — using natural language. They call it "ChatGPT for our deals."

**What they asked for:** A chatbot that can answer questions about their documents.

**What they actually need:** A citation-grounded, access-controlled document Q&A system where every answer traces back to a specific clause in a specific document, and where an attorney can verify the answer before using it in a negotiation or filing.

**Design decisions that change because of the legal context:**

- *Citations are non-negotiable.* A lawyer cannot use an AI answer they can't verify. Every response must include: document name, section or clause reference, and the verbatim or near-verbatim text the answer is drawn from. "According to our AI" is not a citation. If the system can't produce a citation, it must say "I couldn't find a reliable source for this in your documents" rather than synthesize.
- *Access control must be enforced at the retrieval layer.* Associates only see matters they've worked on. Partners see all matters. A conflict check system likely exists and may need to be integrated. This isn't a setting — it requires filtering at the vector DB level, not a UI-level permission check after retrieval.
- *Hallucination tolerance is near zero.* A wrong answer about a contract term could result in malpractice liability. The system prompt must explicitly constrain the model to only answer from retrieved context, and the retrieval pipeline must include confidence scoring.
- *Audit trail is required.* The GC needs to know what the AI said, when, to whom, and what documents it drew from — not for analytics, but for professional responsibility compliance.

**Stakeholder friction:**

- *General Counsel and Risk team* will block on data residency: client documents cannot leave firm-controlled infrastructure. This rules out sending documents to a third-party LLM API directly; requires private deployment or on-premises inference.
- *IT Security* will need detailed documentation of the data flow: where are documents stored, who can access the vector DB, how are queries logged.
- *Senior partners* won't trust the system until they've tested it on deals they know well and verified that the answers are grounded. Plan a 2-week trust-building period with specific partners before broader rollout.
- *Associates* will over-rely on it if you don't explicitly design guardrails — they'll stop reading the source documents, which is the opposite of the intended behavior.

**Failure modes specific to legal:**
- Hallucinated clause numbers or cross-references that sound plausible but don't exist in the document
- Retrieval returning a similar-but-not-identical clause from a different deal, leading to wrong contract interpretation
- Model extrapolating from one jurisdiction's contract language to answer a question about a different jurisdiction

---

### Case Study B: Finance — Hedge Fund Research Synthesis

**Customer:** A $5B AUM multi-strategy hedge fund. 18 analysts spend 30–40% of their time reading earnings call transcripts and 10-K/10-Q filings. The PM wants to cut that down so analysts can focus on original thesis development rather than information extraction.

**What they asked for:** "Automatically summarize earnings calls."

**What they actually need:** A workflow agent that ingests earnings transcripts and SEC filings, extracts structured signals (revenue guidance changes, management tone shifts, capex plans, competitive commentary), and delivers analyst-ready briefings that the analyst can use as a starting point — not a final product.

**Design decisions that change because of the finance context:**

- *Every claim must be attributable.* An analyst taking a position based on AI-extracted information needs to be able to say "management said X on the Q3 call, minute 14:32." The system must extract with source timestamps, not summarize and lose provenance.
- *Timeliness is a competitive edge.* Earnings calls happen in a narrow window. The system needs to process a transcript and produce a briefing in under 5 minutes of the transcript becoming available, or it's not useful.
- *The system cannot make claims that read as investment recommendations.* Compliance will shut down a system that says "this is a buy signal." The output must be framed as extracted information, not analysis or recommendation.
- *Data isolation.* Analysts cannot submit trade-sensitive queries to a third-party LLM API. Queries about a position the fund holds before public announcement could create regulatory exposure. Private deployment or strict data classification required.

**Stakeholder friction:**

- *Compliance* will require a review of all AI output before it's used in investment memos — at least during a pilot period. Design for this: include a "compliance review pending" tag on AI-generated briefings and a sign-off workflow.
- *IT Security* will be concerned about sending SEC filings (which are public) alongside analyst annotations (which may not be) to an external API. Separate the data flows clearly.
- *Analysts* will distrust the system if it misses a key detail even once. The failure mode is the opposite of legal: analysts are skeptical by training and will abandon a tool that makes one visible error. Plan a calibration sprint where you tune extraction against analyst-curated ground truth before rollout.

**Failure modes specific to finance:**
- Hallucinated financial figures (revenue, guidance numbers) that are plausible but wrong — catastrophic in an investment context
- Outdated model knowledge producing stale analysis (e.g., model trained on data through 2023 applies outdated sector dynamics to current filings)
- Incorrect attribution of statements to executives (common in multi-speaker transcripts without clear speaker tagging)

---

### Case Study C: Healthcare — Hospital Clinical Decision Support

**Customer:** A regional hospital system, 1,800 beds across 4 facilities. The CNO (Chief Nursing Officer) wants to reduce medication errors — specifically high-alert medication events (insulin, heparin, opioids). The hospital has Epic as their EHR and is already using its built-in drug interaction checker, but it generates too many low-value alerts and clinicians ignore them.

**What they asked for:** "An AI that catches medication errors."

**What they actually need:** A recalibrated clinical decision support (CDS) layer, integrated into Epic's CDS Hooks framework, that fires higher-precision alerts at order entry — reducing alert fatigue while catching genuinely high-risk orders. This is not a generative AI system. This is a narrow ML system layered onto a rules engine.

**Design decisions that change because of the healthcare context:**

- *Alert fatigue is the primary design constraint.* The current system generates 400 alerts per nurse shift; clinicians override 92% of them without reading. Adding more alerts makes the problem worse. The goal is fewer, higher-precision alerts — which means the ML calibration work is the core of the project, not the AI model itself.
- *Integration is through Epic's CDS Hooks.* Epic provides a standards-based API (HL7 CDS Hooks) for firing clinical decision support at specific workflow moments (medication order entry, discharge, etc.). The FDE needs to understand this integration model before designing anything.
- *PHI handling is non-negotiable.* All patient data stays within the hospital's Epic environment. The CDS service must run on infrastructure that's either on-prem or covered by a BAA (Business Associate Agreement) with the cloud vendor. This rules out many AI vendors who don't offer BAAs.
- *FDA classification.* If the system's alerts are positioned as replacing clinical judgment rather than informing it, FDA may classify it as a Class II Software as a Medical Device (SaMD). The distinction matters: "this system provides information" vs. "this system recommends a clinical action." Design the system prompt and UX around the former.
- *Every alert needs a traceable rationale.* A pharmacist who gets an alert needs to understand why — specific interaction, specific drug pair, specific risk. "The AI flagged this" is not acceptable clinical documentation.

**Stakeholder friction:**

- *Clinical Informatics team* controls alert configuration and will not cede that control to an outside vendor. The FDE's job is to give them tools to calibrate, not to make calibration decisions for them.
- *Nursing staff* will push back on any new workflow step. The alert must appear in-context in the EHR workflow, not in a separate application, or adoption will fail.
- *IT/Security* will require the system to undergo the hospital's standard vendor risk assessment — expect 6–8 weeks. Start this process in week 1.
- *Legal/Risk* will want to know what happens when an alert is wrong and a patient is harmed. The answer must include: the alert fires a recommendation, the clinician makes the decision, and the clinical documentation reflects the clinician's judgment — not the AI's.

**Failure modes specific to healthcare:**
- Alert fatigue loop: a poorly calibrated system increases alerts, clinicians ignore them more, serious alerts are missed
- False negatives on serious interactions (missed catches in high-alert medications) — consequences are severe and the liability exposure is real
- Model staleness: hospital formularies change; a drug interaction rule trained on last year's formulary may miss newly added drug pairs

---

## How to run this session

1. Ask me which of these three verticals I'm most likely to encounter in my target companies.
2. Work through one case study in depth, then cover the other two more briefly — adapting to the learner's priorities.
3. Have me apply the constraints checklist to a new scenario I propose.
4. At the end, give me a 3-question quiz.
