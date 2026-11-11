# Network access

The **Network access** screen controls *which URLs coding agents can reach through
the sandbox*. It's a list of **policies**, each with one or more **rules**, scoped to
the whole Organization or to specific Teams.

List the org's network policies:

```bash terminal-id=host
dgov net ls
```

You'll see the seeded policies - one **deny** and several **allow**:

```text no-run-button
POLICY                 SCOPE         RULES     UPDATED
denyredingtondbteam    Organization  1 deny    Sep 17, 2026
allowdatadog           Organization  1 allow   Sep 09, 2026
allowfirecrawl         Organization  1 allow   Aug 18, 2026
...
```

## A policy's rules

Open the deny policy to see its rule detail - action, protocols, and what it
matches:

```bash terminal-id=host
dgov net show denyredingtondbteam
```

```text no-run-button
  ACTION  NAME            PROTO     MATCHES
  Deny    denyredington   TCP, UDP  All traffic to *.mysql.org
```

In the console, creating a rule lets you pick: **Decision** (Allow / Deny),
**Type** (HTTP, or *All traffic* for any port/method/path), **Protocols**
(TCP / UDP / Both), and **Destinations** (a host or CIDR, e.g. `github.com` or
`10.0.0.0/8`).

> [!IMPORTANT]
> **Deny always overrides allow**, and anything not matched by an allow rule is
> blocked by **default-deny**. So you don't enumerate every bad destination - you
> allow the few you trust, and everything else is denied automatically.

## See it enforced

With governance **enabled**, send two requests the way a sandboxed agent would. The
first hits the MySQL host the deny rule covers; the second hits Datadog, which has an
allow rule:

```bash terminal-id=host
curl https://db.mysql.org:3306
```

```bash terminal-id=host
curl https://api.datadoghq.com
```

The first is **blocked (403)** by `denyredingtondbteam`; the second is **allowed** by
`allowdatadog`. Now turn the master switch off and try the blocked one again:

```bash terminal-id=host
dgov disable
```

```bash terminal-id=host
curl https://db.mysql.org:3306
```

Ungoverned, it returns **200** - the policy still exists, but nothing enforces it.
Re-enable governance before moving on:

```bash terminal-id=host
dgov enable
```

## Create a policy

Block a data-exfiltration paste site org-wide. Deny rules take effect immediately:

```bash terminal-id=host
dgov net create --name blockpaste --deny paste.ee
```

List again - `blockpaste` now sits at the top of the Network access screen:

```bash terminal-id=host
dgov net ls
```

Next: the same model for the **filesystem**.
