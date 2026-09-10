# Changes from stock Postiz

This is a fork of [Postiz](https://github.com/gitroomhq/postiz-app) (AGPL-3.0
licensed). Postiz's own code is unchanged except for one setting:

**File:** `libraries/nestjs-libraries/src/database/prisma/subscriptions/pricing.ts`
**Change:** the `FREE` tier's `posts_per_month` and `channel` (number of social
accounts you can connect) now read from two environment variables —
`FREE_TIER_POSTS_PER_MONTH` and `FREE_TIER_CHANNELS` — instead of being
fixed numbers. They default to 31 and 10 if unset. See `pricing-fork.patch`
for the exact diff, and `docker-compose.yaml` for where the current values
live.

That's the entire modification — nothing else in Postiz was touched: same
dashboard, same scheduling engine, same posting logic, same everything.

**To change the free-tier limit later:** edit the two values in
`docker-compose.yaml` and run `docker compose up -d` — no rebuild, no
GitHub push required. This only needs a rebuild if you want to change
something Postiz's own settings don't expose as an environment variable.

## Platforms included

Four platforms are wired up: Instagram, Facebook, LinkedIn, YouTube — all
genuinely free to use at any volume. X (Twitter) was deliberately left out:
its free tier is limited and heavy usage requires a paid API tier from X
itself (not from Postiz or this fork), which didn't fit the "free forever"
goal. Nothing prevents adding X later if that trade-off changes.

## Why this exists

Samyag Synergy's version of this tool is free for everyone — there's no paid
tier. The 31 posts/month number is a fair-use ceiling (to control our own
hosting/API costs if usage grows), not a paywall. Nobody is asked to pay to
raise it.

## License note

Under AGPL-3.0, running a modified version of this software as a network
service requires making the corresponding modified source available to users
of that service. That's why this fork lives in a public GitHub repository —
see the link in the app's footer. If you're reading this because you're
auditing the fork: `pricing-fork.patch` is the complete, only diff against
upstream Postiz at the commit noted in `UPSTREAM_COMMIT.txt`.
