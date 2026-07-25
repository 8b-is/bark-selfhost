# bark-selfhost

Run your own [Bark](https://github.com/Finb/Bark) push server. Fork this
repo, set 4 secrets, push to `main` — you have your own private push
notification endpoint for the Bark iOS app, on your own box, storing nothing
on anyone else's server.

## The Apple-credentials misconception

You do **not** need an Apple Developer Program membership, an APNs auth key,
or your own iOS app bundle ID to self-host Bark. Bark's iOS app is free on
the App Store, and [bark-server](https://github.com/Finb/bark-server) — the
open-source backend this repo deploys — ships with the credentials needed to
push to that stock app already built in. Self-hosting replaces *where your
messages are processed and stored*, not *how they reach your phone*. The
only Apple involvement is installing the free app; nothing to enroll for,
nothing to sign.

## What this repo actually does

- `docker-compose.yml` — runs [`finab/bark-server`](https://hub.docker.com/r/finab/bark-server)
  (the actual push backend) plus a `caddy` container in front of it that
  gets you free, automatic HTTPS for a domain you point at your box.
- `.github/workflows/deploy.yml` — on every push to `main`, copies those two
  files to your server over SSH and runs `docker compose up -d`. That's the
  entire CI pipeline — there's no build step, the images are pulled
  pre-built from Docker Hub.

## Setup (fork → your own instance)

1. **Fork this repo.**
2. **Get a box.** Any small VPS with Docker installed and ports 80/443 open
   works — this doesn't need real compute, a $5/mo box is overkill already.
3. **Point a domain (or subdomain) at the box's IP** — an A record is
   enough. This is what Caddy uses to issue a free TLS cert automatically.
4. **Add 4 repo secrets** (Settings → Secrets and variables → Actions):

   | Secret | Value |
   |---|---|
   | `DEPLOY_HOST` | Your box's IP or hostname |
   | `DEPLOY_USER` | SSH user with Docker access on that box |
   | `DEPLOY_SSH_KEY` | Private key for that user (generate a dedicated deploy key, don't reuse your personal one) |
   | `DOMAIN` | The domain/subdomain you pointed at the box, e.g. `bark.yourdomain.com` |

5. **Push to `main`** (or run the workflow manually). The Action SSHes in,
   ships the compose files, and brings the stack up.
6. **In the Bark iOS app**: Settings → server address → enter
   `https://<your DOMAIN>`. Register, and you'll get your own device key —
   from your own server, not `api.day.app`.

## Using it

Same API either way, just against your own host:

```sh
curl -X POST "https://<your DOMAIN>/push" \
  -H 'Content-Type: application/json' \
  -d '{"device_key":"<your key>","title":"Hello","body":"From my own server"}'
```

## Why self-host at all

The public `api.day.app` server works fine and is free — this only matters
if you want your notification history to never touch a third party's
database, or you just don't want a dependency on someone else's uptime for
something that pages your phone.
