# Parbhat Kapila

AI Systems Engineer. I build retrieval and agent systems on Postgres, and I care more about what they do when they fail than what they do in the demo.

[parbhat.dev](https://www.parbhat.dev/) · [LinkedIn](https://www.linkedin.com/in/parbhat-kapila/) · [X](https://x.com/Parbhat03) · parbhat@parbhat.dev

Open to full-time remote roles. Full overlap with US or EU hours, whichever the team runs on.

---

## Five systems, deployed

All five are mine. I built them, deployed them, and still run them, which is why the notes below are about trade-offs rather than features.

**[VectorMail](https://vectormail.space)**. Autonomous email agent. Read and write are separate rails on purpose: search returns cited sources, and an AI reply passes two gates (topic gate, then outbound moderation) before it goes out. The model sits upstream of the gate and is never trusted to be the gate. Automation is dry-run by default. A send-intent marker is written before the provider call, so a retried send parks instead of firing again. Embeddings live as a `vector(768)` column on the email row, which means tenancy is enforced in the same `WHERE` clause as retrieval and there's no second store to keep in sync.

**[Sentinel](https://sentinels.in)**. CRM as a permission layer. Ingestion from Gmail, Calendar and Slack is fail-closed: nothing enters the system unless it matches a CRM contact, and the default is deny, including on database errors. HubSpot and Salesforce OAuth. The repo is public, so this one you can read instead of taking my word for it.

**[RepoDoc](https://repodoc.parbhat.dev)**. Codebase RAG that embeds what each file *means*, not what it says. Every file gets an LLM summary capped at 100 words; the summary is what gets embedded (768d) and retrieved by pgvector cosine over an HNSW index. Postgres is also the job queue: indexing rows are claimed by atomic compare-and-swap under five-minute leases with cursored resume, so a job survives serverless 60-second timeouts instead of restarting. Each answered query records its estimated model cost against the project.

**[CUTLINE](https://cutline.cloud)**. One sentence in, one finished MP4 out. Eleven to twelve stages on the slideshow path, in a fixed order, each one timed and logged so a failure names the stage that broke. The stages call outside models, so the same prompt gives a different video; what's fixed is the order and the boundaries. Pipeline over agent, deliberately: no templates, no creative knobs, no model deciding the control flow. Next.js control plane on Vercel with a separate long-running BullMQ worker and Redis as the coordination backbone. Each shot walks Unsplash, then Pexels, then DALL-E 3, then a placeholder, and on a paid plan the job fails outright if more than half the shots end on the placeholder. Payment webhook events are claimed in Postgres with `INSERT ... ON CONFLICT DO NOTHING` before a plan is granted, so a replayed event is ignored.

**[Visura](https://visura.parbhat.dev)**. Versioned document processing. Chunks are keyed by SHA-256, and a chunk whose hash still matches reuses its summary instead of paying the model again. The catch I found later: chunk boundaries depend on all the text before them, so an edit near the start shifts everything after it and reuses almost nothing. That one is on the list. A replay skips chunks that are already done, so re-running a stuck version doesn't re-pay for finished work.

---

## Decisions that cost me something

- **Postgres and pgvector instead of a managed vector database, everywhere I do retrieval.** Costs stay predictable, queries stay inspectable, and tenancy lives in the same clause as retrieval. The trade is that I own index tuning and recall regressions myself.
- **Multi-provider routing through OpenRouter, written before I needed it.** Fallback is cheap to write up front and expensive after.
- **Fail-closed defaults.** Sentinel drops data it can't attribute rather than guessing at it. That makes the product feel stricter than tools that ingest everything, and I'd make the trade again.
- **In-memory idempotency for CUTLINE's job submits rather than a Postgres job-state table**, because BullMQ already owns job lifecycle and two sources of truth is worse than one weaker one. The cost is real and I know what it is: the guard does not survive a restart. I've already been bitten once in the same codebase by a cache that lived on container disk and got wiped on every deploy, so this one is on the list.
- **Shipped ahead of the right abstraction more than once** and paid for it in refactors. Still the correct call while the usage shape was unsettled, but it was a bill, not a free lunch.

---

## Stack

TypeScript · React · Next.js · Node · Postgres · pgvector · Redis · BullMQ · Drizzle · Prisma · Docker · Vercel · OpenRouter · Clerk

---

parbhat@parbhat.dev. I answer fast.
