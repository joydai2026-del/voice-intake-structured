# voice-intake-structured

> Voice-first intake agent: AI interviews users in real-time, collects structured data across any field set, and delivers a validated record — no forms, no typing.

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![LiveKit](https://img.shields.io/badge/Voice-LiveKit-000?style=flat-square)](https://livekit.io)
[![Modal](https://img.shields.io/badge/Serverless-Modal-7C3AED?style=flat-square)](https://modal.com)
[![Tests](https://img.shields.io/badge/tests-113%20passing-brightgreen?style=flat-square)]()

Forms are slow and imprecise. This agent replaces them: the user speaks naturally, the agent asks follow-up questions for any missing fields, reads back a summary for confirmation, and produces a validated structured record — all in under 3 minutes.

---

## Live demo (actual agent transcript)

```
Agent: "What is the title and the name for this record?"
User:  "It's called Sunset Drive, by Sarah Lane."
Agent: "Got it — Sunset Drive by Sarah Lane. What is the date?"
...
Agent: "All set. I have everything I need."
```

---

## Conversation flow

```mermaid
sequenceDiagram
    participant U as User (browser)
    participant FE as Web Frontend
    participant BE as FastAPI backend (Modal)
    participant LK as LiveKit
    participant AI as Voice AI

    U->>FE: Opens intake URL
    U->>FE: Taps "Start"
    FE->>BE: POST /voice/sessions/{sid}/token
    BE->>BE: Auto-dispatch voice worker subprocess
    BE-->>FE: LiveKit JWT
    FE->>LK: Connect audio stream

    loop Intake interview
        AI->>U: Question (field by field)
        U->>AI: Answer (free speech)
        AI->>AI: Extract value, validate, advance
    end

    AI->>U: Summary read-back
    AI->>U: "All set."
    BE->>BE: Emit structured record
    BE->>BE: Route to downstream handler
```

---

## Architecture

```mermaid
graph TD
    URL["Start URL (deep link)"] --> FE["Web Frontend\n(browser)"]
    FE --> BE["FastAPI on Modal\n(serverless, scales to zero)"]
    BE --> LK["LiveKit\n(real-time audio transport)"]
    BE --> AI["Voice AI\n(WebSocket, real-time)"]
    AI --> EXTRACT["Field Extractor\nValidated pydantic model"]
    EXTRACT --> RECORD["Structured Record\n(any schema)"]
    RECORD --> HANDLER["Downstream handler\n(queue, webhook, API)"]
```

---

## Field collection model

The field schema is config-driven — define the fields you need and the agent asks for them in order, handling re-asks and clarifications automatically.

| Field property | Type | Notes |
|---|---|---|
| `key` | string | Stable identifier for the structured record |
| `label` | string | What the agent calls this field in conversation |
| `field_type` | text / date / enum / number | Used for validation |
| `required` | bool | Agent re-asks until satisfied if true |
| `confirmation_read` | bool | Agent reads back this field in the summary |

---

## Implementation status

| Component | Status |
|---|---|
| Voice intake v1.2 | Shipped — 113 tests, confirmed live in browser |
| Auto-dispatch worker | Shipped |
| Cold-start pre-warm | Planned |
| Downstream handler wiring | Configurable per deployment |

---

Built by [Joy Dong](https://www.joydong.org)
