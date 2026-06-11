# muse-voice

> Voice-first music release intake: AI interviews artists about their release in real-time, extracts structured metadata, and queues it for distribution.

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![LiveKit](https://img.shields.io/badge/Voice-LiveKit-000?style=flat-square)](https://livekit.io)
[![Modal](https://img.shields.io/badge/Serverless-Modal-7C3AED?style=flat-square)](https://modal.com)
[![Tests](https://img.shields.io/badge/tests-113%20passing-brightgreen?style=flat-square)]()
[![Part of Muse Agents](https://img.shields.io/badge/Part%20of-Muse%20Agents-1a1a1a?style=flat-square)]()

Getting release metadata from artists is slow: emails, forms, back-and-forth. Muse Voice replaces that with a 3-minute voice conversation. The artist talks, the agent listens and asks follow-up questions, and the release record is ready when the call ends.

---

## Live demo (what the agent actually says)

```
Agent: "Hello, I'm Muse. What is the title and the artist name for this release?"
Artist: "It's called Sunset Drive, by Sarah Lane."
Agent: "Got it. Sunset Drive by Sarah Lane. What is the release date?"
...
Agent: "All set. Sunset Drive by Sarah Lane, a single released October 9th, 2026. 
        Please send your audio and artwork via Telegram."
```

---

## Conversation flow

```mermaid
sequenceDiagram
    participant A as Artist (browser)
    participant FE as Web Frontend
    participant BE as FastAPI backend (Modal)
    participant LK as LiveKit
    participant AI as Voice AI (Gemini)

    A->>FE: Opens intake URL (from Telegram or direct)
    A->>FE: Taps "Start voice intake"
    FE->>BE: POST /voice/sessions/{sid}/token
    BE->>BE: Auto-dispatch worker subprocess
    BE-->>FE: LiveKit JWT
    FE->>LK: Connect audio stream

    loop Intake conversation
        AI->>A: Question (title, artist, date, genre, performer...)
        A->>AI: Answer (free speech)
        AI->>AI: Extract field value, validate, ask next
    end

    AI->>A: Summary read-back + confirmation
    AI->>A: "All set."
    BE->>BE: Create structured release record
    BE->>BE: Queue for distribution handoff
```

---

## Fields collected

| Field | How collected |
|---|---|
| Release title | Direct question |
| Main artist name | Direct question |
| Release date | Direct question + date parsing |
| Release type | Classification (single, EP, album) |
| Primary genre | Direct question |
| Songwriter | Direct question |
| Performer(s) | Direct question, multi-value |
| Producer / Engineer | Direct question |

---

## Architecture

```mermaid
graph TD
    TG["Telegram start URL"] --> FE["Web Frontend\n(browser)"]
    FE --> BE["FastAPI on Modal\n(serverless, auto-scales to zero)"]
    BE --> LK["LiveKit\n(real-time audio transport)"]
    BE --> AI["Gemini Voice AI\n(WebSocket, real-time)"]
    AI --> EXTRACT["Field Extractor\nValidated pydantic model"]
    EXTRACT --> RECORD["Release Record\n(queued for distribution)"]
    RECORD --> DIST["Distribution handoff\n(TuneCore / Paperclip)"]
```

---

## Implementation status

| Component | Status | Notes |
|---|---|---|
| Voice intake v1.2 | Shipped | 113 tests passing, confirmed live in browser |
| Auto-dispatch worker | Shipped | Worker spawned on session create, not on /token |
| Cold-start pre-warm | Planned | Reduce 15-22s initial delay to ~0s |
| Distribution handoff | Planned | Paperclip / TuneCore wiring |

---

Built by [Joy Dong](https://www.joydong.org)
