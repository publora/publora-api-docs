# Grok Bot Integration

Connect [Grok Bot](https://x.ai/bot) to Publora and let it draft, schedule and publish your social media posts. You connect it once, sign in with your Publora account, and then ask in plain words: "Schedule this for Tuesday at 9am on LinkedIn and Bluesky."

## Prerequisites

- A Publora account with at least one social account connected in the Publora dashboard. Grok Bot posts to the accounts you connected there; it can't connect new ones.
- Grok Bot with an eligible plan: any paid individual Cursor plan or the Cursor Teams plan, a linked SuperGrok, SuperGrok Plus, SuperGrok Heavy or X Premium+ subscription, a seat on a Grok Business plan, or the Grok Bot free trial.

The free Publora Starter plan works: 3 social accounts and 15 posts a month on every network except X. Posting to X needs the Pro plan.

## Connect

**1. Ask the bot to add the server.** Open a chat with your bot and send:

```text
Add a custom MCP server called Publora at https://mcp.publora.com/mcp
```

**2. Authorize.** The bot adds the server and shows a **Publora** card with an **Authorize** button. Click it, then click **Continue**.

**3. Sign in to Publora.** Your browser opens Publora. Sign in and click **Approve**.

**4. Verify.** Back in Grok Bot, the card changes to **Publora · Added**, and the bot confirms that Publora is connected.

To see it later, open **Connect apps** in the sidebar and select the connected count next to the title. Publora appears among your installed plugins. An installed plugin is available to every bot on your account.

The sign-in token stays on Grok Bot's connector backend. The bot calls Publora's tools but never receives your password or token.

## Ask before publishing

Grok Bot may publish or delete posts without asking first. To keep the final say, open **Settings → General**, find **Bot**, and add an **Ask first** rule under **Custom Rules**, for example: *Ask first before publishing, scheduling or deleting posts in Publora.* You can also say it in the request: "Draft it and wait for my approval."

## Using with Grok Bot

Start with a request that only reads:

```text
"Which social accounts do I have connected to Publora?"
```

Then:

```text
"Write a LinkedIn post about our new pricing page and keep it as a draft."
"Schedule that post for Tuesday at 9:00 on LinkedIn and Bluesky."
"Add this photo and post it to Bluesky now."
"What is scheduled for next week?"
```

A post stays a draft when `scheduledTime` is omitted and `publishNow` is omitted or `false`. Setting `publishNow: true` publishes immediately, even without a scheduled time. To keep a post unpublished, explicitly ask the bot to save it as a draft.

To check that the bot can call Publora without reaching a real audience, ask it to post to `publora-playground`. Publora acknowledges the request without saving or publishing a post. This checks connectivity only; it does not verify real social-account access, quotas, scheduling, media processing or platform-specific publishing requirements.

## Troubleshooting

**A post to Instagram, TikTok or YouTube won't schedule.** Instagram and TikTok need an image or video. YouTube requires a video; an image alone won't work. Attach the required media and ask again.

**Posting to X fails on the free plan.** X is available on the Pro plan.

**The bot says Publora is missing or tools stop working.** Open **Connect apps** and find Publora. If it shows **Needs auth** or **Disconnected**, choose **Authenticate** and sign in again. If you deleted Grok Bot's key on the **API** page of your Publora dashboard, sign in again the same way.

**You'd rather use an API key.** Use the sign-in above instead. It keeps the token on Grok Bot's connector backend, away from the bot. Never paste a Publora API key into the chat or into the server address; if you already did, delete that key on the **API** page of your Publora dashboard and create a new one.

**`localhost` doesn't work.** Grok Bot works on a cloud computer, so `localhost` there is not your machine. Use `https://mcp.publora.com/mcp`.

Questions: support@publora.com
