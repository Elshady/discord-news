# Discord login, server discovery and push notifications

This upgrade adds a backend Discord OAuth2 flow, a signed seven-day app session,
automatic discovery of servers shared by the user and the bot, and Expo push-token
storage/delivery.

## Discord Developer Portal
1. Create an OAuth2 application and add the exact backend callback URL from
   `DISCORD_REDIRECT_URI` to OAuth2 -> Redirects.
2. Set `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_REDIRECT_URI`,
   `MOBILE_REDIRECT_URI`, and a long random `JWT_SECRET` in the backend environment.
3. The OAuth scopes used are `identify` and `guilds`.
4. Install the bot into each server you want to read. OAuth discovery only lists
   servers the signed-in user belongs to AND where the bot is also installed.

## Mobile sign-in and push registration
The app package includes `expo-auth-session`, `expo-web-browser`,
`expo-notifications`, and `expo-device`; the `discordnews://` deep-link scheme
is configured in app.json.

The login screen should open `${API}/auth/discord` in the system browser and
handle the `discordnews://auth?token=...` deep link. Store the JWT in secure
device storage (for example `expo-secure-store`, which should be added to the
app dependencies) and send it as `Authorization: Bearer <token>` on API calls.

After login, request notification permission on a physical device, obtain an
Expo push token with `Notifications.getExpoPushTokenAsync()`, and POST
`{ "token": "<ExponentPushToken[...]>" }` to `/api/push-token` with the JWT.

## Channel selection and message collection
The bot still only ingests channel IDs in `ANNOUNCEMENT_CHANNEL_IDS`. This is
intentional least-privilege behavior. Server discovery is automatic; selecting
which announcement channels to follow should be a separate authenticated setup
screen. Do not enable broad message collection by default.

## Important security / release notes
- Set a strong, unique JWT_SECRET. The development fallback is not safe.
- Serve the backend only over HTTPS outside local development.
- Implement refresh-token/session revocation and secure token storage before release.
- Add per-user saved-state records before multi-user production; the current
  `saved` flag is global per announcement.
- The demo OAuth callback sends the session JWT in a deep-link URL. For a
  production release, prefer a short-lived one-time exchange code to avoid
  tokens appearing in URL logs.
- Configure Expo EAS project credentials and push notification credentials.
- This is starter code; test OAuth redirect handling on both iOS and Android.
