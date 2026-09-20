---
aliases:
  - Trusted Workstation Staging Deployment
  - Local Vercel Staging Runbook
tags:
  - RPRT
  - software-development
  - deployment
  - vercel
  - bitwarden
  - security
  - runbook
type: operational-runbook
status: draft
updated: 2026-09-20
---
# Local Staging CLI Deployment

> [!abstract] Trusted Workstation Runbook
> This runbook defines the temporary local CLI path for staging deployments while GitHub Actions is unavailable. It keeps secrets in Bitwarden, builds the exact protected `dev` revision, uploads Sentry source maps, deploys verified prebuilt Vercel output, and records secret-free evidence.

**Related notes:** [[Studio Repertório]] · [[Sentry and Source Maps]] · [[Resend Project Initialization]]

> [!warning] Temporary Path
> This method does not prove the canonical GitHub `Staging` workflow. Coordinate each run with RPRT-91 and record the local trusted-workstation exception on the owning Linear issue.

---

## <span style="color:#80352F">Recommendation</span>

Use Bitwarden CLI as the secret source. Do not store secret values in a plaintext `.env` file, Obsidian, the repository, `/mnt/c`, shell history, command arguments, logs, or deployment evidence.

Store only a Bitwarden item ID in a mode-600 reference file under the Linux home directory. Unlock Bitwarden once for the deployment session. A reviewed deployment helper must read the vault item into memory, give each tool only the values it needs, and clear the process environment during cleanup.

| Option | Decision | Reason |
|---|:---:|---|
| Bitwarden CLI session | Use | One interactive vault unlock per session; values remain encrypted at rest |
| Reference-only local file | Use | Contains only one vault item ID, not credentials |
| Plaintext secret file with `chmod 600` | Reject | File copies, backups, indexing, and user-level processes can expose values |
| Vercel CLI saved personal login | Reject | Does not supply Sentry, Resend, or Supabase secrets and can use the wrong identity |
| Secrets in Obsidian | Reject | The vault is documentation, not a credential store |

## <span style="color:rgb(112, 48, 160)">Secret Inventory</span>

Create one Bitwarden vault item named `Studio Repertório Staging Deployment`. Add these values as **Hidden** custom fields:

| Field | Consumer | Lifetime |
|---|---|---|
| `VERCEL_TOKEN` | Vercel pull, deploy, alias, and protected smoke request | Vercel operations only |
| `SENTRY_AUTH_TOKEN` | Staging build, release upload, artifact check, and exact-value scan | Build and Sentry checks only |
| `RESEND_API_KEY` | Staging build validation and deployed runtime | Build and deploy child processes only |
| `SUPABASE_SECRET_KEY` | Staging build validation and deployed runtime | Build and deploy child processes only |

Keep the existing provider restrictions:

| Credential | Required Boundary |
|---|---|
| Vercel token | Approved deployment service identity with the smallest practical team scope |
| Sentry token | `org:ci` only; never store in Vercel |
| Resend key | Approved staging sandbox key |
| Supabase key | Approved staging project only |

> [!important] Rotation
> Update the Bitwarden item and the authoritative GitHub `Staging` secret together after a rotation. Never keep an old value as a fallback field.

## <span style="color:rgb(0, 112, 192)">One-Time Workstation Setup</span>

### 1. Install Bitwarden CLI

Use the official Linux CLI package or the official npm package. Verify the downloaded native binary checksum when that installation method is used.

Official documentation: [Bitwarden Password Manager CLI](https://bitwarden.com/help/cli/)

### 2. Create the reference-only file

Keep this file on the Linux filesystem, not under `/mnt/c`:

```bash
install -d -m 700 "$HOME/.config/repertorio"
umask 077
printf '%s\n' '<BITWARDEN_ITEM_ID>' \
  > "$HOME/.config/repertorio/staging-deploy.item-id"
chmod 600 "$HOME/.config/repertorio/staging-deploy.item-id"
```

The file must contain one exact Bitwarden item ID and a final newline. It must not contain a vault session key, token, API key, password, DSN, or environment assignment.

### 3. Log in interactively

```bash
bw login
```

Use interactive login for workstation sessions. Do not put the Bitwarden master password, personal API key, or two-step code in a command argument or shell file.

## <span style="color:rgb(0, 176, 80)">Session Start</span>

Run this once at the start of an approved deployment session:

```bash
set +x
bw sync
export BW_SESSION="$(bw unlock --raw)"
bw status
```

`BW_SESSION` is a decryption key. Keep it in the current shell environment. Do not pass it with `--session`, because command arguments can be visible to other local processes.

The approved helper must then:

1. Read the Bitwarden item ID from the reference-only file.
2. Run `bw get item <id>` without printing its JSON.
3. Require exactly one Hidden field for each approved name.
4. Validate expected prefixes and project identity without logging values.
5. Keep secret values only in the helper process and narrowly scoped child processes.
6. Refuse to run while shell tracing is enabled.

> [!danger] No Raw Vault Output
> Do not run `bw get item` directly in an ordinary terminal during deployment. Its JSON contains every field value. The helper must capture and parse stdout in memory.

## <span style="color:rgb(255, 140, 0)">Per-Deployment Procedure</span>

### Phase 1: Authorize and pin the source

1. Confirm that local deployment is still approved while GitHub Actions is unavailable.
2. Record the temporary trusted-workstation exception on the owning Linear issue.
3. Fetch `origin/dev`.
4. Record the exact remote commit SHA.
5. Create a new detached worktree under `/tmp/opencode` from that SHA.
6. Require a clean worktree before build and before evidence collection.
7. Do not build from the normal development worktree.

Example source preparation:

```bash
git fetch origin dev
release="$(git rev-parse origin/dev)"
git worktree add --detach \
  "/tmp/opencode/repertorio-staging-$release" \
  "$release"
```

Stop if the release is not an exact 40-character SHA, the expected protected `dev` head changed, or the worktree is dirty.

### Phase 2: Verify tools and public configuration

Use repository-pinned or workflow-pinned versions:

| Tool | Current Required Version |
|---|---|
| Node.js | `24.19.0` |
| pnpm | `11.20.0` |
| Vercel CLI | `59.1.4` |

Set `umask 077` before any tool writes configuration. Give `VERCEL_TOKEN` only to `vercel pull`. Pull Vercel Preview configuration for branch `dev`, then remove every privileged assignment from the local pull artifact.

Verify names and approved destinations only:

| Variable | Required Result |
|---|---|
| `APP_ENVIRONMENT` | `staging` |
| `SUPABASE_PROJECT_REF` | Approved staging project |
| `NEXT_PUBLIC_SENTRY_DSN` | Approved public staging destination |
| `SENTRY_AUTH_TOKEN` | Absent from Vercel configuration |
| `VERCEL_OIDC_TOKEN` | Removed from local configuration artifacts |

Vercel Sensitive variables cannot be downloaded by `vercel env pull` or `vercel env run`. The local helper must obtain `RESEND_API_KEY` and `SUPABASE_SECRET_KEY` from Bitwarden instead.

### Phase 3: Set immutable deployment identity

The build process must receive these non-secret values:

```text
APP_ENVIRONMENT=staging
VERCEL=1
VERCEL_ENV=preview
VERCEL_GIT_PROVIDER=github
VERCEL_GIT_REPO_OWNER=guisaliba
VERCEL_GIT_REPO_SLUG=repertorio
VERCEL_GIT_COMMIT_REF=dev
VERCEL_GIT_COMMIT_SHA=<exact origin/dev SHA>
```

Do not infer application identity from `NODE_ENV`. A Vercel Preview build uses `NODE_ENV=production`.

### Phase 4: Run local quality gates

Run focused checks before the external build write:

```bash
pnpm install --frozen-lockfile
pnpm test:unit
pnpm typecheck
pnpm lint
pnpm format:check
pnpm verify:dependencies
```

Stop on any failure. Do not delete or weaken a failing test to continue the deployment.

### Phase 5: Build and upload Sentry artifacts

Give the build child process only the build-time values it requires:

```text
SENTRY_AUTH_TOKEN
RESEND_API_KEY
SUPABASE_SECRET_KEY
VERCEL_TOKEN only when Vercel CLI requires it
```

Run the pinned Vercel Preview build. The Sentry release must equal the exact deployment SHA. Stop on authentication, release, commit, or source-map errors.

Verify through read-only Sentry API calls:

| Check | Required Result |
|---|---|
| Release endpoint | HTTP 200 |
| Release version | Exact deployment SHA |
| Finalized release | Yes |
| Project link | `repertorio` |
| Artifact bundles | At least one matching bundle |
| Artifact files | More than zero |

Never print a Sentry response body when it can contain unapproved provider data. Report only safe status, counts, IDs, and timestamps.

### Phase 6: Package and scan

Reproduce the package and exact-secret scans from the authoritative workflow. Run the scan after every generated-package change.

Require all of these results:

| Check | Required Result |
|---|---|
| Package roots | Only approved `.vercel`, `.next`, `node_modules`, and metadata paths |
| Symlinks in upload package | None |
| Trace paths | Relative, allowlisted, and inside approved roots |
| Trace sources | Present regular files or approved directories before normalization |
| Exact loaded secret values | Absent from every path and file |
| Client source maps in deployable static output | Zero |
| Sentry upload token in runtime output | Absent |

> [!failure] Current Packaging Defect
> The current workflow converts PNPM trace symlinks into directories but leaves each directory in Vercel `filePathMap` as one file reference. Vercel then omits required package aliases. The first 2026-09-20 local deployment failed because `next/dist/compiled/next-server/server.runtime.prod.js` was not available at runtime.

Before the next repeated local deployment, add a reviewed repository helper that expands traced PNPM aliases into explicit regular-file mappings, includes sibling dependency aliases, validates every mapping, and materializes at least the health function in isolation. Do not repeat the large inline repair manually.

The isolated package test must start the generated function with schema-valid synthetic local values and require:

```text
GET /api/health -> HTTP 200
body -> {"status":"ok"}
```

### Phase 7: Deploy without secret files or secret arguments

Prebuilt CLI deployments do not automatically provide all Vercel system environment values. They also did not select the complete branch-specific staging runtime during the 2026-09-20 proof.

The deploy helper must pass variable **names** through Vercel `--env NAME` options while values exist only in the child process environment. It must explicitly supply these runtime names:

```text
APP_ENVIRONMENT
NEXT_PUBLIC_SENTRY_DSN
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
NEXT_PUBLIC_SUPABASE_URL
RESEND_API_KEY
SUPABASE_PROJECT_REF
SUPABASE_SECRET_KEY
VERCEL_GIT_REPO_OWNER
VERCEL_GIT_REPO_SLUG
```

It must attach these safe metadata fields:

```text
githubDeployment=1
githubCommitRef=dev
githubCommitSha=<exact SHA>
githubCommitOrg=guisaliba
githubCommitRepo=repertorio
```

Do not put `KEY=value` secret pairs in Vercel CLI arguments. Do not write sensitive Vercel values to a temporary environment file.

### Phase 8: Verify before alias assignment

Treat deployment and alias assignment as separate external writes.

1. Create an immutable Preview deployment.
2. Wait for `READY`.
3. Verify project `repertorio`, branch `dev`, and the exact commit SHA.
4. Run authenticated `GET /api/health` against the immutable deployment URL.
5. Require HTTP 200 and `{"status":"ok"}`.
6. Inspect runtime logs for warnings, errors, or fatal events.
7. Assign `dev-studiorepertorio.vercel.app` only after all checks pass.
8. Run the same health request through the stable alias.
9. Verify that the alias resolves to the approved deployment ID.

A `READY` deployment is not sufficient evidence. Vercel can mark a deployment READY while its serverless functions fail during invocation.

## <span style="color:rgb(192, 0, 0)">Stop Conditions</span>

Stop immediately when any condition is true:

| Condition | Required Action |
|---|---|
| Secret appears in chat, logs, history, arguments, URL, or artifact | Revoke and rotate it before continuing |
| Remote `dev` SHA changes after the build starts | Discard the build and restart from the new exact SHA |
| Worktree is dirty | Identify the change; do not claim exact-source evidence |
| Sentry release or artifacts do not match the SHA | Do not deploy or alias |
| Package contains a secret, symlink, unsafe trace, or static source map | Delete the candidate package and fix the packager |
| Deployment runtime returns HTTP 500 | Inspect logs; do not assign the alias |
| Runtime identity resolves as feature Preview | Fix runtime inputs; do not weaken environment validation |
| Alias command targets an unverified URL | Stop and verify the immutable deployment first |

## <span style="color:rgb(0, 176, 240)">Evidence Record</span>

Record only secret-free evidence:

| Evidence | Safe Fields |
|---|---|
| Source | Branch and exact SHA |
| Local checks | Command name, pass/fail, test counts |
| Sentry | Release status, matching SHA, bundle/file counts, safe timestamps |
| Vercel | Deployment ID, immutable URL, alias, READY state, region |
| Smoke test | Route, status, approved response body |
| Logs | Safe counts and error classification; no raw payloads |
| Exceptions | Reason for the local path and remaining canonical-workflow gap |

Do not record DSN keys, tokens, passwords, recipient data, request headers, raw provider payloads, hashes, fingerprints, or recoverable secret fragments.

## <span style="color:rgb(112, 48, 160)">Session Cleanup</span>

The helper must clean up on success, failure, and interruption:

1. Unset all application and provider secrets.
2. Remove temporary Vercel configuration and runtime files.
3. Remove the generated package after evidence is complete.
4. Remove the detached worktree when it is no longer needed.
5. Lock Bitwarden.
6. Unset the Bitwarden session key.

```bash
bw lock
unset BW_SESSION
```

Check that no temporary environment file remains under `/tmp/opencode`.

## <span style="color:#80352F">Automation Target</span>

Do not keep using long, ad hoc shell commands. Add one reviewed repository tool before the next repeated local deployment.

Suggested interface:

```bash
pnpm deploy:staging:local preflight
pnpm deploy:staging:local build
pnpm deploy:staging:local package
pnpm deploy:staging:local deploy
pnpm deploy:staging:local verify
pnpm deploy:staging:local cleanup
```

The helper can persist only non-secret resume state: exact SHA, worktree path, package path, deployment ID, URLs, and completed phase names. It must never persist provider values or the Bitwarden session key.

After the helper exists, the normal operator flow should be:

```bash
set +x
bw sync
export BW_SESSION="$(bw unlock --raw)"
pnpm deploy:staging:local preflight
pnpm deploy:staging:local build
pnpm deploy:staging:local package
pnpm deploy:staging:local deploy
pnpm deploy:staging:local verify
pnpm deploy:staging:local cleanup
bw lock
unset BW_SESSION
```

> [!warning] Draft State
> `pnpm deploy:staging:local` does not exist yet. The next implementation task is to extract the proven local process into this reviewed helper and tests. Until then, use this runbook with an operator and stop before every external write.

## <span style="color:rgb(255, 192, 0)">2026-09-20 Lessons</span>

| Attempt | Result | Lesson |
|---|---|---|
| First candidate | Function could not load Next.js runtime | A symlink-free package needs explicit regular-file alias mappings |
| Second candidate | Runtime environment guard rejected mixed Preview/staging values | Prebuilt deployment needs explicit staging runtime inputs |
| Final candidate | READY, immutable health 200, alias health 200 | Verify invocation before moving the stable alias |

The successful proof used exact source commit `f64b4c8f5773ea5e920b2eef05300f36f3ecb118`, a finalized Sentry release, two artifact bundles with 126 files, and Vercel deployment `dpl_5g3hRfBBDT6cRHaikm7TpkymRBMV`. These values are historical evidence, not defaults for future runs.

## <span style="color:#80352F">Sources of Truth</span>

- [`docs/environments.md`](https://github.com/guisaliba/repertorio/blob/dev/docs/environments.md)
- [`.github/workflows/ci.yml`](https://github.com/guisaliba/repertorio/blob/dev/.github/workflows/ci.yml)
- [Bitwarden Password Manager CLI](https://bitwarden.com/help/cli/)
- [Bitwarden custom fields](https://bitwarden.com/help/custom-fields/)
- [RPRT-70](https://linear.app/guisaliba/issue/RPRT-70/configure-staging-sentry-and-prove-live-delivery)
- [RPRT-91](https://linear.app/guisaliba/issue/RPRT-91/prove-protected-staging-deployment-and-complete-staging-secret)

> [!note] Authority
> Repository policy and reviewed helper code are authoritative. This note explains the temporary operator method and must be updated when the canonical GitHub Actions path returns.
