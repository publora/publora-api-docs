# Manus Integration

Connect [Manus](https://manus.im) (autonomous web agent) to Publora and let it schedule and publish across your social platforms.

Publora ships as a **published connector in Manus' own directory** — there is no config file to edit and nothing to install.

## Prerequisites

- A Manus account, signed in
- A Publora account with an API key (starts with `sk_`) — create one on the **API** page in your dashboard
- At least one social account connected in Publora

> **Do this on a desktop browser.** The setup asks you to move an API key from Publora into Manus. On a phone the key often does not survive the switch between apps, which is the most common reason the connection is never finished.

## Connect

**1. Copy your API key first.** Manus asks for it the moment you press **Connect**, so having it on the clipboard saves a trip back.

**2. Open the Publora connector in Manus.**

- Direct link: [Publora in Manus](https://manus.im/app#connectors/built-in/connector_57244593-6fea-42e4-9368-3a73a06fc8df) (sign in to Manus first — the link only resolves for a signed-in session)
- Or manually: **Settings → Connectors → Browse Connectors**, open the **Apps** tab and search for *Publora*, then press **+**

**3. Authorize.** Press **Connect**, paste the key, then press **Connect** again.

Paste the **bare key** (`sk_...`) — do not prefix it with `Bearer`. The field's placeholder may show `Bearer sk_YOUR_API_KEY`, but the connector adds the scheme itself.

> Some Manus builds offer Publora's **OAuth** sign-in instead of a key field. If a Publora page opens saying *"An application is requesting access to your Publora account"*, just press **Approve** — no key is needed, and Publora mints a dedicated key for the connector that you can revoke on the **API** page.

**4. Verify.** The **Connect** button turns into **Try it out** once Manus accepts the credential.

## Using with Manus

Talk to Manus normally — it calls Publora's MCP tools on its own:

```text
"Check my Publora connection and list any channels you find."
"Draft a LinkedIn post about our launch and schedule it for tomorrow 9am UTC."
"What Publora posts are scheduled this week?"
"Post this update to LinkedIn and Threads at the same time."
```

Ask for the channel list first: Publora identifies targets by `platformId` (for example `linkedin-abc123`), and Manus needs to read those before it can schedule anything.

## Troubleshooting

**The direct link does nothing.** You are signed out of Manus. Sign in, then reopen the link — or reach the connector through **Settings → Connectors → Browse Connectors → Apps**.

**Publora is not in the composer's connector dropdown.** The dropdown lists only connectors you have already added. Add Publora from **Browse Connectors** first.

**"Connect" fails or the tools return 401.** The key was pasted with a `Bearer ` prefix, has a stray space, or was truncated by the copy. Copy it again from the **API** page and retry; if the key was created a while ago and you cannot see its value any more, create a new one.

**The key is gone before you can paste it.** Publora shows a key's value once. Create a fresh key on the **API** page, keep the tab open, and paste it into Manus in the same sitting.

## Next Steps

- [Tools Reference](./tools-reference.md) — all 14 MCP tools
- [Client Setup](./client-setup.md) — other MCP clients
- [Examples](./examples.md) — more conversation examples
- [Troubleshooting](./troubleshooting.md) — server-level issues
