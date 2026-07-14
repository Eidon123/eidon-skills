---
name: eds
description: Eidon Skills (EDS) routing entry for Agent workspace tooling. Use when the user mentions EDS, Eidon Skills, Agent workspace organization, cross-host skill migration, skill source-of-truth, host entry alignment, bridges, or symlink audits and needs the correct EDS capability selected before work begins.
---

# Eidon Skills Router

Route the request to the matching EDS skill before taking action.

## Available Skills

| Request | Skill |
| --- | --- |
| Audit or migrate AGENTS.md, CLAUDE.md, SOURCE_OF_TRUTH.md, project/global skills, host entries, bridges, or symlinks | `eds-agent-migration` |

## Routing Rules

1. Identify the user's concrete workspace outcome.
2. Load the matching leaf skill and follow its workflow in full.
3. If no published leaf skill covers the request, perform only a read-only inventory and explain the missing capability. Do not invent a migration or update procedure.
4. Keep long-term workflow logic in leaf skills; keep this router concise.

## Current Release Boundary

Version 0.1.0 publishes the router and `eds-agent-migration`. Additional `eds-*` capabilities will be added in later releases.
