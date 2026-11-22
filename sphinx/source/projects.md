# Projects

A **project** is a named work area inside a space. It is a label for
organising memories — not a second ACL boundary. Anyone who can read the
space can see its projects and the memories attached to them.

```text
space (personal or group)
  ├── project A
  │     └── memories tagged with that project
  ├── project B
  └── memories with no project
```

Personal spaces and groups both support projects. Membership and roles still
come from the space; see {doc}`groups`.

## Creating and managing

- Create a project from the web UI **Projects** page, or with the MCP tools
  `project_create` / `project_list` / `project_show` / `project_rename` /
  `project_delete`.
- Rename and delete are allowed for the project’s creator, a **group admin**
  of that space, or an instance administrator.
- Deleting a project does **not** delete its memories. Those rows stay; their
  `project_id` is cleared.

## Attaching memories

- When saving an entry, pass optional `project_id` (MCP `remember`, or the
  REST create/update body). The project must belong to the same space as the
  memory.
- Digests and full opens return `project_id` and `project_name` when a memory
  is attached.
- The **Memories** and **Metrics** pages share the same filters, including
  project.

Projects do not change who can read or write a memory. They only help you
slice a large space by effort, repo, or initiative.

## Related

- Spaces and group roles: {doc}`groups`
- Entries, guides, and search: {doc}`memory`
- Agent tools: {doc}`mcp-tools`
