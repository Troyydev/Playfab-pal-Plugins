# PlayFabPal Plugin SDK Documentation

Welcome to the **PlayFabPal Plugin SDK**! This guide will walk you through everything you need to know to build and publish plugins for PlayFabPal.

---

## Table of Contents

1. [Overview](#overview)
2. [Setup & Installation](#setup--installation)
3. [Plugin Structure](#plugin-structure)
4. [Writing Your First Plugin](#writing-your-first-plugin)

   * [Register Function](#register-function)
   * [Slash Commands](#slash-commands)
   * [PluginPayload Usage](#pluginpayload-usage)
   * [Calling PlayFab](#calling-playfab)
5. [Modals & User Input](#modals--user-input)
6. [SafePlayFab Reference](#safeplayfab-reference)
7. [Advanced Features](#advanced-features)

   * [Event Hooks](#event-hooks)
   * [Permissions](#permissions)
8. [Plugin Packaging & Deployment](#plugin-packaging--deployment)
9. [Examples](#examples)
10. [FAQ](#faq)
11. [License](#license)

---

## Overview

The PlayFabPal Plugin SDK enables users to create custom Discord slash-command plugins that interface with PlayFab's Admin and Server APIs. Plugins are sandboxed, per-guild, and leverage convenience helpers for UI elements and API calls.

### Key Components

* **PluginAPI**: Core SDK class passed into each plugin's `register()` function.
* **PluginPayload**: Simplifies replies, modals, and PlayFab calls inside a command.
* **SafePlayFab**: Constructs PlayFab API payloads with permission checks.
* **Decorators**: `@api.command` and `@on_event` to register commands and event handlers.

---

## Setup & Installation

1. **Copy SDK folder** into your bot project:

   ```
   playfabpal_sdk/
     ├── __init__.py
     ├── decorators.py
     ├── plugin_api.py
     └── safe_playfab.py
   ```
2. **Ensure SDK is importable** (e.g., alongside `bot.py`).
3. **Plugins directory** structure:

   ```
   plugins/<guild_id>/<plugin_name>/
     ├── plugin.json
     └── main.py
   ```
4. **Restart** your bot; it will auto-load plugins.

---

## Plugin Structure

Each plugin folder must contain:

* **plugin.json** (metadata):

  ```json
  {
    "name": "MyPlugin",
    "permissions": ["grant_item", "get_player_info"]
  }
  ```
* **main.py**: Defines `register(api: PluginAPI)`.

---

## Writing Your First Plugin

### Register Function

```python
# main.py
from playfabpal_sdk import PluginAPI, PluginPayload

def register(api: PluginAPI):
    # register commands here
    pass
```

The `register(api)` function is called once per guild.

### Slash Commands

Use `@api.command(name, description)`:

```python
@api.command("ping", "Replies with Pong!")
async def ping(interaction: discord.Interaction):
    pl = PluginPayload(interaction, api)
    await pl.send("Pong! 🎉")
```

* Function signature must start with an `Interaction` parameter.
* Additional parameters map to slash options by name and type.

### PluginPayload Usage

Inside a command, instantiate:

```python
def some_cmd(interaction, arg1: str):
    pl = PluginPayload(interaction, api)
    pl.log("debug message")
    await pl.send("Hello!")
```

Methods:

* `pl.send(message: str, ephemeral: bool = True)`
* `pl.send_modal(modal_instance)`
* `pl.playfab(payload: dict) -> dict` to call PlayFab
* `pl.log(msg: str)` to console

### Calling PlayFab

Construct payload via `api.playfab` helpers:

```python
payload = api.playfab.grant_item(player_id, item_id)
result = await pl.playfab(payload)
await pl.send(f"Result: {result}")
```

---

## Modals & User Input

Create interactive forms by subclassing `api.Modal`:

```python
class MyModal(api.Modal):
    def __init__(self):
        super().__init__(title="My Form")
        self.field = api.TextInput(label="Field", placeholder="Enter...")
        self.add_item(self.field)

    async def on_submit(self, interaction):
        pl = PluginPayload(interaction, api)
        value = self.field.value
        await pl.send(f"You typed: {value}")

@api.command("show_form")
async def show_form(interaction: discord.Interaction):
    pl = PluginPayload(interaction, api)
    await pl.send_modal(MyModal())
```

---

## SafePlayFab Reference

| Method                           | Permission              | Description                       |
| -------------------------------- | ----------------------- | --------------------------------- |
| `grant_item(user, item, ...)`    | `grant_item`            | Admin/GrantItemsToUsers           |
| `grant_currency(user, amt, ...)` | `grant_currency`        | Admin/AddUserVirtualCurrency      |
| `subtract_currency(...)`         | `subtract_currency`     | Admin/SubtractUserVirtualCurrency |
| `get_inventory(user)`            | `get_inventory`         | Server/GetUserInventory           |
| `revoke_inventory_item(...)`     | `revoke_inventory_item` | Server/RevokeInventoryItem        |
| `ban_player(user, reason)`       | `ban_player`            | Admin/BanUsers                    |
| `revoke_ban(user)`               | `revoke_ban`            | Admin/RevokeBans                  |
| `get_user_bans(user)`            | `get_user_bans`         | Admin/GetUserBans                 |
| `delete_player(user)`            | `delete_player`         | Admin/DeletePlayer                |
| `find_player(user)`              | `find_player`           | Admin/GetPlayerCombinedInfo       |
| `get_player_info(user)`          | `get_player_info`       | Admin/GetUserAccountInfo          |
| `get_catalog(version)`           | `get_catalog`           | Admin/GetCatalogItems             |
| `set_motd(key, value)`           | `set_motd`              | Server/SetTitleData               |

---

## Advanced Features

### Event Hooks

Use `@on_event(event_name)` to listen:

```python
from playfabpal_sdk import on_event

@on_event("member_join")
async def welcome(api, member):
    # handle event
    pass
```

### Permissions

List required PlayFab permissions in `plugin.json`:

```json
{ "permissions": ["grant_item", "get_inventory"] }
```

SDK enforces these at runtime.

---

## Plugin Packaging & Deployment

1. ZIP your plugin folder (retain `plugin.json` & `main.py`).
2. Upload to your bot’s `plugins/<guild_id>/` directory.
3. Bot will auto-reload on next start.

---

## Examples

Refer to the `examples/` folder in this repo for complete plugins:

* `grantitemv2`
* `inventory_viewer`

---

## FAQ

**Q**: Can I share plugins across guilds?
**A**: Yes—place in each `plugins/<guild_id>/` directory.

**Q**: How to debug?
**A**: Use `pl.log("msg")` to print namespaced logs.

---

## License

MIT © YourName
