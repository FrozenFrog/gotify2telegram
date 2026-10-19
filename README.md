# Gotify 2 Telegram (and Discord)
This Gotify plugin forwards received messages to Telegram and/or Discord.

## Prerequisite
- A Telegram bot, bot token, and chat ID from bot conversation. You can get that information by following this [blog](https://medium.com/linux-shots/setup-telegram-bot-to-get-alert-notifications-90be7da4444).
- Golang, Docker, wget (If you want to build the binary from source).

## Compatibility

Gotify plugins are Go plugins, so a `.so` file only loads on **the exact Gotify version it was built for** (same Go version and same dependency versions). Loading a mismatched plugin makes Gotify exit on startup with an error like `plugin was built with a different version of package ...`.

Prebuilt plugins are available for Gotify `2.5.0`, `2.6.1`, `2.7.3`, `2.8.0`, `2.9.1`, `3.0.0` and `3.1.1`. The Gotify version is part of the file name, e.g. `telegram-plugin-linux-amd64-v3.1.1.so`.

> [!IMPORTANT]
> Pin the Gotify image to a specific version (e.g. `gotify/server:3.1.1`), not `latest`. If you use automatic image updates (Watchtower, Komodo `auto_update`, ...), a new Gotify release will be pulled without a matching plugin and Gotify will fail to start.

## Installation
* **By shared object**

    1. Get the shared object matching your Gotify version and architecture from [release](https://github.com/lekoOwO/gotify2telegram/releases).

    2. Put it into Gotify plugin folder (`data/plugins` by default). Keep only one version of this plugin in the folder.

    3. Restart gotify.

    4. Config the plugin in the Gotify web UI (Plugins tab), see [Configuration](#configuration).

* **Build from source**

    1. Build the binary for your Gotify version. This updates `go.mod` to the dependency versions of that Gotify release and builds inside the official `gotify/build` Docker image.

    ```
    GOTIFY_VERSION=3.1.1 make build
    ```

    Use `make build-linux-amd64` (or `build-linux-arm64`, `build-linux-arm-7`) to build a single architecture.

    2. Follow instructions from step 2 in the shared object installation.

## Upgrading Gotify

1. Back up your Gotify `data` folder (at least `gotify.db` and `plugins`).
2. Get or build the plugin for the new Gotify version.
3. Stop Gotify, replace the old `.so` in the plugin folder with the new one, change the image tag to the new version and start Gotify.

The plugin configuration and enabled state are stored in the Gotify database and keyed by the plugin's module path, so they are kept across upgrades. Existing application and client tokens keep working; you do not need to recreate them.

### Upgrading to Gotify 3.x

- `config.yml` is no longer read by Gotify 3. Convert any non-default settings to environment variables, see the [migration guide](https://gotify.net/docs/migrate-to-3). The plugin configuration is not affected, it lives in the database.
- Tokens are not shown again in the Gotify UI after creation. Keep the client token used in the plugin's `token` setting, or create a new client if you need a new one.
- Gotify 3 can delete clients after a period of inactivity. Existing clients never expire by default; if you create a new client for this plugin, leave its inactivity expiry at `0` (never).

## Configuration

The configuration contains four keys: `clients`, `gotify_host`, `token` and `discord`.

This plugin supports sending to Telegram, Discord, or both. Each `SubClient` may independently enable Telegram and/or Discord.

### Clients

The `clients` configuration key describes which client(channel?) we are going to listen on and which telegram channel (and topic optionally!) we are forwarding the message to.

```yaml
clients:
  - app_id: "The Gotify App ID to be matched. use -1 for all-matching."
    telegram:
      chat_id: "ID of the telegram chat"
      token: "The bot token"
      thread_id: "Thread ID of the telegram topic. Leave it empty if we are not sending to a topic."
    discord:
      webhook_url: "https://discord.com/api/webhooks/..."
      username: "Optional per-client username (falls back to global discord defaults if empty)"
      avatar_url: "Optional per-client avatar URL (falls back to global defaults if empty)"
  - app_id: "Maybe the second Gotify Client Token, yay!"
    telegram:
      chat_id: "ID of the telegram chat"
      token: "The bot token"
      thread_id: "Thread ID of the telegram topic. Leave it empty if we are not sending to a topic."
```

### Global Discord defaults

You can set global Discord defaults (used when per-client username/avatar are empty):

```yaml
discord:
  username: "GotifyBot"
  avatar_url: "https://example.com/avatar.png"
```

### Gotify Host

The `gotify_host` configuration key should be set to `ws://YOUR_GOTIFY_IP` (depending on your setup, `ws://localhost:80` will likely work by default)

### Token

The `token` configuration key should be set to a valid token that can be created in the "Clients" tab.

## Troubleshooting
1. Gotify exits on startup after loading the plugin, with `plugin was built with a different version of package ...`:

    The plugin does not match your Gotify version. Use the `.so` built for your exact Gotify version (see [Compatibility](#compatibility)) and make sure no older version of the plugin is left in the plugin folder.

2. When only the Gotify dashboard receives your message, but not Telegram:

    If, when making the API call to get your bot's chat ID, no data is returned, you may need to change the bot's privacy settings.

    - In the BotFather chat, list your created bots and select the respective bot for which you want to change the Group Privacy setting.
    - Turn off the Group Privacy setting.
