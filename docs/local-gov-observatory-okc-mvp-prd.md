Local Gov Observatory – OKC MVP

Product Requirements Document (PRD)
Version: v0.1
Owner: [Your Name]

1. Problem Statement

Local political behavior (voting, budgeting, contracting) is technically public but practically inaccessible and hard to trust. Citizens and journalists must dig through PDFs, meeting minutes, and scattered portals to understand what their representatives actually do with votes and money.

There is currently no simple, verifiable way to:

See how a specific official behaves over time (attendance, voting patterns, themes).

Understand who financially benefits from public decisions.

Detect patterns associated with elevated corruption risk, in a transparent and non-defamatory way.

Run parallel, transparent governance (community voting + fund distribution) on-chain.

The product addresses this by building a verifiable “local politics + money” observatory, starting with Oklahoma City (OKC).

2. Product Vision

Create a public, mobile- and desktop-friendly platform that:

Tracks and explains local officials’ behavior (votes, meetings, budgets, vendors, projects).

Computes a transparent Integrity Risk Index for each official and vendor, based only on public data and documented rules.

Anchors all data snapshots on blockchain, so anyone can verify tampering has not occurred.

Eventually plugs into on-chain governance rails (DAO treasuries, on-chain voting and funding for local public-good projects).

Long term, this becomes a “local gov terminal” + public integrity layer, reusable across cities and states.

3. Goals & Success Metrics (MVP: OKC)
3.1 Primary Goals

Provide a single public page per official (e.g., OKC City Council members) that shows:

Identity and term information.

Voting behavior and attendance.

Money flows overlapping their ward/jurisdiction.

Integrity Risk Index with clearly explained components.

Make all data independently verifiable:

Every item (vote, budget line, project) has a canonical JSON snapshot, a content hash, and an on-chain registry entry.

Ship a simple, fast web app (mobile + desktop) usable by non-technical citizens.

3.2 Success Metrics

Time-to-answer

From landing on the site, a user can see an official’s recent votes and related money flows in under 10 seconds.

Verification usage

At least 20% of sessions include a user opening a verification element (canonical JSON, hash, or chain transaction).

Data coverage

For OKC City Council:

≥ 1 full fiscal year of budget data.

≥ 6–12 months of complete meeting + vote history.

Trust / clarity (qualitative)

Users can accurately describe, in plain language, what the Integrity Risk Index means and what it does not mean.

4. Scope
4.1 In Scope (MVP)

Geography

Oklahoma City (city level), focusing on:

OKC City Council.

Core departments (e.g., streets, parks, utilities).

Data

Council meetings and votes (roll-call where available).

Annual budget and line items.

Major capital / bond projects.

Approximate ward-level allocation and project overlap.

Features

Official profile pages with:

Integrity Risk Index v1.

Component breakdown (opacity, benefit concentration, process deviation, network risk, context baseline).

Recent key votes table.

Ward-linked money flows summary.

Data verification:

Canonical JSON per record.

SHA-256 hashes.

On-chain registry entries (e.g., on Base or another L2).

UI with “Verify” panels.

Public, read-only web UI (no login required).

4.2 Out of Scope (MVP)

Full campaign finance integration (possible Phase 2+).

ML-based “corruption prediction” (MVP uses deterministic rules only).

Direct integration into city/state internal decision systems.

Running real-money DAO treasuries at scale (beyond a conceptual stub or testnet demo).

5. Personas

Engaged Citizen (Primary)

Wants to know what their council member actually does.

Needs simple visuals (charts, tables) and plain language, not raw CSVs.

Journalist / Watchdog Researcher

Needs exact details: votes, vendors, amounts, timelines, and historical patterns.

Needs exportable data and cryptographically verifiable proof.

Civic Tech / Web3 Builder

Interested in the data model, architecture, and open APIs.

Wants to build bots, dashboards, alerts, or additional tools on top.

Local Official / Staff (Secondary)

May use the site to see how they are represented.

Needs transparent methodology for the risk index; should be able to audit data and computations.

6. Key User Stories

As a citizen, I want to search for my council member, see their attendance, recent votes, and spending overlaps with my ward, so I can decide whether they represent my interests.

As a journalist, I want to open a controversial emergency / no-bid contract and see:

Which officials voted yes/no.

How much money was committed and to whom.

The canonical data snapshot + hash + blockchain reference.

As a researcher, I want to see an Integrity Risk Index breakdown for an official and drill into the exact events used to compute each component, so I can evaluate the methodology.

As a builder, I want to call open APIs to fetch:

Official data and integrity metrics.

Meeting & vote records.

Budget line items and verification metadata.

(Later, Phase 2) As a citizen, I want to vote in a parallel DAO on local projects, where all funds and decisions are traceable on-chain.

7. Functional Requirements
7.1 Data Ingestion & Normalization

The system must periodically fetch data from:

OKC council meetings and agenda/roll-call portals.

OKC budget and bond datasets and/or PDFs.

The system must normalize this into internal entities:

officials

bodies (e.g., “OKC City Council”)

meetings

agenda_items

votes

budgets

budget_line_items

projects

jurisdictions (e.g., wards)

For each ingested record, the system must:

Produce a canonical JSON representation:

Stable field names and ordering.

Compute a SHA-256 content hash of the serialized JSON.

Store in Postgres:

Canonical JSON (JSONB).

Content hash.

Source URL(s).

Retrieval timestamp.

Any parse/ETL metadata (e.g., source version, parsing warnings).

7.2 Blockchain Registry (On-Chain Anchoring)

A simple smart contract (e.g., on Base L2) must support at least:

registerSource(sourceId, metadataURI)

Registers a data source (e.g., okc_meetings, okc_budget_2025) with metadata.

logEvent(sourceId, eventType, contentHash, offchainURI, timestamp)

Appends a verifiable event log entry:

sourceId: identifier of the ingested source.

eventType: e.g., council_vote, budget_line, project_record.

contentHash: SHA-256 hash from canonical JSON.

offchainURI: IPFS CID or backend URL serving the canonical JSON.

timestamp: when the data was retrieved/anchored.

ETL requirements:

For any new or changed record (hash change), ETL must:

Upsert into Postgres.

Anchor via logEvent.

Store the resulting transaction hash / event ID.

UI requirements:

For any record (vote, budget line, project), the UI must:

Show its canonical hash.

Link to the blockchain explorer for the corresponding registry transaction.

Explain succinctly what anchoring means and does not mean.

7.3 Integrity Risk Index (v1 – Deterministic)

The system must compute a per-official Integrity Risk Index (0–100) based on the following components:

Opacity

Attendance rate; proportion of missing/abstain votes.

Frequency of late-added/emergency agenda items supported by the official.

Benefit Concentration

For spend overlapping that official’s ward/jurisdiction:

Share of total dollars going to top vendors (e.g., top 1, top 3, top 5).

Higher concentration = higher risk score on this dimension.

Process Deviation

Share of contracts/authorizations that are:

No-bid.

Emergency or shortened-process.

Weighted by whether the official voted to approve.

Network Risk

Patterns of repeated approvals benefitting the same vendor cluster.

Overlap between “high-concentration” vendors and repeat beneficiaries.

Context Baseline

A jurisdiction-level baseline risk (e.g., for “OKC City Council”) derived from:

Overall use of no-bid/emergency contracting.

External indicators (audit findings, etc.) where available.

Implementation requirements:

Each component must be computed as a deterministic function of stored events.

The overall index must be a weighted combination of components; weights and formulas must be documented.

Each official’s index must be tagged with a scoring_model_version (e.g., v1.0).

UX / language requirements:

The UI must clearly label this as an Integrity Risk Index or Integrity Risk Score, not a corruption label.

A visible disclaimer must state that:

The score is a data-driven risk indicator,

It is not a legal finding or accusation of corruption.

Users must be able to click into each component and see:

The underlying events (votes, budget lines, etc.) that contribute to the score.

Each event’s canonical JSON, hash, and chain tx link.

7.4 Web Application (Frontend)

Views

Official Profile View

Identity and role (name, ward, term dates, party if applicable).

Integrity Risk Index:

Overall score.

Component chart (e.g., bar/radar).

Brief explanations/labels.

Recent key votes table:

Meeting + item.

Vote (yes/no/abstain).

Associated budget impact (if applicable).

Flags (e.g., “emergency”, “no-bid”, “high vendor concentration”).

Ward-linked money flows:

Summary by stream/category (e.g., capital, grants, parks).

Vendor concentration metrics.

Verification panel:

Short explanation of canonical data + hashing + on-chain anchoring.

Links to canonical JSON and blockchain explorer.

Jurisdiction Overview View (OKC City)

Budget overview:

Total budget by category/department for a given year.

Ward or area overview:

Map/list of wards with spending and projects.

Officials list:

All council members with Integrity Risk Index summary.

Record Detail View

For a vote, budget line, or project:

Full contextual details (who, what, when, where, how much).

Canonical JSON (pretty-printed, readable).

Hash and blockchain tx link.

UI/UX Requirements

Fully responsive (mobile-first design) and functional on desktop.

Official profile page should feel like a “dashboard” but remain readable.

Use tooltips and brief inline explanations for jargon (e.g., “no-bid”, “canonical hash”).

Strive for fast initial load, minimal heavy libraries.

7.5 Public API

Provide read-only API endpoints, for example:

GET /api/officials/:id

GET /api/officials/:id/votes

GET /api/officials/:id/money-flows

GET /api/meetings/:id

GET /api/budgets/:year

GET /api/records/:type/:id (returns canonical JSON + hash + on-chain tx + metadata)

Each API response must include:

source_url (original data source).

canonical_hash (SHA-256).

onchain_tx (if anchored).

model_version (for integrity scores where applicable).

8. Non-Functional Requirements

Performance

Official profile page:

Median load time < 2 seconds (when cached).

95th percentile load time < 4 seconds.

API:

95th percentile response time < 500 ms for typical queries.

Availability

Target ≥ 99% uptime for UI and API (MVP level).

Security

Backend is read-only with respect to public data (no privileged write endpoints exposed).

No sensitive personal data beyond publicly available info on public officials.

Transparency & Explainability

Public documentation for:

Data sources and ETL pipelines.

Integrity Risk Index formulas and versions.

Known limitations, caveats, and update frequencies.

Legal / Ethical

Avoid direct labels such as “corrupt” or “clean”; use neutral risk framing.

Ensure wording emphasizes descriptive, factual data and non-definitive risk indicators.

Consult legal guidance (when possible) on framing and disclaimers.

9. Technical Stack & Architecture (Proposed)

Frontend

Next.js (React, TypeScript), App Router.

Tailwind CSS for responsive UI.

Deployed via Vercel (or equivalent).

Backend / API

Next.js API routes, or separate Node/FastAPI service.

Connects directly to Postgres for reads.

Database

Managed Postgres (e.g., Supabase, Neon, RDS).

JSONB columns for canonical snapshots.

Indexed columns for key filters (official, meeting, year, department, vendor).

Jobs / ETL

Python or TypeScript scripts.

Scheduled via GitHub Actions / cron-like runner.

Separate workers per source (e.g., meetings, budget, projects).

Blockchain

Simple registry contract on Base testnet (for dev), then Base mainnet.

Only hashes and basic metadata stored on-chain.

Storage

Canonical JSON in Postgres.

Optional IPFS pinning for long-term public storage of snapshots.

10. Analytics & Observability

User Analytics

Track:

Page views per official profile.

Clicks on “Verify” elements (canonical JSON, hashes, chain tx links).

API request volume by endpoint.

System Health

Dashboards for:

ETL job status (last run, failures, lag).

Data coverage metrics (number of meetings, votes, budget lines ingested).

Chain anchoring success/failure.

Logging

Error and warning logs for ETL and API.

Trace logs for any failed hash/anchoring operations.

11. Risks & Mitigations

Data Quality / Completeness

Risk: City data may be incomplete, inconsistent, or change format.

Mitigation:

Expose data freshness and completeness indicators.

Maintain versioned ETL mappings and clear error logs.

Allow users to see raw source URLs.

Misinterpretation of Risk Index

Risk: Users treat scores as definitive proof of corruption.

Mitigation:

Prominent disclaimers and clear language.

Provide raw indicators and context, not just a single number.

Open-source the scoring model for scrutiny.

Legal Pushback from Officials

Risk: Perceived defamation or unfair portrayal.

Mitigation:

Emphasize factual, sourced data and neutral language.

Provide mechanisms to correct factual errors.

Document methodology and data sources clearly.

Blockchain Complexity / Confusion

Risk: Users may not understand or trust the on-chain aspects.

Mitigation:

Keep the contract simple; avoid complex tokenomics in MVP.

Provide clear, non-technical explanations of anchoring.

Treat on-chain proofs as a backstop for auditors rather than a core UX dependency.

12. Roadmap (High-Level)
Phase 0 – Design & Data Model

Finalize entity schemas for OKC (officials, meetings, votes, budgets, projects).

Define Integrity Risk Index v1 formulas and component definitions.

Build static, non-functional prototype of an official profile page.

Phase 1 – OKC Core MVP

Implement ETL for:

OKC council meetings + votes.

One full budget year + key line items.

Implement blockchain registry contract + anchoring integration.

Implement backend API and frontend official/jurisdiction views.

Release a public beta for OKC City Council.

Phase 2 – Depth & DAO Stub

Extend data history (more years, more departments).

Refine the Integrity Risk Index based on feedback; open-source model.

Implement a small DAO module (testnet) showcasing:

Community proposals.

On-chain voting.

On-chain disbursements for a demo project.

Phase 3 – Expansion

Expand to additional OKC boards/commissions and possibly county or state-level data.

Create a repeatable template for porting this architecture to other cities.

Explore production DAO treasuries and deeper civic collaborations.
