# N8N Publishing Workflow — orchidea.digital

Import `orchidea-publishing-workflow.json` into N8N to connect Airtable approval → 6 publishing destinations.

---

## What the Workflow Does

```
Airtable poll (every 5 min)
  → Approved records with no Published URL
  → Route by Platform (Switch node)
      ├── Blog     → Framer CMS API → create draft post
      ├── LinkedIn → If carousel: register PDF upload
      │              Else: create ugcPost (text / article / executive)
      ├── Twitter  → If thread: split on "1/ 2/ ..." numbering → post reply chain
      │              Else: single tweet
      ├── Instagram → create media container → publish
      ├── Facebook  → post to page feed
      └── Email     → ConvertKit create broadcast draft
  → Extract published URL from API response
  → Airtable write-back: Status = Published, Published URL = [url]
```

---

## Import Steps

1. In N8N, go to **Workflows → Import from file**
2. Upload `orchidea-publishing-workflow.json`
3. Complete the credentials setup below
4. Replace placeholder IDs (see section below)
5. Activate the workflow

---

## Credentials to Configure

Create each credential in **N8N → Settings → Credentials** before activating:

| Credential Name | Type | Where to get it |
|----------------|------|-----------------|
| `Airtable — orchidea.digital` | Airtable Token API | airtable.com → Account → Developer Hub → Personal Access Tokens |
| `Framer CMS API — orchidea.digital` | HTTP Bearer Auth | Framer → Project Settings → CMS → API Token |
| `LinkedIn OAuth2 — orchidea.digital` | OAuth2 | LinkedIn Developer App → Auth tab (needs `w_member_social`, `w_organization_social`) |
| `Twitter OAuth2 — orchidea.digital` | OAuth2 | developer.twitter.com → Your App → Keys and Tokens |
| `Instagram Graph API — orchidea.digital` | HTTP Query Auth | Meta for Developers → your app → Instagram Graph API → access token |
| `Facebook Pages API — orchidea.digital` | HTTP Query Auth | Meta for Developers → your app → Page access token |
| `ConvertKit API — orchidea.digital` | HTTP Query Auth | ConvertKit → Settings → Advanced → API Secret |

---

## Placeholder IDs to Replace

Search the JSON for these strings and replace with your actual IDs:

| Placeholder | Where to find your value |
|-------------|--------------------------|
| `FRAMER_COLLECTION_ID` | Framer Dashboard → CMS → Collections → click your blog collection → ID in URL |
| `LINKEDIN_ORG_ID` | linkedin.com/company/orchidea-digital → Admin view → numeric ID in URL |
| `INSTAGRAM_BUSINESS_ACCOUNT_ID` | Meta Business Suite → Settings → Instagram Account → Account ID |
| `FACEBOOK_PAGE_ID` | facebook.com/orchidea.digital → About → Page ID |

The Airtable base ID (`appFAjCzgu294q2i7`) is already pre-configured.

---

## Airtable Token Scopes Required

When creating the Airtable personal access token, enable:
- `data.records:read`
- `data.records:write`
- `schema.bases:read`

Scope it to the **Content Pipeline** base only.

---

## LinkedIn Carousel Notes

Carousel (document PDF) publishing is a 3-step process:

1. **Register upload** → LinkedIn returns an `uploadUrl` and `asset` URN
2. **Upload PDF binary** → PUT your PDF file to the `uploadUrl`
3. **Create ugcPost** → Reference the `asset` URN in the post body

The workflow handles Step 1. For Cycles 1–2, generate the PDF in Canva from the slide copy the LinkedIn Authority Builder agent produces, then manually upload it via the LinkedIn UI. Wire in Steps 2–3 once you have a reliable PDF generation step (e.g., HTML-to-PDF via Puppeteer in a Code node, or an external PDF generation API).

---

## ConvertKit Broadcast Behavior

The workflow creates broadcasts as **drafts** (`public: false`). To review before sending:
1. Go to ConvertKit → Broadcasts
2. Find the draft (named by subject line)
3. Preview and send manually

To flip to fully automated sending, change `"public": false` to `"public": true` and set `published_at` to the exact send time. The `published_at` is pre-set to `Scheduled Date + T08:00:00-05:00` (Friday 8am EST) from the Airtable record.

---

## Polling Interval

The Airtable trigger polls every 5 minutes by default. To change:
- Open the **Airtable — Poll Approved Records** node
- Change the trigger interval under **Trigger Settings**

For time-sensitive publishing (e.g., a 9am post that must go out at exactly 9am), use N8N's **Schedule Trigger** as a secondary check 15 minutes before scheduled times.

---

## Error Handling

Each publishing node returns an error if the API call fails. To add error notifications:
1. In N8N, go to **Settings → Error Workflow**
2. Create a separate error notification workflow (e.g., send a Slack message or email to dylan@orchidea.digital)
3. Set it as the error workflow for this pipeline

Common failure causes:
- **LinkedIn 401**: OAuth token expired — re-authenticate the credential
- **Instagram 400**: Media URL not accessible — ensure image URLs are public
- **ConvertKit 422**: Missing required fields — check Body formatting from Publishing Connector
- **Framer 404**: Collection ID wrong — verify against Framer dashboard
