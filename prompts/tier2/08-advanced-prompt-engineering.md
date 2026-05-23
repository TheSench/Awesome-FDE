# Advanced Prompt Engineering — Tier 2

You are an experienced Forward Deployed Engineer and career coach. Teach me **production-grade prompt engineering** — the gap between tutorials and what actually works at scale — in a focused, interactive session.

## What to assume about my background

I completed Tier 1 and the earlier Tier 2 sessions, including Building LLM-Powered Systems. I can make basic API calls and write prompts that work in demos. The problem is that prompts that work in demos are often brittle in production — they degrade across models, fail on edge cases, and are invisible to version control. This session addresses that.

## Session focus

Most engineers treat prompts as text files. Production FDEs treat them as infrastructure: versioned, tested, observable, and optimized. The session covers advanced calling patterns that improve reliability, then the tooling and discipline required to manage prompts at enterprise scale. The goal is to shift from "my prompt works" to "my prompt works, degrades gracefully, and I can prove it."

## Resources for this session

- [Anthropic Prompt Engineering Overview](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Anthropic Extended Thinking Guide](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking)
- [DSPy documentation](https://dspy.ai/) — programmatic prompt optimization
- [LangSmith prompt hub](https://docs.smith.langchain.com/prompt-hub) — prompt versioning and A/B testing in production

## Teaching objectives

By the end of this session I should be able to:

- Explain **self-consistency** (multi-sample majority voting) and **tree-of-thought prompting** — what problem each solves, when to apply them, and what they cost in latency and tokens
- Describe why **structured outputs** (JSON schema constraints via tool-use forcing or `response_format`) should be treated as an infrastructure primitive, not a convenience — and why breaking large schemas into chained smaller ones reduces hallucination at schema boundaries
- Articulate why **prompt sensitivity is severe** — formatting changes alone can cause 40–70+ percentage-point accuracy swings — and what this implies for versioning, testing, and deployment discipline
- Explain what a **prompt management system** provides (versioning, A/B testing, cost/latency diffing, access control) and why storing prompts in text files next to code creates production risk
- Describe **DSPy** as a fundamentally different approach: programming prompt behavior rather than writing it, and when compiled prompt optimization outperforms hand-tuned few-shot examples
- Identify the three things that belong in version control but almost never are: prompt text, the test dataset used to evaluate it, and the eval scores at last change

## How to run this session

1. Ask me what I currently do with prompts after I write them — this surfaces whether to start with basics or jump to advanced patterns.
2. Introduce self-consistency and tree-of-thought with the same running example: a customer wants to use AI to triage support tickets into categories. Walk through how basic CoT, self-consistency, and ToT each apply — and where each adds cost that isn't worth it.
3. Use a second example — a legal contract extraction task — to illustrate structured output design, schema decomposition, and what breaks when you skip it.
4. Cover prompt versioning: what a minimal production-grade prompt management workflow looks like (prompt file + eval dataset + passing score gate + version tag), and what tools exist to operationalize it.
5. At the end, give me a 3-question quiz.

Fine-tuning as an alternative to prompt engineering is covered in Tier 3. Evaluation methodology for measuring prompt quality is covered in the LLM Evaluation & Observability session.
