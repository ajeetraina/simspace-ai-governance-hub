# Putting it together

You've driven Docker **AI Governance** the real way - the **Governance API** for the
admin side, **`sbx`** for the developer side, and the **Admin Console** (screenshots)
for the point-and-click view. The model is small:

```text no-run-button
  Manage ............ the switch. Off = nothing is enforced.
  Network .......... allowlist_v0, actions connect:tcp/udp, resources = hosts/CIDR
  Filesystem ....... allowlist_v0, actions read/write,      resources = paths
  MCP .............. cedar_v1, Cedar permit/forbid over MCP::Action
  Audit ............ every decision (console) / sbx policy log (developer)
```

Three ideas carry across all of it:

1. **Two front doors, one source of truth.** Console (UI) and API (`curl`) write to
   the same org policy store; `sbx` syncs it to every sandbox at login.
2. **Default-deny, deny-wins.** Allow the few things a workflow needs; everything
   else is blocked without being listed, and an explicit `deny` always overrides.
3. **The switch is the posture.** Policies are inert until governance is on - so
   `sbx policy ls` ("managed by your org") is the first thing to check.

## One rogue agent, four guardrails

With org policy synced, confirm the posture and watch policy answer each attack:

```bash terminal-id=host
sbx policy ls
```

```bash terminal-id=host
sbx run claude
```

```bash terminal-id=host
curl https://db.mysql.org:3306
```

- **Network:** exfil to MySQL → **403** (`denyredingtondbteam`, `*.mysql.org`).
- **Filesystem:** mounting `~/.aws` → refused (default-deny; only `~/workdemo`
  allowed).
- **MCP:** `delete_repo` on GitHub → denied (`demo-github-readonly` permits read-only
  tools only).
- **Audit:** every attempt recorded - `sbx policy log` locally, Audit logs in the
  console.

```bash terminal-id=host
exit
```

```bash terminal-id=host
sbx policy log
```

## Do it for real

- **Console:** [app.docker.com/accounts/whalecollab](https://app.docker.com/accounts/whalecollab)
  → AI Platform (AI Governance is a licensed product, enabled per org).
- **API:** [docs.docker.com/reference/api/ai-governance](https://docs.docker.com/reference/api/ai-governance/latest/)
  - author policies in Git, `POST` them from CI.
- **Developers:** `sbx login`, then `sbx policy ls` / `reset` / `log`.

> [!NOTE]
> Everything here was **simulated** - no real org, network, or API was contacted -
> but every command and endpoint is the real one, so you'll recognize it instantly
> when you open the console or hit the API for real.
