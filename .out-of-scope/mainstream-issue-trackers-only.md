# Remote issue tracker integrations are out of scope

`setup-matt-pocock-skills` ships one Issue Tracker implementation: local markdown under `.scratch/`. Requests to add GitHub, GitLab, or another remote provider are out of scope.

## Why this is out of scope

Every remote backend hard-codes a CLI and API shape into the distribution. Each one adds permanent maintenance and test surface across `/to-spec`, `/to-tickets`, `/triage`, and the other Issue Tracker consumers. Keeping the interface while shipping one local implementation preserves the existing skill contracts without carrying provider adapters.

## Prior requests

- #99: "Add dex as an issue tracker backend"
