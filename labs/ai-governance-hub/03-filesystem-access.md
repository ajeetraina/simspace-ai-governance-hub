# Filesystem access

The **Filesystem access** screen controls *which local paths coding agents can mount
to*. Same policy/rule model as network access, same default-deny posture - but the
"destination" is a **path** instead of a URL.

List the org's filesystem policies:

```bash terminal-id=host
dgov fs ls
```

```text no-run-button
POLICY           SCOPE         RULES     UPDATED
allowcatalogs    Organization  1 allow   Jul 13, 2026
allowproj        Organization  1 allow   Jul 06, 2026
allowsbxlab      Organization  1 allow   Jul 03, 2026
...
```

Open one to see what it permits:

```bash terminal-id=host
dgov fs show allowcatalog
```

```text no-run-button
  ACTION  NAME              MOUNT       MATCHES
  Allow   allow-allowcatalog  read-write  ./workdemo (and subpaths)
```

## Why this matters

An agent asked to "containerise this app" only needs the **project directory**. It
does **not** need `~/.ssh`, `~/.aws`, or `~/.docker`. With filesystem policies, the
org allows the one working directory and **default-deny blocks everything else** -
so even if the agent (or a prompt injection) tries to mount your home directory to
read your keys, the mount is refused at sandbox creation.

```text no-run-button
  allow:  ./workdemo        → mountable
  (everything else)         → default-deny → mount refused (403), sandbox never starts
```

> [!NOTE]
> This is **defense in depth** alongside *credential isolation* (keeping real API
> keys out of the sandbox entirely). Filesystem policy stops the agent from
> **mounting** secret paths; credential isolation stops a secret from **entering**
> the sandbox even if a path is allowed.

> [!TIP]
> Remember the master switch: with governance disabled, these mount rules don't
> apply either. Run `dgov status` any time to confirm the posture.

Next: governing the **tools** an agent can call, via **MCP access**.
