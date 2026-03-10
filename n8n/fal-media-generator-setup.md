# FAL Media Generator — Setup Guide

## 1. N8N Environment Variables

Set these in N8N → Settings → Environment Variables:

| Variable | Value |
|----------|-------|
| `FAL_KEY` | Your FAL.ai API key — https://fal.ai/dashboard → Keys |
| `AIRTABLE_API_KEY` | Your Airtable personal access token — https://airtable.com/create/tokens |
| `AIRTABLE_BASE_ID` | Your base ID (starts with `app`) — visible in any Airtable API URL |

## 2. Import the Workflow

1. N8N → Workflows → Import from File
2. Select `fal-media-generator.json`
3. Activate the workflow
4. Copy the webhook URL from the **Airtable Webhook** node (looks like `https://your-n8n.com/webhook/fal-media-generator`)

## 3. Airtable Automations (one per table)

Set up the same automation in each of the 6 content tables. The only difference is the `platform` and `table` values in the payload.

**Trigger**: When record matches condition → `Status` is `Ready to Queue`

**Action**: Send a webhook → POST to your N8N webhook URL

**Request body** (JSON):

### YouTube Content table
```json
{
  "recordId": "{Record ID}",
  "table": "YouTube Content",
  "platform": "youtube",
  "contentType": "{ContentType}",
  "title": "{Title}",
  "body": "{Body}",
  "thumbnailConcept": "{ThumbnailConcept}",
  "tags": "{Tags}",
  "tenantId": "{TenantId}"
}
```

### Instagram table
```json
{
  "recordId": "{Record ID}",
  "table": "Instagram",
  "platform": "instagram",
  "contentType": "{ContentType}",
  "title": "{Title}",
  "body": "{Body}",
  "visualDirection": "{VisualDirection}",
  "altText": "{AltText}",
  "hashtags": "{Hashtags}",
  "tenantId": "{TenantId}"
}
```

### LinkedIn Posts table
```json
{
  "recordId": "{Record ID}",
  "table": "LinkedIn Posts",
  "platform": "linkedin",
  "contentType": "{ContentType}",
  "title": "{Title}",
  "body": "{Body}",
  "hook": "{Hook}",
  "postType": "{PostType}",
  "tenantId": "{TenantId}"
}
```

### Twitter/X table
```json
{
  "recordId": "{Record ID}",
  "table": "Twitter/X",
  "platform": "twitter",
  "contentType": "{ContentType}",
  "title": "{Title}",
  "body": "{Body}",
  "tenantId": "{TenantId}"
}
```

### Email Campaigns table
```json
{
  "recordId": "{Record ID}",
  "table": "Email Campaigns",
  "platform": "email",
  "contentType": "{ContentType}",
  "title": "{Title}",
  "body": "{Body}",
  "campaignType": "{CampaignType}",
  "previewText": "{PreviewText}",
  "tenantId": "{TenantId}"
}
```

### SEO / Articles table
```json
{
  "recordId": "{Record ID}",
  "table": "SEO Articles",
  "platform": "seo",
  "contentType": "{ContentType}",
  "title": "{Title}",
  "body": "{Body}",
  "targetKeyword": "{TargetKeyword}",
  "tenantId": "{TenantId}"
}
```

## 4. Video Detection Logic

The workflow generates a **video** (Kling) instead of an image (Flux Pro) when:

| Platform | ContentType contains | Result |
|----------|---------------------|--------|
| `instagram` | `reel` | 9:16 Kling video |
| `youtube` | `video` | 9:16 Kling video |
| Everything else | anything | Flux Pro image |

## 5. What Happens After Generation

On **success**: N8N patches the Airtable record with:
- `ImageUrl` = FAL-hosted asset URL (permanent, no expiry)
- `Status` = `Media Ready`

On **failure**: N8N patches the record with:
- `PublishError` = specific error message (FAL error code + node that failed)
- `Status` unchanged (stays `Ready to Queue` so it can be retried)

## 6. Generated Asset Dimensions

| Platform | Width | Height | Ratio |
|----------|-------|--------|-------|
| YouTube | 1280 | 720 | 16:9 |
| Instagram post | 1080 | 1080 | 1:1 |
| Instagram Reel | 1080 | 1920 | 9:16 (video) |
| LinkedIn | 1200 | 628 | ~1.91:1 |
| Twitter/X | 1200 | 675 | 16:9 |
| Email | 600 | 338 | ~16:9 |
| SEO/Article | 1200 | 630 | ~1.9:1 |
