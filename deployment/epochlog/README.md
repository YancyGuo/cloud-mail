# Epochlog Mail deployment

This fork tracks [maillab/cloud-mail](https://github.com/maillab/cloud-mail).
The `production` branch contains the Epochlog deployment configuration on top of the selected upstream release. The starting application version is v3.4.0 (`c0009ddae326197e22c44986b6537ca7d97e803a`).

## Production

- URL: https://mail.epochlog.xyz
- Worker: `cloud-mail-epochlog`
- Worker configuration: `mail-worker/wrangler.epochlog.jsonc`
- Cloudflare build settings: [build-settings.json](build-settings.json)
- Inbound: Cloudflare Email Routing, epochlog.xyz catch-all to the existing Worker.
- Outbound: Resend; only the epochlog.xyz sending key is used in the application.
- Existing D1 and KV bindings must be preserved. Additional sender addresses are created in Cloud Mail and do not require a new deployment.

## Build and deploy

Cloudflare Workers Builds should watch only `production`. Use the root directory, commands, and non-secret build variables in `build-settings.json`. Disable builds and previews for other branches; they must not use production data bindings.

The runtime `jwt_secret` is already stored as a Cloudflare Worker secret. Resend credentials and mailbox settings are already in the application database/cache. Keep credentials on their existing platforms; do not copy them into this repository or build logs. Builds do not need the administrator password or the Resend management OAuth token.

Use Cloudflare Workers Builds as the production publisher. Do not also configure the upstream GitHub Actions deployment with Cloudflare credentials.

## Upgrade

1. Fetch upstream tags in the existing checkout. Read the selected release notes and database migration requirements. Do not blindly sync the upstream default branch into production.
2. Record the live Worker version and production Git commit. Back up D1 and the non-Git configuration required for this change; keep backups outside the public repository.
3. Create a maintenance branch from the current `production` head, merge the selected upstream release, and preserve the Epochlog configuration, existing resource IDs, secret names, and free-tier Resend sending path.
4. Complete any required database migration separately. Never import a first-install bootstrap database over production.
5. Review changes, then merge into `production` to trigger Cloudflare. Other maintenance branches do not publish production or run against production data.
6. Confirm the build and Worker deployment refer to the expected Git commit, the HTTPS application opens, existing mail remains, and the inbound route and sending configuration are intact. When real delivery verification is requested, send only the explicitly authorized test messages.

## Rollback

For a code-only problem, revert the deployment change on `production` and let the same pipeline redeploy, or temporarily roll the Worker back to the recorded version and then reconcile the Git branch. Do not reset or force-push shared history as a routine rollback.

A Worker version rollback does not undo D1/KV writes or schema migrations. Restore data only from the exact selected backup after assessing mail received since that backup; do not discard newer mail by restoring an entire old database automatically.
