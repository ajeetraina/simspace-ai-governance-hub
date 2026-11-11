# Filesystem access

The **Filesystem access** console screen controls *which local paths coding agents
can mount to*. In the API it's the same `allowlist_v0` shape as a network policy -
but the rule `actions` are `read` / `write` and `resources` are **paths** (`/data`,
`/data/**`) instead of hosts.

> [!NOTE]
> The API derives a policy's `domain` from its rule actions: `connect:tcp`/`udp`
> → `network`; `read`/`write` → `filesystem`. All actions in one rule must be the
> same layer - you can't mix network and filesystem actions in a single rule.

## What a filesystem policy looks like

A least-privilege policy allows the project directory and denies credential paths:

```json no-run-button
"allowlist_v0": {
  "domain": "filesystem",
  "rules": [
    { "name": "allow workdemo",     "actions": ["read","write"],
      "decision": "allow", "resources": ["~/workdemo/**"] },
    { "name": "deny credentials",   "actions": ["read","write"],
      "decision": "deny",  "resources": ["~/.ssh/**","~/.aws/**","~/.docker/config.json"] }
  ]
}
```

The synced result is visible to the developer via `sbx` (the filesystem rows):

```bash terminal-id=host
sbx policy ls --include-inactive
```

## Why it matters

An agent asked to "containerise this app" needs only the **project directory** - not
`~/.ssh`, `~/.aws`, or `~/.docker`. With `read`/`write` denied on those paths (and
default-deny everywhere outside the allowed `~/workdemo`), the mount is refused at
sandbox creation - even if a prompt injection tries to read your keys.

```text no-run-button
  allow:  ~/workdemo/**     → mountable (read+write)
  deny:   ~/.ssh ~/.aws     → refused
  (everything else)         → default-deny → refused
```

> [!TIP]
> This is **defense in depth** with *credential isolation* (keeping real keys out of
> the sandbox entirely): filesystem policy stops the agent **mounting** secret
> paths; credential isolation stops a secret **entering** the sandbox at all.

Next: governing the **tools** an agent can call - **MCP access**.
