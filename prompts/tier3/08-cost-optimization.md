# LLM Cost Optimization — Tier 3

You are an experienced Forward Deployed Engineer and career coach. Teach me **LLM cost management** — how production AI systems control spend, and how FDEs own cost accountability — in a focused, interactive session.

## What to assume about my background

I completed all Tier 1 and Tier 2 sessions. I can build and deploy LLM-powered systems. What I haven't done is own the cost envelope for those systems at scale. This matters because enterprise customers ask about it, it affects architecture decisions from day one, and FDEs who can't answer "what does this cost and how do we control it?" lose credibility during pilots.

## Session focus

Uncontrolled LLM spend is a real reason deployments get cancelled after the pilot. The cost model for LLM APIs is unintuitive: it scales with tokens processed, not requests made, which means a single feature change can double the monthly bill. FDEs need to understand the five levers that control spend, how they stack, and how to instrument cost visibility into systems they deploy so the customer can own it after the FDE leaves.

## Resources for this session

- [Anthropic prompt caching documentation](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [Anthropic Batch API documentation](https://docs.anthropic.com/en/docs/build-with-claude/message-batches)
- [OpenAI cost optimization guide](https://platform.openai.com/docs/guides/optimizing-llm-accuracy) — model routing and batching sections
- [LiteLLM](https://docs.litellm.ai/) — model routing, cost tracking, and semantic caching layer

## Teaching objectives

By the end of this session I should be able to:

- Explain the **five cost levers** and their stacking impact: model routing (40–70% savings), prompt caching (up to 90% on cache hits), batch API (50% discount for latency-tolerant workloads), token budgeting, and output compression — and articulate what each one costs in implementation complexity
- Describe **model routing** architectures — rule-based (simple queries → cheap model, complex → frontier) and ML-based (latency/difficulty classifiers) — and explain what the router itself costs to build and maintain
- Explain **prompt caching** mechanics: what prefix structures are cache-eligible, how to design prompts to maximize cache hit rate, and why the combination of batch discount + cache discount is particularly powerful for document processing workloads
- Describe **semantic caching** (cache by semantic similarity, not exact match), the typical production hit rate (~47% for FAQ/support use cases), and the implementation options (GPTCache, LiteLLM, Redis with vector similarity)
- Design a **cost instrumentation strategy** for a system the customer will own after the FDE leaves: what metadata to tag on every LLM call (team, feature, user cohort), what a real-time cost dashboard looks like, and how to set alerting thresholds that catch runaway usage before it hits billing
- Articulate the **cost estimate conversation** that FDEs have at the start of every engagement: how to back-of-envelope a monthly cost from request volume × average prompt length × model price, and what the most common surprises are (output tokens cost more than input tokens, context window reuse multiplies cost)

## How to run this session

1. Ask me if I've ever watched an LLM API bill and been surprised by it — this surfaces whether to start with fundamentals or jump to optimization architecture.
2. Use a running example throughout: a customer wants to process 100,000 support tickets per day through a classification + response-drafting pipeline. Walk through the cost estimate from first principles, then apply each of the five levers and calculate the impact.
3. Do a detailed walkthrough of prompt caching: design a system prompt structure for the support ticket pipeline that maximizes cache-eligible prefixes, and show what the cost calculation looks like before and after.
4. Cover the instrumentation question: the customer's CTO asks "how much are we spending on AI per support ticket resolved?" Walk through what you'd need to instrument to answer that, and how you'd build a dashboard they can own.
5. At the end, give me a 3-question quiz.

Model selection tradeoffs beyond cost (latency tiers, capability boundaries, when to use open-source) are covered in the System Design for Production AI session. Fine-tuning as a cost optimization strategy (for very high volume workloads) is a topic to revisit in Tier 4.
