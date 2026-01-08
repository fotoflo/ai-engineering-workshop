# Instagram API Setup Guide

## Current Status

✅ **Instagram integration is working with fallback content**

- The app gracefully handles missing Instagram API credentials
- Shows beautiful fallback posts when Instagram API is not configured
- No errors or crashes when Instagram token is missing

## Optional: Enable Live Instagram Posts

To show real Instagram posts mentioning @flexbike, follow these steps:

### 1. Create Instagram App

1. Go to [Facebook Developers](https://developers.facebook.com/apps/)
2. Click "Create App" → "Consumer" → "Next"
3. Enter app name (e.g., "Flexbike Instagram Feed")
4. Add Instagram Basic Display product

### 2. Configure Instagram Basic Display

1. In your app dashboard, go to "Instagram Basic Display"
2. Click "Create New App"
3. Add Instagram Test Users (your Instagram account)
4. Generate User Token for your test user

### 3. Add Environment Variable

Add to your `.env.local` file:

```bash
INSTAGRAM_ACCESS_TOKEN="your_long_lived_access_token_here"
```

### 4. Test Integration

- Restart your development server
- Visit `/add-bike-shop` or home page
- Look for the green "🟢 Live Instagram" indicator in development mode

## How It Works

### Without Token (Current State)

- Shows "🔴 Fallback Content" indicator in development
- Displays curated mock posts that look like real Instagram content
- No API calls made to Instagram
- Perfect for development and demo purposes

### With Token

- Shows "🟢 Live Instagram" indicator
- Fetches real posts mentioning @flexbike or #flexbike
- Falls back to mixed content if not enough real posts found
- Caches responses for 10 minutes to respect rate limits

## Token Management

### Long-lived Tokens

- Instagram Basic Display tokens expire after 60 days
- Set up token refresh in production
- Monitor token expiration in your logs

### Rate Limits

- Instagram Basic Display: 200 requests per hour per user
- Our implementation caches for 10 minutes
- This allows ~6 requests per hour, well within limits

## Troubleshooting

### Common Issues

1. **Token Expired**: Regenerate token in Facebook Developer Console
2. **No Posts Found**: Make sure your Instagram account has posts mentioning @flexbike
3. **Rate Limited**: Wait for rate limit to reset (1 hour)

### Debug Mode

In development, you'll see indicators showing the data source:

- 🟢 Live Instagram: Real posts from Instagram API
- 🟡 Mixed Content: Some real posts + fallback content
- 🔴 Fallback Content: All mock content (current state)

## Production Considerations

1. **Environment Variables**: Ensure `INSTAGRAM_ACCESS_TOKEN` is set in production
2. **Error Monitoring**: Monitor API failures and token expiration
3. **Content Moderation**: Review posts before they appear (consider approval workflow)
4. **Backup Plan**: Fallback content ensures site always works

---

**Note**: The Instagram integration is completely optional. The site works perfectly without it, showing beautiful curated content that matches your brand aesthetic.
