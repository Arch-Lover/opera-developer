# GitHub Actions and AUR publishing

The workflow at `.github/workflows/aur-build.yml` provides the CI/CD pipeline.
It runs a clean Arch Linux build for pull requests and for changes to the
package files on `main`. It verifies source checksums, resolves package
dependencies, builds the package, generates `.SRCINFO`, and uploads the built
package as a seven-day Actions artifact.

After a successful package-file push to `main`, it automatically commits the
following generated/current packaging files to the AUR repository's required
`master` branch:

- `PKGBUILD`
- `.SRCINFO`
- `opera`
- `default`

Documentation-only commits do not trigger the workflow or publish to AUR. The
workflow can also be started from **Actions → Build and publish AUR package →
Run workflow**. Manual runs are build-only unless **publish** is selected.

## One-time GitHub setup

1. Create or register the `opera-developer` package under the AUR account that
   should own it. The package name must match the default `AUR_PACKAGE` value,
   unless you set a different repository variable.
2. Create a dedicated SSH key pair with no passphrase. Add its **public** key
   to that AUR account, then add the **private** key to this GitHub repository's
   Actions secrets as `AUR_SSH_PRIVATE_KEY`.
3. In **Settings → Environments**, create `aur-production`. Add required
   reviewers if you want approval before every AUR push. The workflow itself
   still builds automatically; this environment controls the publishing gate.
4. Optional repository variables:

   | Variable | Default | Purpose |
   | --- | --- | --- |
   | `AUR_PACKAGE` | `opera-developer` | AUR Git repository/package name. |
   | `AUR_GIT_NAME` | `GitHub Actions` | Commit author name in the AUR repository. |
   | `AUR_GIT_EMAIL` | `actions@users.noreply.github.com` | Commit author email in the AUR repository. |

Keep `AUR_SSH_PRIVATE_KEY` in Secrets, never Variables or tracked files. The
workflow retrieves AUR sources over SSH as `aur@aur.archlinux.org`, which is
the AUR Git endpoint.

The secret must contain the complete **private** key, including its
`-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----`
lines. Do not paste the `.pub` file, a public key beginning with `ssh-ed25519`,
or surrounding quotes. Register the matching public key with the AUR account
that owns (or is a co-maintainer of) the package.

## Release workflow

1. Update `pkgver`, source URL/checksums, and (when needed) the NW.js FFmpeg
   version in `PKGBUILD`.
2. Open a pull request and wait for the build job to pass.
3. Merge the package-file change to `main`.
4. The pipeline rebuilds it, regenerates `.SRCINFO`, and publishes the recipe.
   If `aur-production` has reviewers, approve that deployment in GitHub.

The AUR receives source recipes rather than the built `.pkg.tar.*` archive;
the archive remains available from the workflow run's artifacts for seven days.

## Upstream version checks

`.github/workflows/check-updates.yml` is started manually from the **Actions**
tab. It checks Opera's Developer download index and the NW.js FFmpeg project's
latest GitHub release against `pkgver` and
`_nwjs_ffmpeg_version` in `PKGBUILD`.

When an update is found, it creates an update pull request with refreshed
versions and source checksums. Review the FFmpeg compatibility, approve, and
merge that pull request. It never merges the pull request or publishes to AUR;
only the subsequent `main` workflow can do that.
