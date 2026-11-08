# Groups

A **group** is a shared memory space. Every member can read and write memories
in that space. Groups are optional — personal memory works without them.

```text
you
  ├── personal     private; only you; not a group
  └── groups[]     shared spaces you belong to
```

This is not the same thing as **User Groups** in System Settings
(Administrators / Users). Those control who may sign in as admin. Memory
groups are where shared entries and guides live.

## Creating and belonging

- Any authenticated user can create a group (`space_create` / the GUI).
- The creator owns the group: manage roster (add / remove members, rename) and
  delete the group (hard delete; memories in that space go with it).
- Other members can use the group's memories but cannot manage membership or
  delete the group. They may leave.
- There is no role ladder (no owner / member / viewer flags beyond creator vs
  everyone else).
- You cannot remove the group creator from the roster.

Admin (`ADMIN_API_KEY` / admin login) is break-glass: can see and manage across
groups, including delete. Prefer a per-user API key or JWT for normal agent
sessions.

## Writing into a group

`remember` / `guide_write` pick the target in this order:

1. `space_id` — that space (you must be a member).
2. `group` — exact, case-insensitive name among your unarchived groups.
3. `visibility="shared"` — your default shared group, else your only group;
   otherwise an error.
4. Otherwise → personal.

A memory lives in exactly one space. Visibility is that space's roster.

## Tools

| Tool | Job |
| --- | --- |
| `space_list` | Personal space plus groups you belong to |
| `space_create` | Create a group; you become creator |
| `space_show` | Detail and members for a space you can see |
| `me` | Subject, personal space id, and memberships |

See also {doc}`memory-types` and {doc}`mcp-tools`.
