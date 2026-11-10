# Groups

A **group** is a shared place for memories. Everyone in the group can read and
write there. Groups are optional — personal memory works on its own.

```text
you
  ├── personal     private; only you
  └── groups[]     shared spaces you belong to
```

This is not the same as **User Groups** under System Settings
(Administrators / Users). Those control who may sign in as an administrator.
Memory groups are only about sharing entries and guides.

## Creating and belonging

- Any signed-in user can create a group from the web UI (or via a connected
  agent).
- The creator owns the group: invite or remove people, rename it, and delete
  it. Deleting a group also deletes the memories in it.
- An invitation grants nothing until the invited person accepts it on the
  **Groups** page (or declines it there). Until then the group's memories,
  guides and roster stay hidden from them, and their writes never land in it.
- Inviting by email gives the same answer whether or not the address has an
  account, so the roster does not reveal who is registered.
- Other members can use the group’s memories, leave the group, and otherwise
  cannot manage the roster or delete the group.
- Changing a memory (`revise`) needs the same right as forgetting it: its
  creator, the group creator, or an admin. Repeating `guide_write` only
  reinforces your own guide; it never replaces another member's guide.
- There is no separate viewer / editor ladder beyond creator vs everyone else.
- The creator cannot be removed from the roster.

An admin account can manage groups across the site when needed. Everyday use
should be your own user account.

## Where new memories go

When you (or an agent) save a memory, you choose a destination:

1. A specific group you belong to, or
2. Your default shared group (if you set one), or
3. Otherwise your **personal** space.

Each memory lives in exactly one space. Who can see it is whoever belongs to
that space.

## Related

- What entries and guides are: {doc}`memory`
- What a connected agent can do with spaces: {doc}`mcp-tools`
