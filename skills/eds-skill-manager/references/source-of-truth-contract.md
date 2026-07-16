# EDS Source of Truth Contract

Use this contract when creating, migrating, repairing, or extending the global EDS `SOURCE_OF_TRUTH.md`.

## Contents

- Identity and discovery
- Structured Markdown schema
- Intended and verified state
- Workspace registry
- Migration and concurrency
- Derived-data exclusions

## Identity And Discovery

The canonical instance is:

`<confirmed-global-skill-root>/SOURCE_OF_TRUTH.md`

Identify a current `eds-sot/v1` instance by all of these facts:

- Its canonical path is under the user-confirmed global skill root.
- `schema` is `eds-sot/v1`.
- `authority-domain` is `eds-global-structure`.
- `managed-by` is `eds-skill-manager`.

A legacy migration candidate is a file at the canonical location whose self-described global root matches the user-confirmed root and which contains no contradictory authority or manager claim. It does not need fields introduced by `eds-sot/v1`.

Do not treat every file named `SOURCE_OF_TRUTH.md` as a conflict. Resolve logical paths to physical identity first. `canonical-path` is the user-confirmed absolute logical path, without `~`; record a different resolved realpath only as Verified evidence. The canonical EDS scope is `machine`. For conflict checks, `machine` overlaps every narrower scope, identical normalized `workspace:<realpath>` scopes overlap, and distinct workspace scopes do not. An unknown or malformed scope cannot prove coexistence and therefore fails closed.

Two candidates conflict only when they are different physical files and their scopes overlap in the same authority domain. Project business SOT files belong to other authority domains. A distinct file discovered through a verified reference that claims `eds-global-structure` but is outside the canonical root is an invalid competing claimant, not a harmless residue; stop before writing until it is resolved.

Discovery is bounded to the confirmed global skill root, the current project, and paths referenced by verified rules, entries, lock data, or upstream metadata. Do not search the whole home directory for same-named files.

## Structured Markdown Schema

Use stable headings and `key: value` list items. Do not use tables for canonical records. Keep the instance Agent-first and readable in Obsidian.

```markdown
# EDS Global Structure Source of Truth

## Contract

- `schema`: `eds-sot/v1`
- `scope`: `machine`
- `authority-domain`: `eds-global-structure`
- `canonical-path`: `/absolute/path/to/global-skill-root/SOURCE_OF_TRUTH.md`
- `managed-by`: `eds-skill-manager`
- `read-when`: `skill-structure, host-entry-governance, rule-map-governance, explicit-cross-workspace-location`
- `write-policy`: `manager-only`

## Intended Structure

### Skill Roots And Conventions

#### root:global-standard

- `logical-path`: `...`
- `canonical-path`: `/absolute/logical/path/to/global-skill-root`
- `role`: `installed-copy-root`
- `management`: `...`

### Managed Sources And Policies

#### source:example

- `role`: `publication-upstream`
- `location`: `example/repository`
- `management`: `installer-managed`
- `member-policy`: `all-formal-members`

### Host And Rule Mappings

### Workspace Registry Policy

- `setup-decision`: `undecided`
- `prompt-policy`: `after-full-audit`

### Registered Workspaces

#### workspace:example

- `name`: `Example`
- `aliases`: `example-alias`
- `root`: `/absolute/logical/path/to/workspace`
- `purpose`: `example-authority-domain`
- `rules-entry`: `AGENTS.md`
- `lifecycle-intent`: `registered`

### Exceptions And Tombstones

## Verified State

### Verification

#### domain:skill-structure

- `status`: `verified`
- `evidence`: `...`
- `acceptance`: `structure-only`
- `verified-at`: `YYYY-MM-DDTHH:MM:SS+TZ`

#### domain:workspace-registry

- `status`: `active`
- `evidence`: `all registered workspace records passed structural verification`
- `acceptance`: `structure-only`
- `verified-at`: `YYYY-MM-DDTHH:MM:SS+TZ`

#### workspace:example

- `status`: `verified`
- `realpath`: `/resolved/physical/path/to/workspace`
- `evidence`: `root and rules entry are readable`
- `acceptance`: `structure-only`
- `verified-at`: `YYYY-MM-DDTHH:MM:SS+TZ`

### Unmanaged Observations

#### unmanaged:example

- `status`: `unmanaged`
- `evidence`: `...`
- `acceptance`: `audit-only`
- `verified-at`: `YYYY-MM-DDTHH:MM:SS+TZ`

### Unresolved Drift

#### drift:root:global-standard

- `status`: `missing`
- `evidence`: `...`
- `acceptance`: `audit-only`
- `verified-at`: `YYYY-MM-DDTHH:MM:SS+TZ`
```

The three top-level sections and every Contract singleton key must each appear exactly once. Duplicate required sections or keys, missing required values, duplicate record IDs within the same subsection, and unknown parallel top-level sections are invalid and must fail without writes. A paired object intentionally reuses its stable record heading under Intended and Verification; those are two state views, not duplicate records.

The headings are stable. Use `workspace:<id>` for paired workspace records, `unmanaged:<id>` for unmanaged observations, and `drift:<intended-record-id>` for unresolved drift. Add records under the canonical subsections rather than inventing parallel top-level sections. Explanatory prose is allowed only when a terse field cannot preserve the reason for an exception. Use stable serialization: retain the documented section and key order, sort records lexically by stable ID within each subsection, keep user-confirmed logical paths in Intended, keep resolved realpaths in Verified, and do not reserialize an unrelated section.

## Intended And Verified State

`Intended Structure` records user-approved intent that cannot be safely inferred from the current filesystem. A scan must not silently add, remove, or rewrite intended records.

`Verified State` records observations with evidence:

- `status`: `verified`, `missing`, `unmanaged`, `degraded`, `unknown`, or another explicitly defined status.
- `evidence`: the paths, link targets, lock/upstream facts, or host tests supporting the status.
- `acceptance`: `host-smoke-tested`, `structure-only`, or `audit-only`.
- `verified-at`: when that evidence or status last changed materially.

If evidence, status, and acceptance are unchanged, do not rewrite `verified-at`. This keeps repeated verification idempotent. A missing intended object remains in Intended and appears under `Unresolved Drift` with `status: missing`; do not invent a second literal `drift` status. An extra discovered root, mapping, exception, or workspace candidate appears as unmanaged and does not enter Intended without user approval. Ordinary installed members reconstructed from a suite, lock, or upstream are runtime inventory, not one unmanaged record per member; record only structural anomalies that need action.

Never write `healthy` or `verified` from directory existence alone. Use the acceptance levels defined by the manager workflow.

## Workspace Registry

The registry is optional and secondary to skill lifecycle management. Derive one effective state without persisting a duplicate projection:

- Intended stores `setup-decision` as `undecided` or `declined-permanently`.
- If that decision is `declined-permanently`, it is the effective state and setup is not offered unless the user explicitly resets it.
- Otherwise, Verified stores operational `status` as `not-configured`, `configuring`, `active`, or `degraded`.
- `not-configured` offers setup after the next eligible full audit.
- `configuring` means the exact registry manifest was approved and its write began, but verification is incomplete.
- `active` means all registered entries passed the agreed structural verification.
- `degraded` means an approved registration failed its first verification but remains recorded, or an active entry later drifted; repair, rather than setup, is offered.

Choosing "later" leaves `setup-decision: undecided` and operational `status: not-configured`. A permanent decline can be remembered only by an approved Intended update in a writable SOT. Do not create another preference file or rely on model memory.

Use these transitions:

- selecting setup starts discovery but does not write; after the exact registry manifest is approved, one atomic write adds the Intended records and sets Verified to `configuring`, then a re-read confirms that transaction before target verification
- successful write and structural verification changes Verified from `configuring` to `active`
- failed verification rolls Verified back to `not-configured` when no registration remains, or records `degraded` when an approved entry remains but drifts
- "later" changes nothing; an approved permanent decline changes Intended to `setup-decision: declined-permanently`
- an explicit reset changes Intended back to `setup-decision: undecided`; successful repair changes Verified from `degraded` to `active`

No SOT is not a registry state and does not bypass the normal SOT creation workflow.

Partition each registered workspace across the two state sections. Intended contains only:

- stable `id`
- display `name` and optional `aliases`
- user-confirmed absolute logical `root`
- one-line `purpose` or authority domain
- relative or absolute `rules-entry`
- lifecycle intent

Verified contains only observed fields:

- the matching stable `id`
- resolved `realpath` when different
- `status` and `evidence`
- `acceptance`
- `verified-at`

Registration provides location, not blanket authorization. When a workspace becomes relevant to the user's request, resolve it, then read its declared local rules before further work. Continue to obey the user's scope, target rules, and operating-system or service authorization.

Do not scan for workspaces until the user selects setup. Then obtain or propose a finite set of candidate roots, get permission for those roots, perform read-only discovery, show evidence, and require explicit selection before registration. Never hard-code home, Obsidian, iCloud, or a product-specific workspace layout.

Offer registry setup at most once per eligible run, after a top-level full skill audit that completed its declared scope without partial or failed results. Nested manager runs return the offer state to their top-level caller instead of prompting directly. That caller may show the choices only when its overall run independently satisfies the same top-level full skill audit condition. Ordinary inventory does not poll registered workspaces; change `active` to `degraded` only when explicit registry verification or relevant workspace use produces drift evidence.

## Migration And Concurrency

For a legacy SOT:

1. Record the original file hash.
2. Recompute current structural evidence instead of copying counts or member lists.
3. Map explicit policies, identity relationships, exceptions, and tombstones into Intended.
4. Map supported observations into Verified.
5. Stop before writing if an unknown section cannot be classified without losing intent; ask the user instead of silently dropping or persisting it as authoritative data.
6. Re-read the source immediately before writing. If its hash changed, discard the preflight and re-audit.
7. Re-read and validate the written file. A second run with unchanged evidence must produce no diff.

The original hash is transient preflight evidence and is not persisted in the SOT. A legacy requirement to maintain a persistent Catalog or other reconstructable inventory is superseded by this contract: report its removal, preserve any non-derivable policy it contains in the appropriate Intended records, and do not preserve the artifact obligation itself. If mixed content cannot be separated safely, stop and ask.

Only `eds-skill-manager` writes the SOT. Migration and other agents provide exact mappings or evidence. Preserve fields outside the section being updated, and stop on overlapping concurrent changes. Recheck the preflight hash immediately before replacement, write the candidate to a same-directory temporary file, and use an atomic rename. In a multi-write transaction, re-read and record a new baseline hash after each verified write; the next compare must use that new hash, not the original preflight hash. This is optimistic concurrency, not a distributed lock; when concurrent writers are plausible, acquire a short-lived exclusive lock if the environment supports one or stop rather than overwrite.

## Derived-Data Exclusions

Do not persist any of these in the SOT:

- complete installed skill member lists
- suite member expansions when upstream or lock can reconstruct them
- global, lock, suite, host, or project member counts
- full hash inventories
- transient audit logs or backup manifests
- copied `AGENTS.md`, `CLAUDE.md`, or skill bodies
- business module indexes or workspace content
- credentials, cookies, tokens, or authorization grants

Generate inventories in the current response when needed. Do not create or maintain a persistent `CATALOG.md`. Keep source policies, intentional subsets, manual-management exceptions, nonstandard mappings, and tombstones because those are not safely reconstructable from a point-in-time scan.
