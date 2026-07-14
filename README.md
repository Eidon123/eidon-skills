# Eidon Skills

Eidon Skills (EDS) is a growing collection of Agent workspace tools. Use `/eds` as the routing entry.

## Included Skills

- `eds`: routes EDS requests to the appropriate leaf skill.
- `eds-agent-migration`: audits and migrates multi-host Agent rules, skill sources, bridges, and symlinks.

## Install

```bash
npx -y skills add Eidon123/eidon-skills -g --all
```

## Update

Run the same install command again, or update all globally managed skills:

```bash
npx skills update -g
```

## Version

Current release: `v0.1.0`
