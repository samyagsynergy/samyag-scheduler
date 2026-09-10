# Deploying the Samyag Synergy Scheduler

This turns the files in this folder into a live tool — first just for you to
test with your own accounts, later opened up to others once platform
approvals come through. Steps only you can do are marked **YOU DO THIS**.

Two things that changed from earlier drafts of this plan, based on what we
worked through together:
- **No Stripe account needed at all** — see `SECURITY.md` for why the
  placeholder keys already in `docker-compose.yaml` are safe and sufficient.
- **X (Twitter) is left out** — see `CHANGES.md`.

---

## Step 1 — Push the fork to your own GitHub

**YOU DO THIS.** Create a free GitHub account if you don't have one, then a
new **public** repository (public is required — see `CHANGES.md` for why) —
e.g. named `samyag-scheduler`.

Once it exists, run these commands (on your own machine or the server —
either works) to build it from real upstream Postiz plus our one-line patch:

```bash
git clone https://github.com/gitroomhq/postiz-app.git samyag-scheduler-source
cd samyag-scheduler-source
git checkout 36d5fc7b3ac3f17178b1589cf7a7337523017a41   # exact commit this patch applies to
git apply ../pricing-fork.patch                          # makes free-tier limits configurable (defaults: 31 posts, 10 channels)
git remote set-url origin https://github.com/<you>/samyag-scheduler.git
git add -A && git commit -m "Free tier limits: configurable via env vars (default 31 posts/month, 10 channels)"
git push -u origin main
```

(`pricing-fork.patch` and the commit hash above both live in this same
folder — see `CHANGES.md` for what the patch contains.)

## Step 2 — Get a server

Self-hosting this needs more than a tiny free server — Postiz's scheduling
engine runs on Temporal, which itself needs Elasticsearch, so realistically
you want at least 4GB of RAM.

**Recommended: Oracle Cloud "Always Free" tier.** Genuinely free forever, and
gives you an ARM server with up to 24GB RAM. Sign up at oracle.com/cloud/free
— I'll walk you through picking the right shape (Ampere A1, 4 OCPUs/24GB)
when you get there.

**YOU DO THIS:** create the Oracle account and server instance (a credit
card is required for identity verification only — the Always Free tier
itself never charges). Once it's running, I'll help you connect to it and
install Docker.

## Step 3 — Point Cloudflare at the server (free HTTPS, no extra software)

Since your website already lives on Cloudflare, we're reusing exactly that
— no separate "Tunnel" product, just a normal DNS record.

**YOU DO THIS:**
1. In Cloudflare DNS (same dashboard you already use for samyagsynergy.com),
   add an **A record** for your chosen subdomain (e.g.
   `schedule.samyagsynergy.com`) pointing at your Oracle server's IP address
2. Make sure the little cloud icon next to it is **orange** ("Proxied") —
   this is what gives you free HTTPS automatically
3. On the Oracle server's firewall, open port 80 (and only port 80 — nothing
   else needs to be reachable from outside)

That's the entire step — no dashboard beyond the one you already use.

## Step 4 — Register a developer app on each platform

Only needed once you're ready to test with real connections — even for
personal use, you'll need this to connect your own Instagram/LinkedIn/etc.
account. **YOU DO THIS** for each (I can walk through any of these live):

| Platform | Where to register | Typical wait |
|---|---|---|
| LinkedIn | linkedin.com/developers → Create App → apply for "Share on LinkedIn" + "Sign In" products | Days, sometimes instant |
| Facebook + Instagram | developers.facebook.com → Create App → "Business" type → add Instagram Graph API + Pages API | 1–4 weeks for public review — but your own account can connect immediately, unreviewed |
| YouTube | console.cloud.google.com → new project → enable "YouTube Data API v3" → create OAuth credentials | Usually fast for basic scopes |

Each one gives you a Client ID/Secret — these go into the matching spots in
`docker-compose.yaml`.

## Step 5 — Set the domain in your config

**YOU DO THIS:** update `MAIN_URL`, `FRONTEND_URL`, and
`NEXT_PUBLIC_BACKEND_URL` in `docker-compose.yaml` to the subdomain you set
up in Step 3 (e.g. `https://schedule.samyagsynergy.com`).

## Step 6 — Build and launch

Once steps 1–5 are done, this is the part I can walk through with you
command-by-command over a terminal session on your server:

```bash
git clone <your-fork-url>
cd samyag-scheduler-source
docker build --target dist -t <your-dockerhub-or-ghcr>/samyag-scheduler:latest -f Dockerfile.dev .
docker compose up -d
```

Because `DISABLE_REGISTRATION: 'true'` is already set, the very first account
you create will be the only one allowed to register — this is your personal
testing phase (see `SECURITY.md`). Sign up once with your own email, connect
your own social accounts, and try it out.

## Step 7 — When you're ready to open it to others

1. Make sure Facebook/Instagram's app review has come back approved
2. Have `PRIVACY_POLICY_TEMPLATE.md` reviewed and published on the site
3. Flip `DISABLE_REGISTRATION` to `'false'` in `docker-compose.yaml` and
   restart

## Step 8 — Put up the branded landing page

**YOU DO THIS:** follow `landing-page/README.md` — it's a separate free
Cloudflare Pages upload, not something that touches the Oracle server at all.

---

## What's still to come after this

- A short "how it works" walkthrough for first-time users, since they won't
  know this is Postiz under the hood.

Let me know once you've got a GitHub account and an Oracle Cloud account
started, and we'll go step by step from there.
