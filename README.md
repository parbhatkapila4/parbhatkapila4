# Parbhat Kapila

AI Systems Engineer. I build retrieval and agent systems on Postgres, and I care more about what they do when they fail than what they do in the demo.

[parbhat.dev](https://www.parbhat.dev/) · [LinkedIn](https://www.linkedin.com/in/parbhat-kapila/) · [X](https://x.com/Parbhat03) · parbhat@parbhat.dev

Open to full-time remote roles, on your hours. If the team builds in one room, I'd rather be in it.

---

## Five systems, deployed

All five are mine. I built them, deployed them, and still run them, which is why the notes below are about trade-offs rather than features.

**[VectorMail](https://vectormail.space)**. Autonomous email agent. Read and write are separate rails on purpose: search returns cited sources, and an AI reply passes two gates (topic gate, then outbound moderation) before it goes out. The model sits upstream of the gate and is never trusted to be the gate. Automation is dry-run by default. Embeddings live as a `vector(768)` column on the email row, which means tenancy is enforced in the same `WHERE` clause as retrieval and there's no second store to keep in sync.

**[Sentinel](https://sentinels.in)**. CRM as a permission layer. Ingestion from Gmail, Calendar and Slack is fail-closed: nothing enters the system unless it matches a CRM contact, and the default is deny. HubSpot and Salesforce OAuth. The repo is public and MIT licensed, so this one you can read instead of taking my word for it.

**[RepoDoc](https://repodoc.parbhat.dev)**. Codebase RAG that embeds what each file *means*, not what it says. Every file gets an LLM summary capped at 100 words; the summary is what gets embedded (768d) and retrieved by pgvector cosine over an HNSW index. Postgres is also the job queue: indexing rows are claimed by atomic compare-and-swap under five-minute leases with cursored resume, so a job survives serverless 60-second timeouts instead of restarting. Every AI call is metered against a per-project budget, and a query is refused with a 402 once it's spent.

**[CUTLINE](https://cutline.cloud)**. One sentence in, one finished MP4 out, through a deterministic pipeline: twelve stages on the slideshow path, fewer on the talking-character paths. Pipeline over agent, deliberately: no templates, no creative knobs, no model deciding the control flow. Next.js control plane on Vercel with a separate long-running BullMQ worker and Redis as the coordination backbone. Image sourcing falls through three providers and then a placeholder, and the stage fails rather than shipping a render that's mostly placeholder. Payment webhooks claim a `processed_webhook_events` row atomically before granting entitlement, which makes a double-grant structurally impossible rather than unlikely.

**[Visura](https://visura.parbhat.dev)**. Versioned document processing. Chunks are keyed by SHA-256, so re-processing an updated document costs in proportion to what actually changed, not to the size of the file. Stuck versions are detected and replayed idempotently with partial progress preserved.

---

## Decisions that cost me something

- **Postgres and pgvector instead of a managed vector database, in all five.** Costs stay predictable, queries stay inspectable, and tenancy lives in the same clause as retrieval. The trade is that I own index tuning and recall regressions myself.
- **Multi-provider routing through OpenRouter, written before I needed it.** Fallback is cheap to write up front and expensive after.
- **Fail-closed defaults.** Sentinel drops data it can't attribute rather than guessing at it. That makes the product feel stricter than tools that ingest everything, and I'd make the trade again.
- **In-memory idempotency for CUTLINE's job submits rather than a Postgres job-state table**, because BullMQ already owns job lifecycle and two sources of truth is worse than one weaker one.
- **Shipped ahead of the right abstraction more than once** and paid for it in refactors. Still the correct call while the usage shape was unsettled, but it was a bill, not a free lunch.

---

## Stack

TypeScript · React · Next.js · Node · Large Language Models · Postgres · pgvector · Redis · BullMQ · Drizzle · Prisma · Docker · Vercel · OpenRouter · Clerk

---

parbhat@parbhat.dev. I answer fast.
