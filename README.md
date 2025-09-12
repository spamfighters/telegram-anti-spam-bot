# 🛡️ SpamFighters – Open-Source Telegram Antispam Bot

A powerful, modular anti-spam bot for Telegram with customizable filtering strategies per chat.  
Built for communities, powered by AI and flexible rules.

---

## 📚 Table of Contents

- [⚙️ Setup Steps](#️-setup-steps)
  - [1. Create a Log Group](#1-create-a-log-group)
  - [2. Get the Chat ID](#2-get-the-chat-id)
  - [3. Add the Bot to Your Main Chat](#3-add-the-bot-to-your-main-chat)
  - [4. Enable the Bot](#4-enable-the-bot)
  - [5. Set the Log Chat ID](#5-set-the-log-chat-id)
- [🧪 Setup Filters](#-setup-filters)
- [📥 Upload Filter Configuration](#-upload-filter-configuration)
- [🔍 Filters](#-filters)
  - [🌐 Language Injection Detection](#-language-injection-detection)
  - [😊 Emoji Limit Filter](#-emoji-limit-filter)
  - [🤖 Remote Spam Model Filter](#-remote-spam-model-filter)
  - [💰 Strong Restricted Words](#-strong-restricted-words)
- [🧾 `filters.yaml` - Filters logic file](#-example-full-filtersyaml)
- [☁️ Self-Hosted Deployment](#️-self-hosted-deployment)
  - [1. Create Your Own Bot](#1-create-your-own-bot)
  - [2. Pull the Docker Image](#2-pull-the-docker-image)
  - [3. Prepare Local Folders](#3-prepare-local-folders)
  - [4. `config.yaml` – Bot Configuration File](#4-configyaml--bot-configuration-file)
  - [5. Run the Container with Volume Mounts](#5-run-the-container-with-volumes)

---

## ⚙️ Setup Steps

Follow these steps to get your Telegram anti-spam bot up and running:

### 1. Create a Log Group

- Create a **Telegram group** that will serve as the **log/admin chat**.
- Add [`@spam_eater_bot`](https://t.me/spam_eater_bot) to the group.
- Grant the bot **admin permissions**:
  - Set Admin Permissions

This group will receive logs and alerts about spam detection and actions taken.

### 2. Get the Chat ID

Send the following command in the log group:

```text
/get_chat_id
````

* The bot will reply with the group’s ID.
* **Copy this value** — you'll need it in the next steps.

### 3. Add the Bot to Your Main Chat

* Invite `@spam_eater_bot` to the Telegram group you want to protect.
* Grant it **admin permissions**:

  * Can delete messages
  * Can restrict members
  * Can read all messages
  * Add Admin Permission

### 4. Enable the Bot

In your **main chat**, activate the bot:

```text
/enable
```

### 5. Set the Log Chat ID

Back in your main chat, link the log group:

```text
/set_admin_chat_id <your_log_chat_id>
```

Example:

```text
/set_admin_chat_id -1001234567890
```

✅ The bot is now actively protecting your group and logging actions to the admin group.

---

## 🧪 Setup Filters

You can configure the bot using custom filters defined in a YAML file.

## 📥 Upload Filter Configuration

1. Open the **log/admin chat**.
2. Upload your `filters.yaml` file as a **document** (not text/photo).

The bot will:

* ✅ Confirm successful loading.
* ❌ Warn if the file is invalid.

---

## 🔍 Filters

Each filter includes common parameters:

* `enabled`: Enables/disables the filter.
* `quarantineWeight`: Contributes to the user’s quarantine score.
* `banWeight`: Contributes to the user’s ban score.
* `inputTransformer`: Preprocesses the message (e.g., `lowercase`, `remove_unicode`).

### Evaluation Logic

1. The message is transformed.
2. Filters are applied sequentially.
3. Scores accumulate.
4. If a ban score is reached, the message is blocked.
5. If only a quarantine threshold is met, the user is quarantined.

---

### 🌐 Language Injection Detection

```yaml
type: language_injection
name: 'Injection in English words'
strictLanguagePattern: '[A-Za-z]'
quarantineWeight: 2
banWeight: 3
inputTransformer:
  type: remove_unicode
```

---

### 😊 Emoji Limit Filter

```yaml
type: weight
name: 'Emoji Limit'
restrictionPatterns:
  - '[^\\p{L}\\p{M}\\p{N}\\p{P}\\p{Z}\\p{Cf}\\p{Cs}\\s[!-/][:-@][\\[-`][{-~]]'
quarantineWeight: 5
banWeight: 10
inputTransformer:
  type: pass
```

---

### 🤖 Remote Spam Model Filter

```yaml
type: remote_filter
name: 'AI Spam Model'
endpoint: https://your.api.endpoint/check
minMessageLengthForCheck: 40
enabled: true
quarantineWeight: 0.3
banWeight: 0.98
inputTransformer:
  type: pass
```

It will call API endpoint `https://your.api.endpoint/check?text=Message from the chat` and expect the response with the following format:
```json
{ "spam": 0.99 }
```
Where 0.0 -> 1.0 is spam confidence

---

### 💰 Strong Restricted Words

```yaml
type: weight
name: 'Strong restricted words'
restrictionPatterns:
  - '\\d+\\s*[$]'
  - '[$]\\s*\\d+'
  - '18\\s*[+]'
  - '\\s\\d{3}k'
  - 'usd'
  - 'eur'
  - 'income'
  - 'bitcoin'
quarantineWeight: 1
banWeight: 2
inputTransformer:
  type: combine
  transformers:
    - type: remove_unicode
    - type: lowercase
```

---

## 🧾 Example: Full `filters.yaml`

```yaml
filters:
  - type: language_injection
    strictLanguagePattern: '[A-Za-z]'
    name: Injection in English words
    quarantineWeight: 2
    banWeight: 3
    inputTransformer:
      type: remove_unicode

  - type: weight
    name: Emoji Limit
    restrictionPatterns:
      - '[^\p{L}\p{M}\p{N}\p{P}\p{Z}\p{Cf}\p{Cs}\s[!-/][:-@][\[-`][{-~]]'
    quarantineWeight: 5
    banWeight: 10
    inputTransformer:
      type: pass

  - type: remote_filter
    endpoint: 'https://your.api.endpoint/check'
    minMessageLengthForCheck: 40
    name: AI Spam Model
    enabled: true
    quarantineWeight: 0.3
    banWeight: 0.98
    inputTransformer:
      type: pass

  - type: weight
    name: Restricted words
    restrictionPatterns:
      - '\\d+\\s*[$]'
      - '[$]\\s*\\d+'
      - '18\\s*[+]'
      - '\\s\\d{3}k'
      - 'usd'
      - 'eur'
    quarantineWeight: 1
    banWeight: 2
    inputTransformer:
      type: combine
      transformers:
        - type: remove_unicode
        - type: lowercase
```

## ☁️ Self-Hosted Deployment

You can self-host your own instance of the SpamFighters bot.

Thanks! I’ve added your requested steps to the **Setup Section** in the final `README.md`. Here's the updated section including how to create a bot with BotFather and disable group privacy:

---

### 1. 🤖 Create Your Own Bot

If you're **self-hosting** or using your own bot instead of `@spam_eater_bot`, follow these steps:

1. Open [@BotFather](https://t.me/botfather) on Telegram.
2. Send `/newbot` and follow the prompts to name your bot.
3. Copy the **bot token** provided at the end — you'll need it in your `config.yaml` or startup config.
4. **Disable group privacy** by sending this to BotFather:

Then select your bot and choose `Disable` — this allows the bot to **read group messages**, which is essential for spam detection.


Here’s how to pull the `vacxe/telegram-anti-spam-bot` Docker image and run it while **mounting local folders** to the container’s `chats/` and `config/` directories.

---

### 📥 1. Pull the Docker Image

```bash
docker pull vacxe/telegram-anti-spam-bot
```

---

### 📁 2. Prepare Local Folders

Make two directories on your local system to store config and chat data:

```bash
mkdir -p ./config ./chats
```

* `./config` — where you place your config file.
* `./chats` — used for local chat logs or persistent storage (if used by the bot).

---
### ⚙️ `config.yaml` – Bot Configuration File

Example: `config/config.yaml`

```yaml
token: "<your_bot_token>"
pollingTimeout: 10
debug: false
goodBehaviourMessageCount: 2

# Optional: Metrics logging via InfluxDB
influxDb:
  url: "https://my.influx.db"
  token: "<your_influxdb_token>"
  org: "default"
  bucket: "telegram_bot"

```

#### 🔧 Field Descriptions

| Key                         | Description                                                                 |
| --------------------------- | --------------------------------------------------------------------------- |
| `token`                     | Your Telegram Bot API token from [@BotFather](https://t.me/botfather).      |
| `pollingTimeout`            | Timeout (in seconds) for long polling. Recommended: `10–30`.                |
| `debug`                     | Set to `true` to enable verbose debug logging.                              |
| `goodBehaviourMessageCount` | Number of good messages to offset spam score. Helps reduce false positives. |

🧩 Metrics Logging
If you don’t use InfluxDB, simply omit the entire influxDb: block. The bot will run normally without sending metrics.

| Key                         | Description                                                                 |
| --------------------------- | --------------------------------------------------------------------------- |
| `influxDb.url`              | URL to your [InfluxDB](https://www.influxdata.com/) instance for metrics.   |
| `influxDb.token`            | Auth token for accessing InfluxDB.                                          |
| `influxDb.org`              | InfluxDB organization name.                                                 |
| `influxDb.bucket`           | Target bucket where bot metrics are stored.                                 |



Ensure this file is available inside the container at `/config/config.yaml`, typically by mounting the folder:

---

### ▶️ 3. Run the Container with Volumes

```bash
docker run -d \
  --name telegram-antispam-bot \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/chats:/app/chats \
  vacxe/telegram-anti-spam-bot
```

> Replace `$(pwd)` with full paths if you're not on Linux/macOS.
