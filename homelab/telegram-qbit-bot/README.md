# Telegram qBittorrent Bot

A small self-hosted Telegram bot for searching configured Jackett indexers and controlling an existing qBittorrent instance. It does not manage Jellyfin, rename media, scrape metadata, or run subscriptions.

## Features in this starter

- `/search <keyword>`: query Jackett's Torznab `all` endpoint and show selectable results.
- Inline buttons: submit the selected result to qBittorrent.
- Send a magnet URI or direct HTTP(S) torrent URL: add it to qBittorrent.
- Send a `.torrent` file: upload it to qBittorrent.
- `/list`: show current qBittorrent tasks.
- Completion notification for tasks added after the bot's first baseline poll.
- Telegram numeric user-ID allowlist.

## Stack

- Jackett: multi-indexer search API. You add and configure indexers yourself in its Web UI.
- Python Telegram bot: long polling; no inbound Telegram webhook port required.
- Existing qBittorrent: accessed through its Web API.
- Existing Jellyfin and media directories are not changed.

## Deploy

1. Copy this directory to the LXC.
2. Copy `.env.example` to `.env`.
3. Fill in:
   - `TELEGRAM_BOT_TOKEN`: create a bot with Telegram's @BotFather.
   - `TELEGRAM_ALLOWED_USER_IDS`: your numeric Telegram user ID (comma-separated for family members).
   - `JACKETT_API_KEY`: shown on the Jackett dashboard.
   - `QBIT_BASE_URL`, `QBIT_USERNAME`, `QBIT_PASSWORD`: reachable qBittorrent WebUI/API endpoint and credentials.
4. Start:
   ```bash
   docker compose up -d --build
   docker compose logs -f telegram-qbit-bot
   ```
5. Open Jackett Web UI through the LXC localhost binding (default 9117; use an SSH tunnel if accessing remotely), add your own indexers, and test them.
6. Message the bot with `/start`, then try `/search <title>`.

## Connecting to qBittorrent in another Compose project

The qBittorrent service is assumed to remain in its current Compose project. Set `QBIT_BASE_URL` to an address reachable from this bot container, typically:

`http://<LXC-LAN-IP>:<published-qBittorrent-WebUI-port>`

Alternatively, attach both Compose projects to a shared external Docker network and use the qBittorrent container/service name and its internal WebUI port. Do not expose qBittorrent WebUI to the public Internet.

## Notes / current limitations

- This is an initial functional scaffold, not a polished production bot.
- Jackett must have working indexers configured; search cannot discover sites by itself.
- Indexer results may return a magnet URI or a direct torrent download URL. Some private trackers require cookies or site-specific authentication; configure those in Jackett where supported.
- Generic web pages are not automatically parsed. Direct links must be magnet URIs or direct HTTP(S) links accepted by qBittorrent.
- Search results are held in bot memory until restart; old inline result buttons expire after restart.
- Completion detection uses qBittorrent's `uploading` state and is best-effort. Existing tasks present during the first poll are treated as baseline and do not generate notifications.
- Add family Telegram IDs to the allowlist only after testing your own account.
- Keep `.env` private. Never commit bot tokens, API keys, passwords, cookies, or private tracker URLs.
- Only search, download, and share content you are authorized to access.

## Security

- Jackett Web UI is bound to 127.0.0.1 on the LXC by default.
- The Telegram bot uses outbound long polling; no inbound port is published.
- qBittorrent WebUI should remain LAN-only or Docker-network-only, with a strong password.
