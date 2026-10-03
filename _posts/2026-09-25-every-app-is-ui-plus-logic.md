---
layout: post
categories:
  - journal
title: Every App Is Just UI + Logic
subtitle: >-
  A blueprint for a platform where anyone describes the app they want, AI builds
  it, and the boring-but-hard parts (data, security, sync) just work.
tags:
  - ai
  - architecture
author: Tech Tana
date: '2026-09-25'
---
I think we are heading toward a world where nobody picks "the desktop app" or "the iPhone app" or "the watch app" anymore. Everyone will have *their own* app, shaped the way they want to interact with it, on whatever screen happens to be in front of them. If that's true, the interesting product isn't any single app. It's the **builder** that lets people describe the experience they want, uses AI to write the UI and logic, and quietly handles everything people don't want to think about: databases, security, where the data comes from, where it's saved, and how it syncs between phone, PC, and cloud.

This post is my attempt to lay out what that platform looks like, and how to build it so it scales while staying fully customizable.

## The core insight: every app is a black box

Strip away the branding and every application is the same thing: **a UI plus some logic**, processing data that comes from three places:

1. **What the user types in** (inputs)
2. **What lives somewhere else** (data connectors: Stripe, a stock market feed, NOAA weather)
3. **What the app already stored** (the user's own data)

And there are only four things people actually do with an app:

| Pattern | Flow | Example |
|---|---|---|
| **Compute** | input → logic → output | Loan calculator |
| **Store** | input → save | Journal, notes |
| **Store, then process** | input → save → logic later | Expense log → monthly report |
| **View** | connector / stored data → display | Weather dashboard, portfolio |

If that's the whole universe, then an "app" doesn't need to be a compiled binary. It can be a **description**, and one runtime per device renders it.

## Key decision: apps are data, not code

An app on this platform is an **App Spec**, a versioned JSON document. AI writes and edits it, and a single runtime on each device renders it. That decision does most of the heavy lifting:

- **Scalable:** one runtime serves millions of apps, so there's no per-app deployment.
- **Customizable:** every user owns and edits their own spec.
- **Safe:** the spec can only use vetted components and connectors.

Here's what a "money + markets + trip weather" app might look like:

```json
{
  "app": "my-weekend-dashboard",
  "version": 7,
  "data": {
    "connectors": [
      { "id": "bank",    "use": "stripe.balance@2",  "refresh": "15m" },
      { "id": "stocks",  "use": "market.quote@1",    "params": { "symbols": ["AAPL", "NVDA"] }, "refresh": "1m" },
      { "id": "weather", "use": "noaa.forecast@3",   "params": { "location": "$inputs.tripLocation" }, "refresh": "1h" }
    ],
    "inputs": [
      { "id": "tripLocation", "type": "geo" },
      { "id": "tripDate",     "type": "date" }
    ],
    "collections": [
      { "id": "trips", "schema": { "location": "geo", "date": "date", "budget": "money" } }
    ]
  },
  "logic": [
    { "id": "canAfford", "trigger": "on-change",
      "fn": "(bank, trips) => bank.available >= sum(trips.map(t => t.budget))" },
    { "id": "dropAlert", "trigger": "on-connector-update:stocks",
      "fn": "(q) => q.filter(s => s.changePct <= -5).map(s => notify(`${s.symbol} down ${s.changePct}%`))" }
  ],
  "views": {
    "default": [
      { "Card":     { "title": "Balance", "value": "$bank.available" } },
      { "Chart":    { "data": "$stocks", "kind": "line" } },
      { "Forecast": { "data": "$weather", "highlight": "$inputs.tripDate" } }
    ],
    "watch": [
      { "Glance": { "value": "$bank.available", "sub": "$weather.summary" } }
    ]
  },
  "permissions": ["stripe:balance.read", "market:quote.read", "noaa:forecast.read"]
}
```

Nobody writes this by hand. The user says *"Show my Stripe balance, my watchlist, and whether it'll rain on my Saturday hike. On my watch, just the balance and the weather."* The AI produces the spec.

## The architecture, layer by layer

```
┌────────────────────────────────────────────────────────────────────┐
│  1. EXPERIENCE LAYER   Web · iOS · Android · Watch · TV · Voice      │
│     Universal renderer + local-first data replica (offline, CRDT)   │
├────────────────────────────────────────────────────────────────────┤
│  2. AI BUILDER         Conversation → App Spec diff → preview       │
│     Constrained to component catalog + connector registry            │
├────────────────────────────────────────────────────────────────────┤
│  3. LOGIC RUNTIME      Sandboxed WASM functions · triggers · jobs   │
│     Runs on device, edge, or cloud depending on what it needs        │
├────────────────────────────────────────────────────────────────────┤
│  4. CONNECTOR GATEWAY  Registry · credential vault · shared cache   │
│     Stripe · Plaid · Market data · NOAA · Calendar · 3rd-party SDK   │
├────────────────────────────────────────────────────────────────────┤
│  5. PLATFORM           Identity · storage · encryption · sync ·     │
│                        audit · billing · metering                    │
└────────────────────────────────────────────────────────────────────┘
```

### 1. Experience layer (any device)

- **Universal renderer.** The same spec renders on the web (React), on iOS/Android (React Native), on watchOS/Wear OS (a reduced "glance" component set), and even as voice or chat (a text rendering of the same view tree).
- **Graceful degradation per form factor.** Every component declares smaller versions of itself: *Chart → Sparkline → single number*. The renderer picks based on screen size and capability. Users can override per device ("on my watch, only today's balance").
- **Local-first.** Each device holds a local replica (SQLite / IndexedDB) and syncs through CRDTs (Automerge/Yjs-style). Apps work offline, and phone ↔ PC ↔ cloud sync never produces a "which version wins?" dialog.

### 2. AI builder

- **Conversation → spec.** An LLM agent turns natural language into a *spec diff*. It is constrained with tool use and JSON-schema output to the component catalog and connector registry, so it composes vetted building blocks instead of inventing infrastructure.
- **Logic generation.** Custom algorithms are generated as small TypeScript functions, type-checked against connector and collection schemas, compiled to WASM, and run against generated test fixtures before they're published.
- **Iterate by talking.** "Make it darker." "Move weather to the top." "Alert me if AAPL drops 5%." Each change is a new spec version with a live preview, a diff, and one-click rollback.
- **Remix.** Published specs become templates that others can fork. This is the growth loop.

### 3. Logic runtime (the "algorithm" box)

- **Sandboxed.** WASM isolates (Wasmtime / Workers-style) with CPU, memory, and time limits. There's no raw network access. The only I/O goes through platform APIs such as `connectors.call()` and `store.get()/put()`.
- **Placement is the platform's job.** Cheap pure logic runs on the device. Anything that needs secrets, schedules, or heavy compute runs at the edge or in the cloud. The spec declares *what* the logic needs, and the platform decides *where* it runs.
- **Triggers and workflows.** An event bus (NATS/Kafka) plus durable workflows (Temporal-style) handle schedules, alerts, and multi-step pipelines.

### 4. Connector gateway (the moat)

This is where the real value, and the real business, lives.

- **Connector registry.** Each connector is a versioned manifest listing its auth type (OAuth2 / API key / none), its operations with typed input/output schemas, rate limits, cost per call, cache TTL, license terms, and data-sensitivity class.
- **Normalized schemas.** Every weather provider maps to one `Forecast` type, and every brokerage to one `Quote`. Providers become swappable, and AI only has to learn one shape.
- **Credential vault.** User OAuth tokens and platform API keys live in a KMS-backed vault with per-tenant envelope encryption. *Apps and AI-generated code never see a secret.*
- **Shared cache and fan-in.** If 10,000 users ask for the NOAA forecast for the same grid cell, that's **one** upstream call. The cache hit rate is where the margin on paid data comes from.
- **Resilience.** Rate limiting, retries, circuit breakers, and fallback providers.
- **Two modes:**
  - *Bring your own account* (Stripe, a bank through Plaid): the user authorizes, and the platform charges a small per-call or per-month fee.
  - *Platform-licensed data* (market data, premium weather): the platform buys wholesale, meters usage, and resells with a markup.
- **Connector SDK.** Data providers publish their own connectors and earn a revenue share. It's an app store, but for *data*.

### 5. Platform infrastructure (what users never see)

- **Identity.** One account with passkeys/OAuth. Each app gets **scoped, revocable grants** ("this app can read my Stripe *balance*, not my *transactions*").
- **Storage.** Multi-tenant Postgres with row-level security for structured collections, object storage for files, a time-series store for connector history, and a vector index so AI can search over the user's own data.
- **Security.** Encryption in transit and at rest, per-tenant keys, optional end-to-end encryption for sensitive collections, an audit log of every connector call, and data-residency controls. There's also a path to SOC 2 and PCI/GLBA posture for financial connectors.
- **Scale.** Stateless services on Kubernetes, an edge runtime for rendering and light logic, specs and static assets on a CDN, and connector caches in Redis. The cost that grows with each user is mostly storage and metered connector calls, and both of those are billed.

## A request, end to end

I glance at my watch on Saturday morning:

1. The watch renderer loads my spec's `watch` view from its local replica and shows the cached balance and forecast **instantly**.
2. In the background it asks the gateway to refresh `bank` and `weather`.
3. The gateway checks my grants, pulls my Stripe token from the vault, and calls Stripe. NOAA is already in the shared cache because a few thousand other hikers asked for the same grid cell an hour ago.
4. The `canAfford` logic reruns on the device. The new state syncs through CRDT to my phone and laptop.
5. The metering service records one Stripe call and one cache hit, and those feed my bill.

I never picked a database, managed an API key, or thought about sync. I just asked for an app.

## Business model

| Revenue stream | Mechanism |
|---|---|
| Subscription | Free tier (a few apps, basic connectors) → Pro (more apps, schedules, devices) |
| Connector metering | Pass-through cost + margin per call; margin grows with cache hit rate |
| Marketplace take-rate | % of paid templates and third-party connectors |
| Teams / enterprise | Shared apps, SSO, private connectors to internal databases |

## Roadmap

1. **MVP (0–6 months).** Web and phone renderer, about 30 components, 5 connectors (Stripe, Plaid, one market-data API, NOAA, Google Calendar), the AI spec builder, Postgres storage, and cloud-only logic. *Goal: someone builds a "personal finance + trip weather" dashboard in under five minutes.*
2. **Growth (6–18 months).** Local-first CRDT sync, watch and voice renderers, on-device logic, scheduled triggers and alerts, a template marketplace, and the connector SDK for third parties.
3. **Scale (18 months+).** Edge execution, multi-region data residency, enterprise private connectors, end-to-end-encrypted collections, compliance certifications, and connector revenue share at marketplace scale.

## What could go wrong

- **Trust.** The moment you touch someone's bank account, security has to be excellent from day one. The vault and gateway are tier-0 systems.
- **AI correctness.** Generated logic that touches money must be constrained, typed, tested, and previewed before it runs on real data.
- **Connector terms of service.** Some providers forbid caching or resale, so licensing is tracked per connector in the manifest, and the gateway enforces it.
- **The component ceiling.** Some apps will need UI the catalog can't express. Sandboxed custom components are the escape hatch, and they get the same isolation as custom logic.

## Closing thought

Most of what makes software expensive to build isn't the part the user sees. It's the plumbing: auth, storage, sync, integrations, and security. If a platform owns that plumbing once, really well, and lets AI assemble the visible part per person, then "building an app" becomes as casual as describing what you want to see. That's the future I'd like to build toward.
