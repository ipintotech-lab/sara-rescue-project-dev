# SARA Platform Skeleton

A working skeleton of SARA: one thin path from any data source to an object on the map, with every tool behind a generic connector.

The point is to compare tools (OpenDroneMap, Hugin, Metashape, Polycam, authority feeds and others) on the same inputs **without changing the core**.

> **Not for operational use.** This skeleton has no offline sync, war rooms, damage engine or real authorization. Do not use it in a real incident.

The presentation demo lives in a separate repository (`project-rescue`). This repository is where the skeleton is built.

## Status

Planning is done and implementation has not started. The work is tracked as [issues](../../issues) and [milestones](../../milestones).

## What the skeleton settles early

These are the parts that are expensive to change later:

- **Object model:** Incident, Building, RawItem, EvidenceItem, SpatialAsset, CandidateLink, ConnectorInstance, IngestRun/ProcessingJob, Lineage, Event
- **Connector contract:** one manifest and lifecycle for all five kinds (pull feed, push, file import, processor, exporter)
- **Ingest pipeline:** fetch → store raw → validate → normalize → resolve → deduplicate → persist and emit
- **Lineage:** every object traces back to its raw inputs, connector version and processing parameters
- **Asset store:** private storage, with files addressed by ID and checksum

## Non-goals

- Offline-first sync, map packages and Starlink handling
- War rooms, an incident cockpit and decision actions (the data model leaves room for them)
- Damage calculation, a social media verification workflow and biometric data
- Production authentication (the skeleton uses a development login with fixed roles)

## Planned stack

| Part | Choice |
|---|---|
| Repository | TypeScript monorepo (pnpm workspaces) |
| API | Node.js, Fastify, OpenAPI; schemas validated with Zod or TypeBox |
| Database | PostgreSQL with PostGIS |
| Job queue | PostgreSQL-based (pg-boss or Graphile Worker) |
| Object storage | S3-compatible (MinIO locally) |
| Web client | MapLibre GL JS with a deck.gl overlay |
| Heavy processors | Separate containers (NodeODM, Hugin, PDAL) |
| Local environment | docker compose, started with one command and seeded with data |
| Hosting | Railway for API, web and worker; processors on a separate VM |

## Planned layout

```
apps/
  api/             Fastify API
  web/             MapLibre web client
  worker/          Ingest and job runner
packages/
  shared/          Schemas and types shared by all apps
  connector-sdk/   Connector contract, registry types, contract test suite
connectors/
  */               One package per connector (vendor code lives only here)
```

## Milestones

| Milestone | Gate | Estimate |
|---|---|---|
| M0 Foundation | One command starts the stack; CI runs tests and migrations | 1 week |
| M1 Core pipeline | A mock item flows from fetch to map with full lineage; bad input stays in quarantine | 2 weeks |
| M2 Feeds and devices | Recorded USGS traffic with duplicates and an outage produces correct records; replay gives the same result | 1–2 weeks |
| M3 Processors | The test drone set produces an orthophoto and point cloud with lineage; a blurred set is rejected | 2 weeks |
| M4 Viewer and comparison | Two tools compared on the same building, with metrics stored and visible | 1–2 weeks |

## Rules

- The core (API, pipeline, web) never imports a vendor SDK. Vendor code lives only inside a connector package.
- Swap a tool by configuration, not by editing core code.
- Raw input is stored, immutable and checksummed, before anything else happens to it. Bad input goes to quarantine and stays visible.
- Outputs are observations at capture time, not structural assessments.
- AI outputs are labeled suggestions and never feed a measurement.
- Gaussian splats are display-only and never feed a calculation.
- Each connector records its licenses. Data that cannot be used offline or analyzed is excluded.
- A missing or failed layer is shown as unavailable, never as empty.
