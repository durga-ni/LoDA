# Day 3 — Trainer Reference
## Running Example: Meeting Email Responder — Tools & Memory

**Purpose:** This document maps the Meeting Email Responder to every agenda block on Day 3. Use it as your live demo script and trainer prep sheet. Candidates work on LODA; you explain each concept on this example first.

**The day in one line:** on Day 1 and Day 2 the prompt was judged on what it *said*. Today it is judged on what it *does* — it gets hands (tools) and a past (memory).

**The two workflows this day runs on** (both in `examples-n8n/example-email-responder/`):

| File | What it is |
|---|---|
| `1_Email Responder.json` | The Day 1–2 prompt, wired end to end. **No tool.** |
| `2_Email Responder - ReAct.json` | The same workflow with **one tool added**, on a second agent |

The difference between those two files *is* the morning's concept. Have both imported and runnable before the session starts.

**Platform note:** run this in n8n with live Gmail/Calendar credentials if access is confirmed and working. If not, keep the integration steps pinned — n8n's pin-data feature lets you run the agent and the tool against saved sample data with no live credential. Do not lose the day to OAuth setup.

---

## Agenda (1 day, 2×2hr sessions)

### Morning (2h) — Tools

| Time | Block |
|---|---|
| 0:00–0:10 | Recap Day 1 & 2 + today in one line |
| 0:10–0:30 | 1. What a tool is — you don't call it, the model does. A tool description is a prompt. |
| 0:30–1:00 | 2. Generic vs custom tools, on the email example |
| 1:00–1:25 | Hands-on 1: write tool contracts — paper, no canvas |
| 1:25–1:35 | 3. Method: build and test in chunks · choose your model empirically |
| 1:35–1:50 | Hands-on 2: wire one tool, test it alone before anything else connects |
| 1:50–2:00 | Recap |

### Evening (2h) — Memory

| Time | Block |
|---|---|
| 2:00–2:20 | 4. Why memory: does the right answer depend on something that happened earlier? |
| 2:20–2:45 | 5. Three kinds — none / conversation buffer / persistent — each on the email example |
| 2:45–3:10 | 6. How memory fails: stale, leaked across senders, poisoned |
| 3:10–3:50 | Hands-on 3: add memory — part A make the follow-up work, part B break it |
| 3:50–4:00 | Recap + close |

---

## Before the day — trainer prep checklist

- [ ] Both example workflows imported into n8n and opened side by side.
- [ ] Run `1_Email Responder` once. Note where it succeeds and where it stops.
- [ ] Run `2_Email Responder - ReAct` once. Watch the ReAct agent's tool call in the execution log — that log is your main visual aid all morning.
- [ ] **Verify one thing specifically:** in `1_Email Responder`, the `Create an event2` node is a plain Google Calendar node, but its `start` and `end` use `$fromAI(...)`. `$fromAI` is designed to be filled by an agent when the node is attached as a tool. Run it and see what those fields actually resolve to. Whatever you find is your opening for Block 1 — either it works and you explain why, or it doesn't and you have just shown the room, in one node, why a tool has to be *connected as a tool*.
- [ ] Have the test emails from Appendix C ready to send, or pinned as sample data.
- [ ] Bring blank tool contract sheets (Appendix B) — printed, one per pair.

---
---

# DAY 3 — MORNING (2h) — Tools

> **Shape of the morning:** first you see the model choosing to call something. Then you learn the two kinds of thing it can call. Then you design one on paper before anyone touches the canvas.

---

## Recap + today in one line (0:00–0:10)

### Recap (5 min)

One sentence per pair: which guardrail did you add yesterday that you would not have thought of on your own?

### Today in one line (5 min) — say this to the room

"Yesterday your prompt was judged on what it said. A wrong answer was a wrong field in a JSON object. Today your prompt gets hands. A wrong answer becomes a meeting in somebody's calendar, or an email that has already left the building. Everything you learned about guardrails still applies — but now it applies to actions, not to text."

Put the two workflow files on the projector, side by side, and leave them there.

---

## Block 1: What a tool is — you don't call it, the model does (0:10–0:30)

### Explain with this example

Open `1_Email Responder` on the projector. Walk the chain out loud:

```
Manual trigger
  → Gmail: Get many messages (unread, limit 1)
  → AI Agent (your Day 1–2 triage prompt, Gemini as the chat model)
  → Code in JavaScript (parse the model's JSON, fall back safely if it isn't valid)
  → Gmail: Send a message (sendAndWait — a human approves)
  → If (approved?)
  → Google Calendar: Create an event
```

Then ask the room one question: **"Where in this workflow does the model decide to check the calendar?"**

It doesn't. The calendar node sits in the main flow after the If. It runs because a wire says it runs. The model produced text; n8n did everything else.

**The point to land:** this is a workflow. It is a good workflow. But there is no tool in it, because nothing here is *chosen* by the model.

### Now open the second file

`2_Email Responder - ReAct` is the same file plus one idea. Everything up to the approval step is identical. After the If, it adds:

```
If (approved)
  → ReAct Agent  ←── ai_tool ──  Get many events in Google Calendar
  → Google Calendar: Create an event
```

Look at the connection type. The calendar node is not wired to `main`. It is wired to **`ai_tool`**.

**That single connection is the whole concept.** On a `main` connection, a node runs because the flow reached it. On an `ai_tool` connection, a node runs *only if the model decides to call it*, with arguments the model decides to pass.

| | `main` connection | `ai_tool` connection |
|---|---|---|
| Who decides it runs | Your wiring | The model |
| Who decides the arguments | Your expressions | The model |
| Runs how many times | Once, in order | Zero, once, or several times, in whatever order the model chooses |
| When it's the right choice | The step always has to happen | The step depends on what the model concluded |

### A tool description is a prompt

If the model chooses, the only thing steering that choice is what the model can read about the tool: its **name** and its **description**.

Look at the ReAct agent's system prompt in the file:

```
2. ACT: Use the "check availability" tool to see if that exact slot is free.
```

Now look at the node it is talking about. The node is called **`Get many events in Google Calendar`**.

**Live demo (5 min, and it is the best five minutes of the morning):**
1. Run it as-is, a few times. Watch the execution log — does the agent reach for the tool every time?
2. Rename the node to `check availability`. Run again, same input.
3. Ask the room what changed and why.

**Key line to say:** "The model does not read your workflow. It reads a list of tool names and descriptions, and picks one. A vague tool description is not a documentation problem — it's a routing bug."

Same rule shows up one more time in the file, in the `Create an event` node:

```
$fromAI("event_start", "the final confirmed meeting start time in ISO 8601")
```

The second argument is a sentence written for the model to read, so it knows what to put in that field. Nobody typed that timestamp. **Descriptions are how you program a tool call.**

**Two ways to fill a tool's parameters, both visible in this one file:**

| Way | In the file | Use when |
|---|---|---|
| **Pin it with an expression** | The availability tool's `timeMin` / `timeMax` are set from the incoming item | You already know the value and don't want the model inventing one |
| **Let the model fill it** | `Create an event` uses `$fromAI(...)` | The value depends on what the model worked out |

Pinning is a guardrail. It is the tool-level version of yesterday's "never infer a value that isn't there."

---

## Block 2: Generic vs custom tools (0:30–1:00)

### The two kinds

| | Generic tool | Custom tool |
|---|---|---|
| What it is | A prebuilt integration action | Logic you write yourself |
| In n8n | Any app node attached on `ai_tool` — Gmail, Calendar, Slack, hundreds of others | Code node, HTTP Request node, Workflow-as-tool |
| What it costs you | A credential and a choice of action | You have to write it, and you own it when it breaks |
| When it's right | An integration already does the thing | Nothing covers it, or you need a *guaranteed* answer |

### On the email example

| Need | Kind | How |
|---|---|---|
| Is that slot free? | **Generic** | Google Calendar → Get many events, attached as a tool — exactly what `2_Email Responder - ReAct` does |
| Book the meeting | **Generic** | Google Calendar → Create an event |
| Send the reply | **Generic** | Gmail → Send a message |
| Is this inside working hours, Mon–Fri 09:00–17:00 CET? | **Custom** | No integration knows your company's hours. This is a few lines in a Code node |
| Is the model's JSON actually valid? | **Custom** | A Code node that parses it, and returns something safe if it can't |

**Start with generic.** People reach for a Code node far too early. The rule to say out loud: *if an integration already does it, you are not writing code to do it again.* You write custom logic for your own policy, and for hard guarantees.

### The custom logic already in the file — and why it is not a tool

`1_Email Responder` already contains a Code node:

````javascript
const raw = $input.item.json.output;

let parsed;
try {
  const cleaned = raw.replace(/```json\n?|```/g, '').trim();
  parsed = JSON.parse(cleaned);
} catch (e) {
  parsed = {
    sender: "unknown",
    topic: null,
    proposed_time: null,
    urgency: "medium",
    decision: "need_more_info",
    missing_info: ["Could not parse AI response: " + e.message],
    draft_response: null,
    event_start: null,
    event_end: null
  };
}

return { json: parsed };
````

Walk the room through what it does: it takes whatever the model emitted, strips the code fences models like to add, and tries to parse it. If that fails, it does not crash and it does not pass garbage downstream — it returns a well-formed object whose decision is `need_more_info`, with the reason recorded.

Now the important part. **This node is on a `main` connection, so it is not a tool.** The model cannot call it, and — more to the point — the model *cannot skip it*. It runs on every single execution, whatever the model did.

**Key line to say:** "Yesterday I told you a guardrail narrows the range of answers but cannot make a language model deterministic, and that where you need a hard guarantee it has to be code outside the prompt. This node is that promise, kept. Ten lines of JavaScript, and a malformed response can no longer reach your calendar."

**The distinction worth writing on the board:**

| | Custom **tool** | Custom **logic** |
|---|---|---|
| Connection | `ai_tool` | `main` |
| Who runs it | The model, if it decides to | The flow, always |
| Good for | Something the model should reach for when it judges it's needed | A check that must never be skipped |

### Read tools and write tools

The single most useful way to sort your tools:

| | Read tool | Write tool |
|---|---|---|
| Example here | Get many events | Create an event · Send a message |
| A wrong call costs | A wrong answer | A real event, a sent email, an external side effect |
| If called twice | Nothing | **Two invites** |

**Blast radius is already handled in this workflow, and nobody has named it.** `Send a message` is `sendAndWait` with double approval, and it gates everything downstream. A human sees the triage, approves, and only then does anything get written to a calendar.

**Key line to say:** "Read tools you can hand out freely. Write tools you gate. This workflow gates them with a human in the loop — that is a design decision somebody made, not a default, and you should make it deliberately every time."

---

## Hands-on 1: Write tool contracts (1:00–1:25)

**In pairs. Paper or a shared doc. Nobody opens n8n.**

**Presenter note:** the instinct is to start dragging nodes. Don't allow it. A tool that is wrong on paper costs a pen mark; a tool that is wrong on the canvas costs a rebuild plus the credential setup. This is the same "sketch before you build" discipline as Day 1's "no canvas yet."

Design **three** tools for the Meeting Email Responder. At least one generic, at least one custom. For each, fill in this contract:

| Field | What goes here |
|---|---|
| **Name** | What the model sees in its tool list. Short, verb-first |
| **Description** | One or two sentences, written *for the model*. What it does, what it returns, and when to use it |
| **Inputs** | Each parameter, its type and format. Mark each as **pinned** (you set it) or **model-filled** (`$fromAI`) |
| **Output** | What comes back, in what shape |
| **Read or write** | Read / Write |
| **Called twice** | What happens? If the answer is bad, what stops it? |

A worked example and blank sheet are in Appendix B.

**Checkpoint at the 15-minute mark:** ask two pairs to read out just their *descriptions*, and ask the room, "If that were the only thing you could see, would you know when to call it?"

---

## Block 3: Method — build and test in chunks · choose your model empirically (1:25–1:35)

### Build and test in chunks, never the whole workflow first

```
Add one node → Test it alone → Add the next node → Test the connection
→ Add the next → Test again → Only now, run the full workflow
```

**Why it matters:** left alone, people build all seven nodes, run once, and have no idea which one broke. This is unit testing before integration testing, applied to a canvas instead of a codebase.

**Concretely on this workflow:**
1. Trigger — fire it, confirm a message actually arrives.
2. AI Agent alone — test the prompt against a pinned sample email before anything is connected after it.
3. Code node — feed it a *deliberately broken* model output and confirm the fallback fires.
4. Each tool, alone — call it with fixed parameters, outside the agent, before the agent is allowed to choose it.
5. Only now, the whole chain, end to end, on a real email.

**Say this:** "Step 4 is the one people skip. If you attach a tool you have never run, and the agent doesn't call it, you now have two possible causes — a bad description or a broken tool — and no way to tell them apart."

### Choose your model empirically

The same prompt across different models does not behave the same. Decide by running the comparison on a task you actually care about, not by picking the newest name.

**On this workflow:** both agents use `gemini-3.5-flash-lite`. That is a choice, and it is a reasonable one for a cheap high-volume triage step — but it should be a tested choice, not an inherited one. Tool-calling reliability in particular varies a lot between models: a model that writes perfect JSON may still be sloppy about *when* to reach for a tool.

**Signal to state plainly:** if you don't yet know which model to use, you are not ready to build the workflow. Go back and test the LLM call on its own until you do. That testing *is* the model choice.

---

## Hands-on 2: Wire one tool, test it alone (1:35–1:50)

**In pairs, on the canvas. One tool. Not three.**

Start from `1_Email Responder` (the one with no tool) and add the availability check yourself — ending up at what `2_Email Responder - ReAct` already does.

1. Add a **Google Calendar Tool** node — the *tool* variant, not the regular one.
2. **Test it alone first.** Run it with fixed dates and confirm it returns events. Do not connect it yet.
3. Connect it to the agent on the `ai_tool` port.
4. Write its description from your Hands-on 1 contract.
5. Run the agent on one email and **open the execution log**. Did it call the tool? With what arguments?

**If it didn't call the tool:** good — that is the lesson, not a failure. Change the description, not the model. Run again.

**Presenter note:** keep everyone on the availability tool. The temptation is to add the create-event tool too, which is a write tool, which means the room starts creating real calendar entries fifteen minutes before a break. Write tools come after the guardrail conversation, not before.

---

## Recap (1:50–2:00)

Three questions:
1. In one sentence: what makes something a tool rather than just a node?
2. Whose tool description got called reliably, and what did it say?
3. Which of your three contracts is a write tool, and what gates it?

---
---

# DAY 3 — EVENING (2h) — Memory

> **Shape of the evening:** first, whether you need memory at all. Then the three kinds. Then the three ways it fails — and one of those failures is the same bug you guardrailed yesterday, reappearing one level down.

---

## Block 4: Why memory (2:00–2:20)

### Start with what's missing

Put either workflow back on the projector and point at the AI Agent node. It has a chat model attached underneath it. The **memory port next to it is empty.**

Then point at the Gmail node: `getAll`, `limit: 1`, filter `unread`.

**Say this:** "One unread email, one run, no memory. This workflow starts from zero every single time — that is not an oversight, it is the design. And for triage, it is the right design."

### The one question

**Does the correct answer depend on something that happened earlier?**

If no, memory is cost and risk with nothing in return. Every triage decision in this workflow is a function of one email plus fixed rules. That needs no memory.

Memory stops being optional the moment the workflow has to **follow up**:

```
Email 1 (Monday)  — "Can we meet next week to go through the numbers?"
                    → decision: need_more_info, missing: proposed time

Email 2 (Tuesday) — "Tuesday 3pm works for me."
```

Without memory, the agent reads email 2 alone: no topic, no context, and it asks Marcus what the meeting is about — the thing he already told it yesterday. Annoying from a person. Unacceptable from a system that is meant to save time.

**Key line to say:** "Memory is not a feature you switch on because agents have it. It is the answer to one question, and if the answer is no, leave the port empty."

### State is not memory

One distinction that saves a lot of confusion later. Look at what the ReAct agent receives in `2_Email Responder - ReAct`:

```
Sender: {{ $json.sender }}
Topic: {{ $json.topic }}
Decision: {{ $json.decision }}
Proposed event_start: {{ $json.event_start }}
```

It knows the sender and topic. It has no memory. Those facts were **handed to it down a wire** by the previous step, as text in its prompt.

| | State | Memory |
|---|---|---|
| Where it comes from | Passed in explicitly by you | Recalled by the agent itself |
| How long it lives | This execution | Across turns, or across runs |
| Who controls it | You | Configuration, and whatever went in before |

**Say this:** "If you can pass it in, pass it in. Explicit state is easier to test, easier to debug, and cannot be poisoned. Reach for memory when you *can't* pass it in — because you don't know, at build time, what the agent will need to remember."

---

## Block 5: Three kinds of memory (2:20–2:45)

### On the email example

| Kind | What it does | On this workflow | What it costs |
|---|---|---|---|
| **None** | Every run starts blank | What is built today — each unread email judged on its own | Cannot follow up. Repeats questions already answered |
| **Conversation buffer** | Keeps the last N turns of *this* conversation | Marcus replies "Tuesday 3pm works" and the agent still knows the topic from Monday | Window fills up and old turns drop out **silently** — no error, the agent just quietly stops knowing |
| **Persistent** | Facts that survive across runs, in a store | "Marcus prefers afternoons." "Priya's team is on-call and their outages are always high urgency" | Facts go stale. Storage to run. And you now hold personal data you have to justify |

### How this looks in n8n

Attach a **Simple Memory** (buffer window) sub-node to the AI Agent's memory port. Two settings matter, and both are teaching moments:

| Setting | What it means | The trap |
|---|---|---|
| **Session ID / key** | Which conversation this is | Default behaviour assumes a chat trigger. This workflow is triggered by *email*, so there is no session — **you must decide what the key is**, and that decision is the whole of Block 6 |
| **Context window length** | How many turns are kept | Too small and it forgets mid-thread; too large and you pay for, and expose, more history on every call |

**Scope fence to state out loud:** persistent memory is a store you look things up in. When that store gets big enough that you need to search it by meaning rather than by key, you are doing retrieval — and that is RAG, which is its own day later in this track. Today stops at: key it correctly, and know what's in it.

---

## Block 6: How memory fails (2:45–3:10)

> This is where advanced earns its name. Everyone can attach a memory node. The value is knowing the three ways it turns on you.

### Failure 1 — Stale

Persistent memory stored "Marcus prefers afternoons" six months ago. Marcus has since changed teams and now runs a standup every afternoon. The agent keeps proposing 3pm and keeps being declined — and nothing in the output ever looks wrong.

**The property to name:** memory has no expiry unless you give it one. A fact written once is asserted forever.

**Mitigations:** timestamp what you store · store the *source*, not just the conclusion ("said on 14 March" beats "prefers afternoons") · prefer facts that don't rot (a working-hours policy) over facts that do (a personal preference).

### Failure 2 — Leaked across senders

This is the important one, so demo it rather than describe it.

Attach a buffer memory with **one fixed session key** for the whole workflow. Now run two emails through:

```
Email A — Priya: "Production outage on the payments service,
                  need 15 minutes tonight."

Email B — Nina:  "Fancy grabbing lunch Thursday?"
```

The agent handling Nina's email can now see Priya's outage in its history. It may mention it. It may raise Nina's urgency because the recent history is full of incident language. Either way, **a fact from one sender has reached another sender's reply.**

**Say this:** "You already fixed this bug yesterday. Guardrail G5 — no leakage across emails. You wrote it as a rule in the system prompt and it worked. Today you have reintroduced exactly the same bug from underneath, at infrastructure level, where no prompt rule can reach it. The prompt guardrail is still there. It still says don't leak. And the leak happens anyway, because the leaking data is being handed to the model as its own memory."

**The fix:** the session key must be the thing that defines the conversation — the email thread ID, or the sender. One key per conversation, never one key per workflow.

**The general lesson, and write this one on the board:** *a guardrail written at one level does not protect the level below it.*

### Failure 3 — Poisoned

Bring back Card 4 from yesterday: the forwarded thread carrying an instruction aimed at the assistant reading it.

Yesterday, that injection broke one run. One bad JSON object, one wrong field, and the next email started clean.

Write that same email into memory, and it is no longer one run. It sits in the agent's history and is re-read as trusted context on every subsequent call. **A one-shot attack becomes a persistent one.**

And now stack this morning on top: this agent has a write tool downstream. An injected instruction that survives in memory, in a workflow that can create calendar events and send mail, is an instruction with hands and a memory.

**Mitigations:**
- Memory is not a place you put raw input. Store the *extracted, validated* result — the output of the Code node, not the email body.
- Keep write tools behind the approval step, as this workflow already does.
- If a run gets a malformed or suspicious result, don't write it to memory at all.

**Key line to close the block:** "Memory makes an agent better at its job and better at repeating your mistakes. Both, always, at the same time."

---

## Hands-on 3: Add memory, then break it (3:10–3:50)

**In pairs, on your workflow. Two parts. Do not skip to part B.**

### Part A — Make the follow-up work (20 min)

1. Attach a **Simple Memory** node to your AI Agent.
2. Set the session key to the **email thread** (or the sender address — state which you chose and why).
3. Send **Thread email 1** (Appendix C). Record the output.
4. Send **Thread email 2** from the same sender.
5. **Check:** did the agent still know the topic, or did it ask again?

If it asked again, debug it in this order: is the memory actually connected · is the key the same for both emails · is the context window long enough.

### Part B — Break it (20 min)

1. Change the session key to a **fixed string** — `meeting-agent`, anything constant.
2. Run **Priya's outage email**, then **Nina's lunch email**, in that order.
3. Read Nina's output carefully. What leaked? Urgency? A mention? Tone?
4. Record it on the sheet:

| What I changed | What I expected | What I actually got |
|---|---|---|
| Session key → fixed | Nina's email judged on its own | |

5. Now fix it. Put the key back. Re-run both. Confirm Nina's output is clean.

**Facilitation note — if nothing visibly leaks.** With a short context window and two very different emails it may not show on the first try. Do not let the pair conclude the bug isn't real. Three escalations:
- Run three or four emails before Nina's, so the history is genuinely full of the other conversations.
- Ask them to look at what was actually *sent to the model*, not just what came back. The leak is in the input either way.
- Then ask the closing question: "Would you sign this off to run unattended against a shared inbox, where the previous sender is whoever happened to email first?"

**Say this:** "Not leaking today because the window was short is not the same as not leaking. Yesterday's line still holds — right by luck and right by rule look identical until the day they don't."

---

## Recap + close (3:50–4:00)

Four questions:
1. What makes something a tool rather than a node? *(the model chooses it)*
2. What steers which tool the model picks? *(the name and the description — nothing else)*
3. What is the one question that decides whether you need memory? *(does the right answer depend on something earlier?)*
4. Which guardrail from yesterday came back today, and why couldn't the prompt stop it?

**Closing line:** "Two days ago you wrote a prompt. Yesterday you broke it and guarded it. Today it can act and it can remember. Everything from here is about what happens when there is more than one of them — which is tomorrow."

---
---

# Appendix A — The two reference workflows, node by node

### `1_Email Responder.json` — no tools

| # | Node | Type | What it does |
|---|---|---|---|
| 1 | When clicking 'Execute workflow' | Manual trigger | Starts the run by hand |
| 2 | Get many messages | Gmail | `getAll`, `limit: 1`, unread only |
| 3 | AI Agent | LangChain Agent | The Day 1–2 triage system prompt. User message carries the current date/time, the sender, and the email body |
| 4 | Google Gemini Chat Model | Chat model sub-node | `gemini-3.5-flash-lite` |
| 5 | Code in JavaScript | Code | Strips code fences, parses JSON, returns a safe `need_more_info` object if parsing fails |
| 6 | Send a message | Gmail | `sendAndWait`, double approval — a human reviews the triage before anything is written |
| 7 | If | If | `{{ $json.data.approved }}` is true |
| 8 | Create an event2 | Google Calendar | Creates the invite. `start` / `end` use `$fromAI(...)`; attendee and summary come from the original message |

**What to point out:** no `ai_tool` connection anywhere. The model produces text and nothing else.

### `2_Email Responder - ReAct.json` — one tool added

Everything above, unchanged, plus:

| # | Node | Type | What it does |
|---|---|---|---|
| 9 | ReAct Agent | LangChain Agent | Receives the triage result as text. Instructed to THINK → ACT → OBSERVE → THINK → ACT, check availability before booking |
| 10 | Google Gemini Chat Model2 | Chat model sub-node | Its own model instance |
| 11 | Get many events in Google Calendar | **Google Calendar Tool** | Connected on **`ai_tool`**. `timeMin` / `timeMax` pinned from the incoming item |

Flow change: `If (approved) → ReAct Agent → Create an event2`.

**The three things to demo from this file:**
1. The `ai_tool` connection — the model chooses, the wire doesn't.
2. The name mismatch — the prompt says "check availability", the node is called "Get many events in Google Calendar". Rename it live and compare.
3. `$fromAI("event_start", "the final confirmed meeting start time in ISO 8601")` — a description written for the model, filling a real parameter.

**Also worth naming:** this file has two agents, each with its own model — triage first, then action. That split is deliberate and it is a sensible pattern, but multi-agent orchestration is the next day's subject. Name it, don't teach it.

---

# Appendix B — Tool contract sheet

### Worked example

| Field | Value |
|---|---|
| **Name** | `check_availability` |
| **Description** | Returns all calendar events between a start and end time. Call this before proposing or confirming any meeting slot, to check whether the slot is already busy. Returns an empty list if the slot is free. |
| **Inputs** | `timeMin` (ISO 8601, pinned from the proposed start) · `timeMax` (ISO 8601, pinned from the proposed end) |
| **Output** | List of events. Empty list = free |
| **Read or write** | Read |
| **Called twice** | Harmless — same answer, one extra API call |

### Blank

| Field | Tool 1 | Tool 2 | Tool 3 |
|---|---|---|---|
| **Name** | | | |
| **Description** (written for the model) | | | |
| **Inputs** (pinned or `$fromAI`) | | | |
| **Output** | | | |
| **Read or write** | | | |
| **Called twice** | | | |
| **What gates it** | | | |

---

# Appendix C — Test emails for the day

### Thread email 1 — Marcus, Monday

```
"Hi, can we get some time next week to go through the Q3
timeline before the client presentation? — Marcus"
```

Expected: `decision: need_more_info`, `missing_info: ["proposed time"]`, topic captured.

### Thread email 2 — Marcus, Tuesday (same thread)

```
"Tuesday 3pm works for me."
```

Expected **with memory**: topic still "Q3 timeline", `decision: accept`, no repeated question.
Expected **without memory**: topic missing, `need_more_info` again. That contrast is the point of the exercise.

### Priya — high urgency

```
"Production outage on the payments service. Need 15 minutes
tonight at 11pm to walk through the rollback. — Priya"
```

Used in part B as the email whose context leaks.

### Nina — out of scope, low stakes

```
"Fancy grabbing lunch Thursday? No agenda, just catching up.
— Nina"
```

Run this **immediately after Priya's**. Any trace of the outage, or any urgency above low, is the leak.

### Lars — injection (carried over from Day 2, Card 4)

The forwarded thread whose signature block contains an instruction addressed to any assistant processing the email. Used in Block 6 to show the difference between breaking one run and poisoning every run after it.

---

# Appendix D — Concept ↔ Block mapping

| Day 3 concept | Block | Where it lives in the example |
|---|---|---|
| A tool is chosen by the model | 1 | `ai_tool` connection in workflow 2 vs `main` in workflow 1 |
| Tool description is a prompt | 1 | "check availability" vs the node's real name · `$fromAI` second argument |
| Pinned vs model-filled parameters | 1 | Tool's `timeMin`/`timeMax` vs `Create an event`'s `$fromAI` |
| Generic tools | 2 | Google Calendar Tool, Gmail |
| Custom tools and custom logic | 2 | The Code node — parse, fall back, never skippable |
| Hard guarantees live in code | 2 | Code node fallback to `need_more_info` |
| Read vs write tools, blast radius | 2 | `sendAndWait` double approval gating the calendar write |
| Tool contracts | Hands-on 1 | Appendix B |
| Build and test in chunks | 3 | Test the tool alone before attaching it |
| Choose the model empirically | 3 | `gemini-3.5-flash-lite` on both agents — a choice to be tested |
| Wire one tool, test it alone | Hands-on 2 | Turning workflow 1 into workflow 2 by hand |
| When memory is needed | 4 | Empty memory port · `limit: 1, unread` |
| State vs memory | 4 | ReAct agent receives sender/topic as prompt text |
| Three kinds of memory | 5 | None today · buffer for the Marcus thread · persistent for sender preferences |
| Session key | 5, 6 | No chat trigger here, so the key is your decision |
| Stale memory | 6 | "Marcus prefers afternoons", six months old |
| Leakage across senders | 6, Hands-on 3B | Priya's outage reaching Nina's reply — Day 2's G5, one level down |
| Poisoned memory | 6 | Day 2's Card 4 injection, now persisted, in a workflow with write tools |
