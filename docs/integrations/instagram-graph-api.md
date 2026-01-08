# Instagram Graph API Integration Setup

This document outlines the complete setup process for integrating Instagram Graph API to fetch mentions (tagged posts) instead of the deprecated Instagram Basic Display API.

## Prerequisites

1. **Instagram Business or Creator Account**: You need an Instagram Business or Creator account connected to a Facebook Page.
2. **Facebook App**: Created with Instagram Basic Display and Instagram Graph API products enabled.
3. **Environment Variables**: Set in `.env.local`:
   ```env
   INSTAGRAM_APP_ID=your_app_id
   INSTAGRAM_APP_SECRET=your_app_secret
   NEXT_PUBLIC_BASE_URL=https://your-domain.com
   ```

## Setup Steps

### 1. Initial Authentication

Visit the admin page at `/admin/instagram` and click "Connect Instagram Account". This will:

1. Redirect to Instagram OAuth flow
2. Request necessary permissions:
   - `instagram_graph_user_profile`
   - `instagram_graph_user_media`
   - `pages_show_list`
   - `pages_read_engagement`
3. Exchange authorization code for short-lived access token
4. Convert to long-lived access token (60 days validity)
5. Store token data securely

### 2. Token Management

The system automatically manages tokens:

- **Storage**: Tokens are stored in-memory (for demo). In production, use a database.
- **Refresh**: Vercel cron job runs every 15 days to refresh tokens before expiry
- **Validation**: API checks token validity before making requests

### 3. API Endpoints

#### Main Instagram API

- **Route**: `GET /api/instagram`
- **Function**: Fetches mentions (tagged posts) from your Instagram account
- **Fallback**: Uses static content when API is unavailable

#### Authentication Routes

- **Route**: `GET /api/instagram/auth?action=login` - Start OAuth flow
- **Route**: `GET /api/instagram/auth?action=status` - Check token status
- **Route**: `POST /api/instagram/auth` - Manual token refresh

#### Cron Job

- **Route**: `GET /api/instagram/cron` - Automatic token refresh (runs every 15 days)

#### Admin Interface

- **Route**: `/admin/instagram` - Manage Instagram integration

## Token Refresh Schedule

The Vercel cron job is configured in `vercel.json`:

```json
{
  "path": "/api/instagram/cron",
  "schedule": "0 3 1,15 * *"
}
```

This runs at 3:00 AM on the 1st and 15th of every month, ensuring tokens are refreshed well before the 60-day expiry.

## Permissions Required

The OAuth flow requests these Instagram Graph API permissions:

- `instagram_graph_user_profile`: Access basic profile information
- `instagram_graph_user_media`: Access media posted by the user
- `pages_show_list`: Access list of Facebook Pages
- `pages_read_engagement`: Read engagement metrics for Pages

## Data Flow

1. **User tags @flexbike in Instagram post**
2. **Instagram Graph API** detects the tag
3. **Our system fetches mentions** using `/api/instagram`
4. **InstagramFeed component** displays the posts
5. **Tokens auto-refresh** via cron job

## Error Handling

- **Token Expired**: Falls back to static content
- **API Rate Limits**: 10-minute cache prevents excessive requests
- **Network Issues**: Graceful fallback with fallback posts
- **Auth Failures**: Clear error messages in admin interface

## Monitoring

Check token status at `/admin/instagram` to see:

- Connection status (Connected/Not Connected)
- Token expiry date and time
- Instagram User ID
- Manual refresh option

## Security Notes

- Never commit tokens to version control
- Use environment variables for app credentials
- Tokens are encrypted in transit (HTTPS)
- Refresh tokens automatically to minimize security risks

## Troubleshooting

### Common Issues

1. **"Instagram Business Account Required"**

   - Ensure your Instagram account is connected to a Facebook Page
   - Convert to Business or Creator account if needed

2. **"Invalid Scopes"**

   - Check that all required permissions are approved in Facebook App settings

3. **"Token Refresh Failed"**

   - Verify app credentials in `.env.local`
   - Check Facebook App configuration

4. **No Mentions Showing**
   - Confirm users are tagging your account correctly (@flexbike)
   - Check that posts are public
   - Verify token has proper permissions

### Debug Commands

```bash
# Check token status
curl "http://localhost:3000/api/instagram/auth?action=status"

# Test Instagram API
curl "http://localhost:3000/api/instagram"

# Manual token refresh
curl -X POST "http://localhost:3000/api/instagram/auth" \
  -H "Content-Type: application/json" \
  -d '{"action": "refresh"}'
```

## Migration from Basic Display API

The system automatically falls back to the old Basic Display API if Graph API tokens aren't configured, ensuring zero downtime during migration.

