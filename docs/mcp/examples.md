# MCP Conversation Examples

Real-world examples of using Publora MCP with AI assistants like Claude, Cursor, and others.

## Getting Started

### First Connection Test

> **You:** Show my connected social media accounts

> **Claude:** Let me check your Publora connections.
>
> You have 3 connected accounts:
> 1. **Twitter/X** - @yourcompany (twitter-abc123)
> 2. **LinkedIn** - Your Company Page (linkedin-xyz789)
> 3. **Bluesky** - yourcompany.bsky.social (bluesky-def456)

---

## Scheduling Posts

### Schedule a Single Post

> **You:** Schedule a LinkedIn post for tomorrow at 9am saying "Excited to announce our new feature!"

> **Claude:** I'll schedule that post for you.
>
> Done! Post scheduled:
> - **Platform:** LinkedIn
> - **Time:** Tomorrow at 9:00 AM EST
> - **Content:** "Excited to announce our new feature!"
> - **Post ID:** 67a1b2c3d4e5f6a7b8c9d0e1

### Schedule with Specific Timezone

> **You:** Schedule "Join our webinar!" to LinkedIn for March 1st at 2pm Pacific time

> **Claude:** Scheduled! The post will publish on March 1, 2026 at 2:00 PM PST (10:00 PM UTC).
>
> Post ID: 67b2c3d4e5f6a7b8c9d0e1f2

### Post to Multiple Platforms

> **You:** Post "We're hiring senior engineers! Check our careers page" to Twitter, LinkedIn, and Bluesky right now

> **Claude:** I'll schedule that for immediate publishing. (To post immediately via MCP, I set `scheduledTime` a few seconds in the future. A time already in the past is clamped to server time and returns a `SCHEDULED_TIME_COERCED` warning -- and from 2026-08-25, times 5+ minutes stale are rejected with `SCHEDULED_TIME_IN_PAST`.)
>
> Published successfully:
> - **Twitter:** https://twitter.com/yourcompany/status/...
> - **LinkedIn:** https://linkedin.com/feed/update/...
> - **Bluesky:** https://bsky.app/profile/...

### Create a Draft for Later

> **You:** Create a draft post for Instagram: "Behind the scenes at our office today"

> **Claude:** I can't create a draft via MCP -- the `create_post` tool requires a `scheduledTime` (it's not optional). I'll need a specific date and time to schedule this post. When would you like it published?
>
> **Limitation:** The MCP `create_post` tool requires `scheduledTime` — it is marked as required in the MCP schema, even though the REST API accepts it as optional (omitting it in REST creates a draft). Drafts cannot be created via MCP; use the [REST API](/endpoints/create-post) directly or the Publora dashboard instead.

### Schedule a Thread

> **You:** Schedule a Twitter thread for tomorrow at 10am:
> 1. "We've been working on something big..."
> 2. "After months of development, we're ready to share..."
> 3. "Introducing our new AI-powered feature!"

> **Claude:** I'll create a scheduled thread with 3 tweets for tomorrow at 10am.
>
> Thread scheduled:
> - **Tweet 1:** "We've been working on something big..."
> - **Tweet 2:** "After months of development, we're ready to share..."
> - **Tweet 3:** "Introducing our new AI-powered feature!"
>
> Post ID: 67d4e5f6a7b8c9d0e1f2a3b4

---

## Managing Your Calendar

### View Scheduled Posts

> **You:** What do I have scheduled for next week?

> **Claude:** You have 5 posts scheduled for next week:
>
> 1. **Monday 9am** - LinkedIn: "Product update announcement..."
> 2. **Tuesday 2pm** - Twitter + LinkedIn: "New blog post about..."
> 3. **Wednesday 10am** - LinkedIn: "Case study: How..."
> 4. **Thursday 3pm** - Twitter: "Quick tip for..."
> 5. **Friday 9am** - All platforms: "Week in review..."

### Reschedule a Post

> **You:** Move Monday's LinkedIn post to Tuesday at 11am

> **Claude:** Done! Post rescheduled from Monday 9am to Tuesday 11am.

### Cancel a Specific Post

> **You:** Cancel the Thursday Twitter post

> **Claude:** Post deleted. Your remaining scheduled posts:
> 1. Monday 9am - LinkedIn
> 2. Tuesday 2pm - Twitter + LinkedIn
> 3. Wednesday 10am - LinkedIn
> 4. Friday 9am - All platforms

### Cancel All Drafts

> **You:** Delete all my draft posts

> **Claude:** Deleted 3 draft posts:
> - Draft about product launch
> - Draft about team event
> - Draft about holiday message
>
> Your scheduled posts remain unchanged.

### View Posts by Platform

> **You:** Show me all my scheduled LinkedIn posts

> **Claude:** You have 4 LinkedIn posts scheduled:
>
> 1. **Feb 20, 9am** - "Excited to share our Q1 results..."
> 2. **Feb 22, 10am** - "Meet our new team member..."
> 3. **Feb 25, 2pm** - "Product roadmap update..."
> 4. **Feb 28, 9am** - "Monthly newsletter recap..."

---

## Cross-Platform Workflows

### Morning Social Media Routine

> **You:** What's on my social media agenda today?

> **Claude:** Today's schedule (February 19):
>
> **9:00 AM** - LinkedIn: "Monday motivation post"
> **12:00 PM** - Twitter: "Lunch break poll"
> **3:00 PM** - All platforms: "Product tip of the day"
>
> You also have 2 draft posts ready for review.

### Weekly Planning

> **You:** Help me plan next week's social media content

> **Claude:** I'll help you plan! Looking at your best performing times:
>
> **Suggested schedule:**
> - **Monday 9am** - LinkedIn: Week kickoff / motivation
> - **Tuesday 10am** - LinkedIn: Product update or feature highlight
> - **Wednesday 2pm** - Twitter: Quick tip or poll
> - **Thursday 9am** - LinkedIn + Twitter: Blog post promotion
> - **Friday 3pm** - All platforms: Week recap / weekend message
>
> Want me to create draft posts for any of these slots?

### Batch Content Creation

> **Important:** The MCP `create_post` tool requires `scheduledTime` — it is marked as required in the MCP schema, even though the REST API accepts it as optional (omitting it in REST creates a draft). Drafts cannot be created via MCP; use the [REST API](/endpoints/create-post) directly or the Publora dashboard instead.

> **You:** Create 5 posts about our new feature for different platforms, scheduled hourly starting tomorrow at 9am

> **Claude:** I've created and scheduled 5 platform-tailored posts:
>
> 1. **LinkedIn (9:00 AM):** "We're excited to announce a major update that will change how you..."
> 2. **Twitter (10:00 AM):** "Big news! Our new feature is here. Here's what it means for you..."
> 3. **Bluesky (11:00 AM):** "Hey! We just shipped something cool..."
> 4. **Instagram (12:00 PM):** "Swipe to see our latest feature in action..."
> 5. **Threads (1:00 PM):** "Want to know what we've been working on? Let me tell you..."
>
> All 5 posts scheduled for tomorrow. You can reschedule or cancel any of them.

---

## LinkedIn Engagement

### Comment on a Post

> **You:** Comment "Great insights!" on this LinkedIn post urn:li:ugcPost:7429953213384187904

> **Claude:** Done! I've posted your comment:
>
> **Post:** Product announcement by Tech Company
> **Your comment:** "Great insights!"
> **Comment ID:** 7434695495614312448

### Reply to a Comment

> **You:** Reply to that comment with "Thanks for the detailed response!"

> **Claude:** Done! Your reply has been posted:
>
> **Parent comment:** Great insights!
> **Your reply:** "Thanks for the detailed response!"

### React to a Post

> **You:** Like this LinkedIn post urn:li:share:7123456789

> **Claude:** Done! I've added a LIKE reaction to the post.

### Add an Insightful Reaction

> **You:** React with "insightful" to my colleague's LinkedIn post

> **Claude:** Done! I've added an INTEREST (lightbulb) reaction to the post.
>
> Available reaction types:
> - LIKE (thumbs up)
> - PRAISE (clapping hands)
> - EMPATHY (heart)
> - INTEREST (lightbulb) ← used
> - APPRECIATION (hands together)
> - ENTERTAINMENT (laughing)

### Remove a Reaction

> **You:** Unlike that post

> **Claude:** Done! I've removed your reaction from the post.

### Delete a Comment

> **You:** Delete my comment from that post

> **Claude:** Comment deleted successfully.

### Mention a Person in a Post

> **You:** Schedule a LinkedIn post for tomorrow at 9am thanking John Smith (urn:li:person:ACoAABcD1234EfG) for his help

> **Claude:** Done! I've scheduled the post with the mention:
>
> **Content:** "Thanks to @John Smith for the amazing collaboration on this project!"
> **Scheduled:** Tomorrow at 9:00 AM
>
> The mention will appear as a clickable link to John's profile, and he'll receive a notification.

### Mention a Company in a Post

> **You:** Create a LinkedIn post announcing our partnership with Acme Corp (urn:li:organization:98765432)

> **Claude:** Done! Post created with the company mention:
>
> **Content:** "Excited to announce our partnership with @Acme Corp Inc! Together we're building the future of..."
>
> **Note:** I used the exact company name "Acme Corp Inc" to ensure the mention renders correctly.

### Multiple Mentions

> **You:** Post to LinkedIn: "Shoutout to @{urn:li:person:ACoAADeFgHi5678|Jane Doe} and the team at @{urn:li:organization:98765432|Tech Startup Inc} for the collaboration!"

> **Claude:** Posted! Both mentions will appear as clickable links:
> - Jane Doe → links to her profile
> - Tech Startup Inc → links to their company page

---

## LinkedIn Feed Retrieval (Coming Soon)

> **Status: NOT YET AVAILABLE** - These features require the `r_member_social` permission, which is **RESTRICTED** and requires LinkedIn approval. The implementation is ready and will be enabled once LinkedIn approves the permission for Publora.

### View Your Recent Posts

> **You:** Show my recent LinkedIn posts

> **Claude:** Here are your last 10 LinkedIn posts:
>
> 1. **Feb 28** - "Excited to announce our new feature!" (1,250 impressions)
> 2. **Feb 25** - "Behind the scenes at our office..." (890 impressions)
> 3. **Feb 22** - "Product roadmap update for Q1..." (2,100 impressions)
> 4. **Feb 20** - "Meet our newest team member..." (1,450 impressions)
> 5. **Feb 18** - "Weekly tips for content creators..." (780 impressions)
>
> Would you like details on any specific post?

### View Comments on a Post

> **You:** Show comments on my latest LinkedIn post

> **Claude:** Your post "Excited to announce our new feature!" has 12 comments:
>
> 1. **John Smith:** "This is amazing! Can't wait to try it out."
> 2. **Sarah Johnson:** "Great work team! When is the beta available?"
> 3. **Tech Company:** "Congratulations on the launch!"
> 4. **Mike Williams:** "How does this compare to the competition?"
> 5. *...and 8 more comments*
>
> Would you like me to reply to any of these?

### View Reactions on a Post

> **You:** Who reacted to my LinkedIn announcement?

> **Claude:** Your post has 156 reactions:
>
> **By type:**
> - 89 LIKE (👍)
> - 45 PRAISE (👏)
> - 15 INTEREST (💡)
> - 5 EMPATHY (❤️)
> - 2 APPRECIATION (🙏)
>
> **Recent reactors:**
> - Jane Doe (LIKE)
> - Acme Corp (PRAISE)
> - Tech Startup Inc (INTEREST)
> - *...and 153 more*

### Analyze Post Engagement

> **You:** Analyze the engagement on my last 5 LinkedIn posts

> **Claude:** Here's your engagement analysis:
>
> | Post | Date | Impressions | Reactions | Comments | Rate |
> |------|------|-------------|-----------|----------|------|
> | "New feature..." | Feb 28 | 1,250 | 28 | 12 | 3.2% |
> | "Behind scenes..." | Feb 25 | 890 | 45 | 8 | 6.0% |
> | "Roadmap update..." | Feb 22 | 2,100 | 62 | 23 | 4.0% |
> | "New team member..." | Feb 20 | 1,450 | 89 | 15 | 7.2% |
> | "Weekly tips..." | Feb 18 | 780 | 23 | 5 | 3.6% |
>
> **Best performer:** "Meet our newest team member" with 7.2% engagement.
> **Insight:** Personal/team content gets 2x more reactions than product updates.

---

## Advanced Workflows

### Content Repurposing

> **You:** Take my LinkedIn post 67a1b2c3d4e5f6a7b8c9d0e1 and adapt it for Twitter

> **Claude:** Here's your LinkedIn post:
> "We're thrilled to announce that our team has grown to 50 people! This milestone represents years of hard work and dedication. Thank you to everyone who believed in us from the start..."
>
> Adapted for Twitter (280 chars):
> "Big milestone: Our team just hit 50 people! Years of hard work led us here. Thank you to everyone who believed in us from day one."
>
> Want me to schedule this to Twitter?

### Quick Status Check

> **You:** Quick status: anything failed or pending?

> **Claude:** Status check:
>
> **Failed posts:** 0
> **Pending review:** 2 drafts
> **Scheduled (next 24h):** 3 posts
>
> All systems healthy!

---

## Tips for Better Conversations

### Be Specific with Dates

```text
Good: "Schedule for March 1st at 2pm EST"
Good: "Schedule for tomorrow at 9am"
Bad: "Schedule for sometime next week"
```

### Reference Posts Clearly

```text
Good: "Delete post 67a1b2c3d4e5f6a7b8c9d0e1"
Good: "Reschedule my 9am LinkedIn post"
Bad: "Delete that post" (which one?)
```

### Specify Platforms When Needed

```text
Good: "Post to LinkedIn and Twitter"
Good: "Show my LinkedIn analytics"
Bad: "Post this everywhere"
```

### Use Natural Time References

```text
Claude understands:
- "tomorrow at 9am"
- "next Monday at 2pm"
- "in 3 hours"
- "March 1st at 10am EST"
```
