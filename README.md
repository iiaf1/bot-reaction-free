<!--
============================================================
 Bot Reaction Free
 حقوق الملكية © 2026 iaf1 — جميع الحقوق محفوظة.
 Copyright © 2026 iaf1 — All rights reserved.
============================================================

[AR] إشعار المشروع:
 - هذا المشروع مجاني بالكامل.
 - هذا المشروع غير قابل للبيع، ولا يجوز بيعه أو بيع كوده أو أي نسخة معدّلة منه.
 - يجب الإبقاء على إشعار حقوق الملكية هذا في أي نسخة أو جزء من الكود.
 - الرخصة: PolyForm Noncommercial 1.0.0 (استخدام غير تجاري فقط) — راجع LICENSE.md.

[AR] إخلاء المسؤولية:
 هذا الملف مقدَّم "كما هو" (AS IS) بدون أي ضمانات من أي نوع.
 المالك (iaf1) غير مسؤول عن أي أضرار أو خسائر أو مشاكل في السيرفر
 أو فقدان بيانات أو ثغرات أمنية أو سوء استخدام ناتج عن استخدام هذا الكود
 أو تعديله أو استضافته. المستخدم هو المسؤول الوحيد عن استخدامه.

[EN] Project Notice:
 - This project is completely free.
 - This project is NOT for sale. Selling it, its source code, or any
   modified version of it is not permitted by the owner.
 - This copyright notice must be retained in all copies and derivatives.
 - License: PolyForm Noncommercial 1.0.0 (noncommercial use only) — see LICENSE.md.

[EN] Disclaimer:
 This file is provided "AS IS", without warranty of any kind. The owner
 (iaf1) is not liable for any damage, loss, server issues, data loss,
 security issues, or misuse resulting from the use, modification, or
 hosting of this code. The user is solely responsible for its use.

Contact / للتواصل:
 Discord: iaf0
 GitHub:  https://github.com/iiaf1
-->

# Bot Reaction Free

A **free** Discord bot built with Node.js and discord.js. It automatically adds an emoji to **every new message** in the channels you choose, with independent settings per channel.

> The slash commands and the bot's replies are in Arabic.

## Features

- Slash commands restricted to administrators (`Administrator` permission required).
- Standard emojis (🔥) and custom emojis (static or animated).
- Multi-channel support; each channel has its own emoji and status (on/off).
- Settings are stored in SQLite and survive restarts.
- Ignores messages from bots, webhooks, and system messages.
- Per-channel reaction queue; handles Discord API errors and rate limits without stopping the bot.
- Automatic reconnection, plus a watchdog that restarts the bot if the connection is down for too long.
- Daily event and error logs with automatic cleanup of old files.
- 24/7 operation with PM2.
- Minimal permissions: **no Privileged Intents** and no "Send Messages" permission.
- Owner-only commands via `OWNER_ID`.

## Requirements

- Node.js **20 or later**
- npm
- A Discord account and a bot application from the [Developer Portal](https://discord.com/developers/applications)

> The project uses `better-sqlite3`, which installs prebuilt on most systems. If installation fails on Linux, install the build tools:
> `sudo apt install -y build-essential python3`

## 1) Create the bot

1. Open the Developer Portal and click **New Application**.
2. In the **Bot** tab, click **Reset Token** and copy the token (never share it).
3. Under **Privileged Gateway Intents**, leave all three options **disabled**. The bot does not need them.

## 2) Get your OWNER_ID

Enable **Developer Mode** (Discord Settings → Advanced), then right-click your profile and choose **Copy User ID**.

## 3) Install

```bash
git clone https://github.com/iiaf1/bot-reaction-free.git
cd bot-reaction-free
npm install
cp .env.example .env
```

Edit `.env` so it contains only these two variables:

```env
DISCORD_TOKEN=put_bot_token_here
OWNER_ID=put_owner_id_here
```

`CLIENT_ID` and `GUILD_ID` are not needed. The bot reads the application ID after logging in and registers its commands globally.

## 4) Run

```bash
npm start
```

On startup the bot prints a **ready-to-use invite link** with the correct permissions, starting with
`https://discord.com/oauth2/authorize?client_id=...`

Open it and add the bot to your server. To build the link manually, use `permissions=328768` and `scope=bot applications.commands`.

The bot needs only these 4 permissions:

| Permission | Reason |
|---|---|
| View Channel | See the channel |
| Read Message History | Required by Discord to add a new reaction |
| Add Reactions | Add the reaction |
| Use External Emojis | For custom emojis from another server |

## 5) Run 24/7 with PM2

```bash
npm install -g pm2
pm2 start ecosystem.config.cjs
pm2 save
pm2 startup        # run the command it prints (Linux/macOS) so the bot starts after a reboot
```

Useful commands:

```bash
pm2 logs bot-reaction-free       # view logs
pm2 restart bot-reaction-free    # restart
pm2 stop bot-reaction-free       # stop
pm2 status                       # status
```

Notes:

- Do not set `instances` above 1, or several instances will react to the same message.
- PM2 does not support `pm2 startup` on Windows; use Task Scheduler or `pm2-installer`.

## Commands

| Command | Description |
|---|---|
| `/تفاعل` | Create or edit a reaction. Options: channel (`الروم`), emoji (`الإيموجي`), status (`الحالة`: on/off, default on) |
| `/تفاعل-حذف` | Delete the reaction from a channel |
| `/تفاعل-عرض` | List all reactions in the server and their status |
| `/تفاعل-إيقاف` | Pause one channel, or the whole server if no channel is given |
| `/تفاعل-تشغيل` | Resume one channel, or the whole server if no channel is given |
| `/مالك-احصائيات` | (Owner) Bot statistics: servers, reactions, queue, memory, uptime |
| `/مالك-تنظيف` | (Owner) Remove settings of servers the bot left and of deleted channels |

- Admin commands require the `Administrator` permission, and this is checked again at execution time.
- The bot owner (`OWNER_ID`) can run admin commands in any server the bot is in, and is the only one who can run the owner commands.
- A custom emoji must belong to a server the bot is in.
- Reactions apply to text and announcement channels. Thread messages have a different channel ID and are not covered.

## How it works

- **Intents:** only `Guilds`, `GuildMessages`, and `GuildEmojisAndStickers`. The bot never reads message content.
- **messageCreate:** ignores bots, webhooks, and system messages, checks whether the channel has an enabled reaction, then queues the job.
- **Queue:** one queue per channel (max 200 jobs). Reactions run sequentially and discord.js waits out rate limits automatically.
- **Errors:** deleted messages, blocked reactions, and the 20-reaction limit are ignored quietly. Network and 5xx errors are retried. Missing permissions are logged at most once per 10 minutes per channel and do not cause failing requests. If an emoji becomes unusable (for example it was deleted), reactions in that channel are disabled automatically and a warning is logged.
- **Connection:** discord.js reconnects automatically. If the bot is not ready for more than 5 minutes, or the session is invalidated, it exits so PM2 restarts it.
- **Unexpected errors:** `unhandledRejection` is logged and the bot continues. `uncaughtException` is logged and PM2 restarts the bot within seconds in a clean state.

## Logs and data

- `logs/combined-YYYY-MM-DD.log`: all events.
- `logs/error-YYYY-MM-DD.log`: warnings and errors only (files older than 14 days are deleted).
- `data/bot.sqlite`: the database. Back up the `data/` folder regularly.

## Project structure

```text
bot-reaction-free/
├── src/
│   ├── index.js                 # entry point: client, events, login, graceful shutdown
│   ├── config/                  # env, constants, paths
│   ├── commands/
│   │   ├── admin/               # setup, remove, list, pause, resume
│   │   └── owner/               # stats, cleanup
│   ├── events/                  # clientReady, messageCreate, interactionCreate, connection, ...
│   ├── services/                # reactionQueue, commandRegistrar, stats
│   ├── database/                # SQLite, migrations, repositories
│   └── utils/                   # logger, emoji, permissions, embeds, loader, ...
├── ecosystem.config.cjs         # PM2 config
├── .env.example
└── package.json
```

## Adding a command or event

- **Command:** add a file under `src/commands/admin/` or `src/commands/owner/` that has a `default` export `{ scope, data, execute }`. It is loaded and registered automatically.
- **Event:** add a file under `src/events/` with a `default` export `{ name, once?, execute }` (or an array of them).
- **Table or column:** add a new migration at the end of the array in `src/database/index.js`.
- Files whose names start with `_` are ignored.

## Troubleshooting

| Problem | Fix |
|---|---|
| Bot exits immediately with an `.env` message | Make sure `.env` is in the project root and both values are set |
| `Fatal login error` | The token is wrong. Reset it in the Developer Portal |
| Commands do not appear | Invite the bot with the `applications.commands` scope, refresh Discord with `Ctrl+R`, and check the log for `Registered ... slash commands` |
| Commands appear but cannot be used | They require `Administrator`; or adjust command permissions in Server Settings → Integrations |
| The bot does not react | Check the 4 permissions in that channel (channel overrides can block them) and make sure the status is on in `/تفاعل-عرض` |
| Custom emoji rejected | It must belong to a server the bot is in; pick it from the emoji picker while typing the command |
| `better-sqlite3` fails to install | Use Node 20+ and install the build tools (see Requirements) |
| PM2 keeps stopping the bot | Usually a `.env` or token problem. Run `npm start` to see the error |

## Security

- No token or sensitive data is stored in the code. Everything lives in `.env`, which is listed in `.gitignore`, as are `data/` and `logs/`.
- If your token ever leaks, click **Reset Token** in the Developer Portal immediately.

## Rights and License

Copyright © 2026 iaf1. All rights reserved.

- This project is completely free and **not for sale**: selling it, its source code, or any modified version is not permitted.
- The copyright notice must be kept in every file and in any copy or part of the code.
- License: **PolyForm Noncommercial 1.0.0** (non-commercial use only). See [LICENSE.md](./LICENSE.md).
- Provided "AS IS" without warranty. The owner is not liable for any damage, data loss, security issues, or misuse. The user is solely responsible for its use.

Contact: Discord `iaf0` | GitHub: https://github.com/iiaf1
