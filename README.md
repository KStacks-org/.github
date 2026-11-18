# KStacks workflows

Application repositories use `dev` and `prod` branches. `dev` is the default for
new work; PRs into `prod` promote it. Container workflows build PRs without
publishing. Pushes publish images; applications registered in infra update only
the matching environment's Helm values on `infra/prod`.

`container.yml` holds the common GHCR build and Helm image-update steps. Callers
pin its commit so a shared workflow edit is reviewed before applications adopt it.
Public frontend settings live in `.env.dev` and `.env.prod`; `frontend: true`
builds both variants from each commit. Backend images use a single `sha-<sha>`
variant. Existing repository-specific checks remain in their own CI jobs.

Optional inputs cover existing Dockerfile differences, Maven JAR builds,
GitHub Packages dependencies, and multi-platform builds. Only the eight apps
registered in infra set `infra-app`; other repositories publish without deploying.
The JWT library retains Maven publishing, and schedules-service has a JAR build
because it has no Dockerfile or registered deployment.

## Branch cutover

Merge the infra migration and ensure the live bootstrap and child Applications
track `prod` before selecting `dev` as infra's default branch. Both environment
values live on `infra/prod`, so app CI can deploy dev without changing production
values. Dev remains disabled until its databases, credentials, DNS, and TLS exist.

Rename current production branches (`main`, `master`, or `production`) to `prod`.
For devs-service, `prod` already exists alongside `main`: reconcile their histories
before removing the old branch. Preserve existing `dev` history; rename
`development` to `dev` where present, and create `dev` from `prod` where absent.
Merge the workflow changes into both branches before setting `dev` as default.
The `.github` repository follows the same naming and keeps its organization profile.
