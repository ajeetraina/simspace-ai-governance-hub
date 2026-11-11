# The AI Governance console

Docker Hub gives an organization one place to **set guardrails for how AI coding
agents behave**: which URLs they can reach, which local paths they can mount, which
MCP tools they can call - and an audit log of every decision. In the real product
it lives at
[app.docker.com/accounts/$$org$$](https://app.docker.com/accounts/$$org$$) under
**AI Platform**:

```text no-run-button
AI Platform
  ├─ Manage            ← the master switch: AI governance on / off
  ├─ Network access    ← which URLs agents can reach through the sandbox
  ├─ Filesystem access ← which local paths agents can mount
  ├─ MCP access        ← which MCP servers/tools are reachable (Cedar policy)
  └─ Audit logs        ← every policy-enforcement event (who, what, when, decision)
```

> [!NOTE]
> **This lab is fully simulated.** There is no real Docker org, no network, and no
> Governance API behind the terminal - every command is scripted so that everyone
> sees the same allow/deny decisions, with nothing to install. The console is
> point-and-click; this lab gives you an equivalent CLI, **`dgov`**, so you can
> explore the same five screens from a terminal and *see* enforcement happen.

## Why it exists

An AI agent running in a sandbox can make network calls, mount directories, and
call tools. Left ungoverned, a single rogue or confused agent can exfiltrate data,
read your credentials, or call a destructive tool. **AI Governance** lets the org
define the rules **once**, centrally, so every developer's sandbox enforces them -
and developers can't turn them off.

## Meet `dgov`

`dgov` is this lab's thin CLI over the **same control plane the console writes to**.
Start by asking it for help, then check the current posture:

```bash terminal-id=host
dgov help
```

```bash terminal-id=host
dgov status
```

You'll see governance is **DISABLED** right now - exactly like the console
screenshot, where the master toggle is off. That's our starting point.

> [!TIP]
> The two "front doors" to governance - the **console** (point-and-click) and the
> **API** (`curl`, covered near the end) - write to the **same source of truth**.
> `dgov` is simply a readable stand-in for both so you can learn the model quickly.

Next: the **Manage** screen - the master switch that makes every other policy real.
