# Connecting channels — credentials per provider

Each official-API channel needs ITS OWN developer app. One app's keys serve **many
connected accounts** of that channel (Postiz allows multiple `Integration`s per provider).
Put the keys in the **gitignored** `overlays/webrobot/secret.yaml`, then `kubectl apply -k`
+ rollout restart.

**Prerequisite:** `postiz.webrobot.eu` must resolve + serve HTTPS — every OAuth app
registers a **redirect/callback URL** of the form:

```
https://postiz.webrobot.eu/integrations/social/<provider>
```

## Easy (start here — no app review)
| Channel | Where to get it | Redirect URL | Env vars |
|---|---|---|---|
| **Telegram** | Chat **@BotFather** → `/newbot` → copy the bot token | — (none) | `TELEGRAM_TOKEN` |
| **Bluesky** | Account → Settings → **App passwords** → add | — (entered in UI per account) | none (no global app) |
| **Mastodon** | Your instance → Preferences → **Development** → New application (scopes: read, write) | `…/integrations/social/mastodon` | `MASTODON_CLIENT_ID`, `MASTODON_CLIENT_SECRET`, `MASTODON_URL` (instance base URL) |

## Medium (create an OAuth app)
| Channel | Developer portal | Redirect URL | Env vars |
|---|---|---|---|
| **X / Twitter** | developer.x.com → Project + App, permissions **Read+Write** | `…/integrations/social/x` | `X_API_KEY`, `X_API_SECRET` |
| **LinkedIn** (+ pages) | linkedin.com/developers → app; products *Sign In with LinkedIn* + *Share on LinkedIn* | `…/integrations/social/linkedin` | `LINKEDIN_CLIENT_ID`, `LINKEDIN_CLIENT_SECRET` |
| **Reddit** | reddit.com/prefs/apps → create app type **web app** | `…/integrations/social/reddit` | `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET` |
| **Pinterest** | developers.pinterest.com → app | `…/integrations/social/pinterest` | `PINTEREST_CLIENT_ID`, `PINTEREST_CLIENT_SECRET` |
| **Discord** | discord.com/developers → app (OAuth2 + Bot) | `…/integrations/social/discord` | `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_BOT_TOKEN_ID` |
| **Slack** | api.slack.com/apps → app | `…/integrations/social/slack` | `SLACK_ID`, `SLACK_SECRET` |

## Heavy (need platform app review / verification)
| Channel | Developer portal | Redirect URL | Env vars |
|---|---|---|---|
| **Facebook + Instagram** | developers.facebook.com → Business app, *Facebook Login* (review: `pages_manage_posts`, `instagram_content_publish`) | `…/integrations/social/facebook` | `FACEBOOK_APP_ID`, `FACEBOOK_APP_SECRET` |
| **Instagram (standalone)** | developers.facebook.com (Instagram API) | `…/integrations/social/instagram` | `INSTAGRAM_APP_ID`, `INSTAGRAM_APP_SECRET` |
| **Threads** | developers.facebook.com (Threads API) | `…/integrations/social/threads` | `THREADS_APP_ID`, `THREADS_APP_SECRET` |
| **TikTok** | developers.tiktok.com → app, **Content Posting API** | `…/integrations/social/tiktok` | `TIKTOK_CLIENT_ID`, `TIKTOK_CLIENT_SECRET` |
| **YouTube** | console.cloud.google.com → OAuth client, **YouTube Data API v3** | `…/integrations/social/youtube` | `YOUTUBE_CLIENT_ID`, `YOUTUBE_CLIENT_SECRET` |

> Channels with **no/blocked API** (or to skip all of this): use the planned WebRobot
> **agentic-browser provider** (Axis B in `agentic-backend/POSTIZ_WEBROBOT_INTEGRATION.md`)
> — login once via the HITL mirror, no per-channel OAuth app.

## How to apply keys
1. Add the env vars under `stringData:` in `overlays/webrobot/secret.yaml` (gitignored).
2. `kubectl apply -k infra/postiz/overlays/webrobot`
3. `kubectl -n nuvolaris rollout restart deploy/postiz`
