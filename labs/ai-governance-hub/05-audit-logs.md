# Audit logs

Policies set the guardrails; **Audit logs** is how you *review policy enforcement
events across your organization* - every allow and deny, with who, what, when, and
the decision.

> *Audit logs are stored in Docker Cloud by default, capturing member actions,
> timestamps, and IPs.*

The console groups this under tabs - **Audit events**, **CSV exports**, **Audit
delivery**, **History**, **Export & connectors** - with filters for **Date range**,
**Activity**, **User**, **Harness**, and **Decision**. That org-wide view is a
console (and export/SIEM) feature.

## The developer-side view: `sbx policy log`

On a developer's machine, `sbx` exposes the local enforcement decisions the sandbox
proxy made. After the network test earlier, look at what was blocked:

```bash terminal-id=host
sbx policy log
```

```text no-run-button
SANDBOX   TYPE     HOST          RULE                 REASON    COUNT
catalog   network  db.mysql.org  denyredingtondbteam  explicit  1
catalog   network  example.com   default-deny         implicit  1
```

Each row is a policy decision: an **explicit** deny (the MySQL rule) and an
**implicit** one (default-deny). Machine-readable output for piping into tooling:

```bash terminal-id=host
sbx policy log --json
```

```json no-run-button
{"sandbox":"catalog","host":"api.datadoghq.com","decision":"allow","rule":"allowdatadog","count":12}
{"sandbox":"catalog","host":"db.mysql.org","decision":"deny","rule":"denyredingtondbteam","count":1}
{"sandbox":"catalog","host":"example.com","decision":"deny","rule":"default-deny","count":1}
```

> [!TIP]
> In the console you'd filter **Decision = Deny** to triage what agents are *trying*
> to do that policy stops - a fast way to spot a misconfigured workflow or a rogue
> agent - and stream those events to a SIEM via **Export & connectors**. `sbx policy
> log --json` is the same signal, locally, for a single machine.

## Turn recording off

Audit logging is on by default (the Manage screen stores events in Docker Cloud).
Open **Settings** (⚙), turn **"Audit logging"** off, then look again:

```bash terminal-id=host
sbx policy log
```

Decisions are still **enforced** - but nothing is **recorded**, so you lose the
trail. In the real console, audit storage and delivery are configured on the Manage
screen and the Audit logs tabs. Toggle it back on to resume recording.

A `deny` spike is the bridge to alerting. Next, the full programmatic flow -
**governance as code**.
