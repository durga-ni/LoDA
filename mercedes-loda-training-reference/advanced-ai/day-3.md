# Advanced Day 3: LLM Fundamentals, Benchmarking & Local LLMs
**Track:** Advanced AI

---

## Scope note

This day is deliberately kept at awareness level, not depth. The room does not need to be able to build an LLM, so they are not being taught to. The goal is just enough of a mental model that "which model, and why" and "should we run this ourselves" stop being black-box questions. Explain this day mostly verbally and on a whiteboard, this is not meant to be read off a slide.

**Morning:** finish whatever practical from Advanced Day 2 did not get completed, this day's own concept material only needs one to two hours, not the whole day.

---

## Part 1: What is an LLM

**Purpose:** give the room one honest, simple mental model, so nothing later in the day is mysterious.

- Trained on a vast amount of text already out there, Wikipedia, books, forums, basically all the text available.
- What it is actually doing, at every single step, is predicting the next word. Given everything so far, what is the single most likely next word. That is the whole mechanism, repeated over and over to produce a full answer.
- One line worth naming directly: the paper "Attention Is All You Need" (2017) is the one that changed everything, it introduced the architecture, built around something called attention, that every modern LLM is still built on. Attention is what lets the model weigh which of the earlier words actually matter most for predicting the next one, rather than treating everything before it equally.

---

## Part 2: Benchmarking, how models actually get compared

**Purpose:** once the room knows what an LLM is, the natural next question is why one model gets chosen over another. Benchmarking is the honest answer, not vibes, not marketing.

Models get compared on benchmarks, standard tests everyone runs the same way, so results are actually comparable model to model. Two worth knowing by name, without going deep on either, point the room to look further on their own if they want:

- **SWE-bench**, a coding benchmark built from real GitHub issues, the model has to actually generate a patch that fixes the real problem.
- **AIME**, a hard, competition-level maths benchmark, testing multi-step mathematical reasoning, not simple arithmetic.

The pattern to land: different benchmarks test different things, a model that leads on a coding benchmark is not automatically the best choice for a maths-heavy task, and vice versa. Choosing a model means matching the benchmark to what you actually need it to do.

---

## Part 3: Local LLMs

**Purpose:** local is one option among several, not the default. Lay out all the ways to actually use AI first, so "run it yourself" is understood as a deliberate choice with real tradeoffs, not the "advanced" or "correct" option by default.

### Three ways to actually use AI

1. **Call a hosted model over an API**, Claude, ChatGPT, and others, paying per token used. The default for most day-to-day use.
2. **RAG**, when you already have a large set of your own documents or data you want the model working from, covered properly on Day 4.
3. **Fine-tuning**, one line worth knowing: adjusting the model's own weights on your own data, so its behaviour itself changes, not just what it is handed at the time.

### Or, host the model yourself

Past roughly 8 billion parameters, running a model locally on a normal laptop stops being realistic. Bigger models, 70 billion parameters and up, genuinely produce better output, but they need real infrastructure to run, not a laptop.

Worth stating as a personal, practitioner's opinion, not a universal rule: running LLMs locally usually costs more than it looks like on paper, a skill set most teams do not already have, an ongoing server cost, and more that can quietly go wrong compared to calling a hosted API.

Where local still genuinely earns its place: governance and security-sensitive situations, some organisations self-host on infrastructure they control, a dedicated instance on AWS, for example, specifically because the data cannot leave their own environment.

### Experiment

As a light, hands-on taste, not the focus of the day: run one small model locally.

```
ollama pull llama3:8b
ollama run llama3:8b
```

---

## Resources

- [Andrej Karpathy, Deep Dive into LLMs like ChatGPT](https://www.youtube.com/watch?v=7xTGNNLPyMI), a general-audience, verbal-and-visual explanation of how these models actually work, worth drawing on directly for this day's delivery style
- [Attention Is All You Need (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762), the paper that introduced the transformer architecture
- [SWE-bench](https://www.swebench.com), the coding benchmark referenced in Part 2
- [Ollama](https://ollama.com), the tool for the local-model experiment
