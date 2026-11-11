# Discord News — free mobile-first deployment

This starter uses a polling architecture designed for a phone-only, $0 hobby deployment:

- **GitHub Actions** runs `server/src/poll.js` every 5 minutes and reads configured Discord channels through the Discord REST API.
- **Neon** provides free PostgreSQL storage.
- **Render** hosts the API. Its free web service sleeps after 15 minutes idle and may take about a minute to wake up.
- **Expo** runs the iOS/Android app during development.

## Important privacy choice
The GitHub repository must be **public** for standard GitHub-hosted Actions runner usage to remain free without consuming the private-repository minutes quota. The source code is public, but keep `DISCORD_BOT_TOKEN`, `DATABASE_URL`, and `ANNOUNCEMENT_CHANNEL_IDS` in GitHub Actions Secrets only. Never commit secrets or paste them into chat.

GitHub documents that standard hosted runners are free for public repositories: https://docs.github.com/en/billing/concepts/product-billing/github-actions

## A. Create the database
1. Create a free Neon project: https://neon.com/
2. Copy its PostgreSQL connection string (use the pooled connection string if available).
3. Keep the connection string private.

Neon free plan limits apply; see https://neon.com/docs/introduction/plans

## B. Upload this project to GitHub from your phone
1. Create a **Public** repository named `discord-news`.
2. Upload the extracted project files and folders, preserving folder paths (including `.github/workflows/poll-discord.yml`).
3. Do not upload `.env` files or secrets. The example env files contain placeholders only.

GitHub mobile can create repositories; for uploading a folder tree from a phone, use github.com in your browser and the repository's Add file / Upload files flow. If your browser cannot select a whole folder, upload files in batches while preserving the directory structure.

## C. Add GitHub Actions secrets
In the repository, open Settings -> Secrets and variables -> Actions -> New repository secret.
Create:
- `DATABASE_URL` = Neon connection string
- `DISCORD_BOT_TOKEN` = bot token from Discord Developer Portal
- `ANNOUNCEMENT_CHANNEL_IDS` = comma-separated channel IDs, e.g. `123...,456...`

Enable Actions for the repository. The workflow can be started manually from Actions -> Poll Discord announcements -> Run workflow. Scheduled runs are every 5 minutes, but GitHub may delay scheduled jobs during busy periods.

## D. Create and invite the Discord bot
1. Create an application at https://discord.com/developers/applications
2. Add a Bot user and copy its token directly into the GitHub secret (never into source code).
3. Invite the bot to each server with View Channel and Read Message History permissions for the selected channels.
4. Copy the IDs of announcement channels and put them in the `ANNOUNCEMENT_CHANNEL_IDS` secret.
5. Do not enable Message Content Intent for this REST polling setup unless you later change the implementation to use Gateway message events. The bot still needs permission to view/read the chosen channels.

## E. Deploy the API to Render
1. Create a free Render account: https://render.com/
2. Choose New -> Blueprint and connect the public `discord-news` GitHub repository.
3. Render should detect `render.yaml`. Create the service.
4. When prompted, set `DATABASE_URL` to the same Neon connection string.
5. Wait for the build to finish. Open the service URL plus `/health`; it should return `{"ok":true}`.

Render's free web service sleeps after 15 minutes without inbound traffic. It may take around a minute to wake up. This does not stop the separate GitHub Actions poller.

## F. Configure the phone app
1. Open the `mobile` folder in the repository.
2. Edit `mobile/.env.example` into `.env` (or set the Expo public environment variable in the build environment) and set:
   `EXPO_PUBLIC_API_URL=https://YOUR-RENDER-SERVICE.onrender.com`
3. Expo Go and Expo's core development tools are free. To run the project itself, a Node-capable environment is still needed; since you have only a phone, use GitHub-connected cloud tooling or a trusted helper for the initial Expo start/build. EAS has a limited free build allowance, but store publishing may require developer program fees.
4. The mobile client is a starter; complete Discord OAuth login, per-user authorization, device push registration, and production security before public release.

## API
- `GET /health`
- `GET /api/servers`
- `GET /api/announcements?serverId=<discord-server-id>&q=<search>&saved=true`
- `PATCH /api/announcements/:id/save` with `{ "saved": true }`

## Honest limitations
- Polling is not instant; scheduled workflow runs may be delayed.
- Each poll checks the latest 100 messages in each configured channel.
- Render's free API may sleep; first app request after idle may be slow.
- Neon and Render free plans have usage, restart, and data-retention limits. Check current provider terms.
- The API currently has incomplete user authentication and authorization. Keep this as a private personal test only; do not expose real users' feeds or private server content publicly until authorization is finished.
