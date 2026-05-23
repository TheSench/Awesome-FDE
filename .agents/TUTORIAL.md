# FDE Tutorial — Agent Entry Point

`@` this file to start or resume your FDE tutorial session.

---

## What the agent should do on load

### Step 1 — Load context (do this silently, without narrating)

Read both files:
- `learner/profile.md`
- `learner/relevance.md`

On missing file (error or empty result), treat as first-time learner:
- Missing `learner/profile.md` → no sessions completed, no background known. Proceed to [initialization](#initialization) instead of Steps 2–3.
- Missing `learner/relevance.md` → treat all topics as MED until file created.

Attempt read and branch on result; don't check existence first.

### Initialization

Run only when `learner/profile.md` doesn't exist (first session ever).

1. Ask learner short background questions:
   - What's your current engineering background? (e.g. backend, crypto/blockchain, infrastructure, full-stack)
   - Have you worked directly with customers or external stakeholders in a technical capacity?
   - What's drawing you to the FDE role — is it a career pivot, interview prep, or general curiosity?
2. Create `learner/profile.md` using template in [`.agents/docs/TEMPLATES.md`](.agents/docs/TEMPLATES.md).
3. If `learner/relevance.md` also missing, create it using relevance file template in [`.agents/docs/TEMPLATES.md`](.agents/docs/TEMPLATES.md) with MED for all topics. Note it was auto-generated and can be customized.
4. Proceed to Step 2 (start session 1).

### Step 2 — Determine next session

Using `Tiers completed` table in learner profile + [Session Sequence](#session-sequence), identify:
- Last completed session (topic + tier)
- Next session (file path + topic + relevance rating)

### Step 3 — Show progress summary and begin

Show brief progress summary, then immediately start next session without waiting for confirmation.

**Progress summary format:**

```
**Progress so far:** [N] of 23 sessions complete.
Last completed: [Topic] — Tier [N] ([date])

**Starting:** [Topic] — Tier [N] ([HIGH / MED / LOW] relevance)
[One sentence on what this session covers and why it matters for this learner's context.]
```

### Step 4 — Run the session

1. Load prompt file from `prompts/`
2. Run session
3. Apply depth calibration from `learner/relevance.md` for topic

### Step 5 — Update the learner profile

Update `learner/profile.md`. Add entry under `## Session log`:

```markdown
### [Topic] — Tier [N] · [YYYY-MM-DD]

**Covered:** [1–2 sentence summary of what was taught]

**Strengths:** [Concepts the learner grasped quickly or explained back correctly]

**Gaps / needs reinforcement:** [Concepts that needed multiple attempts, were answered incorrectly in the quiz, or the learner flagged as uncertain]

**Open questions:** [Questions raised that weren't fully resolved — carry these forward]

**Notes:** [Anything else relevant: learning style observations, areas of strong interest, analogies that landed well]
```

Update top-level profile sections if learner background or preferences were revealed. Don't ask for permission — just write the file.

#### Post-session follow-up questions

Answer follow-up questions normally. Evaluate if exchange revealed anything worth tracking:

- Gap resolved → update **Gaps / needs reinforcement**
- New open question → add to **Open questions** or **Carried-forward open questions**
- Concept grasped more deeply → note under **Strengths**
- Illuminating observation → add **Extended discussion** paragraph in session log

Only write file if something worth carrying forward.

#### Update progress chart

After updating `learner/profile.md`, regenerate `learner/progress.md` to reflect the new session count. The file has two sections:

**Header:** `**[N] of 23 sessions complete** · Updated [YYYY-MM-DD]`

Count total sessions by summing entries in the `Tiers completed` table.

**Overview chart** (`xychart-beta`): bar chart with four bars — one per tier — showing percentage complete. Tier totals are always 5 / 7 / 6 / 5. Round to nearest integer.

```
xychart-beta
    title "% Complete by Tier"
    x-axis ["Tier 1", "Tier 2", "Tier 3", "Tier 4"]
    y-axis "% Complete" 0 --> 100
    bar [<t1_pct>, <t2_pct>, <t3_pct>, <t4_pct>]
```

**Current tier chart** (`graph LR`): show all sessions in the active tier (the lowest tier not yet fully complete) as a linear chain of nodes. Use these classes:

- `done` (green `#2d6a4f`) — completed sessions
- `next` (orange `#f4a261`) — the immediately upcoming session (first incomplete)
- `pending` (gray `#dee2e6`) — remaining sessions after next

Node IDs: `T<tier>_<position>` (e.g., `T2_5`). Labels: short topic name (2 lines max). Connect all nodes left-to-right with `-->`.

If Tier 4 is complete, replace the current tier chart with a completion message.

The session order within each tier follows the [Session sequence](#session-sequence) table.

Commit changes:

```
git add learner/profile.md learner/relevance.md learner/progress.md
git commit -m "Session log: [Topic] — Tier [N] ([YYYY-MM-DD])"
```

Only stage modified files. Don't ask — just commit.

### Step 6 — Prompt to start the next session

After saving progress, display exactly content inside `<closing_message>`, nothing after. Don't include `<closing_message>` tags.

<closing_message>
Session saved. To continue, open a new conversation and send:

```
Continue
```
</closing_message>

Only `Continue` inside code block so learner can copy-paste directly.

---

## Depth calibration

Calibrate on two axes before session:

**Learner history** (`learner/profile.md`): skip mastered concepts, dwell on gaps, connect to prior strengths, surface open questions, match explanation style. If no session log yet, check background for stated strengths/gaps.

**Topic relevance** (`learner/relevance.md`): HIGH → go deeper, focus on real-world application and interview implications; MED → follow prompt as written; LOW → core concepts only, compress peripheral details, note what was skipped.

---

## Templates

See [`.agents/docs/TEMPLATES.md`](.agents/docs/TEMPLATES.md) for learner profile template and relevance file template.

---

## Progress tracking

Progress stored in `learner/profile.md`. `Tiers completed` table is canonical record. Session complete when session log entry written.

### Learner profile folder structure

Profile starts as single file. When session log exceeds ~15 sessions, split it:

```
learner/
  profile.md                  ← keep: background, tiers table, recurring strengths/gaps, open questions
  sessions/
    tier1-01-what-is-fde.md   ← move: individual session logs go here
    tier1-02-fde-in-ai-era.md
    ...
```

When splitting, update `learner/profile.md` to add `## Session index` with links to session files. Write new sessions to `learner/sessions/` once folder exists; otherwise append to `learner/profile.md`.

---

## Session sequence

Complete all sessions at one tier before advancing. Within tier, follow order below. Relevance from `learner/relevance.md` affects depth, not whether to complete.

### Tier 1 — Orientation

| # | File | Topic |
|---|------|-------|
| 1 | `prompts/tier1/01-what-is-fde.md` | What is a Forward Deployed Engineer |
| 2 | `prompts/tier1/02-fde-in-ai-era.md` | The FDE Role in the AI Era |
| 3 | `prompts/tier1/03-technical-skills.md` | Core Technical Skills |
| 4 | `prompts/tier1/04-customer-skills-mindset.md` | Customer-Facing Skills & Mindset |
| 5 | `prompts/tier1/05-fde-interview-overview.md` | The FDE Interview — Overview |

### Tier 2 — Breadth

| # | File | Topic |
|---|------|-------|
| 6 | `prompts/tier2/01-role-deep-dive.md` | FDE vs SE vs Solutions Engineer |
| 7 | `prompts/tier2/02-llm-systems.md` | Building LLM-Powered Systems |
| 8 | `prompts/tier2/03-rag-vector-dbs.md` | RAG & Vector Databases |
| 9 | `prompts/tier2/04-production-system-design.md` | System Design for Production AI |
| 10 | `prompts/tier2/05-palantir-style-interview.md` | The Palantir-Style Problem Interview |
| 11 | `prompts/tier2/06-interview-prep-storytelling.md` | Interview Preparation & Storytelling |
| 12 | `prompts/tier2/07-crypto-to-fde.md` | Crypto/Blockchain → FDE Transition |

### Tier 3 — Application

| # | File | Topic |
|---|------|-------|
| 13 | `prompts/tier3/01-production-deployment-patterns.md` | Production AI Deployment Patterns |
| 14 | `prompts/tier3/02-llm-evaluation-observability.md` | LLM Evaluation & Observability |
| 15 | `prompts/tier3/03-stakeholder-management.md` | Stakeholder Management & Communication |
| 16 | `prompts/tier3/04-advanced-interview-practice.md` | Advanced Interview Practice |
| 17 | `prompts/tier3/05-industry-case-studies.md` | Case Studies: Healthcare, Finance, Legal AI |
| 18 | `prompts/tier3/06-fde-portfolio.md` | Building Your FDE Portfolio |

### Tier 4 — Specialist

| # | File | Topic |
|---|------|-------|
| 19 | `prompts/tier4/01-enterprise-architecture.md` | Enterprise Architecture for AI |
| 20 | `prompts/tier4/02-change-management.md` | Change Management & Adoption |
| 21 | `prompts/tier4/03-apac-international.md` | APAC & International FDE Market |
| 22 | `prompts/tier4/04-fde-career-progression.md` | FDE Career Progression |
| 23 | `prompts/tier4/05-community-contribution.md` | Contributing to the FDE Community |
