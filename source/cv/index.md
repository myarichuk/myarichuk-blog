---
title: CV
type: cv
comments: false
top_img: false
---

## Michael Yarichuk

Senior Software Engineer · Systems & Performance · .NET

[linkedin.com/in/myarichuk](https://linkedin.com/in/myarichuk) | [github.com/myarichuk](https://github.com/myarichuk) | [graymatterdeveloper.com](https://www.graymatterdeveloper.com)

---

### Profile

Senior software engineer and systems programmer with 13+ years of .NET experience, specialising in high-performance and low-level engineering: zero-allocation patterns, database storage internals, crash and core dump analysis, and memory subsystem design. Spent seven years contributing to RavenDB and its Voron B-Tree storage engine at Hibernating Rhinos. Builds production libraries benchmarked against hand-optimised baselines. Autodidact since age 12 — the kind who goes straight to the source: RBIL, processor manuals, WinDbg.

---

### Experience

#### Stealth Startup (Cybersecurity)
**2022 – present**

*.NET 6/7/8 · C# · Docker · containerised microservices on-prem · Node.js*

*Company name and product details are NDA/MoD export-constrained. Listed as "stealth startup" on LinkedIn at company's request.*

**Team Lead / Engineering Manager — 2022–2023**
- Managed a small agile team: goal-setting, KPIs, sprint planning, code reviews, and mentoring 3–4 engineers across .NET and Node.js stacks
- Stayed hands-on throughout — no pure-management overhead; continued designing and shipping features in parallel
- After ~1.5 years, transitioned to Tech Lead by mutual agreement — IC contribution was the higher-leverage use of time

**Tech Lead / Senior Backend Engineer — 2023–present**
- Technical lead for an on-premises containerised .NET microservice platform; responsible for architectural decisions, cross-service integration design, and performance strategy
- Sole engineer with Node.js experience on the team — took full ownership of the Node.js service: feature development, maintenance, performance work, and production incidents
- Post-mortem investigation of production **hangs** and **managed memory leaks** — including timer-queue-rooted `CancellationTokenSource` leaks and finalizer-thread starvation — using WinDbg/SOS and memory heap snapshots
- Triaged **SELECT N+1** issues across microservice boundaries; identified query fan-out patterns causing latency spikes under real production load
- Established BenchmarkDotNet-based benchmarking culture; systematically identified and eliminated allocation hotspots in hot-path .NET services

#### Snyk
**2021 – 2022**

Senior Backend Software Developer

*TypeScript · Node.js · .NET · dependency graph analysis*

- Contributed to open-source dependency scanning across three language ecosystems: **.NET** (NuGet, `.csproj`/lock file resolution), **Python** (pip, `requirements.txt`, Poetry, `pyproject.toml`), and **Node.js** (npm/yarn, transitive resolution)
- Navigated and extended a large-scale legacy TypeScript codebase; maintained correctness of multi-layer dependency graph resolution logic under active feature development

#### Gigya / SAP
**2020 – 2022**

Senior Backend Software Developer

*.NET · C# · microservices · identity management at scale*

- Designed and built features across a high-volume microservice platform handling millions of identity events per day
- Performance analysis and post-mortem investigation of production issues; full-stack work spanning backend services and internal tooling

#### Hibernating Rhinos
**2013 – 2020**

Server/Client Developer · Support Engineer

*.NET · C# · RavenDB · Voron B-Tree storage engine · LMDB · memory-mapped I/O*

- One of the longest-tenured engineers on the RavenDB core team — **7 years** on a production NoSQL database and its storage engine
- Implemented and maintained features in the **Voron** B-Tree storage engine: tree structure, transaction management, memory-mapped I/O, page management
- Designed and built client-side and server-side API components across multiple RavenDB major versions; worked across the full stack from the wire protocol down to storage
- Provided direct technical support to enterprise customers; diagnosed and resolved storage-layer bugs under production pressure

#### Early Career
**2010 – 2013**

Junior developer roles including internal tooling and web development; concurrent with intensive self-directed study in systems programming and .NET internals.

---

### Education

**Technion – Israel Institute of Technology**

Physics B.Sc. · Attended ~4 semesters, did not complete · Left to work; family financial constraints

**Self-taught since ~age 12.**

12th-grade final project: a fully self-implemented paint application in C++ under DOS — trigonometry-based shape rendering, INT 10h, AX=0x0013 (320×200, 256 colour), writing pixels directly into VGA framebuffer memory at 0xA000:0000.

No libraries. No tutorials. Just Peter Norton's book, the RBIL, a compiler, and a lot of time.

---

### Projects

**[SharpArena](https://github.com/myarichuk/SharpArena)**

Zero-allocation arena allocator for high-performance .NET. Linked-list-of-segments bump allocator backed by `NativeMemory`, with built-in collections: `ArenaList<T>`, `ArenaPtrStack<T>`, `ArenaUtf16String` (content-based equality, zero managed allocation). Ships with a symbolic math parser as a real-world stress test. Fully compatible with **Blazor WebAssembly** — no virtual memory APIs in the hot path. 53 releases.

**[MongoZen](https://github.com/myarichuk/MongoZen)**

Identity Map + Unit of Work ORM for MongoDB in .NET. Roslyn source generators wire up `DbSet<T>` at compile time — zero reflection on the hot path. Change tracking uses an unmanaged arena-backed diffing engine with a zero-alloc `DocId` discriminated union fingerprint struct. Benchmarked against hand-optimised raw driver code: **~50–100× faster** on repeated reads (Identity Map cache), **33% faster and 7% less allocated** on ReadAndModify vs. hand-written `BulkWrite`.

**[CampaignVault](https://github.com/myarichuk/campaignvault)** (WIP)

AI-assisted D&D campaign management tool — session tracking, NPC and world-state consistency, AI-driven narrative assistance. Exploring multi-agent orchestration for DM support workflows.

---

### Writing

[graymatterdeveloper.com](https://www.graymatterdeveloper.com) — 33 posts covering .NET debugging and post-mortem analysis, production systems engineering, performance profiling, design patterns, and C++ internals. Highlights include a multi-part *Production-Ready* series (memory dump tooling, structured logging, performance metrics, testing strategy) and hands-on debugging write-ups. Cadence is uneven — a daughter recently arrived who has discovered that closing a laptop lid is the most effective scheduling mechanism available to her.

---

### Skills

**Languages:** C#, .NET, C++, TypeScript, Node.js

**Low-level .NET:** Span<T>, NativeMemory, unsafe code, stackalloc, unmanaged generics, struct layout, pinned memory, memory-mapped I/O

**Platform:** .NET Core, .NET 6–10, Blazor WASM, ASP.NET Core, WCF

**Databases:** Voron (B-Tree), RavenDB, MongoDB, MS-SQL

**Tooling:** Roslyn Source Gen, BenchmarkDotNet, WinDbg / SOS, Testcontainers, Docker, Git

**Specialisations:** Zero-alloc design, Perf profiling, Crash dump analysis, Core dump analysis, Concurrent systems, Storage engine internals, Compile-time codegen

---

### Interests

Systems programming for fun. Fantasy & sci-fi literature. Strategy games — chess, 4X, CRPGs. TTRPGs; currently building tooling for them.

---

### Get in Touch

Have a project in mind or want to discuss systems engineering, debugging, or performance optimization? Send me a message.

<!-- TODO: Add Formspree contact form here when theme supports it (planned for NexT migration) -->

```
Name:
Email:
Message:

[Send]
```
