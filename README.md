# Docker AI Governance - Console + API (Simspace lab)

An interactive, fully in-browser **Simspace** lab for Docker's **AI Governance**,
taught the real way: the **Governance API** (`curl` to `hub.docker.com/v2`) for the
admin side, the **`sbx` CLI** for the developer side, and the **Admin Console**
(real screenshots) for the point-and-click view. You read and create policies,
watch the sandbox proxy allow and deny live, and inspect the enforcement log - with
**nothing real** behind it, so every learner sees the same decisions and there's
nothing to install.

Two real front doors, one source of truth:

| Front door | How you use it here |
| --- | --- |
| **Admin Console** (`app.docker.com` → AI Platform) | Real screenshots of Manage / Network / Filesystem / MCP / Audit |
| **Governance API** (`https://hub.docker.com/v2`) | Real `curl` calls: `/auth/token`, `/orgs/{org}/governance/policies`, `/rules` |
| **`sbx` CLI** (developer side) | `sbx policy ls` / `reset` / `log`, and enforcement inside `sbx run` |

Policy model: network/filesystem policies are `allowlist_v0` (rule `actions`
`connect:tcp`/`udp` or `read`/`write`, `decision` allow/deny, `resources`
hosts/CIDRs or paths); MCP policies are `cedar_v1` (Cedar `permit`/`forbid`).
Reference: [docs.docker.com/reference/api/ai-governance](https://docs.docker.com/reference/api/ai-governance/latest/).

## The arc

| # | Section | What you do |
| --- | --- | --- |
| 0 | The AI Governance console | The two front doors; `curl /auth/token`; `sbx policy ls` shows it's unmanaged |
| 1 | Manage: the master switch | Sync org policy with `sbx policy reset`; see managed vs not |
| 2 | Network access | `GET`/`POST` policies via the API; `sbx run` + a MySQL deny (403) and Datadog allow (200) |
| 3 | Filesystem access | `allowlist_v0` with `read`/`write` on paths; allow `~/workdemo`, deny creds |
| 4 | MCP access | `GET`/`POST` a `cedar_v1` policy - read-only GitHub tools only |
| 5 | Audit logs | `sbx policy log` (+ `--json`) - the enforcement decisions |
| 6 | Governance as code | The full API flow: auth → create policy → add rule → list |
| 7 | Putting it together | One rogue agent, four guardrails, one audit trail |

The lab lives under [`labs/ai-governance-hub/`](labs/ai-governance-hub/):
`labspace.yaml` (config, seeded API request-body files, sections),
`simulator.yaml` (the real `curl`/`sbx` command behaviour and enforcement), and one
Markdown file per section. It's loaded at runtime by a prebuilt image - there's no
build step for content.

> **Everything in the terminal is simulated** - no real Docker, org, network, or
> Governance API is contacted. But every command and endpoint is a real one
> (`curl` against the documented API, real `sbx policy` commands), so the lab maps
> exactly onto the live product.

## Author / preview locally

```bash
docker compose up dev                 # live preview at http://localhost:5173
docker compose run --rm validate      # validate the lab + regenerate labs.json
```

See [AGENTS.md](AGENTS.md) for the full authoring guide.
