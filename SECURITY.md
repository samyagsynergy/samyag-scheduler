# Security notes

Everything in this file reflects things actually checked in Postiz's own
source code, not assumptions — see the file/line references.

## What this setup already handles

- **One port open, fronted by Cloudflare.** Port 80 is open on the server,
  but it sits behind Cloudflare's proxy (the same orange-cloud DNS you
  already use for your main site) — Cloudflare gives free automatic HTTPS
  and hides your server's IP from casual visitors. This is the same setup
  most small self-hosted sites use; it's one port, not "wide open."
- **Personal-use lock, verified in code.** `DISABLE_REGISTRATION: 'true'`
  makes Postiz check `apps/backend/src/services/auth/auth.service.ts`,
  function `canRegister`: when this flag is on, only the very first account
  ever created is allowed to register — every signup after that is refused.
  This means only you can create an account while it's set this way, even if
  someone found the URL.
- **Real random secrets, not placeholders.** `JWT_SECRET` and both database
  passwords were generated with `openssl rand`, not left as guessable text.
- **The Stripe keys are genuinely inert.** Confirmed in
  `libraries/nestjs-libraries/src/services/stripe.service.ts`: `const stripe
  = new Stripe(process.env.STRIPE_SECRET_KEY || 'sk_nothing')`. Postiz's own
  developers fall back to a fake key if none is given — it only contacts
  Stripe's servers if someone actually tries to check out, which this
  product never offers. No account, no risk of any charge.

## What you (or I, guiding you) still need to do

- **Keep the server's operating system updated** — this is the one ongoing
  task that doesn't have a "set once and forget" version. A monthly check-in
  is a reasonable minimum.
- **Turn on Oracle Cloud's firewall properly** — even though no ports are
  published in Docker, the underlying server should still only allow SSH
  access (and ideally only from your own IP, not "anywhere").
- **Back up the database volume periodically**, once real content is in
  there, so a server problem doesn't mean losing everyone's scheduled posts.
- **When you're ready to open registration to others** (`DISABLE_REGISTRATION:
  'false'`), have the privacy policy (see `PRIVACY_POLICY_TEMPLATE.md`) live
  on the site first, not after.
- **Don't read or export user drafts out of curiosity**, even though the
  database's structure makes it technically possible for whoever runs the
  server — worth stating this as a policy to yourself and, eventually, in
  writing to users.

## What no setup can fully remove

- **You are still the one holding user's platform access tokens** once
  registration opens to others. Good security practice reduces this risk;
  it can't be brought to zero. This is true of every scheduling tool that
  exists, not a weakness specific to this one.
