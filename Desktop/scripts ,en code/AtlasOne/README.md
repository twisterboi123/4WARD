# AtlasOne Discord Bot

A Discord.js v14 slash-command only bot providing FAQs, reminders, polls, moderation, leveling (XP), tags, math evaluation, unit conversion, color info, auto-responses, welcome/goodbye messages, and logging.

## Features Overview
- FAQ system: `/faq add|list|get`
- Reminders: `/remind set|list|cancel` (restores after restart)
- Polls with buttons: `/poll create|close`
- Server/user/role/channel info: `/server stats`, `/user info`, `/role info`, `/channel info`
- Moderation: `/mod warn|warnings|clearwarnings|timeout|purge`
- Auto-responder: `/auto add|list|remove` + passive triggers
- Welcome/Goodbye messages: `/welcome set`, `/goodbye set`
- Leveling: passive XP per message + `/level rank`, `/level leaderboard`
- Tags: `/tag add|get|list|remove`
- Utilities: `/math evaluate`, `/convert units`, `/color info`
- Persistent JSON storage in `data/` with safe helper
- Action logging to rotating files in `logs/`

## Project Structure
```
index.js
commands/
events/
utils/
data/
logs/
```

## Local Setup
1. Install dependencies:
```powershell
npm install
```
2. Set environment variables (PowerShell example):
```powershell
$env:DISCORD_TOKEN="YOUR_BOT_TOKEN"
$env:DISCORD_APP_ID="YOUR_APPLICATION_ID"
# Optional for guild-scoped registration (faster propagation)
$env:DISCORD_GUILD_ID="YOUR_GUILD_ID"
```
3. Register slash commands:
```powershell
npm run deploy
```
4. Start the bot:
```powershell
npm start
```

## Deploy to Render.com (24/7 Hosting)
1. Push your code to GitHub (already done).
2. Go to [Render Dashboard](https://dashboard.render.com/) and sign in.
3. Click **New** → **Web Service**.
4. Connect your GitHub account and select the `twisterboi123/AtlasOne` repository.
5. Render will auto-detect `render.yaml`. Confirm the settings:
   - **Name**: atlasone-bot
   - **Environment**: Node
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Plan**: Free (or upgrade for always-on)
6. Add environment variables in the Render dashboard:
   - `DISCORD_TOKEN` = your bot token
   - `DISCORD_APP_ID` = your application ID
   - `DISCORD_GUILD_ID` = (optional) your guild ID
7. Click **Create Web Service**.
8. Once deployed, run slash command registration manually from your local machine:
```powershell
npm run deploy
```
9. Your bot will now run 24/7 on Render's free tier (sleeps after 15 min inactivity; wakes on events).

**Note**: Render's free tier spins down after inactivity. For true 24/7 uptime, upgrade to a paid plan or use a cron job to ping the service.

## Notes
- Use a guild ID during development for instant command updates.
- Reminder and poll IDs are incremental integers stored in JSON.
- Welcome/goodbye message supports `{user}` placeholder.
- Avoid editing JSON files manually while the bot runs.

## Logs
- Stored in `logs/YYYY-MM-DD.log` as JSON lines.

## Future Improvements
- Add pagination for large lists.
- Add permissions checks to more commands.
- Refine XP curve and cooldown configuration.

## License
No license specified; private/internal use assumed.
