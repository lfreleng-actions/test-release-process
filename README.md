<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# Test Release Process

Repository that exercises the organisation's release tooling end-to-end. A
scheduled workflow raises a trivial pull request (containing the current date)
once a week; manual invocation via `workflow_dispatch` does the same on
demand. If an automated pull request already exists, it gets updated rather
than duplicated.

Merging a pull request feeds the
[release-drafter](https://github.com/release-drafter/release-drafter) action,
which maintains a draft release. Test the rest of the release process by
pushing a signed, semantic-versioned tag: the workflow checks the tag
(versioning scheme and signature), and on success promotes the matching draft
release to a published release.

## test-release-process

## Workflow Overview

1. **Generate Pull Request** (`pull-request.yaml`) — on a weekly schedule or
   `workflow_dispatch`, writes the current date to `date.txt` and raises (or
   updates) a pull request via the local composite action (`action.yaml`).
2. **Release Drafter** (`release-drafter.yaml`) — on every push to `main`,
   updates the draft release notes.
3. **Release on Tag Push** (`tag-push.yaml`) — on a tag push, validates the
   tag then promotes the associated draft release.

## Notes

While exercising the release process, this repository tests the following
actions:

- [lfreleng-actions/tag-validate-action][tag-validate] — unified tag
  validation (versioning scheme plus SSH/GPG signature verification, and
  optional GitHub/Gerrit signing-key registration checks). This supersedes the
  older `tag-push-verify-action`, `tag-validate-semantic-action`, and
  `tag-validate-calver-action`.
- [lfreleng-actions/draft-release-promote-action][promote] — promotes a draft
  release to a published release.
- [release-drafter/release-drafter][drafter] — maintains draft release notes.
- [peter-evans/create-pull-request][cpr] — raises the automated pull request.
- [step-security/harden-runner][harden] — runner egress hardening.

The unified `tag-validate-action` handles both Semantic Versioning and
Calendar Versioning tags (`require_type: semver`, `calver`, or `both`), so the
separate `tag-validate-semantic-action` and `tag-validate-calver-action`
repositories no longer need dedicated coverage here.

## Automated Pull Request Permissions

The scheduled workflow raises a pull request from within GitHub Actions. When
an organisation enforces the *"Allow GitHub Actions to create and approve pull
requests"* restriction (recommended for supply-chain hardening), the default
`GITHUB_TOKEN` **cannot** open pull requests and the workflow fails with:

```text
GitHub Actions is not permitted to create or approve pull requests.
```

To keep the organisation hardened while still allowing this repository to
raise its automated pull request, the workflow uses a dedicated GitHub App
token when you configure one. Provide the following repository (or
organisation) configuration:

| Kind     | Name                        | Description                  |
| -------- | --------------------------- | ---------------------------- |
| Variable | `LF_RELENG_BOT_APP_ID`      | GitHub App ID                |
| Secret   | `LF_RELENG_BOT_PRIVATE_KEY` | GitHub App private key (PEM) |

The GitHub App needs `contents: write` and `pull requests: write` repository
permissions. Install the App on this repository. Without
`LF_RELENG_BOT_APP_ID`, the workflow falls back to `GITHUB_TOKEN`, which
works where the organisation permits Actions to create pull requests.

[tag-validate]: https://github.com/lfreleng-actions/tag-validate-action
[promote]: https://github.com/lfreleng-actions/draft-release-promote-action
[drafter]: https://github.com/release-drafter/release-drafter
[cpr]: https://github.com/peter-evans/create-pull-request
[harden]: https://github.com/step-security/harden-runner
