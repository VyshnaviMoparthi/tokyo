# System Design Prompt Kit

How to use it in the interview:
1. Ask the interviewer your clarifying questions first (users, data size, how often data changes, security, latency, budget). Write their answers down.
2. Copy **Prompt 1**, paste the problem statement and the answers where marked, and run it.
3. Read the output against the checklist at the bottom. Say out loud what you agree with and what you would change.
4. Use the follow-up prompts to go deeper on whatever the interviewer asks about.
5. Explain the final design in your own words. Never read the AI output aloud.

---

## Prompt 1: Main design prompt

```
You are a principal AI solutions architect helping me design a production GenAI system
in a technical interview. The panel will judge three things most: SCALABILITY,
EFFECTIVENESS (quality and accuracy) and SECURITY. Cost and reliability also matter.

PROBLEM STATEMENT:
<<< paste the problem statement exactly as given >>>

ANSWERS TO MY CLARIFYING QUESTIONS (may be partial):
<<< paste what the interviewer told you: users, data sources, data size, update
frequency, access rules, latency, budget, cloud/hosting rules. Write "unknown" if not given >>>

DESIGN RULES (follow these unless the problem clearly says otherwise):
- Choose the right access method per data type: structured or numeric data that
  needs exact filtering, calculation or freshness is read through tools (SQL / APIs),
  NOT a vector store. Use RAG only for unstructured text where meaning matters.
- Enforce authorization in code: the user's identity comes from login (SSO), is
  injected into every tool call, and filters data at query/retrieval time. Never rely
  on the prompt for access control, and never let the LLM choose whose data it reads.
- Every number or fact in an answer must come from a tool or retrieved source.
  Include a post-generation check, citations, and an "I don't know" path.
- Respect data residency: if sensitive data must stay in the company environment,
  say which model hosting option satisfies it.
- Prefer the simplest design that meets the requirements. Single agent with tools
  before multi-agent; workflow before agent when the steps are fixed.

OUTPUT FORMAT (use these exact headings, be concise, use bullet points):

0. Assumptions - list every assumption you made where information was missing,
   with a sensible number (users, requests/day, data volume).

1. Clarifying questions I should still ask - top 5, most important first.

2. Capacity estimate - back-of-the-envelope math: users -> requests/day ->
   peak requests/second -> LLM calls and tokens per request -> rough daily cost.
   Data volume per source and how often it changes. Show the arithmetic.

3. Architecture overview - one paragraph, then a Mermaid flowchart
   (```mermaid flowchart TD ...```) showing users, auth, API, agent/orchestrator,
   tools, retrieval, data stores, ingestion jobs, LLM, monitoring.

4. Data handling per source - a table: source | type | how it's ingested |
   where it's stored | how it's queried (tool vs RAG) | freshness | why.

5. Query flow - numbered steps from user question to final answer.

6. Actions (only if the system must change things) - tools/MCP servers,
   user confirmation before writes, permission model, idempotency.

7. SECURITY - authentication, authorization (row, column, document level and
   where each is enforced), PII handling, data residency and model hosting,
   prompt injection (direct and indirect) defenses, secrets management,
   encryption, audit logging, least privilege for tools.

8. SCALABILITY - what is the first bottleneck at 10x and 100x load and how to
   remove it: stateless API + autoscaling, caching (semantic, embedding, prompt),
   model routing, async queues for heavy jobs, index/partition strategy for the
   data stores, rate limits, timeouts, retries with backoff, fallback model.

9. EFFECTIVENESS - how quality is measured: golden dataset (size and how it is
   built), retrieval metrics (recall@k, MRR), routing/tool-selection accuracy,
   answer correctness and faithfulness (LLM-as-judge + human review), latency
   targets (p95), user feedback, regression tests in CI, online monitoring.

10. Reliability and operations - tracing/observability, alerting, prompt and
    model versioning, canary rollout and rollback, failure modes and what the
    user sees when each one fails.

11. Cost - main cost drivers and the top 4 ways to reduce them without
    hurting quality.

12. Trade-offs - a table: decision | chosen option | alternative | why chosen.

13. Phased plan - MVP (what ships first), phase 2, phase 3.

14. Top risks - 5 risks with mitigation.

15. 60-second summary - a spoken summary I can say to the panel in plain words.

After writing the design, review it yourself as a strict senior interviewer:
list anything that violates the DESIGN RULES or the stated requirements, and fix it
in the design before showing me the final version.
```

---

## Follow-up prompts

**Challenge the design (use right after Prompt 1)**
```
Act as a skeptical interviewer. Find the 5 weakest points in this design for
scalability, effectiveness and security. For each, explain the risk and give
the fix. Be specific to this problem, not generic.
```

**Scale it up**
```
The load grows 10x in users and 100x in data. Walk through what breaks first,
in order, and the exact change for each. Include updated capacity numbers.
```

**Security deep dive**
```
Go deeper on security only. Walk through 3 attack scenarios (a user trying to see
another team's data, a prompt injection hidden inside a document or tool result,
a leaked API key). For each: how it is prevented, detected, and contained.
```

**Evaluation deep dive**
```
Design the evaluation plan in detail: how to build the golden dataset, the exact
metrics with target values, how evaluation runs in CI, and how we monitor
quality in production after launch.
```

**Explain it simply**
```
Explain this design as if I'm drawing it on a whiteboard for a non-technical
business stakeholder. 6 sentences maximum, no jargon.
```

**Predict the panel's questions**
```
List the 10 follow-up questions a senior interviewer is most likely to ask about
this design, with a 2-3 sentence answer for each.
```

**Reduce cost**
```
The monthly LLM bill is 3x over budget. Give the changes in order of impact,
with the expected saving for each and how we confirm quality didn't drop.
```

---

## Checklist: read the AI's answer against this

Say out loud what you find. Catching a mistake impresses more than a perfect output.

- [ ] Numbers and live data go through tools/SQL, not embeddings.
- [ ] Access is enforced in code at query or retrieval time, not in the prompt or after generation.
- [ ] Row, column and document level rules are all covered (if the problem has them).
- [ ] Data residency rule is respected by the model hosting choice.
- [ ] There is a real capacity estimate with numbers, not just "it scales."
- [ ] Evaluation has a golden set, named metrics, and runs before every release.
- [ ] Failure handling: timeouts, retries, fallback, what the user sees.
- [ ] Old or stale data is handled (versions, sync jobs, freshness shown to users).
- [ ] Write actions need confirmation and use the user's own permissions.
- [ ] There is an MVP: not everything is built on day one.
- [ ] Nothing is generic. The design uses the numbers from the problem statement.
