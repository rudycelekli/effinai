---
name: llm-wiki
description: "Second brain for AI agents + Obsidian where the LLM incrementally ingests sources into a persistent, interlinked markdown vault. Knowledge compounds — cross-references, contradictions, and synthesis are already there when you query."
version: "1.0.0"
tags: [knowledge-management, obsidian, second-brain, wiki, research, pkm]
createdBy: "built-in"
status: "active"
---

# LLM Wiki — Second Brain for AI Agents + Obsidian

Inspired by Andrej Karpathy's LLM Wiki pattern. This skill turns an AI agent into a disciplined wiki maintainer that **incrementally builds and maintains** a persistent, interlinked Obsidian vault as you feed it sources. The knowledge compounds — cross-references, contradictions, and synthesis are already there when you query.

## Core Principle

Most LLM+docs workflows are **RAG**: retrieve fragments at query time, synthesize from scratch, forget. The wiki is **compounding**: sources are read once, integrated into a persistent markdown knowledge base, and kept current. You curate and ask; the LLM reads, files, cross-references, and maintains.

> Obsidian is the IDE. The LLM is the programmer. The wiki is the codebase.

## When to Use

- **Personal**: track goals, health, psychology, journaling, self-improvement
- **Research**: deep dives over weeks on a topic — papers, articles, reports, evolving thesis
- **Book companion**: file chapters as you read; build a fan-wiki-style companion
- **Business/team**: internal wiki fed by Slack, meeting notes, calls — LLM does maintenance nobody else wants to do
- **Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives**

**Do NOT use when:** you need one-shot Q&A over a fixed document (use RAG), you don't plan to add sources over time, or you don't want Obsidian in the loop.

## Architecture (Three Layers)

```
vault/
├── raw/                    # Layer 1 — IMMUTABLE source of truth
│   ├── <source files>      # Articles, papers, PDFs, images, data
│   └── assets/             # Downloaded images from clipped articles
├── wiki/                   # Layer 2 — LLM-owned knowledge base
│   ├── index.md            # Content catalog (LLM updates every ingest)
│   ├── log.md              # Append-only timeline
│   ├── entities/           # Person/Org/Place pages
│   ├── concepts/           # Ideas, theories, frameworks
│   ├── sources/            # One summary page per ingested source
│   ├── comparisons/        # Cross-source analysis pages
│   └── synthesis/          # High-level syntheses, theses, overviews
└── CLAUDE.md               # Schema + conventions
```

- **Layer 1 (raw/)** — you own. LLM only reads; never writes.
- **Layer 2 (wiki/)** — LLM owns. It creates, updates, and cross-references pages. You read it.
- **Layer 3 (CLAUDE.md)** — the schema. Conventions, workflows, frontmatter rules.

## Three Core Operations

### 1. Ingest
LLM reads a source, discusses takeaways with you, writes a source summary, updates 10-15 relevant pages, updates index, appends to log.

### 2. Query
LLM reads index.md first, drills into relevant pages, synthesizes with citations. Good answers get **filed back into the wiki** so explorations compound.

### 3. Lint
Health check: contradictions, stale claims, orphan pages, missing cross-refs, concepts mentioned but lacking their own page, data gaps to fill with web search.

## Slash Commands

| Command | Purpose |
|---|---|
| `/wiki-init` | Bootstrap a fresh vault with schema files + starter structure |
| `/wiki-ingest <path>` | Read a source, discuss, update wiki, log it |
| `/wiki-query <question>` | Search wiki, synthesize answer, offer to file back |
| `/wiki-lint` | Run health check — contradictions, orphans, stale claims, gaps |
| `/wiki-log` | Show recent log entries |

## Sub-Agents

| Agent | When dispatched |
|---|---|
| `wiki-ingestor` | Delegated ingest flow — reads source, proposes updates, applies after approval |
| `wiki-linter` | Runs the health-check workflow independently, reports findings |
| `wiki-librarian` | Answers queries using index-first search, synthesizes with citations |

## Why This Works (vs Plain RAG)

| Plain RAG | LLM Wiki |
|---|---|
| Rediscover knowledge each query | Knowledge accumulates |
| Cross-references re-computed every time | Cross-references pre-written and maintained |
| Contradictions surface only if you ask | Contradictions flagged during ingest |
| Exploration disappears into chat history | Good answers re-filed as new pages |
| Scales by embeddings infrastructure | Scales by markdown + index.md + optional local search |

## Obsidian Setup (Recommended)

- **Obsidian Web Clipper** — browser extension; converts web articles to markdown into raw/
- **Download images locally** — Settings -> Files and links -> Attachment folder path = raw/assets/
- **Graph view** — see hubs/orphans; essential for spotting structural problems
- **Dataview plugin** — dynamic tables/lists over page frontmatter
- **Git** — the vault is a plain markdown repo; version it

## Iron Rule

**The LLM never edits files in `raw/`.** Ever. Sources are immutable. All LLM writes go to `wiki/`. If you need to correct a source, do it in `raw/` yourself — then re-ingest.
