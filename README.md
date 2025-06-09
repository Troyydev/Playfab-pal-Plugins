# PlayFabPal SDK

> A lightweight, pluggable SDK for building Discord slash-command plugins that call PlayFab admin/server APIs.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Core Concepts](#core-concepts)

   * [PluginAPI](#pluginapi)
   * [PluginPayload](#pluginpayload)
   * [SafePlayFab](#safeplayfab)
   * [Decorators](#decorators)
5. [Full API Reference](#full-api-reference)

   1. [PluginAPI Methods](#pluginapi-methods)
   2. [PluginPayload Methods](#pluginpayload-methods)
   3. [SafePlayFab Methods](#safeplayfab-methods)
6. [Example Plugin](#example-plugin)
7. [Advanced: Modals & UI](#advanced-modals--ui)
8. [FAQ](#faq)
9. [License](#license)

---

## Introduction

The **PlayFabPal SDK** makes it easy to write Discord slash-command plugins that interact with PlayFab’s Admin & Server APIs.
It handles:

* **Slash command registration** per-guild
* **Payload construction & permission checks** via `SafePlayFab`
* **HTTP call injection** (so your bot can configure keys securely)
* **Modal dialogs** for richer input
* A simple **decorator** interface

---

## Installation

Copy the `playfabpal_sdk/` folder into your bot project.
Ensure your directory structure:

```
bot.py
playfabpal_sdk/
  ├── __init__.py
  ├── decorators.py
  ├── plugin_api.py
  └── safe_playfab.py
plugins/
  <guild_id>/
    myplugin/
      plugin.json
      main.py
```

---

## Quick Start

1. **In your bot loader** (e.g. `bot.py`), inject HTTP handler:

   ```python
   def call_handler(endpoint: str, body: dict) -> dict:
       url = f"https://{config['title_id']}.playfabapi.com/{endpoint}"
       headers = {
           "X-SecretKey": config["dev_key"],
           "Content-Type": "application/json"
       }
       return requests.post(url, json=body, headers=headers).json()

   api = PluginAPI(bot, guild, config, permissions)
   api.set_call_handler(call_handler)
   module.register(api)
   api.finalize_commands(guild=guild)
   ```

2. **In your plugin**:

   ```python
   from playfabpal_sdk import PluginAPI, PluginPayload

   def register(api: PluginAPI):

       @api.command("grantitem", "Grant an item to a player")
       async def grantitem(interaction, player_id: str, item_id: str):
           p = PluginPayload(interaction, api)
           p.log(f"Granting {item_id} → {player_id}")
           payload = api.playfab.grant_item(player_id, item_id)
           result = await p.playfab(payload)
           await p.send(f"✅ {result}")
   ```

---

## Core Concepts

### PluginAPI

The main entrypoint for plugins. Responsible for:

* Registering slash commands
* Injecting HTTP handler
* Exposing UI helpers

### PluginPayload

A thin wrapper around a single `Interaction`:

* `.send()` to reply
* `.send_modal()` to pop up a modal
* `.playfab()` to call your PlayFab handler

### SafePlayFab

Builds PlayFab request payloads **with permission checks**:

```python
payload = api.playfab.grant_item(user_id, item_id)
# => { "endpoint": "...", "body": { ... } }
```

### Decorators

* `@command(name, description)`: mark a function as a slash command
* `@on_event(event_name)`: (future) hook for events

---

## Full API Reference

### PluginAPI Methods

| Method                        | Signature                                  | Description                                  |
| ----------------------------- | ------------------------------------------ | -------------------------------------------- |
| `__init__`                    | see code                                   | Initialize SDK instance for a guild          |
| `set_call_handler`            | `(endpoint, body) -> dict`                 | Inject your HTTP handler                     |
| `command`                     | `(name, description) -> decorator`         | Register a function as `/name`               |
| `finalize_commands`           | `(guild=None)`                             | Called after register, no-op when using tree |
| `reply`                       | `(interaction, message, ephemeral=True)`   | Safely reply or follow up                    |
| `text_input`                  | `(label, placeholder, style) -> TextInput` | Shorthand for modal inputs                   |
| `create_modal`                | `(title, inputs, callback) -> Modal`       | Build & return a modal instance              |
| `Modal, TextInput, TextStyle` | —                                          | Shortcuts to `discord.ui` classes            |

### PluginPayload Methods

| Method       | Signature                   | Description            |
| ------------ | --------------------------- | ---------------------- |
| `send`       | `(message, ephemeral=True)` | Send a response        |
| `send_modal` | `(modal)`                   | Pop up a modal         |
| `playfab`    | `(payload) -> dict`         | Calls injected handler |
| `log`        | `(msg)`                     | Prefixed console log   |

### SafePlayFab Methods

| Method                  | Permissions             | Description                       |
| ----------------------- | ----------------------- | --------------------------------- |
| `grant_item`            | `grant_item`            | Admin/GrantItemsToUsers           |
| `grant_currency`        | `grant_currency`        | Admin/AddUserVirtualCurrency      |
| `subtract_currency`     | `subtract_currency`     | Admin/SubtractUserVirtualCurrency |
| `get_inventory`         | `get_inventory`         | Server/GetUserInventory           |
| `revoke_inventory_item` | `revoke_inventory_item` | Server/RevokeInventoryItem        |
| `ban_player`            | `ban_player`            | Admin/BanUsers                    |
| `revoke_ban`            | `revoke_ban`            | Admin/RevokeBans                  |
| `get_user_bans`         | `get_user_bans`         | Admin/GetUserBans                 |
| `delete_player`         | `delete_player`         | Admin/DeletePlayer                |
| `find_player`           | `find_player`           | Admin/GetPlayerCombinedInfo       |
| `get_player_info`       | `get_player_info`       | Admin/GetUserAccountInfo          |
| `get_catalog`           | `get_catalog`           | Admin/GetCatalogItems             |
| `set_motd`              | `set_motd`              | Server/SetTitleData               |

---

## Example Plugin

```python
# plugins/<guild_id>/grantbundle/main.py

from playfabpal_sdk import PluginAPI, PluginPayload
import discord

```
