# Narrowed prep options (RADISE / SmartHub)

Use this file when you want a **smaller** study set than the full guide in `interview-prep-smart-structures-radise.md`.

---

## Choose your focus

**Pick one (or combine):**

1. **20-minute verbal script** — word-for-word intro + one project story you can rehearse end-to-end.
2. **Deep dives only** — NestJS + PostgreSQL + Redux (concepts, patterns, and interview Q&A without the full 50-question spread).

Say which you want, and trim the rest of the main doc accordingly.

---

## Option A — 20-minute verbal script (outline)

**Time box (~20 minutes total in a real round, adjust if they split rounds):**

| Block | Minutes | Content |
|-------|---------|---------|
| Intro + motivation | ~2–3 | Who you are, stack, why RADISE/SmartHub |
| One deep project | ~8–10 | SwagShots (or Corpusvision platform): problem, your role, architecture, NestJS + queues + data stores, one metric/outcome |
| Technical strengths | ~3–4 | NestJS, PostgreSQL/MongoDB, Redis/BullMQ, RBAC, CI/CD — tied to JD |
| IoT/AEC bridge | ~2 | Honest curiosity + analogy (async pipelines → sensor pipelines) |
| Questions for them | ~2–3 | 2–3 sharp questions from the main doc |

**Word-for-word intro (adapt to your voice):**

> “Hi, I’m Pulkit. I’m a senior backend engineer with a bit over five years building B2B and consumer-scale products—mostly on Node and NestJS with TypeScript. I’ve owned APIs, authentication and RBAC, async workloads with Redis and BullMQ, and integrations like payments and notifications. I’ve also worked with PostgreSQL and MongoDB depending on whether we needed strong relational models or flexible analytics. On the frontend I’ve shipped React and Next for admin and product surfaces, though my deepest strength is the backend. I’m interested in RADISE because SmartHub sits at the intersection of serious infrastructure data, reliability, and user-facing tooling—that’s where I like to work. I’d love to walk through one system I led end-to-end, then hear how your team splits the stack and handles sensor data from ingestion to the UI.”

**One project story skeleton (SwagShots — fill with your real numbers):**

1. **Situation:** Short-video platform; uploads, playback, engagement, analytics.
2. **Task:** Own backend architecture and critical paths for scale and reliability.
3. **Action:** NestJS modules; JWT + RBAC; FFmpeg + BullMQ for transcoding/HLS; MongoDB for engagement and aggregations; object storage + CDN; FCM; monitoring and deployments (Docker/Nginx/PM2, CI/CD).
4. **Result:** Scale to active users/downloads/ratings you’re comfortable stating; what you learned about queues and observability.

Rehearse this story **out loud** until it fits ~8 minutes without rushing.

---

## Option B — Deep dives only (NestJS + PostgreSQL + Redux)

Study **only** these sections from `interview-prep-smart-structures-radise.md`:

| Topic | Sections / question numbers |
|-------|----------------------------|
| **NestJS** | Part 3 — **C** (questions 16–27); skim **E** for API patterns (36–40) |
| **PostgreSQL** | Part 3 — **D** (28–35); **E** pagination/caching/N+1 (36–39) |
| **Redux** | Part 3 — **B** (6–15); add RTK Query (12) if JD stresses Redux |

**Extra focus if “deep dive”:**

- **NestJS:** module boundaries, Guards/Pipes/Interceptors, validation DTOs, testing layers.
- **PostgreSQL:** indexes for time-range queries, transactions, migrations, JSONB vs separate tables.
- **Redux:** when to use vs server state (TanStack Query); RTK Query vs manual slices; one example of normalizing list data.

**Skip for this mode:** long IoT system-design lists unless they ask—keep **one** sensor pipeline sentence ready (see main doc, section H).

---

## After you choose

- **Option A:** Practice with a timer; record yourself once and cut filler words.
- **Option B:** For each topic, write **one** example from Corpusvision/DandD and **one** hypothetical SmartHub example (e.g. project → sensors → readings).

---

## Links

- Full Q&A and flow: `interview-prep-smart-structures-radise.md`
- System design (20 Q&A): `system-design-20-questions.md`
