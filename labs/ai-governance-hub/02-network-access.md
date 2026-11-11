# Network access

The **Network access** console screen controls *which URLs coding agents can reach
through the sandbox* - a list of **policies**, each a set of **rules**. In the API,
a network policy is an `allowlist_v0` object whose rules carry `actions`
(`connect:tcp` / `connect:udp`), a `decision` (`allow` / `deny`), and `resources`
(hosts, `*.wildcards`, or CIDRs).

## Read a policy with the API

The org `whalecollab` already has a deny policy, `denyredingtondbteam`. Fetch it by
id (send your bearer token from the intro):

```bash terminal-id=host
curl https://hub.docker.com/v2/orgs/whalecollab/governance/policies/pol_06gawnt8tw1pzxe3e69y58jsr5df -H "Authorization: Bearer $TOKEN"
```

```json no-run-button
"allowlist_v0": {
  "domain": "network",
  "rules": [ { "name": "denyredington",
              "actions": ["connect:tcp","connect:udp"],
              "decision": "deny", "resources": ["*.mysql.org"] } ]
}
```

> [!IMPORTANT]
> **Deny always wins over allow**, and anything no `allow` rule matches is blocked by
> **default-deny**. So you allow the few destinations you trust and everything else
> is refused automatically.

The same rules, synced to the developer's machine, are visible via `sbx`:

```bash terminal-id=host
sbx policy ls
```

## See it enforced

Start a sandboxed agent and make the requests an agent would. The proxy evaluates
each against the org policy:

```bash terminal-id=host
sbx run claude
```

```bash terminal-id=host
curl https://db.mysql.org:3306
```

```bash terminal-id=host
curl https://api.datadoghq.com
```

The MySQL host is **403** (matched `denyredingtondbteam`); Datadog is **200**
(allowed by `allowdatadog`).

> [!TIP]
> Open **Settings** (⚙) and turn **"Network policy: denyredingtondbteam"** off, then
> re-run the MySQL `curl` - it returns **200** now that the deny rule is gone. Toggle
> it back on to restore the block. (Turning the master **AI governance** switch off
> has the same effect for *every* rule at once.)

Leave the sandbox:

```bash terminal-id=host
exit
```

## Create a policy with the API

Block data-exfiltration paste sites org-wide. The request body is seeded as
`network-policy.json` (a real `allowlist_v0` deny rule) - take a look, then POST it:

```bash terminal-id=host
cat network-policy.json
```

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/orgs/whalecollab/governance/policies -H "Authorization: Bearer $TOKEN" -d @network-policy.json
```

The API returns **201** with the new policy and its server-assigned `id`. To add
another rule to an existing policy, POST to its `/rules` sub-resource:

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/orgs/whalecollab/governance/policies/pol_06gawnt8tw1pzxe3e69y58jsr5df/rules -H "Authorization: Bearer $TOKEN" -d @allow-rule.json
```

Next: the same model for the **filesystem**.
