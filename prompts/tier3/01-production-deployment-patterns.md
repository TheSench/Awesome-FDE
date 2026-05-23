# Production AI Deployment Patterns — Tier 3

You are an experienced Forward Deployed Engineer and career coach. Teach me **production AI deployment patterns** at an applied, hands-on level in a focused, interactive session.

## What to assume about my background

I completed all Tier 1 and Tier 2 sessions. I understand LLM APIs, RAG, agent frameworks, and production system design tradeoffs. Now I want to go from knowing the components to knowing how they're assembled and operated in real customer deployments — the patterns that experienced FDEs reach for, and why.

## Session focus

Knowing the components isn't enough — FDEs need pattern recognition. Most enterprise AI deployments follow recognizable shapes, and an FDE who can quickly identify which pattern applies can move faster, avoid known failure modes, and set better expectations with customers. This session catalogs the core patterns.

## Resources for this session

- README.md: Technical skills section (as background) plus interview question 6: "How would you design a RAG system for a law firm's internal document library? What are the failure modes?"
- Patterns to cover: document Q&A (RAG), workflow automation agents, real-time data enrichment, human-in-the-loop review pipelines, multi-modal document processing

## Pattern Reference

*Use this as the teaching foundation for the session. Each pattern is described with its component stack, integration points, and production failure modes.*

---

### Pattern 1: Document Q&A (RAG)

**Customer need:** Employees need to query large document repositories (contracts, policies, internal wikis, manuals) with natural language and get cited answers.

**Component stack:**
- Document store (SharePoint, Confluence, Google Drive, S3) → ingestion pipeline (PDF parsing, chunking, embedding) → vector database → retrieval layer → re-ranker → LLM with system prompt enforcing citation → response with source references

**Integration points:**
- SSO/identity system — access control must be enforced at retrieval (users should only see documents they're permitted to access)
- Document management system — ingestion must be incremental and handle updates/deletions
- Existing ticketing/search tools — the output often feeds into or replaces an existing search workflow

**Production failure modes:**
1. *Access control not enforced in retrieval* — the vector DB returns semantically relevant documents that the user isn't permitted to see. This is a compliance disaster in legal, finance, and healthcare. Fix: filter by user permissions before or during retrieval, not after.
2. *Chunk boundary errors* — a contract clause is split across two chunks; the answer is in neither chunk individually. Fix: overlapping chunks + parent document retrieval.
3. *Hallucination over retrieved content* — the LLM elaborates beyond the source material and presents it as a citation. Fix: strict prompting ("only answer from the provided context; if the answer isn't in the context, say so") and citation grounding validation.

---

### Pattern 2: Workflow Automation Agents

**Customer need:** Multi-step processes that previously required human coordination — intake, triage, routing, response drafting — need to run automatically or with minimal intervention.

**Component stack:**
- Trigger (email/webhook/schedule/form submission) → stateful agent (LangGraph or similar) → tool calls (CRM API, database reads/writes, email send, internal APIs) → output handler (human review queue, direct action, or notification)

**Integration points:**
- CRM and ticketing systems (Salesforce, Zendesk, ServiceNow) — usually the source of truth for workflow state
- Internal APIs — FDEs frequently discover these are underdocumented or have no sandbox environment
- Email/Slack/notification layer — for handoffs to humans and status updates

**Production failure modes:**
1. *Agent loops* — the agent retries a failed tool call indefinitely, or gets stuck in a clarification cycle. Fix: max retry limits, explicit failure states that route to human review.
2. *Non-determinism at scale* — the same input produces different outputs on different runs. Customers notice this when the agent handles 1,000 tickets and 5% are handled inconsistently. Fix: lower temperature for action-taking steps; human review queue for low-confidence actions.
3. *Latency making synchronous use impractical* — chaining 4 LLM calls and 6 API calls takes 30+ seconds; the customer expected a 3-second response. Fix: async execution with status updates; identify which steps need synchronous output and which can be deferred.

---

### Pattern 3: Real-Time Data Enrichment

**Customer need:** Live events flowing through a system (customer support tickets, trading signals, transaction records, log events) need to be enriched with AI-generated context as they arrive.

**Component stack:**
- Event stream (Kafka, Pub/Sub, SQS, or webhook) → enrichment service (LLM call, often with a lookup against a context store) → enriched event output → downstream consumer (analytics, alerting, CRM update)

**Integration points:**
- Message queue or streaming infrastructure — understanding backpressure and consumer lag is required
- Context store (database or vector DB) — enrichment usually requires retrieving entity context (customer history, product details) alongside the event
- Downstream systems — the enriched output often feeds into existing BI, alerting, or workflow tools

**Production failure modes:**
1. *LLM latency breaking real-time SLAs* — if the customer expects sub-second enrichment and the LLM call takes 1.5 seconds, the architecture is wrong. Fix: async enrichment with best-effort delivery, or use a smaller/faster model for latency-sensitive paths.
2. *Cost spiraling at volume* — enriching 100,000 events/day with GPT-4-class models at $0.01/call is $1,000/day. Customers rarely anticipate this. Fix: model tiering (cheap model for filtering, expensive model only for high-priority events), caching for repeated inputs.
3. *Prompt drift creating inconsistent enrichment* — when a prompt is updated mid-stream, historical and new enrichments are no longer comparable. Customers using enrichment for analytics only discover this 3 months later. Fix: version prompts; log which prompt version produced each output.

---

### Pattern 4: Human-in-the-Loop (HITL) Review Pipelines

**Customer need:** AI handles volume but humans need to catch edge cases — especially in compliance, legal, medical, or high-stakes decision contexts where automated error has real consequences.

**Component stack:**
- AI draft or recommendation → confidence scoring → routing logic (auto-approve above threshold / human queue below threshold) → human review interface → approval/rejection → audit trail + optional feedback signal for model improvement

**Integration points:**
- Existing review workflows — the HITL queue often replaces or augments something humans were already doing manually; the UX must fit into their existing flow or adoption fails
- Audit/compliance systems — every decision and its reviewer must be logged with timestamp
- Model pipeline — feedback from reviewers needs a path back to improving the model, or the system doesn't get better over time

**Production failure modes:**
1. *Humans rubber-stamping* — reviewers approve AI recommendations without reading them because the queue volume is too high or the review UI doesn't surface the reasoning clearly. Fix: require a brief rationale input for overrides; sample and audit reviewer behavior.
2. *Confidence threshold miscalibration* — set too high, everything goes to humans and you've just built an expensive inbox; set too low, errors slip through. Fix: calibrate on historical data before launch; revisit thresholds at 30 and 90 days.
3. *No feedback loop* — the system captures reviewer decisions but the data never flows back to improve the model. Fix: build the feedback pipeline at the start, not as a post-launch improvement.

---

### Pattern 5: Multi-Modal Document Processing

**Customer need:** Extract structured information from documents that combine text, tables, charts, and images — invoices, medical forms, engineering drawings, financial statements — at scale.

**Component stack:**
- Document input (upload, email attachment, scan) → document parsing layer (OCR for scanned PDFs, direct extraction for digital PDFs) → multi-modal model for layout understanding → structured extraction (field-by-field or schema-based) → validation layer → output to target system (ERP, database, CRM)

**Integration points:**
- Document management or inbound processing systems (email, FTP, document capture portals)
- ERP/CRM for structured output destination — field mapping is often non-trivial
- Compliance/audit systems — original document must be preserved alongside extracted output

**Production failure modes:**
1. *OCR errors on poor-quality scans* — fax-quality or photographed documents have high error rates; the extraction looks plausible but contains wrong values. Fix: confidence scores on extracted fields; flag low-confidence fields for human review rather than silently passing wrong data.
2. *Table extraction failures* — complex multi-header tables, merged cells, and rotated tables break most extraction pipelines. Fix: test specifically on the worst-quality documents in the customer's corpus before committing to accuracy SLAs.
3. *No validation layer* — extracted values (totals, dates, account numbers) are passed directly to downstream systems without sanity checks. A wrong invoice amount that passes straight into AP is a costly error. Fix: rule-based validation (totals must sum, dates must be in range, account numbers must match a known format) before any extracted value reaches a system of record.

## Teaching objectives

By the end of this session I should be able to:
- Describe five common enterprise AI deployment patterns (document Q&A, workflow automation, real-time enrichment, human-in-the-loop review, multi-modal processing) and explain what customer need each serves
- For each pattern, identify the standard component stack, the typical integration points with customer systems, and the two or three failure modes that actually occur in production
- Apply the "which pattern is this?" diagnostic to a new customer problem — given a vague customer description, identify the closest pattern and what additional information you'd need to confirm
- Explain what "going live" actually involves for an AI deployment: staging environment, user acceptance testing, monitoring setup, rollback plan
- Describe what a post-deployment review contains and who it's for — the README explicitly names this as a common interview question

## How to run this session

1. Ask me which deployment patterns I've built or been close to — this determines where to spend more time.
2. Walk through each pattern with a component diagram described in words: "the customer data flows into X, which connects to Y, with Z as the failure point most people miss."
3. Have me apply the diagnostic to one new scenario: given a description, identify the pattern and its failure modes.
4. At the end, give me a 3-question quiz.

Enterprise-level multi-system architecture and change management are covered in Tier 4 sessions 1 and 2.
