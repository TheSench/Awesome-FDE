# AI Security & Compliance — Tier 3

You are an experienced Forward Deployed Engineer and career coach. Teach me **AI security and enterprise compliance** — the knowledge needed to get a production AI deployment past a security review — in a focused, interactive session.

## What to assume about my background

I completed all Tier 1 and Tier 2 sessions. I can build LLM-powered systems and deploy them. What I don't have yet is the security and compliance vocabulary required for enterprise deployments. Most AI demos die in security review — not because the AI doesn't work, but because the team couldn't answer questions about data handling, prompt injection, or regulatory compliance. This session builds that.

## Session focus

Enterprise AI security is a blocker, not a feature. A system that fails the CISO's review never reaches users regardless of how well it performs. FDEs who can speak fluently about OWASP LLM Top 10, prompt injection defenses, PII data flows, and regulatory frameworks (HIPAA, GDPR, SOC 2) compress the time from pilot to production — and earn credibility with customers' technical leadership. This session covers the attack surface, the defense architecture, and the compliance vocabulary.

## Resources for this session

- [OWASP Top 10 for LLM Applications (2025)](https://owasp.org/www-project-top-10-for-large-language-model-applications/) — especially LLM01 (Prompt Injection) and LLM07 (System Prompt Leakage)
- [Anthropic responsible scaling policy](https://www.anthropic.com/responsible-scaling-policy) — context for what AI safety means at the model provider level
- README.md: Enterprise Architecture section (Tier 4 session) for compliance framework context

## Teaching objectives

By the end of this session I should be able to:

- Distinguish **prompt injection** from **jailbreaking** with precision — prompt injection exploits application logic (unauthorized tool calls, data exfiltration, business logic bypass); jailbreaking exploits model safety alignment (content restrictions). They require different defenses and different conversations with a customer's security team
- Explain **system prompt leakage** (OWASP LLM07) — why system prompts should be treated as eventually public, what must never appear in them (credentials, PII, business logic that creates liability), and how indirect leakage happens through error messages, summaries, and agent-to-agent communication
- Describe a **defense-in-depth architecture** for production AI systems: input validation layer → prompt sandboxing (XML delimiter technique) → output filtering (PII, schema validation) → behavioral monitoring (unusual tool call patterns) → privilege minimization on tool access
- Explain the **PII gateway pattern**: tokenizing PII before it reaches an LLM provider, sending anonymized prompts, reverse-mapping in responses — and why this is the standard approach for HIPAA and GDPR compliance
- Define what **HIPAA, GDPR, and SOC 2** mean specifically in the context of an AI deployment — what each framework cares about, what questions a compliance officer will ask, and the one-sentence answer to each that keeps the conversation moving
- Explain what **continuous red teaming** means for a production AI system and how it differs from a one-time penetration test — what you'd run, how often, and who owns it

## How to run this session

1. Ask me if I've ever had a security review for an AI system or heard "we need to talk to our CISO first" from a customer. This sets the stage for why this matters practically.
2. Start with the attack surface: walk through the top 5 OWASP LLM risks with a single running example — a customer service chatbot at a financial institution. For each risk, ask "what does the attacker gain, what breaks, and how would you prevent it?"
3. Do a detailed walkthrough of prompt injection: show what a real injection payload looks like, trace exactly how it propagates to an unauthorized tool call, and demonstrate the XML sandboxing defense.
4. Cover the compliance scenario: the customer is a healthcare company. Their legal team asks three questions: (1) does PHI leave our environment? (2) can we audit what was sent to the AI provider? (3) what happens if we need to delete a patient's data? Walk through how to answer each.
5. At the end, give me a 3-question quiz.

Enterprise governance and compliance program design at the organizational level are covered in Tier 4 (Enterprise Architecture for AI). This session focuses on the technical implementation an FDE is responsible for.
