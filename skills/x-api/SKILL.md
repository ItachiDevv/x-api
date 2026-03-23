---
name: x-api
description: >
  X (Twitter) API v2 — posting tweets (text, image, video), searching, managing users,
  direct messages, streaming, media upload, and production patterns for scheduled posting
  with retry logic, rate limit handling, and auth failure recovery.
  Use when integrating with X/Twitter, posting content, building bots, or scheduling tweets.
metadata:
  author: ItachiDevv
  version: "2.0"
  tags: x, twitter, api, posting, scheduling, oauth, media-upload
---

# X (Twitter) API — Posts, Users, Search & Streaming

Expert-level knowledge of the X API v2 for posting tweets, searching, managing users, direct messages, and streaming. Includes battle-tested production patterns for scheduled posting with retry logic.

## Authentication

### OAuth 2.0 Bearer Token (App-only)
```bash
# Read-only access to public data
export X_BEARER_TOKEN="AAAA..."
curl -H "Authorization: Bearer $X_BEARER_TOKEN" \
  "https://api.x.com/2/tweets/1234567890"
```

### OAuth 1.0a (User Context)
```bash
# Read + write access on behalf of a user
export X_API_KEY="..."
export X_API_SECRET="..."
export X_ACCESS_TOKEN="..."
export X_ACCESS_TOKEN_SECRET="..."
```

### OAuth 2.0 PKCE (User Context, modern)
- Authorization Code Flow with PKCE
- Scopes: `tweet.read`, `tweet.write`, `users.read`, `dm.read`, `dm.write`, `offline.access`

## API Tiers & Rate Limits

| Tier | Price | Key Capabilities |
|------|-------|------------------|
| Free | $0 | Post tweets (1,500/mo), 1 app, basic read |
| Basic | $100/mo | 10K reads/mo, 3K posts/mo, 1 app |
| Pro | $5,000/mo | 1M reads/mo, 300K posts/mo, 3 apps, full-archive search |

## Posts (Tweets)

### Create Tweet
```bash
POST https://api.x.com/2/tweets
Authorization: OAuth 1.0a or OAuth 2.0 PKCE (tweet.write)

# Simple text tweet
{ "text": "Hello from the API!" }

# Reply to tweet
{ "text": "Reply text", "reply": { "in_reply_to_tweet_id": "1234567890" } }

# Quote tweet
{ "text": "Check this out", "quote_tweet_id": "1234567890" }

# Tweet with media
{ "text": "Photo!", "media": { "media_ids": ["1234567890"] } }

# Tweet with poll
{
  "text": "Favorite color?",
  "poll": { "options": ["Red", "Blue", "Green"], "duration_minutes": 1440 }
}
```

### Delete Tweet
```bash
DELETE https://api.x.com/2/tweets/{id}
```

### Lookup Tweets
```bash
# Single tweet
GET https://api.x.com/2/tweets/{id}?tweet.fields=created_at,public_metrics,author_id

# Multiple tweets
GET https://api.x.com/2/tweets?ids=1234,5678&tweet.fields=created_at,public_metrics

# Tweet fields: id, text, created_at, author_id, public_metrics, entities, attachments
# Expansions: author_id, referenced_tweets.id, attachments.media_keys
```

### Like / Unlike
```bash
POST https://api.x.com/2/users/{user_id}/likes
{ "tweet_id": "1234567890" }

DELETE https://api.x.com/2/users/{user_id}/likes/{tweet_id}
```

### Retweet / Undo
```bash
POST https://api.x.com/2/users/{user_id}/retweets
{ "tweet_id": "1234567890" }

DELETE https://api.x.com/2/users/{user_id}/retweets/{tweet_id}
```

### Bookmark
```bash
POST https://api.x.com/2/users/{user_id}/bookmarks
{ "tweet_id": "1234567890" }

GET https://api.x.com/2/users/{user_id}/bookmarks
```

## Users

### Lookup
```bash
# By ID
GET https://api.x.com/2/users/{id}?user.fields=name,username,description,public_metrics,profile_image_url

# By username
GET https://api.x.com/2/users/by/username/{username}

# Multiple IDs
GET https://api.x.com/2/users?ids=1234,5678

# Authenticated user
GET https://api.x.com/2/users/me
```

### Followers / Following
```bash
GET https://api.x.com/2/users/{id}/followers?max_results=100
GET https://api.x.com/2/users/{id}/following?max_results=100

# Follow user
POST https://api.x.com/2/users/{user_id}/following
{ "target_user_id": "1234567890" }

# Unfollow
DELETE https://api.x.com/2/users/{user_id}/following/{target_user_id}
```

### Block / Mute
```bash
POST https://api.x.com/2/users/{user_id}/blocking
{ "target_user_id": "1234567890" }

POST https://api.x.com/2/users/{user_id}/muting
{ "target_user_id": "1234567890" }
```

## Timelines

```bash
# User tweets
GET https://api.x.com/2/users/{id}/tweets?max_results=10&tweet.fields=created_at,public_metrics

# User mentions
GET https://api.x.com/2/users/{id}/mentions?max_results=10

# Reverse chronological (home timeline, requires user auth)
GET https://api.x.com/2/users/{id}/reverse_chronological
```

## Search

```bash
# Recent search (last 7 days)
GET https://api.x.com/2/tweets/search/recent?query=from:elonmusk&max_results=10&tweet.fields=created_at,public_metrics

# Full-archive search (Pro tier only)
GET https://api.x.com/2/tweets/search/all?query=...

# Count tweets matching query
GET https://api.x.com/2/tweets/counts/recent?query=...
```

### Query Operators
```
keyword               # Contains keyword
"exact phrase"        # Exact match
from:username         # From user
to:username           # To user
@username             # Mentioning user
#hashtag              # Contains hashtag
url:"example.com"     # Contains URL
has:media             # Has media attachment
has:images            # Has images
has:videos            # Has videos
has:links             # Has links
is:retweet            # Is a retweet
is:reply              # Is a reply
is:quote              # Is a quote tweet
lang:en               # Language
-is:retweet           # NOT a retweet (exclude)
(cat OR dog)          # Boolean OR
```

## Direct Messages

```bash
# Send DM
POST https://api.x.com/2/dm_conversations/with/{participant_id}/messages
{ "text": "Hello!" }

# Send DM to group
POST https://api.x.com/2/dm_conversations
{
  "conversation_type": "Group",
  "participant_ids": ["user1", "user2"],
  "message": { "text": "Group chat" }
}

# Get DM events
GET https://api.x.com/2/dm_events?dm_event.fields=created_at,text,sender_id

# Get conversations
GET https://api.x.com/2/dm_conversations/{id}/dm_events
```

## Media Upload

```bash
# Step 1: Upload media (v1.1 endpoint, still used)
POST https://upload.x.com/1.1/media/upload.json
Content-Type: multipart/form-data
media_data=<base64-encoded>

# Step 2: Use media_id in tweet
POST https://api.x.com/2/tweets
{ "text": "Photo!", "media": { "media_ids": ["returned_media_id"] } }
```

## Filtered Stream (Real-time)

```bash
# Add rules
POST https://api.x.com/2/tweets/search/stream/rules
{
  "add": [
    { "value": "cat has:images", "tag": "cats with images" },
    { "value": "from:elonmusk", "tag": "elon tweets" }
  ]
}

# Get rules
GET https://api.x.com/2/tweets/search/stream/rules

# Connect to stream (SSE)
GET https://api.x.com/2/tweets/search/stream?tweet.fields=created_at,author_id

# Delete rules
POST https://api.x.com/2/tweets/search/stream/rules
{ "delete": { "ids": ["rule-id-1"] } }
```

## Pagination

All list endpoints use cursor-based pagination:
```bash
GET /2/users/{id}/followers?max_results=100&pagination_token=<next_token>

# Response includes:
{
  "data": [...],
  "meta": {
    "next_token": "abc123",   # Use as pagination_token
    "result_count": 100
  }
}
```

## Fields & Expansions

Customize response data:
```bash
# Tweet fields
tweet.fields=created_at,public_metrics,entities,author_id,conversation_id

# User fields
user.fields=name,username,description,public_metrics,profile_image_url,verified

# Expansions (include related objects)
expansions=author_id,referenced_tweets.id,attachments.media_keys

# Media fields (requires media expansion)
media.fields=url,preview_image_url,type,width,height
```

## Rate Limit Headers

Every response includes:
```
x-rate-limit-limit: 300          # Max requests in window
x-rate-limit-remaining: 299      # Remaining requests
x-rate-limit-reset: 1234567890   # Unix timestamp when window resets
```

Handle 429 responses:
```typescript
if (response.status === 429) {
  const resetTime = response.headers.get('x-rate-limit-reset');
  const waitMs = (parseInt(resetTime) * 1000) - Date.now();
  await new Promise(r => setTimeout(r, waitMs));
  // Retry request
}
```

## Production Node.js Patterns (twitter-api-v2)

### CRITICAL: Fresh Client Per Call

**NEVER reuse a long-lived TwitterApi instance.** OAuth 1.0a signatures include timestamps — stale clients cause sporadic 401s in long-running services. Build a fresh client per request:

```typescript
import { TwitterApi, EUploadMimeType, ApiResponseError } from 'twitter-api-v2';

function getClient(): TwitterApi {
  // Fresh client per call — avoids stale OAuth timestamps
  return new TwitterApi({
    appKey: process.env.X_API_KEY!,
    appSecret: process.env.X_API_SECRET!,
    accessToken: process.env.X_ACCESS_TOKEN!,
    accessSecret: process.env.X_ACCESS_TOKEN_SECRET!,
  });
}
```

### CRITICAL: Request Timeouts

Twitter API calls can hang indefinitely (especially media uploads). Always wrap with a timeout:

```typescript
const REQUEST_TIMEOUT_MS = 30_000;

function withTimeout<T>(promise: Promise<T>, ms: number, label: string): Promise<T> {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(
      () => reject(new Error(`Request timed out after ${ms}ms (${label})`)),
      ms
    );
    promise.then(
      (val) => { clearTimeout(timer); resolve(val); },
      (err) => { clearTimeout(timer); reject(err); }
    );
  });
}
```

### Retryable Errors & Backoff

401s from Twitter are often transient (clock skew, OAuth nonce reuse, infrastructure blips). Always retry them with a fresh client:

```typescript
function isRetryableError(err: unknown): boolean {
  if (err instanceof ApiResponseError) {
    // 401 = transient auth (retry with fresh client), 5xx = server errors
    return err.code === 401 || err.code === 500 || err.code === 502 || err.code === 503;
  }
  if (err instanceof Error) {
    const msg = err.message.toLowerCase();
    return msg.includes('econnreset') || msg.includes('etimedout') ||
           msg.includes('econnrefused') || msg.includes('request failed') ||
           msg.includes('socket hang up') || msg.includes('fetch failed');
  }
  return false;
}

async function withRetry<T>(
  context: string,
  fn: (twitter: TwitterApi) => Promise<T>
): Promise<T> {
  const maxAttempts = 4;
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const twitter = getClient(); // Fresh client each attempt
      return await withTimeout(fn(twitter), REQUEST_TIMEOUT_MS, context);
    } catch (err) {
      logTwitterError(`${context} (attempt ${attempt})`, err);
      if (attempt < maxAttempts && isRetryableError(err)) {
        const backoffMs = 1500 * Math.pow(2, attempt - 1); // 1.5s, 3s, 6s
        await new Promise(r => setTimeout(r, backoffMs));
        continue;
      }
      throw err;
    }
  }
  throw new Error(`${context}: exhausted retry attempts`);
}
```

### Posting Tweets (Text, Image, Video)

```typescript
// Text tweet
export async function postTweet(text: string): Promise<string> {
  const result = await withRetry('postTweet', (twitter) =>
    twitter.v2.tweet({ text })
  );
  return result.data.id;
}

// Image tweet (base64 input)
export async function postTweetWithImage(
  text: string, imageBase64: string, mimeType: string
): Promise<string> {
  const result = await withRetry('postTweetWithImage', async (twitter) => {
    const imageBuffer = Buffer.from(imageBase64, 'base64');
    const mediaId = await twitter.v1.uploadMedia(imageBuffer, {
      mimeType: mimeType as EUploadMimeType,
    });
    return twitter.v2.tweet({ text, media: { media_ids: [mediaId] } });
  });
  return result.data.id;
}

// Video tweet (Buffer input — twitter-api-v2 auto-chunks)
export async function postTweetWithVideo(
  text: string, videoBuffer: Buffer
): Promise<string> {
  const result = await withRetry('postTweetWithVideo', async (twitter) => {
    const mediaId = await twitter.v1.uploadMedia(videoBuffer, {
      mimeType: EUploadMimeType.Mp4,
    });
    return twitter.v2.tweet({ text, media: { media_ids: [mediaId] } });
  });
  return result.data.id;
}
```

### Error Logging for Debugging

Log full error context including rate limit headers and clock skew for 401 diagnosis:

```typescript
function logTwitterError(context: string, err: unknown): void {
  if (err instanceof ApiResponseError) {
    console.error(`[twitter] ${context} — HTTP ${err.code}`, {
      data: err.data,
      rateLimit: err.rateLimit,
    });
    if (err.code === 401) {
      // Log clock skew to diagnose OAuth timestamp drift
      const serverDate = (err.headers as Record<string, string>)?.date;
      const localTime = new Date();
      const clockSkewMs = serverDate
        ? localTime.getTime() - new Date(serverDate).getTime()
        : null;
      console.error('[twitter] 401 debug:', {
        localTime: localTime.toISOString(),
        twitterServerDate: serverDate || 'not in headers',
        clockSkewMs,
      });
    }
  } else {
    console.error(`[twitter] ${context}:`, err);
  }
}
```

## Scheduled Posting Architecture

For bots that post on a schedule, use this battle-tested pattern:

### Rate Limit Cooldown

```typescript
let rateLimitedUntil = 0;

// On 429: cool down until the longer of 1h or next midnight UTC (daily cap reset)
if (isRateLimited) {
  const nextMidnight = new Date();
  nextMidnight.setUTCDate(nextMidnight.getUTCDate() + 1);
  nextMidnight.setUTCHours(0, 10, 0, 0);
  rateLimitedUntil = Math.max(Date.now() + 3600_000, nextMidnight.getTime());
}

// Before posting, check cooldown
if (Date.now() < rateLimitedUntil) return; // skip
```

### Post Verification Loop

Tweets can silently fail (API returns 200 but tweet doesn't appear). Schedule verification checks:

```
Post attempt → Schedule verify at +3 min
  Verify #1: Posted? → Done. Failed? → Retry, schedule verify at +6 min
  Verify #2: Posted? → Done. Failed? → Retry, schedule verify at +9 min
  Verify #3: Posted? → Done. Failed? → Mark as failed permanently
```

Key: schedule next verification BEFORE retrying (crash-safe — if retry crashes, next check catches it).

### Auth Retry vs Rate Limit

Distinguish between retryable auth failures and rate limits:

```typescript
const isRateLimited = msg.includes('429') || msg.includes('rate limit') ||
                      msg.includes('too many requests') || msg.includes('usage cap');
const isAuthFailure = msg.includes('401') || msg.includes('unauthorized') ||
                      msg.includes('authenticate') || msg.includes('oauth');

// Auth failures: retry up to 6 times with 30s delays (withRetry handles first 4)
// Rate limits: stop ALL posting, defer remaining posts, cool down 1h+
// Other errors: mark as failed immediately
```

### Stuck Post Recovery (on startup)

After server restart, recover posts stuck in "posting" status (crashed mid-upload):

```typescript
// Reset posts stuck in "posting" for >5 min back to "approved"
// Short threshold (5 min not 0) avoids racing with an in-flight upload during fast redeploy
```

## Common Failure Modes

| Error | Cause | Fix |
|-------|-------|-----|
| Sporadic 401 | Stale OAuth client | Fresh `getClient()` per call |
| 401 clock skew | Server clock drift | Log clock skew, NTP sync |
| Hanging requests | Twitter API stalls | `withTimeout()` wrapper |
| 429 "usage cap" | Daily post limit hit | Cooldown until midnight UTC |
| 429 rate limit | Per-endpoint burst | Respect `x-rate-limit-reset` header |
| Silent post failure | 200 returned but no tweet | Post verification loop |
| ECONNRESET | Connection dropped | Retry with backoff |

## Env Vars Used

- `X_API_KEY` — App API key (consumer key)
- `X_API_SECRET` — App API secret (consumer secret)
- `X_ACCESS_TOKEN` — User access token (OAuth 1.0a)
- `X_ACCESS_TOKEN_SECRET` — User access token secret
- `X_BEARER_TOKEN` — App-only bearer token
