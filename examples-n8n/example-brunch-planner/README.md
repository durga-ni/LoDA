# Brunch planner — n8n workflows (importable)

Four workflows that turn *"Brunch this Sunday with Alice, Bob and Charlie"*
into a booked table and emailed invites, built as an agentic workflow:
**the AI agent decides (date, guests, candidate, invite text), the graph
executes (lookup, human approval, booking, sending).** Restaurants are mocked
inside the workflows; invites go out through Gmail for real.

Exported from n8n **2.12.3**. The files are the same shape as the editor's
*Download* button produces. Problem statement, design notes and the Python
version live in the source repo (`everyday-ai-agents/reservation`, branch `n8n`).

| File | What | Kind |
|---|---|---|
| `1_Brunch - WF4 Eval Menu.json` | Scores a menu against diners' restrictions | one LLM call + structured output |
| `2_Brunch - WF3 Book Table.json` | Books a slot, capacity check | plain code, no LLM |
| `3_Brunch - WF2 Reservations Agent.json` | Finds and ranks tables; never books, never sees guest names | AI Agent (ReAct loop) |
| `4_Brunch - WF1 Host.json` | Chat entry point; calls the three above as tools/steps; Approve/Decline in chat; Gmail invites | AI Agent + workflow |

## Import (about 10 minutes)

You need: an **Anthropic** API key and a **Gmail OAuth2** credential
(Google Cloud *Web application* OAuth client with n8n's redirect URL; Gmail
API enabled). Node versions in these files exist on n8n ≥ 2.x; older
instances silently drop nodes whose version they don't know — if a node
appears blank after import, that's why.

1. **Import in file order, 1 → 4.** Workflows → *Create* → ⋯ → *Import from
   file…*. Children first, because WF1 references the others by id. Save each.
2. **Credentials.** Attach your Anthropic credential on the model node in
   WF4 (`Anthropic (eval)`), WF2 (`Anthropic (reservations)`) and WF1
   (`Anthropic (host)`). Attach Gmail on WF1's `Send invite`. The files carry
   credential *names* from the source instance; n8n will flag them as missing.
3. **Re-select the sub-workflows in WF1** (three places — ids are
   instance-specific): tool `find_tables` → *WF2*, tool `eval_menu` → *WF4*,
   node `Book table` → *WF3*. Each is a dropdown on the node.
4. **Activate all four.** A sub-workflow must be active before another
   workflow can call it, and WF1 needs production mode for the approval wait
   and chat memory.
5. **Point the guests at yourself.** `get_friend_profiles` in WF1 holds
   Alice/Bob/Charlie with `@example.com` addresses; edit them (Gmail
   plus-addressing works: `you+alice@gmail.com`).

## Run

Open WF1 → *Chat* → *"Brunch this Sunday with Alice, Bob and Charlie"*.
Answer the date question if asked. The agent presents a plan with **Approve —
book & send / Decline** buttons. Approve books the table and emails three
invites; Decline leaves nothing booked and nothing sent.

To test a piece on its own: open WF3 or WF4, click the trigger, *Set output
data* with e.g. `{"restaurant_id":"r3","date":"2026-09-20","time":"11:00","party_size":4}`,
*Execute workflow*.

## Things to know

- The mock booking ledger persists in WF3's static data. After a few
  successful runs a slot fills up and the agent's pick fails with *"Only 0
  seats left"*: open WF3, deactivate, clear its static data (or re-import),
  reactivate.
- Don't enable *Thinking* on the Anthropic model nodes: n8n sends a fixed
  token budget that `claude-opus-5` rejects.
- The Chat Trigger must stay on *Response Mode: Using Response Nodes* — the
  Approve/Decline buttons come from Chat nodes downstream.
