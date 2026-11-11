# Governance as code (the API)

The console is one front door. The **Docker AI Governance API** is the other - the
same control plane over HTTP. Both write to the **same source of truth**, so a policy
created by either shows up everywhere.

```text no-run-button
   ┌──────────────┐
   │ Admin Console│──┐
   │ (point/click)│  │
   └──────────────┘  ├─▶  one org policy store  ──sync──▶  every sandbox
   ┌──────────────┐  │    (network · fs · mcp)
   │ Governance   │──┘
   │ API (curl)   │
   └──────────────┘
```

Use the API when you want **governance as code**: policies in version control,
created in CI, managed by admin tooling instead of by hand.

## Authenticate, then write a policy

Log in with org credentials (simulated):

```bash terminal-id=host
docker login
```

The lab seeds a policy document - take a look:

```bash terminal-id=host
cat blockpaste.json
```

POST it to the Governance API. This is the exact same `blockpaste` policy you could
create from the Network access screen:

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/orgs/whalecollab/governance/policies -d @blockpaste.json
```

```text no-run-button
> POST /v2/orgs/whalecollab/governance/policies
< HTTP/1.1 201 Created
{ "id": "pol_7hk2...", "name": "blockpaste", "type": "network", "scope": "organization", "rules": 1 }
```

Because it's the same store, the policy is immediately visible through `dgov`:

```bash terminal-id=host
dgov net ls
```

> [!NOTE]
> This mirrors a real pattern: a `setup-policies.sh` script (or a CI job) that
> authenticates once, then `POST`s each policy and its rules. The console is for
> one-off human changes; the API is for repeatable, reviewable, automated ones.

> [!IMPORTANT]
> The API is still **governed by the master switch**. Creating a policy by API
> doesn't enforce it any sooner - `dgov status` must say **ENABLED** for sandboxes
> to apply it, exactly as in the console.

Finally, let's tie the whole model together.
