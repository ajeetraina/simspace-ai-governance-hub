# Manage: the master switch

The **Manage** screen (*Manage AI governance → "Monitor and control AI usage across
your organization"*) has a single, decisive control: the **AI governance** toggle.

> *Set guardrails for how AI features are used across your organization. Track usage,
> apply policies, and review activity. Audit logs are stored in Docker Cloud by
> default, capturing member actions, timestamps, and IPs.*

Every other screen - Network, Filesystem, MCP - repeats the same warning:

> *Policies do not take effect when AI governance is disabled.*

This is a **console action** (an org admin flips the toggle in the web UI); there
is no separate API call to flip it. What it changes for developers is whether their
sandboxes **sync and enforce** the org policy.

## See both states from the developer side

With governance off (or before you've logged in), `sbx` only has local defaults:

```bash terminal-id=host
sbx policy ls
```

Now sync the org policy to this machine - the developer-side effect of the switch
being on. `sbx login` fetches it at authentication; `sbx policy reset` forces a
re-sync at any time:

```bash terminal-id=host
sbx policy reset
```

List again - now it's **managed by your org**, with remote rules active:

```bash terminal-id=host
sbx policy ls
```

> [!NOTE]
> In this lab you can also flip the state from **Settings** (the ⚙ gear, top-right,
> next to Reset): the **"AI governance (Manage switch)"** toggle is the in-lab
> analog of the console switch, wired to the same state that `sbx policy reset`
> sets. Turn it off and `sbx policy ls` goes back to *not managed*.

## Why the switch matters

```text no-run-button
  governance OFF                         governance ON
  ─────────────                          ────────────
  policies exist in the console          policy synced to every sandbox
  but are never evaluated        ──▶     and evaluated on every request
  sandboxes run UNGOVERNED               deny → allow → default-deny
```

You'll feel this directly next: the same request is **200** ungoverned and **403**
once the org's deny rule is in effect.

With org policy synced, continue to **Network access**.
