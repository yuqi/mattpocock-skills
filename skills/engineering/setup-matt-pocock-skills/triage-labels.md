# Triage Labels

The skills speak in terms of five canonical triage state roles. This file maps those roles to the values recorded in local ticket `Status:` fields. Category values remain `bug` and `enhancement`.

| Canonical role    | Local field value | Meaning                                 |
| ----------------- | ----------------- | --------------------------------------- |
| `needs-triage`    | `needs-triage`    | Maintainer needs to evaluate this issue |
| `needs-info`      | `needs-info`      | Waiting for more information            |
| `ready-for-agent` | `ready-for-agent` | Fully specified, ready for an AFK agent |
| `ready-for-human` | `ready-for-human` | Requires human implementation           |
| `wontfix`         | `wontfix`         | Will not be actioned                    |

When a skill mentions a state role, record the corresponding value from this table in the ticket's `Status:` field.

Edit the middle column to match whatever local status vocabulary you actually use.
