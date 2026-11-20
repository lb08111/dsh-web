# Agent Note: Source-built Linux container and authenticated loopback entry

Status: implemented

## Problem

The local source deployment needs a reproducible Linux amd64 delivery that includes the operations workspace, retains application state during updates, and makes SSH and operations routes reachable from a remote browser with the same authenticated owner authority. A host-only Docker recipe does not establish those plugin access boundaries or package the local workspace additions.

## Decision

The root Dockerfile builds the current workspace on Node 24.22.3 with the official DSH 0.2.2-rc.3 host. The runtime image contains the official host and the independent production closure created by pnpm deploy. The preparation step repairs the aggregate self-reference, verifies that dsh-ops is present, and pins every host-supplied SDK dependency to the same installed cohort. Credentials and local dependency trees are excluded from the build context.

The workspace install uses an amd64-specific BuildKit pnpm store cache. It records the selected Node, libuv and pnpm versions and only the presence of Node IPC environment variables. The canonical install writes to a regular temporary log file, which is printed after the process finishes. Its original failure status still stops the build before compilation and packaging.

Application state lives in `/data`, and work files live in `/workspace`. Startup creates the Web profile and mounts the packaged family. Existing real package directories remain intact; package symlinks are atomically refreshed to the current image's production paths. A fresh profile enables SSH, while subsequent profile settings persist.

The DSH host binds container loopback. A public proxy checks the configured public Host and Origin, verifies the native signed browser-owner cookie by requesting the native index, and only then translates the request to the loopback authority. A launch-token navigation reaches the native index to establish that cookie. HTTP and WebSocket requests share the same owner check. An API peer credential grants no management access through this proxy.

The installer reads the private delivery's registry.credentials as literal data and sends its password through standard input. It serializes updates, retains the two bind-mounted directories, and restarts the previous managed container when a replacement fails startup. Readiness requires the native entry, an empty degradation ledger, the operations tree and the SSH host repository.

When replacing a managed container, the installer adopts its existing `/data` and `/workspace` bind sources unless that directory was explicitly configured. It also preserves the container's configured runtime user. A change in the invoking user's home or sudo context therefore does not select empty default directories or change the application UID and GID.

## Alternatives considered

Copying the developer machine's node_modules was considered and rejected because native modules and absolute workspace links must match Linux amd64 and the runtime installation. An independent production closure provides a build-time check of those paths.

Exposing the DSH listener directly was considered and rejected because the SSH and operations routes classify remote sockets as outside their loopback scope. Removing those fences would change plugin security behavior. The container entry authenticates the browser owner before translating its validated request.

The earlier [container-assets proposal](../../rejected/architecture/2026-09-13-remote-web-ui-docker-assets.md) was rejected when the project had no verified Docker execution environment and its scope was the remote-web plugin. This delivery packages the official host with source-built application plugins and is exercised through container build and access checks. Its boundary is documented in the [deployment guide](../../../../deploy/docker/README.md).

## Consequences

The artifact runs on Linux amd64 with a fixed host version. Model access and SSH secrets are configured after deployment and persist outside the image. Updates retain application state, while the image remains the authority for packaged plugin code. The deployment serves one configured browser origin and requires its value to follow address or proxy changes.

The shared [loopback classification](../bug-fix/2026-10-19-reverse-proxy-loopback-fence.md) remains unchanged. The proxy owns its public authentication and same-origin decision, and only authenticated owner traffic is presented as local to the private listener. Native container health checks also detect a reachable Web shell whose management plugins failed to mount.

The first amd64 build under ARM emulation completed pnpm package linking, then its install process exited with SIGABRT in libuv. The regular-file log output is a diagnostic change to test inherited standard I/O behavior, not a verified fix. Node and pnpm remain pinned, and unsuccessful installs are never accepted as complete.
