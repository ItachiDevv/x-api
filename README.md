# x-api

An [Agent Skill](https://agentskills.io) that teaches AI agents how to work with the X (Twitter) API v2.

## What's Included

- **Full API v2 reference** — Posts, users, search, timelines, DMs, streaming, bookmarks
- **Authentication** — OAuth 1.0a, OAuth 2.0 Bearer, OAuth 2.0 PKCE
- **Media upload** — Images (base64), videos (chunked), using v1.1 upload endpoint
- **Production posting patterns** — Battle-tested from real scheduled posting systems
- **Retry & error handling** — Fresh client per call, request timeouts, exponential backoff
- **Rate limit management** — Cooldown timers, daily cap resets, per-endpoint burst limits
- **Post verification** — 3-check chain to catch silent failures
- **Scheduling architecture** — Stuck post recovery, auth retry chains, rate limit deferral

## Install

```bash
npx skills add nirholas/x-api
```

## Key Production Lessons

1. **Always create a fresh `TwitterApi` client per request** — OAuth 1.0a timestamps go stale in long-running services, causing sporadic 401s
2. **Wrap all API calls with a timeout** — Twitter can hang indefinitely, especially media uploads
3. **401s are retryable** — Twitter returns 401 for transient reasons (clock skew, nonce reuse). Retry with fresh client + backoff.
4. **Rate limit cooldown should extend to midnight UTC** — Twitter's daily posting cap resets at midnight UTC, not after a fixed window
5. **Post verification is essential** — API can return 200 but the tweet doesn't appear. Schedule verification checks.

## License

MIT
