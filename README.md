# renovate-config

A shared [Renovate](https://docs.renovatebot.com/) configuration preset, applied across my repositories.

## Usage

In any repo, drop a `renovate.json` (or `.github/renovate.json`) at the root:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>eddgrant/renovate-config"]
}
```

Renovate resolves the preset by reading [`default.json`](./default.json) from this repository.

## What you get

### Noise reduction

- **`minimumReleaseAge: "21 days"`** — Renovate waits three weeks after a release before opening a PR for it. Catches packages that get superseded by another point release shortly after publication.
- **No schedule** — once a release clears the age gate, Renovate opens the PR immediately. The age gate alone provides the noise reduction; layering a weekly schedule on top would only add latency.
- **`prConcurrentLimit: 10`, `prHourlyLimit: 4`** — caps the open-PR floor and the rate at which new PRs appear.
- **Dependency Dashboard** issue automatically created in every consumer repo so updates have one centralised view.

### Security

- **Vulnerability alerts bypass the age gate** — CVE-driven updates flow immediately, labelled `security`.

### Grouping

Coherent release trains and tightly-coupled packages are batched into one PR each, instead of the default one-PR-per-package fragmentation:

| Group | What it covers |
|---|---|
| **Kotlin** | `org.jetbrains.kotlin*`, `com.google.devtools.ksp*`, Kover |
| **Gradle wrapper** | The `gradle-wrapper` manager |
| **Micronaut** | All `io.micronaut*` packages |
| **Next.js + React** | `next`, `react`, `react-dom`, `@types/react*`, `eslint-config-next` |
| **TypeScript** | `typescript`, `ts-node`, `tsx`, `@types/node` |
| **JS lint tooling** | `eslint`, `prettier`, `@typescript-eslint/*`, `eslint-*` |
| **Terraform providers / modules** | One group per datasource type |
| **GitHub Actions** | All `uses:` pins in workflows |
| **Docker digest pins** | Base-image digest refreshes |
| **non-major dev dependencies** | npm `devDependencies` (minor/patch); same for Python `dev-dependencies` |

### Other defaults

- **Semantic commits** — `chore(deps): …` prefix on every PR.
- **Lockfile maintenance** — runs monthly (first day of the month, before 9am Europe/London) with the age gate disabled.
- **`internalChecksFilter: "strict"`** — Renovate suppresses PRs that fail its internal checks (e.g. age gate not yet met) instead of pre-creating them.
- **`rebaseWhen: "behind-base-branch"`** — branches rebase only when the base moves on, not on every commit.
- **Major versions split** — bumps that span more than one major version get a separate PR per major (`:separateMultipleMajorReleases`).
- **Auto-assigned to `@eddgrant`**.

## Per-repo overrides

The preset is just a base — any repo can override or add rules in its own `renovate.json`. A few common patterns:

### Enable automerge for low-risk updates

```json
{
  "extends": ["github>eddgrant/renovate-config"],
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "matchDepTypes": ["devDependencies"],
      "automerge": true
    }
  ]
}
```

### Drop the age gate for a fast-moving project

```json
{
  "extends": ["github>eddgrant/renovate-config"],
  "minimumReleaseAge": "0 days"
}
```

### Pin a package to a specific major

```json
{
  "extends": ["github>eddgrant/renovate-config"],
  "packageRules": [
    {
      "matchPackageNames": ["some-package"],
      "allowedVersions": "<2.0.0"
    }
  ]
}
```

## Repository management

This repository, like all of mine, is created and managed via [`eddgrant-IaC`](https://github.com/eddgrant/eddgrant-IaC) — see `terraform/configurations/management/github/eddgrant-user/terragrunt.hcl`.
