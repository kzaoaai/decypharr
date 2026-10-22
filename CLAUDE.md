@FORK.md

# kzaoaai/decypharr

A fork of [sirrobot01/decypharr](https://github.com/sirrobot01/decypharr) that exists to
build our own Decypharr image. Branch `main-patched` is
[h1n054ur/decypharr](https://github.com/h1n054ur/decypharr) `main-patched` **frozen at
`caca7380`** (upstream v2.7 plus the fixes `FORK.md` lists), plus our image workflow on top.
The code under `pkg/` is identical to `caca7380`. `main` and the other branches are
upstream's, untouched.

## Build and deploy

- Push a tag matching `v*-aa*` on `main-patched`: `.github/workflows/image.yml` builds the
  stock `Dockerfile` for linux/amd64 and pushes `ghcr.io/kzaoaai/decypharr:<tag>`. The run's
  summary prints the digest. The package is public, so pulling needs no credentials.
- Upstream's release, beta and docs workflows are deleted on this branch so a tag here never
  publishes upstream's artifacts. Keep it that way.
- Deployments pin the image **by digest**, never by tag. Where it runs and how to roll back is
  in the operator's own documentation, not here.
- Current image: tag `v2.7-patched-aa1` = commit `3389f0e`.

## Our own commits on top of `caca7380`

- `3389f0e` ci: our GHCR image workflow. The only change that matters.
- `87a049f` and its revert `87aa739`: a queue-cleanup change that never took effect; reverted
  so the code matches `caca7380`. Its tag `v2.7-patched-aa2` was deleted. Don't re-apply it:
  the Arrs report a download client's error with `trackedDownloadStatus` ok, which queue
  cleanup skips before matching, so a fix would also have to accept `status` warning.

## Conventions

- **Nothing from h1n054ur's branch arrives unreviewed.** Taking newer fork or upstream
  commits means: compare them against upstream (`gh api repos/sirrobot01/decypharr/compare/<base>...h1n054ur:decypharr:main-patched`),
  check each cherry-pick exists upstream with the same changes and each "(upstream #N)"
  squash matches that PR's diff, read every commit of the fork author's own, then rebase this
  branch, run the tests, and push a new `v*-aa<N>` tag (the next free number is 3).
- Once upstream releases a version containing the #434 fix (PR #450), the plan is to return
  to the stock upstream image and retire this fork.
- No lab-specific data (hosts, addresses, names) in this public repo.

## Test

```sh
go test ./pkg/manager/link/ ./pkg/server/qbit/ ./pkg/debrid/providers/torbox/
```

These cover the patched link validation (#450), qBittorrent hash handling and TorBox
naming. They pass on `3389f0e`. A full `go test ./...` needs the CGO/FUSE toolchain the
`Dockerfile` installs.

## Open items

- Review scheduled for 2026-11-04: what upstream merged or released since v2.7 (#434/#450,
  #444, #442, #400) and what h1n054ur added after `caca7380`; then stay, rebase, or retire.
- Upstream #400 (TorBox `queued` state treated as an error) is not fixed in this build.
