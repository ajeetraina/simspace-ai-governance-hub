# Audit logs

Policies set the guardrails; the **Audit logs** screen is how you *review policy
enforcement events across your organization* - every allow and deny, with **who**,
**what**, **when**, and the **decision**.

> *Audit logs are stored in Docker Cloud by default, capturing member actions,
> timestamps, and IPs.*

The console groups this under tabs - **Audit events**, **CSV exports**, **Audit
delivery**, **History**, and **Export & connectors** - and lets you filter by
**Date range**, **Activity**, **User**, **Harness**, and **Decision**.

## Nothing to see when governance is off

If AI governance is disabled, no policies are enforced, so nothing is recorded.
Confirm by turning it off and reading the log:

```bash terminal-id=host
dgov disable
```

```bash terminal-id=host
dgov audit ls
```

You get **0 matching records** - exactly what the console shows with the master
switch off.

## Turn it on, generate events, read the trail

Re-enable governance and replay the two network requests from earlier so there's
something to see:

```bash terminal-id=host
dgov enable
```

```bash terminal-id=host
curl https://db.mysql.org:3306
```

```bash terminal-id=host
curl https://api.datadoghq.com
```

Now read the audit log:

```bash terminal-id=host
dgov audit ls
```

```text no-run-button
TIME (UTC)   USER          HARNESS  RESOURCE            ACTIVITY     DECISION
just now     michelangelo  claude   db.mysql.org:3306   net:connect  DENY
just now     michelangelo  claude   api.datadoghq.com   net:connect  ALLOW
2 min ago    leonardo      codex    ./workdemo          fs:mount:rw  ALLOW
5 min ago    leonardo      codex    ~/.aws              fs:mount:rw  DENY
```

Each row is a policy decision. The **DENY** rows are your deny rules and
default-deny doing their job - a blocked MySQL connection, a refused `~/.aws` mount.

> [!TIP]
> In the real console you'd filter **Decision = Deny** to triage what agents are
> *trying* to do that policy stops - a fast way to spot a misconfigured workflow or
> a genuinely rogue agent. Those events can be exported (CSV) or streamed to a SIEM
> via **Export & connectors**.

The audit log is also the bridge to alerting: a `DENY` spike can page on-call. Next,
the programmatic side - **governance as code**.
