# Eidon Skills

Eidon Skills (EDS) provides cross-host Agent workspace migration and skill lifecycle management.

## Included Skills

- `eds-agent-migration`: migrates shared rules and host adapters between Agent environments, then coordinates skill reconciliation with the manager.
- `eds-skill-manager`: inventories, normalizes, installs, updates, removes, and repairs global and project skills.

Invoke either leaf skill directly. EDS does not publish a routing skill at this stage.

## Install

Install every published EDS skill through the universal Agent Skills target:

```bash
npx -y skills add Eidon123/eidon-skills -g -a universal -s '*' -y
```

After installation, use `eds-skill-manager` to verify the host discovery entries required by the local Agent environments.

## Update

```bash
npx skills update -g -y
```

## Version

Current release: `v0.2.0`
