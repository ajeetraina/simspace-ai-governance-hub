# MCP access

Agents don't only reach the network and filesystem - they call **tools**, usually
over **MCP** (the Model Context Protocol). The **MCP access** screen controls *which
MCP servers are accessible through Docker's MCP gateway*, and which **tools** on them
an agent may call.

Unlike the network and filesystem screens, MCP policies are written in **Cedar**, a
purpose-built authorization language, so you can express fine-grained rules like
"read-only GitHub tools, nothing that writes".

List the org's MCP policies:

```bash terminal-id=host
dgov mcp ls
```

```text no-run-button
POLICY                 SCOPE         CEDAR POLICY                                  UPDATED
allowalltools          Organization  permit(principal, action == MCP::Action::...  Aug 03, 2026
allowikimcp            Organization  permit(principal, action == MCP::Action::...  Aug 03, 2026
demo-github-readonly   Organization  permit(principal, action == MCP::Action::...  Jul 06, 2026
```

Open the read-only GitHub policy to see the full Cedar document:

```bash terminal-id=host
dgov mcp show demo-github-readonly
```

```text no-run-button
  permit(
    principal,
    action == MCP::Action::"call_tool",
    resource
  )
  when {
    resource.server == "github" &&
    resource.tool in ["search_repositories", "get_file_contents", "list_issues"]
  };
```

## Reading the policy

- **`permit(...)`** - this is an *allow* rule. Cedar is default-deny too: a tool
  call that no `permit` covers is refused.
- **`action == MCP::Action::"call_tool"`** - it governs tool invocations.
- **`when { ... }`** - the guard. Only the `github` server, and only the three
  **read-only** tools listed, are allowed.

So an agent governed by `demo-github-readonly` can *search* repos and *read* files,
but a call to `create_pull_request`, `merge_pull_request`, or `delete_repo` is
**denied** - there's no `permit` for it.

> [!IMPORTANT]
> One governed gateway + Cedar means you grant **exactly** the tools a workflow
> needs and nothing more - the principle of least privilege, applied to AI tools.

Every allow and deny you've seen is recorded. Next: the **Audit logs**.
