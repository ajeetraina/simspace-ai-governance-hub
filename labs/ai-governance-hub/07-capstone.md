# Putting it together

You've driven every screen of Docker Hub's **AI Governance** console from the
terminal. The whole model is small:

```text no-run-button
  Manage ............ one master switch. Off = nothing is enforced.
  Network access .... which URLs agents reach    (allow/deny, default-deny)
  Filesystem access . which paths agents mount   (allow/deny, default-deny)
  MCP access ........ which tools agents call     (Cedar permit, default-deny)
  Audit logs ........ every decision: who, what, when, allow/deny
```

Three ideas carry across all of it:

1. **Define once, enforce everywhere.** The org sets policy centrally; every
   developer's sandbox syncs and applies it. Developers can't opt out.
2. **Default-deny.** You allow the few things a workflow needs; everything else is
   blocked without being listed. Deny always overrides allow.
3. **The master switch is the posture.** Policies are inert until **Manage → AI
   governance** is on - so it's the first thing to check.

## One rogue agent, four guardrails

Imagine a single confused (or compromised) agent trying four things at once. With
governance **enabled**, confirm the posture and watch policy answer each one:

```bash terminal-id=host
dgov status
```

```bash terminal-id=host
curl https://db.mysql.org:3306
```

- **Network:** the exfil-to-MySQL attempt → **DENY** (`denyredingtondbteam`).
- **Filesystem:** mounting `~/.aws` to read keys → **DENY** (default-deny; only
  `./workdemo` is allowed).
- **MCP:** calling `delete_repo` on GitHub → **DENY** (`demo-github-readonly`
  permits read-only tools only).
- **Audit:** all four attempts recorded with user, time, and decision.

Read the trail one more time:

```bash terminal-id=host
dgov audit ls
```

## What you can do next

- In the **real console** at
  [app.docker.com/accounts/$$org$$](https://app.docker.com/accounts/$$org$$), the
  same screens are point-and-click, and AI Governance is a licensed product you
  enable per org.
- Stream **Audit logs** to a SIEM (Datadog, Splunk) via **Export & connectors** and
  alert on `DENY` spikes.
- Manage policies as code with the **Governance API** in CI.

> [!NOTE]
> Everything in this lab was **simulated** - no real org, network, or API was ever
> contacted. The commands, policy names, and decisions mirror the live console so
> you can recognize it instantly when you open the real thing.

That's AI Governance: set the guardrails once, and every agent in the org plays
inside them.
