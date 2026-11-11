# The AI Governance console

Docker's **AI Governance** lets an organization set guardrails for how AI coding
agents behave: which URLs they can reach, which paths they can mount, which MCP
tools they can call - plus an audit log of every decision. There are **two real
front doors**, and both write to the same source of truth:

```text no-run-button
  ┌─────────────────────────────┐
  │ Admin Console (web UI)       │   app.docker.com → AI Platform
  │   point-and-click            │   Manage · Network · Filesystem · MCP · Audit
  └──────────────┬──────────────┘
                 │   same org policy store
  ┌──────────────┴──────────────┐
  │ Governance API (HTTP)        │   https://hub.docker.com/v2
  │   curl / CI / automation     │   /orgs/{org}/governance/policies ...
  └─────────────────────────────┘
                 │  synced at login / `sbx policy reset`
                 ▼
         every developer sandbox  →  deny · allow · default-deny
```

> [!NOTE]
> **This lab is fully simulated.** No real org, network, or Governance API is
> contacted - every command is scripted so everyone sees the same decisions, with
> nothing to install. The console screens are shown as the **real screenshots**
> (it's a web UI). The terminal uses the **real** tools: `curl` against the
> Governance API, and the `sbx` CLI developers already use.

## The two interfaces you'll use

**Admin side - the Governance API** (`https://hub.docker.com/v2`). Authenticate a
username + Personal Access Token for a bearer token:

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/auth/token -H "Content-Type: application/json" -d @creds.json
```

Take the `access_token` from the response and send it as `Authorization: Bearer
$TOKEN` on every later call.

**Developer side - `sbx`.** A developer's sandbox fetches the org policy at login
and the daemon caches it. Right now nothing is synced, so there's no org policy:

```bash terminal-id=host
sbx policy ls
```

It reports **not managed** - only local sbx defaults. That's our starting point:
governance exists in the console, but this machine isn't enforcing it yet.

> [!TIP]
> Reference: the full API is documented at
> [docs.docker.com/reference/api/ai-governance](https://docs.docker.com/reference/api/ai-governance/latest/).
> Everything below uses only those real endpoints and real `sbx` commands.

Next: the **Manage** screen - the switch that turns enforcement on.
