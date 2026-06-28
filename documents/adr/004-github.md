# Architecture Decisions

---

# ADR-004: GitHub Integration Is a Game Page Block

## Decision

GitHub integration starts in v2 and exists as a block inside Game Page.

## Reason

- GitHub is supporting context, not the main workflow.
- Users are unlikely to browse PRs as a separate primary screen.
- Pull Requests should be connected to games and stages.
- Developers should be able to link PRs for games they are assigned to.
- Linking PRs does not require manager/admin approval.

## Alternatives

- Standalone GitHub screen.
- Full GitHub project management integration.
- GitHub integration in MVP.

## Consequences

- No separate GitHub navigation in early versions.
- Game Page contains the Pull Requests section.
- Manual PR linking is enough at first.
- GitHub webhooks and PR summaries can be added later.
