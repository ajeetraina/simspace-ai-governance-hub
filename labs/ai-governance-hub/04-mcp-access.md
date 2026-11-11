# MCP access

Agents also call **tools**, usually over **MCP** (the Model Context Protocol). The
**MCP access** console screen controls *which MCP servers are reachable through
Docker's MCP gateway*, and which tools an agent may call.

Unlike network and filesystem policies (`allowlist_v0`), MCP policies use the
`cedar_v1` type: rules are written in **Cedar**, a purpose-built authorization
language, carried in a single `text` field against a published `schema_url`.

## Read the MCP policy with the API

```bash terminal-id=host
curl https://hub.docker.com/v2/orgs/whalecollab/governance/policies/pol_07hk2mcpdemo0github0readonly -H "Authorization: Bearer $TOKEN"
```

The `cedar_v1` body (unescaped for reading):

```text no-run-button
schema_url: https://schemas.docker.com/policy/cedar/v1/mcp/v0

permit(principal, action == MCP::Action::"call_tool", resource is MCP::Tool)
  when {
    resource.server == "github" &&
    resource.tool in ["search_repositories", "get_file_contents", "list_issues"]
  };
```

## Reading the policy

- **`permit(...)`** - an *allow* statement. Cedar is default-deny: a tool call no
  `permit` covers is refused. (`forbid(...)` statements deny, and win.)
- **`action == MCP::Action::"call_tool"`** - it governs tool invocations.
- **`when { ... }`** - the guard: only the `github` server, and only the three
  **read-only** tools listed.

So a governed agent can *search* repos and *read* files, but `create_pull_request`,
`merge_pull_request`, or `delete_repo` are **denied** - no `permit` covers them.

## Create an MCP policy with the API

Same endpoint as any policy; the body is a `cedar_v1` object (seeded as
`mcp-policy.json`):

```bash terminal-id=host
cat mcp-policy.json
```

```bash terminal-id=host
curl -X POST https://hub.docker.com/v2/orgs/whalecollab/governance/policies -H "Authorization: Bearer $TOKEN" -d @mcp-policy.json
```

> [!IMPORTANT]
> One governed gateway + Cedar means you grant **exactly** the tools a workflow
> needs and nothing more - least privilege, applied to AI tools.

Every allow and deny you've seen is recorded. Next: the **Audit logs**.
