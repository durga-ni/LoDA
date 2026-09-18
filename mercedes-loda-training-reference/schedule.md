# LoDA Training, Mercedes Team

## Programme Schedule

This is the high-level index: titles and session flow only. Full detail, mechanism, resources, and diagrams for each day live in their own files, "From Prompting to Workflows" under `from-prompting-to-workflows/`, "Advanced AI" under `advanced-ai/`.

---

## Standing structure (applies across every day of the programme)

- Every day opens with a short recap of what was covered and done the previous day, before moving into the new day's content.
- At the end of every logical step of the programme, run a reflection session looking back on it.

---



## Track 1: From Prompting to Workflows (3 days)


| Day   | Topic                                       |
| ----- | ------------------------------------------- |
| Day 1 | Hands-On Prompting & Workplace Applications |
| Day 2 | Practice & Coaching on Real Use Cases       |
| Day 3 | Introduction to Agentic Workflows           |


**Day 1 session flow:**

- Opening (introductions, ice-breaker, engagement format, hopes and concerns, key pain points, norms, survey)
- Prompting-changes-by-context framing
- Using AI (identifying patterns, prompt size scales with the problem)
- Human-conversation analogy
- Building AI Systems & LLM calls (top techniques)
- Common mistakes

**Day 2 session flow:**

- Recap
- Use-case gathering
- Coach demo (drawing on Day 1)
- Practice in pairs (describe, don't prescribe), mid-session showcase from the coach-paired pair
- Outcome (skill, agent, or script)
- Close-of-day showcases (2-3 pairs)

**Day 3 session flow:**

- Recap
- Concept block (prompt to skill to workflow ladder, two ways to build the pattern)
- Hands-on build (same email example, parallel pair)
- Close-of-day showcases

Full detail: [Day 1](from-prompting-to-workflows/day-1.md) · [Day 2](from-prompting-to-workflows/day-2.md) · [Day 3](from-prompting-to-workflows/day-3.md)

---



## Track 2: Advanced AI (5 days)


| Day   | Topic                                              |
| ----- | --------------------------------------------------- |
| Day 1 | Deep dive on agentic workflows                     |
| Day 2 | Multi-agent orchestration                          |
| Day 3 | LLM fundamentals, benchmarking & local LLMs        |
| Day 4 | RAG fundamentals                                   |
| Day 5 | AI workflows for enterprise software development   |


**Day 1 session flow (Deep dive on agentic workflows):**

- Recap
- What any agentic workflow is actually made of (LLM calls, integrations, custom tools)
- How n8n represents these as nodes (trigger, action, core, AI cluster nodes)
- Method: choose your model empirically, not by assumption
- Method: build and test in chunks, never the whole workflow first
- Hands-on: build a real use case node by node, testing at each step

**Day 2 session flow (Multi-agent orchestration, concept portion ~2-3 hours):**

- Recap
- The orchestrator-worker pattern, with the organisation/team human analogy
- The tradeoff, stated honestly (performance gain vs token cost)
- Two failure stories, and the delegation-briefing fix
- Two named multi-agent patterns (manager, handoff)
- Reflexion, self-evaluation and improvement over successive attempts
- Evaluating a multi-agent system (LLM-as-judge on 3 simple metrics, plus human testers reading raw production queries)
- Rest of day: hands-on with an existing multi-agent example, extend rather than build from scratch

Full detail: [Day 1](advanced-ai/day-1.md) · [Day 2](advanced-ai/day-2.md)

**Day 3 session flow (concept portion ~1-2 hours, mostly verbal and whiteboard):**

- Morning: finish Advanced Day 2's practical first
- What is an LLM (trained on internet text, next-word prediction, one line on "Attention Is All You Need")
- Benchmarking, how models actually get compared (SWE-bench, AIME)
- Local LLMs: the three ways to use AI (API, RAG, fine-tuning), then hosting it yourself, why and when, and the tradeoffs
- Light experiment: run one small model locally with Ollama

Full detail: [Day 3](advanced-ai/day-3.md)

**Day 4 session flow:**

- Recap
- What RAG actually is, plus the SQL-query analogy
- Why naive RAG quietly fails
- Chunking techniques, fixed-size vs semantic, with a real example
- Contextual Retrieval, the fix, with the published numbers
- Hands-on, part 1: rewrite naive chunks with context
- Hands-on, part 2: build a real RAG pipeline in n8n

Full detail: [Day 4](advanced-ai/day-4.md)

**Day 5 session flow:**

*First half, one concept plus a capstone build:*
- Recap
- Explore, plan, implement, commit
- Capstone build: one real system combining an LLM call, a real integration, RAG, and ideally a second agent

*Second half, recap and reflection:*
- Closing image: the "AI" and "Human Developer" mindmaps side by side, as the final synthesis
- Closing reflection, three questions for the room

Full detail: [Day 5](advanced-ai/day-5.md)

---

