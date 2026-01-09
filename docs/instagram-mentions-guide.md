# Instagram Graph API Mentions Setup

With your **App ID** and **App Secret**, you can now generate the tokens
you'll need for mentions (and anything else via the Instagram Graph
API).

------------------------------------------------------------------------

## 1. Exchange App ID + Secret → App Access Token

    GET https://graph.facebook.com/oauth/access_token?
      client_id={app-id}
      &client_secret={app-secret}
      &grant_type=client_credentials

This gives you an **App Token**.\
👉 Useful for debugging, not for fetching IG content.

------------------------------------------------------------------------

## 2. Log in a User (your IG Business/Creator account)

Use **Facebook Login** (OAuth) to get a **short-lived User Access
Token** (valid \~1h).\
- Scopes you must request:\
`instagram_graph_user_profile`, `instagram_graph_user_media`,
`pages_show_list`, `pages_read_engagement`.\
- Redirect back to your site with `code`.

------------------------------------------------------------------------

## 3. Exchange Code → Short-Lived Token

    GET https://graph.facebook.com/v21.0/oauth/access_token?
      client_id={app-id}
      &client_secret={app-secret}
      &redirect_uri={redirect-uri}
      &code={code}

------------------------------------------------------------------------

## 4. Exchange Short-Lived → Long-Lived Token

    GET https://graph.facebook.com/v21.0/oauth/access_token?
      grant_type=fb_exchange_token
      &client_id={app-id}
      &client_secret={app-secret}
      &fb_exchange_token={short-lived-token}

-   Returns a **60-day token**.\
-   You can refresh it before expiry with the same call.

------------------------------------------------------------------------

## 5. Get Your IG User ID

    GET https://graph.facebook.com/v21.0/me/accounts?
      access_token={user-access-token}

-   Find the FB Page connected to your IG account.\
-   From there:\
    `GET /{page-id}?fields=instagram_business_account`\
-   That gives `{ "instagram_business_account": { "id": "1784..." } }`.

------------------------------------------------------------------------

## 6. Fetch Mentions (tagged media)

    GET https://graph.facebook.com/v21.0/{ig-user-id}/tags?
      fields=id,media_type,media_url,caption,permalink,username,timestamp
      &access_token={long-lived-token}

------------------------------------------------------------------------

### Key Notes

-   **App ID + Secret alone are not enough** --- they're just step 1 in
    the token exchange.\
-   You'll always need a **User Access Token** tied to your
    Business/Creator IG account.\
-   Best practice: run this token logic server-side, store the
    long-lived token in your DB or secrets manager, and refresh every
    \~60 days.

------------------------------------------------------------------------

👉 Do you want me to give you a **copy-paste Next.js API route** that
uses your App ID + Secret to automatically refresh and serve a valid
long-lived token (so your frontend can just call `/api/ig/mentions`
without worrying about OAuth)?
