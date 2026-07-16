# Eidon Skills

Eidon Skills (EDS) provides cross-host Agent workspace migration and skill lifecycle management.

## Included Skills

- `eds-agent-migration`: migrates shared rules and host adapters between Agent environments, then coordinates skill reconciliation with the manager.
- `eds-skill-manager`: inventories, normalizes, installs, updates, removes, and repairs global and project skills; it also owns the global structural `SOURCE_OF_TRUTH.md` and its optional workspace registry.

Invoke either leaf skill directly. EDS does not publish a routing skill at this stage.

## Install

Install every published EDS skill through the universal Agent Skills target:

```bash
npx -y skills add Eidon123/eidon-skills -g -a universal -s '*' -y
```

After installation, use `eds-skill-manager` to verify the host discovery entries required by the local Agent environments.

## Source Of Truth

`eds-skill-manager` is the only writer of `<confirmed-global-skill-root>/SOURCE_OF_TRUTH.md`. The file stores intended structural policy, exceptions, verified evidence, and optional user-approved workspace locations. Reconstructable member inventories are generated on demand rather than persisted.

Workspace discovery is opt-in and remains separate from the manager's core skill lifecycle audit. Cross-host rule migration remains the responsibility of `eds-agent-migration`.

## Update

Re-run the EDS-scoped install command so unrelated global skills are not updated:

```bash
npx -y skills add Eidon123/eidon-skills -g -a universal -s '*' -y
```

## Version

Current release: `v0.2.1`
