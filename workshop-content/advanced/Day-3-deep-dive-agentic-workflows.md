# Deep Dive into Agentic Workflows
**All examples: LODA (extends the ad-hoc client request triage from AI Systems & LLM Calls)**

**Platform note:** run the integration step in **n8n with Teams/Outlook**, if access is confirmed and working. If not, keep the integration step abstract (sketch it, describe the action, don't wire a live credential) rather than losing time to setup on the day.

---

## Agenda (1 day, 2×2hr sessions)

### Morning (2h)

| Time | Block |
|---|---|
| 0:00–0:15 | Recap of AI Systems & LLM Calls |
| 0:15–0:45 | 1. What a workflow is made of: LLM calls, integrations, custom tools |
| 0:45–1:15 | 2. How n8n represents these as nodes |
| 1:15–1:45 | Hands-on 1: sketch your LODA workflow (no canvas yet) |
| 1:45–2:00 | Recap |

### Evening (2h)

| Time | Block |
|---|---|
| 2:00–2:20 | 3. Method 1: choose your model empirically |
| 2:20–2:40 | 4. Method 2: build and test in chunks |
| 2:40–3:45 | Hands-on 2: build the workflow, node by node |
| 3:45–4:00 | Recap + close |

---
---

# Morning

## Recap (15 min)

One sentence each: what did your triage system prompt do, and what did you learn testing it against clean/ambiguous/gap inputs?

---

## 1. What a workflow is made of (30 min)

Every agentic workflow, no matter how complex, is some combination of three things wired together:

| Piece | What it is | Simple example |
|---|---|---|
| **LLM calls** | A system prompt + user prompt — exactly what you already built | "What should I wear today?" — the reasoning step |
| **Integrations** | A connection to an external app, each exposing its own actions and needing its own credential | Checking a live weather API before answering |
| **Custom tools** | Logic you write yourself, for when no integration covers what you need | A script that converts the API's raw response into "bring an umbrella" |

**Apply to LODA:** your triage system prompt from last session **is** the LLM call component. Nothing new to build there. What's missing to make it a workflow:

| Piece | LODA version |
|---|---|
| **LLM call** | Your existing triage prompt (or the reference one) |
| **Integration** | Something that starts the workflow (a request arriving) and something it acts on afterward (post the ticket somewhere — Teams channel, or an email via Outlook) |
| **Custom tool** | Only if a rule needs logic no integration provides — e.g. checking today's date to flag SLA breaches |

**The point to land:** you already built the hardest part last session. Today is about the wiring around it.

---

## 2. How n8n represents these as nodes (30 min)

| n8n node type | What it's for |
|---|---|
| **Trigger nodes** | What starts the workflow — schedule, webhook, form, chat message |
| **Action/app nodes** | The integrations — Teams, Outlook, and hundreds of others, each with its own actions |
| **Core nodes** | Logic that isn't an integration (see below) |
| **AI cluster nodes** | An AI Agent root node, with sub-nodes for the chat model, memory, and tools it can call |

**Core nodes worth knowing:**

| Node | Its one job |
|---|---|
| If | Split into two paths based on one condition |
| Switch | Split into more than two paths |
| Merge | Combine outputs from separate branches |
| Set | Shape/transform data, no external call |
| Code | Write JavaScript directly — where a custom tool lives |
| HTTP Request | Call any REST API with no dedicated node |
| Webhook | Expose a URL that starts the workflow |

**Apply to LODA — the shape of the triage workflow:**

```
Trigger (Teams message / Outlook email arrives with a client request)
  → AI Agent node (your triage system prompt as the system message,
     the incoming message as the user message)
  → If node (urgency == "high" or missing_info is non-empty?)
       → Teams/Outlook action: post to on-call channel / send email
       → else: log and continue
```

**AI cluster node structure, for the AI Agent piece:**
- Chat model sub-node: which LLM to call
- Memory sub-node: conversation history across turns (not needed for single-shot triage — worth noting when it *would* matter, e.g. if the agent needs to follow up)

**Why memory matters, briefly:** without it, an agent starts fresh every call — it can't recall what it already asked the client, what it tried last time, or what it learned from a past mistake. Memory is what lets an agent hold context across turns instead of treating every input as if it's the first. **LODA read:** your triage system prompt is single-shot by design — no memory needed. But if you extended it to *follow up* with a client ("you said no error message last time — do you have one now?"), that follow-up only works if the workflow remembers the first exchange. That's the moment memory stops being optional.
- Tool sub-nodes: what it can act on — e.g. a lookup of which product a client is provisioned on

---

## Hands-on 1: sketch your LODA workflow (30 min)

**No canvas yet.** In pairs, on paper or a shared doc:

1. Name the three components for your workflow: which LLM call (yours or the reference), which integration(s), any custom tool.
2. Sketch the node sequence — trigger → LLM call → decision → action — the way the diagram above is drawn.
3. Decide: does this need an If/Switch node? What's the condition?

**Presenter note:** don't let anyone touch n8n yet. A wrong sketch costs a pen mark; a wrong workflow costs a rebuild.

---

## Recap (15 min)

One pair shares their sketch. Ask the room: what would break first if this were built exactly as sketched?

---
---

# Evening

## 3. Choose your model empirically, not by assumption (20 min)

The same prompt run across different models won't perform identically. Decide by actually running the comparison on a task you care about — not by assuming the newest or most talked-about model is right for this job.

**Signal worth stating plainly:** if you don't have a clear idea which model to use yet, you're not ready to build the workflow. Go back and test the LLM call alone, in chunks, until you know which model handles it well. That testing *is* the empirical choice — not a separate step.

**Apply to LODA:** you already did a version of this in Hands-on 3 last session (clean/ambiguous/gap inputs). If your prompt handled all three well, you already know your model choice works for this task.

---

## 4. Build and test in chunks, never the whole workflow first (20 min)

```
Add one node → Test it alone → Add the next node → Test the connection
→ Add the next node → Test again → Only now, run the full workflow
```

**Why:** left alone, people build every node, run it once, and have no idea which one broke. Same discipline as testing a model alone before wiring it into a pipeline.

**Concretely, for the LODA workflow:**
1. Add the trigger. Confirm it actually fires — send a real test message.
2. Add the AI Agent node alone. Test the prompt against a sample input before connecting anything else.
3. Add the If node. Test both branches with different sample outputs.
4. Add the Teams/Outlook action. Test its credential and its one action in isolation — send one test message before it's wired to the rest.
5. Only now, run the full chain, end to end, on a real input.

---

## Hands-on 2: build the workflow, node by node (65 min)

**In pairs.** Build exactly in the order above — one node, one test, before the next goes in. Use your own triage prompt if it held up well in testing; otherwise use the reference prompt from AI Systems & LLM Calls.

**If n8n + Teams/Outlook access is working:** wire it for real — trigger, AI Agent node, If node, Teams/Outlook action, tested at each step.

**If integration access isn't available:** keep the trigger and AI Agent node live and testable in n8n; for the Teams/Outlook step, describe the action node you'd add and what you'd test, without wiring the credential.

**Checkpoint at the 35-minute mark:** hands up — who has the trigger + LLM call tested and working alone? Anyone who doesn't, a coach goes to them next, before they add anything further.

---

## Recap + close (15 min)

1. Which node broke first, and was it the prompt, the trigger, or the connection?
2. What's the one thing you'd test differently if you built another workflow tomorrow?
