# Zapier Integration

Publora has a native Zapier app, so anything that can start a Zap can also publish to your social accounts. A new row in a spreadsheet, a fresh item in an RSS feed, a message in Slack. Any of them can turn into a scheduled post across your connected platforms, without writing a line of code.

![The Publora app page on Zapier, under the Social Media Marketing category](../../images/zapier/zapier-a-app-page.png)

## Before you start

- A Publora account with at least one connected social account.
- A Zapier account. The free plan is enough for single-step Zaps.

## Step 1 — Get your Publora API key

In the Publora dashboard, open **API** in the left sidebar (or go straight to [app.publora.com/dashboard/api](https://app.publora.com/dashboard/api)) and click **+ Create New Key**. Copy it right away; the full key is shown once, and the list afterwards only keeps a short prefix so you can tell your keys apart.

![The API page in the Publora dashboard, with the API Keys list and the Create New Key button](../../images/zapier/zapier-b-api-key.png)

## Step 2 — Open the Publora app on Zapier

Go to [zapier.com/apps/publora/integrations](https://zapier.com/apps/publora/integrations) and click **Connect Publora to 9,000+ apps**, or start a Zap from scratch and search for **Publora** in the editor.

## Step 3 — Connect your account

When Zapier asks to connect Publora, paste the key into the **Publora API Key** field and click **Connect**. Zapier checks it by fetching your connected accounts, so a wrong key fails immediately instead of later.

![The Zapier connection window with the Publora API Key field](../../images/zapier/zapier-c-connect.png)

Once it goes through, the account shows up in the step and can be reused by every other Zap you build. You can manage the connection any time at [zapier.com/app/connections](https://zapier.com/app/connections).

![A connected Publora account in the Zap editor, in use by four Zap workflows](../../images/zapier/zapier-d-connected.png)

## Step 4 — What you can build

**Triggers** (Publora starts the Zap). All four are instant: Publora pushes the event to Zapier the moment it happens, instead of Zapier polling for changes on a schedule.

- **New Published Post** — a post was published to a connected account
- **New Scheduled Post** — a post was scheduled for publishing
- **New Failed Post** — a post failed to publish to a connected account
- **Post Returned to Draft** — a scheduled post went back to draft and won't publish until it's scheduled again

**Actions** (Publora does something):

- **Create Post** — schedules a post, saves a draft, or publishes right away
- **Update Post** — changes a draft or scheduled post's text, accounts, timing, media, or settings
- **Delete Post** — deletes a post
- **Create LinkedIn Comment** — adds a comment to a LinkedIn post
- **Add LinkedIn Reaction** — adds your reaction to a LinkedIn post
- **Reshare LinkedIn Post** — reshares a post to your profile or Company Page

The LinkedIn actions take the post's URN, which **New Published Post** provides as Posted ID.

**Searches:**

- **Find Connected Account** — looks up a connected social account
- **Find Posts** — finds posts by status, platform, or scheduled date

## Example: publish new blog posts automatically

Wire up **RSS by Zapier → New Item in Feed** as the trigger, then **Publora → Create Post** as the action. Picking the event takes one click:

![Choosing a Publora trigger event in the Zap editor](../../images/zapier/zapier-g-trigger-event.png)

Create Post starts with Content, Platforms, Schedule Time, Media URLs and Publish Now. The optional fields sit below them:

![The Create Post form showing the Media URLs, Publish Now and First Comment fields](../../images/zapier/zapier-h-create-post.png)

Pick your accounts under **Platforms**. The dropdown lists the accounts you've connected in Publora, so there are no IDs to copy by hand. Map the feed's title and link into **Content**, then leave **Schedule Time** empty to create a draft, set a time to schedule it, or turn on **Publish Now** to post right away. Test the step, then publish the Zap. Every new post in the feed now becomes a post on your networks.

**First Comment 1 to 3** add up to three comments under your own post on LinkedIn, X, Threads, Bluesky and Mastodon. **Platform Options** shows extra settings for the platforms you choose, such as who can view a TikTok, a YouTube title, or who can reply on Threads.

## Example: get alerted when something publishes

Use **Publora → New Scheduled Post** or **New Published Post** as the trigger, then send it wherever you keep an eye on things: **Email by Zapier** for a notification, **Google Sheets → Create Row** for a running log, or **Slack → Send Channel Message** for a ping.

To catch problems, use **New Failed Post** or **Post Returned to Draft** the same way, so a post that didn't go out reaches you right away.

![A live two-step Zap: New Scheduled Post in Publora followed by Send Outbound Email](../../images/zapier/zapier-i-zap-live.png)

## More ideas

- New row in Google Sheets → Create Post, scheduled from a column
- New item in a Notion database → Create Post
- New WordPress post → Create Post, announcing it on your networks
- New Airtable record with status Approved → Create Post
- Schedule by Zapier, weekly → Create Post for a recurring digest

## Tips and troubleshooting

- Leaving **Schedule Time** empty creates a draft unless **Publish Now** is on.
- **Publish Now** can't be combined with a future **Schedule Time**.
- Instagram, TikTok, and YouTube require media. Pass public `https` URLs in **Media URLs**.
- In Update Post, **Media URLs** adds to the media already on the post.
- You can't update a post that has already published. Deleting a published post removes it from Publora, but not from the platforms it already went out to.
- Times are ISO 8601, in UTC, and **Schedule Time** must be in the future. Zaps can queue behind rate limits — if the time you mapped has already passed when the step runs, Publora clamps a small delay to "now" (returning a `SCHEDULED_TIME_COERCED` warning) and rejects anything five or more minutes stale. Add a few minutes of buffer.

## Calling the API directly

The Zapier app covers scheduling, editing, and publishing events. If you need an endpoint it doesn't expose, you can still call the Publora API from a **Webhooks by Zapier** step: send a POST to `https://api.publora.com/api/v1/create-post` with your key in the `x-publora-key` header. See [Authentication](../../authentication.md) for the key setup and [Core Workflows](../curl/all-endpoints.md) for every endpoint in curl form.
