# Bloom AI — Closira Customer Support Agent

An AI-powered customer support workflow for **Bloom Aesthetics Clinic**, built as part of the Closira AI Engineering Intern assignment.

The agent handles inbound customer enquiries across four stages:
**FAQ Answering → Lead Qualification → Escalation Detection → Conversation Summary**

---

## Project Structure

```
closira/
├── agent.py                  # Main workflow — run this
├── data/
│   └── sop.json              # Knowledge base (SOP) for Bloom Aesthetics Clinic
├── prompt_design.md          # Full prompt design documentation
├── test_transcripts/
│   ├── 01_in_sop_botox_price.md
│   ├── 02_out_of_scope_question.md
│   ├── 03_escalation_angry_customer.md
│   ├── 04_lead_qualification.md
│   └── 05_conversation_summary.md
├── logs/                     # Session logs (auto-created, JSON)
└── README.md
```

---

## Setup

### Prerequisites

- Python 3.9+
- An Anthropic API key

### Install dependencies

```bash
pip install anthropic
```

### Set your API key

```bash
export ANTHROPIC_API_KEY=your_api_key_here
```

Or create a `.env` file and load it before running.

---

## Running the Agent

```bash
python agent.py
```

The agent runs as a CLI conversation. Type your message and press Enter.

### Available Commands

| Command | Description |
|---|---|
| `/qualify` | Switch to lead qualification mode |
| `/summary` | Generate and display session summary |
| `/status` | Show current session state (stage, escalation, qualification data) |
| `/quit` | Generate summary, save log, and exit |
| `/help` | Show command list |

---

## How It Works

### Stage 1 — FAQ Answering
The agent answers customer questions using only the data in `data/sop.json`. It will not invent prices, services, staff names, or policies. If a question is outside the SOP, it acknowledges this and offers to escalate.

### Stage 2 — Lead Qualification
Triggered via `/qualify`. The agent asks one question at a time across three dimensions:
1. Treatment interest
2. New or returning customer
3. Timing / urgency

When all three are collected, a `[QUALIFICATION_SUMMARY]` is stored in session state.

### Stage 3 — Escalation Detection
Escalation is detected at two levels:
- **Model-side**: The system prompt instructs the model to append `[ESCALATE: reason]` when any trigger is met (complaint, medical question, pricing negotiation, angry sentiment, explicit request for human)
- **System-side**: If the agent gives uncertain/unanswered responses to 2+ consecutive questions, auto-escalation fires regardless

Once escalated, the AI stops generating responses. The customer is held with a fixed message while subsequent inputs are logged for the human agent.

### Stage 4 — Conversation Summary
Generated via `/summary` or automatically on `/quit`. Produces a structured report covering:
- Customer intent
- Key details collected
- SOP gaps identified
- Escalation status and reason
- Recommended next action

All sessions are logged to `logs/session_<timestamp>.json`.

---

## SOP

The knowledge base (`data/sop.json`) covers:

- **Business**: Bloom Aesthetics Clinic
- **Hours**: Monday–Saturday, 9am–7pm
- **Services**: Botox (from £200), Dermal Fillers (from £250), Free Initial Consultation
- **Booking**: Via WhatsApp or website; 24hr cancellation policy; £50 deposit for treatments
- **Aftercare**: Post-treatment guidance for Botox and Fillers
- **Escalation triggers**: Defined explicitly

To use a different SOP, edit `data/sop.json` and update the business name in the system prompt in `agent.py`.

---

## Dependencies

| Package | Purpose |
|---|---|
| `anthropic` | Claude API client |
| `json`, `re`, `os`, `datetime` | Standard library — no extra install needed |

---

## Trade-offs and Known Limitations

| Area | Decision / Limitation |
|---|---|
| **No frontend** | CLI only, as specified in the brief |
| **No persistent memory** | Each session is independent; history is not carried across runs |
| **Full SOP injected per turn** | Increases token usage slightly; acceptable for this SOP size. For a larger SOP, RAG retrieval would be more scalable |
| **Escalation relies on model compliance** | Mitigated by a system-side unanswered-question counter, but not 100% guaranteed |
| **Qualification is manual** | `/qualify` must be invoked explicitly. A production system would blend qualification naturally into the FAQ flow |
| **No streaming** | Responses appear after full generation. Streaming could be added using `client.messages.stream()` |
| **Single model** | Uses `claude-sonnet-4-20250514` throughout. The summary stage could use a cheaper/faster model |

---

## Test Transcripts

Five test transcripts covering all required scenarios are in `test_transcripts/`:

| File | Scenario |
|---|---|
| `01_in_sop_botox_price.md` | In-SOP question answered accurately |
| `02_out_of_scope_question.md` | Out-of-scope question escalated without guessing |
| `03_escalation_angry_customer.md` | Angry/complaint sentiment detected and escalated |
| `04_lead_qualification.md` | Structured qualification with summary produced |
| `05_conversation_summary.md` | Full session summary with all five required fields |
