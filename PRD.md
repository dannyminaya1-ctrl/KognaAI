# KognaAI — Technical Product Requirements Document

**Version:** 1.0  
**Status:** Draft  
**Date:** May 2026  
**Product Type:** Consumer Mental Health AI Companion  

---

## Table of Contents

1. [Mission & Vision](#1-mission--vision)
2. [Problem Statement](#2-problem-statement)
3. [Product Definition](#3-product-definition)
4. [Differentiators](#4-differentiators)
5. [System Architecture](#5-system-architecture)
6. [Tech Stack](#6-tech-stack)
7. [Core API Contract](#7-core-api-contract)
8. [Feature Backlog](#8-feature-backlog)
9. [Success Metrics](#9-success-metrics)
10. [Roadmap](#10-roadmap)
11. [Safety Architecture](#11-safety-architecture)
12. [Compliance & Privacy](#12-compliance--privacy)
13. [Risk Register](#13-risk-register)
14. [Open Questions](#14-open-questions)

---

## 1. Mission & Vision

KognaAI is a culturally intelligent AI companion that meets Black and brown individuals — especially those from religious communities where therapy carries stigma — exactly where they are.

**It does not replace therapy. It builds the bridge to it.**

KognaAI normalizes emotional conversation, reinforces healthy habits, and gently primes users to feel comfortable seeking professional care. The core metaphor: *"That friend who always picks up"* — someone who gets your background, holds no judgment, and shows up every day.

---

## 2. Problem Statement

| Problem | Description |
|---|---|
| **Access gap** | Millions lack affordable therapy — cost, geography, and insurance are real barriers |
| **Cultural stigma** | "We don't go to therapy" — faith communities often frame mental health struggles as spiritual weakness |
| **Trust deficit** | Historical trauma with healthcare systems creates reluctance to engage with clinical tools |
| **Market gap** | No existing product centers Black and brown cultural context with warmth, not clinical coldness |

---

## 3. Product Definition

| Attribute | Detail |
|---|---|
| **What it is** | A warm, friend-like AI chatbot that listens, affirms, habit-builds, and nudges toward professional care |
| **What it is not** | A therapist, diagnostic tool, or crisis service — Kogna is a companion, not a clinician |
| **Platform** | Mobile-first web app (PWA) · Native iOS & Android in Phase 2 |
| **Audience** | Black and brown consumers, 18–45, especially those in religious communities where therapy is stigmatized |
| **Monetization** | Freemium — free daily check-ins; premium unlocks memory, journaling, and habit programs |

---

## 4. Differentiators

| Differentiator | Detail |
|---|---|
| **Cultural grounding** | System prompt co-designed with Black and brown therapists; aware of faith, family, and community dynamics |
| **Therapy bridge** | Explicit product goal: reduce stigma and build readiness for professional care — not app dependency |
| **Habit engine** | CBT + positive psychology micro-habits tailored to user context, not generic wellness clichés |
| **Privacy-first** | Zero third-party data sharing; end-to-end encryption; no training on user conversations without consent |

---

## 5. System Architecture

```
┌─────────────────────────────────────────────────┐
│              CLIENT LAYER                        │
│   Next.js 15 App Router · React 19 · Tailwind   │
│         shadcn/ui · PWA-ready                   │
└────────────────────┬────────────────────────────┘
                     │ HTTPS / WebSocket stream
┌────────────────────▼────────────────────────────┐
│               API LAYER                          │
│  Next.js Route Handlers · Vercel AI SDK          │
│         (streaming) · Edge Runtime               │
└──────────┬──────────────────────┬───────────────┘
           │ Orchestration        │
┌──────────▼──────────┐  ┌───────▼───────────────┐
│      AI CORE        │  │      RAG LAYER         │
│  Claude Sonnet 4    │  │  pgvector (Supabase)   │
│  Cultural system    │  │  Therapeutic knowledge │
│  prompt · MIND-SAFE │  │  Cultural embeddings   │
│  LangChain.js       │  │                        │
└──────────┬──────────┘  └───────┬───────────────┘
           │ Data persistence    │
┌──────────▼──────────┐  ┌───────▼───────────────┐
│     DATABASE        │  │   AUTH & STORAGE       │
│  Supabase Postgres  │  │  Supabase Auth         │
│  Row-level security │  │  OAuth (Google/Apple)  │
│  Encrypted convos   │  │  Supabase Storage      │
│  pgvector index     │  │  (voice files)         │
└──────────┬──────────┘  └───────────────────────┘
           │
┌──────────▼──────────────────────────────────────┐
│         SAFETY LAYER (wraps all AI outputs)      │
│  Real-time crisis classifier · 988 / NAMI        │
│  escalation · Human review queue                 │
│  Content moderation API                          │
└─────────────────────────────────────────────────┘
```

### Data Flow — One Conversation Turn

1. **User message** → Encrypted in transit (TLS 1.3), stored with RLS in Supabase
2. **Safety check** → Crisis classifier runs first — if risk detected, escalate before LLM call
3. **RAG retrieval** → Message embedded → pgvector similarity search → pulls cultural + therapeutic context
4. **LLM call** → Claude Sonnet 4 receives: system prompt + retrieved context + conversation history + user message
5. **Output filter** → Response passes content moderation before streaming to client
6. **Memory update** → Key facts, mood signal, habit status written back to Supabase user profile

---

## 6. Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Frontend** | Next.js 15 + React 19 | App Router, Server Components, streaming UI. Default for AI apps in 2026. |
| **Styling** | Tailwind CSS + shadcn/ui | Composable, accessible components. Fast to build warm, mobile-first UI. |
| **AI / LLM** | Anthropic Claude Sonnet 4 | Best-in-class safety controls, nuanced emotional tone, strong instruction-following. |
| **AI SDK** | Vercel AI SDK 4 | Streaming chat UI, tool use, multi-provider support. Native Next.js integration. |
| **Orchestration** | LangChain.js | RAG pipelines, memory chains, retrieval agents. Standard for LLM workflows. |
| **Database** | Supabase (PostgreSQL) | Auth, storage, real-time, pgvector for RAG — all in one. RLS built in. |
| **Vector store** | pgvector (Supabase) | Semantic similarity search for RAG. IVFFlat index for fast retrieval at scale. |
| **Embeddings** | text-embedding-3-small | 1536-dim embeddings for therapeutic knowledge base and cultural context retrieval. |
| **Auth** | Supabase Auth | OAuth (Google, Apple), email/password, MFA. HIPAA-compatible with proper config. |
| **Deployment** | Vercel + Edge Runtime | Sub-100ms cold starts globally. Streaming response support built in. |
| **Monitoring** | LangSmith + Sentry | LLM trace observability + error tracking. Required for safety auditing. |
| **Payments** | Stripe | Freemium → premium subscription. Billing portal, usage metering. |
| **Voice (Phase 2)** | Whisper API + ElevenLabs | STT for voice input. TTS for warm, human-sounding audio responses. |
| **Safety** | Custom crisis classifier | Fine-tuned classifier to detect risk signals before LLM generates any output. |

---

## 7. Core API Contract

### Chat Endpoint

```
POST /api/chat
Authorization: Bearer {session_token}
Content-Type: application/json

Body:
{
  "messages": [...conversationHistory],
  "userId": "uuid",
  "sessionId": "uuid",
  "stream": true
}

Response: ReadableStream (Vercel AI SDK format)
Headers:
  X-Crisis-Flag: none | low | high
```

### Database Schema (core tables)

```sql
-- Users
create table users (
  id uuid primary key default gen_random_uuid(),
  email text unique not null,
  created_at timestamp default now(),
  cultural_context jsonb,
  therapy_readiness_score int default 0
);

-- Conversations
create table conversations (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id),
  created_at timestamp default now(),
  mood_signal text,
  crisis_flag text default 'none'
);

-- Messages
create table messages (
  id uuid primary key default gen_random_uuid(),
  conversation_id uuid references conversations(id),
  role text check (role in ('user', 'assistant')),
  content text not null,
  created_at timestamp default now()
);

-- RAG knowledge base
create table knowledge_chunks (
  id bigserial primary key,
  content text not null,
  category text, -- 'cultural', 'cbt', 'habit', 'faith'
  embedding vector(1536),
  metadata jsonb
);

create index on knowledge_chunks
  using ivfflat (embedding vector_cosine_ops)
  with (lists = 100);

-- Habits
create table habits (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id),
  name text not null,
  streak int default 0,
  last_completed date
);
```

---

## 8. Feature Backlog

### Must Have (Phase 0 MVP)

| Feature | Description |
|---|---|
| **Conversational chat interface** | Warm, streaming text chat. Mobile-first. Culturally fluent tone. No clinical cold language. |
| **Cultural system prompt engine** | Co-designed with BIPOC therapists. Aware of faith, family loyalty, code-switching, and stigma. |
| **Daily check-in flow** | Mood pulse, gratitude prompt, one micro-habit. Builds the daily ritual that drives retention. |
| **Crisis detection & escalation** | Real-time classifier. If risk detected: surface 988, NAMI, Crisis Text Line immediately. |
| **User auth & secure data store** | Supabase Auth with MFA. All conversations encrypted at rest. RLS prevents cross-user access. |
| **Freemium billing** | Stripe integration. Free tier with usage caps. Premium subscription unlocks full features. |

### Should Have (Phase 1)

| Feature | Description |
|---|---|
| **Cross-session memory** | Kogna remembers your name, family context, recurring themes — not a fresh start every session. |
| **Habit tracker** | Streak tracking for sleep, movement, prayer/meditation, journaling. Positive reinforcement framing. |
| **Therapy readiness score** | Internal metric tracking emotional openness. When high, Kogna gently introduces therapy. |
| **Guided reflection journaling** | AI-prompted journaling with CBT techniques in everyday, non-clinical language. |
| **RAG knowledge base** | Cultural + CBT content embedded and retrieved to ground LLM responses. |

### Nice to Have (Phase 2+)

| Feature | Description |
|---|---|
| **Voice mode** | Whisper STT + ElevenLabs TTS. Talk to Kogna like calling a friend. Critical for low-literacy users. |
| **Therapist referral directory** | Surface culturally competent BIPOC therapists when user is ready. |
| **Community spaces** | Group wellness, church/org white-label, faith-specific modes. |
| **Clinical hand-off API** | Structured summary passed to therapist when user makes the transition. |

---

## 9. Success Metrics

### North Star Metric

> **7-day active users who complete at least 3 check-ins per week**

This proves habit formation — the core product promise — not just novelty usage.

### KPIs

| Category | Metric | Target |
|---|---|---|
| **Retention** | D7 retention | 40%+ |
| **Retention** | D30 retention | 25%+ |
| **Monetization** | Free → paid conversion | 5–8% |
| **Engagement** | Avg check-ins / week | 3+ |
| **Safety** | Crisis escalation SLA | < 1 second |
| **Performance** | LLM response time | < 2 seconds |
| **Quality** | Thumbs up ratio | > 80% |
| **Mission** | % users who click therapy referral | Track & improve |

### Measurement Dimensions

- **Engagement:** Messages/session, sessions/week, check-in completion rate, habit streaks
- **Safety:** Crisis detection accuracy, false negative rate, escalation completion rate
- **Therapy readiness:** % users who click referral, % who self-report starting therapy
- **Cultural fit:** NPS segmented by community, qualitative audit by BIPOC therapist advisory board
- **Revenue:** MRR, ARPU, LTV:CAC ratio, churn rate

---

## 10. Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 0 — Foundation** | Weeks 1–8 | Cultural system prompt · Core chat UI · Daily check-in · Crisis detection · Supabase auth + encrypted storage · Freemium billing |
| **Phase 1 — Habit & Memory** | Weeks 9–20 | Cross-session memory · Habit tracker with streaks · Guided journaling · Therapy readiness score · RAG knowledge base · LangSmith observability |
| **Phase 2 — Voice & Referrals** | Weeks 21–36 | Voice mode (Whisper + ElevenLabs) · Native iOS & Android (React Native) · BIPOC therapist referral directory · Mood trend insights |
| **Phase 3 — Community & Growth** | Week 37+ | Group wellness spaces · Church/org white-label · Faith-specific modes · Clinical partnership API |

---

## 11. Safety Architecture

### Crisis Protocol

| Step | Action |
|---|---|
| **Detection** | Custom classifier runs before every LLM call. Detects suicidal ideation, self-harm signals, abuse disclosures. SLA: < 1 second. |
| **Response** | Never leaves user without a resource: 988 Suicide & Crisis Lifeline, Crisis Text Line (text HOME to 741741), NAMI Helpline (1-800-950-6264) |
| **Escalation** | High-risk sessions flagged for human clinical review within 24 hours |
| **Logging** | All crisis-flagged sessions retained in encrypted audit log regardless of user deletion requests |

### System Prompt Guardrails

- Kogna cannot diagnose, prescribe, or position itself as a therapist
- All system prompts co-designed with licensed clinical psychologists
- Explicit disclosure in onboarding and persistent in-app: "Kogna is a companion, not a therapist"
- Socratic questioning and empathic reflection techniques — no medical advice

### Human Review

- Flagged sessions reviewed by clinical advisors weekly
- BIPOC therapist advisory board audits system prompt quarterly
- Patterns from review feed back into prompt improvement cycle

---

## 12. Compliance & Privacy

| Area | Requirement |
|---|---|
| **Data encryption** | AES-256 at rest (Supabase) · TLS 1.3 in transit · Zero plaintext conversation logs |
| **Data sharing** | Zero third-party data sharing. No ad targeting. Explicit policy in ToS and onboarding. |
| **Training opt-out** | Users can opt out of conversation use for model improvement. Default: opted out. |
| **GDPR / CCPA** | Right to deletion, data export, consent flows. Supabase RLS enforces per-user data isolation. |
| **CA SB 243 (Jan 2026)** | Companion chatbot disclosure requirements met. Guardrails for minors. Operational documentation maintained. |
| **HIPAA posture** | Not a covered entity in v1. Architecture designed for HIPAA-ready upgrade (Supabase Business + BAA). |

---

## 13. Risk Register

| Risk | Level | Mitigation |
|---|---|---|
| Crisis miss — harmful output to at-risk user | 🔴 High | Pre-LLM crisis classifier with <1% false negative target. Weekly clinical review. Immediate escalation pathway. |
| Cultural misalignment — tone feels generic or offensive | 🔴 High | BIPOC therapist advisory board co-designs and audits system prompt quarterly. Community beta before launch. |
| Over-reliance — users avoid therapy indefinitely | 🟡 Medium | Therapy readiness nudges built into product logic. Kogna explicitly and warmly encourages professional care. |
| Regulatory action under AI mental health rules | 🟡 Medium | Legal review before launch. SB 243 compliance documented. Not positioned as clinical care. |
| Inference cost at scale | 🟢 Low | Usage caps on free tier. Claude Haiku for low-stakes turns, Sonnet for emotional depth turns. |

---

## 14. Open Questions

- [ ] Who are the 2–3 licensed Black and brown therapists joining the advisory board?
- [ ] What is the Phase 0 launch community (church org, community center, direct consumer)?
- [ ] Will the app support Spanish in v1 for Latino communities?
- [ ] What is the pricing model for the premium tier? ($9.99/mo? $14.99/mo?)
- [ ] Is there a partnership play with organizations like Therapy for Black Girls or NAMI?
- [ ] What is the policy on minor users (under 18)?

---

*KognaAI Technical PRD v1.0 · Built with purpose for communities that deserve better access to care.*
