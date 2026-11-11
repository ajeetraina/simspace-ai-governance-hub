# Governance as code (the API)

You've used the Governance API piecemeal; here's the **whole flow** in order - the
pattern you'd put in a script or CI job so policies live in version control, not just
in someone's browser. Base URL: `https://hub.docker.com/v2`. Reference:
[docs.docker.com/reference/api/ai-governance](https://docs.docker.com/reference/api/ai-governance/latest/).

The pipeline - author once, enforce everywhere:

```text no-run-button
   policy.json in Git
        │   CI job  (or you, with curl)
        ▼
   POST /v2/.../governance/policies ──┐
                                      ├──▶  ORG POLICY STORE  ◀── Admin Console writes here too
   (auth → create policy → add rule) ─┘        (one source of truth)
                                                     │
                                       sbx login / sbx policy reset
                                                     ▼
                                   every developer sandbox  →  deny · allow · default-deny
```

Both front doors (API and Console) write to the **same store**; `sbx` pulls it to
every machine. Now walk the four API steps.

## 1. Authenticate → bearer token

`creds.json` holds a Docker username and a Personal Access Token (`dckr_pat_…`) or
Organization Access Token (`dckr_oat_…`):

```bash terminal-id=host
cat creds.json
```

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/auth/token -H "Content-Type: application/json" -d @creds.json
```

Capture the `access_token` as `$TOKEN` and send it as `Authorization: Bearer $TOKEN`
on every call.

## 2. Create a policy

A policy is `allowlist_v0` (network/filesystem) or `cedar_v1` (MCP). POST the body to
the `/policies` collection; the API returns **201** with the server-assigned `id`:

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/orgs/whalecollab/governance/policies -H "Authorization: Bearer $TOKEN" -d @network-policy.json
```

## 3. Add a rule to a policy

Rules are a sub-resource. POST to `…/policies/{id}/rules`:

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/orgs/whalecollab/governance/policies/pol_06gawnt8tw1pzxe3e69y58jsr5df/rules -H "Authorization: Bearer $TOKEN" -d @allow-rule.json
```

## 4. List what the org has

```bash terminal-id=host
curl https://hub.docker.com/v2/orgs/whalecollab/governance/policies -H "Authorization: Bearer $TOKEN"
```

> [!NOTE]
> The console and the API write to the **same source of truth**. A policy created
> here shows up in the Admin Console immediately, and vice-versa. The full verb set
> is `GET`/`POST` on the collection, and `GET`/`PATCH`/`DELETE` on a policy
> (`…/policies/{id}`) and on a rule (`…/policies/{id}/rules/{rule_id}`).

## 5. Developers pick it up at login

Nothing you POST is enforced until a developer's machine syncs it - which happens at
`sbx login`, or on demand:

```bash terminal-id=host
sbx policy reset
```

That's governance-as-code: author in Git → `POST` from CI → `sbx` syncs → every
sandbox enforces. Finally, let's tie the model together.
