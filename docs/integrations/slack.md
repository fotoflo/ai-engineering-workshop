# Slack Webhook Setup for Booking Notifications

This document explains how to set up Slack webhook notifications for booking events in the Flexbike Firebase Functions.

## Overview

The system now sends Slack notifications for:

- New booking creation
- Booking status changes (confirmed, declined, cancelled, expired)

## Setup Instructions

### 1. Create a Slack App and Webhook

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click "Create New App" → "From scratch"
3. Give your app a name (e.g., "Flexbike Notifications") and select your workspace
4. In the left sidebar, go to "Incoming Webhooks"
5. Toggle "Activate Incoming Webhooks" to On
6. Click "Add New Webhook to Workspace"
7. Choose the channel where you want to receive notifications
8. Copy the webhook URL (format: `https://hooks.slack.com/services/[TEAM_ID]/[BOT_ID]/[TOKEN]` - **DO NOT commit actual webhook URLs to version control**)

### 2. Configure Firebase Functions

Set the Slack webhook URL as a Firebase Functions configuration:

```bash
firebase functions:config:set slack.webhook="YOUR_WEBHOOK_URL_HERE"
```

Replace `YOUR_WEBHOOK_URL_HERE` with the webhook URL you copied from Slack.

### 3. Deploy the Functions

Deploy the updated functions to Firebase:

```bash
firebase deploy --only functions
```

## Notification Types

### New Booking Created

- **Trigger**: When a new booking document is created in Firestore
- **Content**: Booking details including rider info, bike info, dates, and pricing
- **Format**: Detailed message with booking ID, rider details, bike details, and booking information

### Booking Status Changes

- **Trigger**: When a booking's status field changes
- **Content**: Status change details with before/after status
- **Supported Statuses**:
  - `confirmed` ✅ - Booking confirmed by host
  - `declined` ❌ - Booking declined by host
  - `cancelled` 🚫 - Booking cancelled
  - `expired` ⏰ - Booking expired (24-hour timeout)

## Message Format

Messages are formatted in Slack's markdown format with:

- Emojis for visual distinction
- Bold headers for sections
- Code formatting for booking IDs
- Bullet points for details
- Currency formatting for prices (Indonesian Rupiah)
- **Contact Information**: WhatsApp links and email addresses for both rider and company
- **Bot Identity**: Flexbike branding with custom icon and username

## Bot Identity

The Slack notifications are sent with:

- **Username**: "Flexbike"
- **Icon**: Flexbike favicon from https://flexbike.app/favicon.png
- **Professional Branding**: Consistent with the Flexbike brand identity

## Contact Information Features

Each notification includes contact information for both the rider and company:

### WhatsApp Links

- **Rider Contact**: Direct WhatsApp link with pre-filled message about the booking
- **Company Contact**: Direct WhatsApp link with pre-filled message about the booking, displays actual phone number
- **Phone Number Formatting**: Automatically formats Indonesian phone numbers (+62)
- **Fallback**: If WhatsApp number not available, shows email instead

### Email Fallbacks

- **Rider Email**: Shows rider's email address (primary contact method for riders)
- **Company Email**: Shows company's email address if available
- **No Contact**: Shows "No contact info available" if neither WhatsApp nor email is available

### Note on Rider Phone Numbers

Currently, rider phone numbers are not stored during user registration. To enable WhatsApp contact for riders, you would need to:

1. Update the user registration process to collect and store phone numbers
2. Add a phone number field to the user document structure
3. Update the frontend registration forms to include phone number input

For now, riders can be contacted via email, which is always available.

### Message Templates

- **Rider Message**: "Hi [Name]! I'm contacting you about your Flexbike booking (ID: [BookingID])."
- **Company Message**: "Hi! I'm contacting you about a Flexbike booking (ID: [BookingID]) for [CompanyName]."

## Troubleshooting

### Check Function Logs

View function logs to debug issues:

```bash
firebase functions:log
```

### Verify Configuration

Check if the webhook URL is properly configured:

```bash
firebase functions:config:get
```

### Test Webhook

You can test the webhook manually by sending a POST request:

```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Test message"}' \
  YOUR_WEBHOOK_URL_HERE
```

## Security Notes

- Keep your webhook URL secure and don't commit it to version control
- The webhook URL is stored in Firebase Functions configuration
- Only marketplace bookings trigger notifications (non-marketplace bookings are filtered out)
- Failed notifications are logged but don't break the booking process

## Customization

To modify the notification format or add new notification types:

1. Edit `slackBookingNotifications.js`
2. Update the message formatting functions
3. Add new trigger conditions if needed
4. Deploy the updated functions

## Environment Variables

The system uses the following configuration:

- `functions.config().slack.webhook` - The Slack webhook URL
