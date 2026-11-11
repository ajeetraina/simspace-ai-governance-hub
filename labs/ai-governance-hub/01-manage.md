# Manage: the master switch

The **Manage** screen (*Manage AI governance → "Monitor and control AI usage across
your organization"*) has a single, powerful control: the **AI governance** toggle.

> *Set guardrails for how AI features are used across your organization. Track usage,
> apply policies, and review activity. Audit logs are stored in Docker Cloud by
> default, capturing member actions, timestamps, and IPs.*

Everything else in the console - network, filesystem, and MCP policies - **only
takes effect when this switch is on**. Every policy screen repeats the warning:

> *Policies do not take effect when AI governance is disabled.*

## See the two states

Right now it's off. Confirm:

```bash terminal-id=host
dgov status
```

Now flip the master switch on - the CLI equivalent of toggling **AI governance** in
the console:

```bash terminal-id=host
dgov enable
```

Check the posture again - it now reports **ENABLED**, and audit logging has started:

```bash terminal-id=host
dgov status
```

> [!NOTE]
> In the real console this is a toggle an **org admin** flips. Developers can't
> change it, and they can't opt their sandboxes out - that's the entire point of
> *governance*. In this lab you can also flip it from **Settings** (the ⚙ gear,
> top-right, next to Reset): the **"AI governance (master switch)"** toggle is wired
> to the same state as `dgov enable` / `dgov disable`.

## Why the switch matters

```text no-run-button
  governance OFF                         governance ON
  ─────────────                          ────────────
  policies exist in the console          policies are synced to every sandbox
  but are never evaluated        ──▶     and evaluated on every request
  sandboxes run UNGOVERNED               deny → allow → default-deny
```

You'll feel this directly in the next sections: with governance **off**, a request
that *should* be blocked sails through; with it **on**, the same request is denied.

Leave governance **enabled** and continue to **Network access**.
