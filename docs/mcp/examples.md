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
> - **Post ID:** pg_abc123

### Schedule with Specific Timezone

> **You:** Schedule "Join our webinar!" to LinkedIn for March 1st at 2pm Pacific time

> **Claude:** Scheduled! The post will publish on March 1, 2026 at 2:00 PM PST (10:00 PM UTC).
>
> Post ID: pg_def456

### Post to Multiple Platforms

> **You:** Post "We're hiring senior engineers! Check our careers page" to Twitter, LinkedIn, and Bluesky

> **Claude:** I'll post that to all three platforms now.
>
> Published successfully:
> - **Twitter:** https://twitter.com/yourcompany/status/...
> - **LinkedIn:** https://linkedin.com/feed/update/...
> - **Bluesky:** https://bsky.app/profile/...

### Create a Draft for Later

> **You:** Create a draft post for Instagram: "Behind the scenes at our office today"

> **Claude:** Draft created! You can edit it in the Publora dashboard or ask me to schedule it when ready.
>
> Draft ID: pg_xyz789

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
> Post ID: pg_thread123

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

## Analytics and Insights

### Check Single Post Performance

> **You:** How did my last LinkedIn post perform?

> **Claude:** Your post from February 15 received:
> - **1,250** impressions
> - **680** unique viewers
> - **28** reactions (likes, celebrates, etc.)
> - **5** comments
> - **3** reshares
> - **2.9%** engagement rate

### Weekly Performance Summary

> **You:** Give me a summary of my LinkedIn performance this week

> **Claude:** This week's LinkedIn metrics:
>
> **Posts:** 5 published
> **Total impressions:** 4,230
> **Total reactions:** 89
> **Comments:** 12
> **Reshares:** 7
>
> **Best performer:** "Product launch announcement" (1,850 impressions)
> **Follower growth:** +47 new followers

### Compare Post Performance

> **You:** Which of my posts got the most engagement this month?

> **Claude:** Top 3 posts by engagement rate:
>
> 1. "Announcing our Series A" - 3.4% engagement, 2,100 impressions
> 2. "Meet our new CTO" - 2.8% engagement, 1,890 impressions
> 3. "Product roadmap 2026" - 2.5% engagement, 1,650 impressions

### Follower Growth

> **You:** How many LinkedIn followers do I have and how's the growth?

> **Claude:** Your LinkedIn page has **12,847** followers.
>
> Growth breakdown:
> - **This week:** +47 followers
> - **This month:** +234 followers (+1.9%)
> - **Last 90 days:** +687 followers (+5.6%)

### Profile Overview

> **You:** Give me an overall summary of my LinkedIn presence

> **Claude:** LinkedIn Profile Summary:
>
> **Profile:** Your Company Page
> **Followers:** 12,847
>
> **Last 30 days:**
> - Posts published: 18
> - Total impressions: 15,400
> - Average engagement: 2.3%
> - Best day: Tuesdays at 10am
> - New followers: +234
>
> **Top performing content types:**
> 1. Announcements (3.1% avg engagement)
> 2. Behind-the-scenes (2.8% avg engagement)
> 3. Industry insights (2.4% avg engagement)

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

### End of Day Review

> **You:** How did today's posts perform?

> **Claude:** Today's performance summary:
>
> - **LinkedIn (9am):** 450 impressions, 23 reactions
> - **Twitter (12pm):** 1,200 views, 45 votes on poll
> - **All platforms (3pm):** 890 total impressions
>
> Best engagement: Twitter poll with 3.8% engagement rate.

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

> **You:** Create 5 draft posts about our new feature for different platforms

> **Claude:** I've created 5 draft posts:
>
> 1. **LinkedIn (long-form):** "We're excited to announce a major update that will change how you..."
> 2. **Twitter (concise):** "Big news! Our new feature is here. Here's what it means for you..."
> 3. **Bluesky (casual):** "Hey! We just shipped something cool..."
> 4. **Instagram (visual focus):** "Swipe to see our latest feature in action..."
> 5. **Threads (conversational):** "Want to know what we've been working on? Let me tell you..."
>
> All saved as drafts. Ready to schedule when you approve them!

---

## Team and Workspace Management

### View Team Members

> **You:** Who's on my social media team?

> **Claude:** Your workspace has 3 members:
>
> 1. **you@company.com** - Admin
> 2. **marketing@company.com** - Member
> 3. **social@company.com** - Member

### Add a Team Member

> **You:** Add newperson@company.com to my workspace

> **Claude:** Done! Invitation sent to newperson@company.com.
>
> They'll receive an email to join your workspace. Once accepted, they can:
> - View scheduled posts
> - Create and schedule content
> - Access analytics

### Remove a Team Member

> **You:** Remove the user social@company.com from my workspace

> **Claude:** User removed from workspace. They no longer have access to:
> - Scheduled posts
> - Connected accounts
> - Analytics data
>
> Their past activity remains in the audit log.

---

## Advanced Workflows

### Content Repurposing

> **You:** Take my LinkedIn post pg_abc123 and adapt it for Twitter

> **Claude:** Here's your LinkedIn post:
> "We're thrilled to announce that our team has grown to 50 people! This milestone represents years of hard work and dedication. Thank you to everyone who believed in us from the start..."
>
> Adapted for Twitter (280 chars):
> "Big milestone: Our team just hit 50 people! Years of hard work led us here. Thank you to everyone who believed in us from day one."
>
> Want me to schedule this to Twitter?

### Analytics-Driven Scheduling

> **You:** When's the best time to post on LinkedIn based on my analytics?

> **Claude:** Based on your last 90 days of LinkedIn posts:
>
> **Best times for engagement:**
> 1. Tuesday 10am - 3.2% avg engagement
> 2. Wednesday 9am - 2.9% avg engagement
> 3. Thursday 2pm - 2.7% avg engagement
>
> **Best days overall:**
> - Tuesday: Highest impressions
> - Thursday: Most comments
>
> Want me to reschedule your upcoming posts to these optimal times?

### Campaign Tracking

> **You:** Show me all posts from my product launch campaign last week

> **Claude:** Product launch campaign (Feb 12-16):
>
> | Date | Platform | Content | Impressions | Engagement |
> |------|----------|---------|-------------|------------|
> | Feb 12 | LinkedIn | "Exciting news coming..." | 890 | 2.1% |
> | Feb 13 | Twitter | "Tomorrow is the day!" | 1,200 | 1.8% |
> | Feb 14 | All | "Introducing..." | 3,400 | 3.4% |
> | Feb 15 | LinkedIn | "Deep dive into..." | 1,100 | 2.6% |
> | Feb 16 | Twitter | "FAQ thread..." | 780 | 2.0% |
>
> **Campaign totals:**
> - Total impressions: 7,370
> - Average engagement: 2.4%
> - New followers: +156

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
Good: "Delete post pg_abc123"
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
