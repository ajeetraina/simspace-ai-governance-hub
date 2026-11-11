# Docker AI Governance - the Admin Console, in a terminal (Simspace lab)

An interactive, fully in-browser **Simspace** lab that recreates Docker Hub's
**AI Governance** console as a simulated terminal. You drive all five console
screens with one small CLI (`dgov`), watch policies allow and deny requests live,
and read the audit trail - with **nothing real** behind it, so every learner sees
the same decisions and there's nothing to install.

The real feature lives at **app.docker.com → AI Platform**:

| Console screen | What it governs |
| --- | --- |
| **Manage** | The master switch - AI governance on / off |
| **Network access** | Which URLs agents can reach through the sandbox |
| **Filesystem access** | Which local paths agents can mount |
| **MCP access** | Which MCP servers/tools agents can call (Cedar policy) |
| **Audit logs** | Every policy-enforcement event (who, what, when, decision) |

## The arc

| # | Section | What you do |
| --- | --- | --- |
| 0 | The AI Governance console | Tour the five screens; meet `dgov`; see governance is off |
| 1 | Manage: the master switch | `dgov enable` - make every policy real |
| 2 | Network access | Read/create policies; watch a MySQL deny and a Datadog allow |
| 3 | Filesystem access | Allow `./workdemo`, default-deny `~/.aws` and friends |
| 4 | MCP access | Read a Cedar policy that permits read-only GitHub tools only |
| 5 | Audit logs | Generate events, then review allow/deny with user + timestamp |
| 6 | Governance as code | Create the same policy via the Governance API (`curl`) |
| 7 | Putting it together | One rogue agent, four guardrails, one audit trail |

The lab lives under [`labs/ai-governance-hub/`](labs/ai-governance-hub/):
`labspace.yaml` (config, seeded files, sections), `simulator.yaml` (the `dgov`
command behaviour and enforcement), and one Markdown file per section. It's loaded
at runtime by a prebuilt image - there's no build step for content.

> **Everything in the terminal is simulated** - no real Docker, org, network, or
> Governance API is contacted. The command names, policy names, and allow/deny
> decisions mirror the live console so the lab is instantly recognizable.

## Author / preview locally

```bash
docker compose up dev                 # live preview at http://localhost:5173
docker compose run --rm validate      # validate the lab + regenerate labs.json
```

See [AGENTS.md](AGENTS.md) for the full authoring guide.
