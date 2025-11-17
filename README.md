# GitHub ➜ GitHub Issue Ops

This repository contains the Issue Ops automation used to migrate GitHub repositories to GitHub with the [`gh gei`](https://github.com/github/gh-gei) CLI.

The automation is triggered from an issue template and a slash command so that migrations can be queued by collaborators without giving them direct access to administrative tokens.

## Prerequisites

1. **Repository secrets**
	- `TARGET_ADMIN_TOKEN`: GitHub personal access token (classic or fine-grained) with the permissions required to create repositories, set secrets, and run migrations in the destination organization.
	- `SOURCE_ADMIN_TOKEN`: GitHub personal access token (classic or fine-grained) with access to the source organization and repositories being migrated.
2. **Issue template enabled** – the template in `.github/workflows/ISSUE_TEMPLATE/github-enterprise-cloud-migration.yml` must remain checked in so that issues can be created from it.

## How the workflow works

| File | Purpose |
| --- | --- |
| `.github/workflows/migration-github-to-github.yml` | Listens for issue comments and calls the shared workflow when the Issue Ops conditions are met. |
| `.github/workflows/shared-github-to-github.yml` | Runs the migration. It parses the issue payload, installs the `gh gei` extension, and calls `gh gei migrate-repo` for each mapping. |
| `.github/workflows/ISSUE_TEMPLATE/github-enterprise-cloud-migration.yml` | Captures the GitHub source and target organization inputs used by the workflow. |

Each migration run publishes an artifact named `statuses` that lists the source and destination repositories together with their success state. Success or failure is also reported back to the originating issue.

## Running a migration via Issue Ops

1. **Create an issue** using the *Repo Migration from GitHub to GitHub* template.
	- List one mapping per line in the `source-repo,target-repo` format (no URLs).
	- Provide the source GitHub organization and target GitHub organization.
2. **Submit the issue.**
3. **Queue the migration** by commenting on the issue with:

	```
	/run-gei-migration
	```

4. **Monitor progress.** The workflow posts a comment with a link to the run. When the run completes, a success or failure comment is added along with per-repository details.

## Notes & troubleshooting

- The workflow currently runs a maximum of five repository migrations in parallel to balance throughput and rate limiting.
- If a repository mapping fails, review the run logs and the `statuses` artifact for the failing entry, fix the underlying issue, and re-run by commenting `/run-gei-migration` again.