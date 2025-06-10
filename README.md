# PlayFabPal SDK Guide for Plugin Authors

Welcome to the **PlayFabPal SDK** — a plugin framework to expand your Discord bot with custom commands and PlayFab logic. This guide will help you build and publish plugins using the `/plugin load` and `/plugin remove` commands.

---

## Table of Contents

1. [Overview](#overview)
2. [Installation & Loading](#installation--loading)
3. [Plugin Folder Structure](#plugin-folder-structure)
4. [plugin.json Metadata](#pluginjson-metadata)
5. [Plugin Logic (main.py)](#plugin-logic-mainpy)

   * [The `register(api)` Function](#the-registerapi-function)
   * [Registering Commands](#registering-commands)
   * [PluginPayload Utilities](#pluginpayload-utilities)
   * [Making PlayFab Calls](#making-playfab-calls)
6. [UI: Modals and Inputs](#ui-modals-and-inputs)
7. [PlayFab Permissions](#playfab-permissions)
8. [Plugin Example](#plugin-example)
9. [FAQ](#faq)
10. [License](#license)
11. [Publishing Plugins](#publishing-plugins)

---

## Overview

Each plugin is a self-contained folder with its own commands and UI. Once loaded, plugins automatically register slash commands tied to your guild and gain access to PlayFab API calls — securely and safely.

---

## Installation & Loading

1. Prepare your plugin folder:

   ```
   myplugin/
   ├── plugin.json
   └── main.py
   ```
2. In Discord, run:

   ```
   /plugin load pluginID:versioncode
   ```
3. Confirm the upload. On success, commands will register and your plugin will activate.
4. To remove a plugin:

   ```
   /plugin remove myplugin
   ```

---

## Plugin Folder Structure

Each plugin must have:

```
<plugin_name>/
  plugin.json
  main.py
```

---

## plugin.json Metadata

```json
{
  "name": "grantitems_multi",
  "permissions": ["grant_item"]
}
```

Fields:

* `name`: Unique plugin name (used for `/plugin remove`)
* `permissions`: Allowed PlayFab operations (used by `SafePlayFab`)

---

## Plugin Logic (main.py)

### The `register(api)` Function

```python
from playfabpal_sdk import PluginAPI, PluginPayload

def register(api: PluginAPI):
    # Your logic here
```

* Called automatically on load
* `api` is scoped to the guild loading it

### Registering Commands

```python
@api.command("ping", "Replies pong")
async def ping(interaction):
    p = PluginPayload(interaction, api)
    await p.send("Pong!")
```

* Commands are defined with decorators
* First arg is always `interaction: discord.Interaction`

### PluginPayload Utilities

`PluginPayload` simplifies:

* `send(message, embed=None, ephemeral=False)`
* `send_modal(modal)`
* `playfab(payload)`
* `log(msg)`

```python
p = PluginPayload(interaction, api)
p.log("Hello")
await p.send("✅ Done")
```

### Making PlayFab Calls

```python
payload = api.playfab.grant_item(player_id, item_id)
result = await p.playfab(payload)
```

* `api.playfab` constructs the payload
* `p.playfab()` sends the request using your plugin's permissions

---

## UI: Modals and Inputs

You can show a modal using:

```python
inputs = [
  api.text_input("Player ID", "Enter the Player ID"),
  api.text_input("Item IDs", "Separate with commas")
]

async def handle_modal(p2, values):
    await p2.send(f"Got: {values}")

modal = api.create_modal("Grant Items", inputs, handle_modal)
await p.send_modal(modal)
```

---

## PlayFab Permissions

Define only what your plugin needs:

```json
"permissions": ["grant_item", "get_inventory"]
```

The SDK uses this to restrict unsafe access.

---

## Plugin Example

```python
# plugin.json
{
  "name": "grantitems_multi",
  "permissions": ["grant_item"]
}

# main.py
from playfabpal_sdk import PluginAPI, PluginPayload
import discord

def register(api: PluginAPI):

    @api.command("grant_multiple_items", "Grant multiple items")
    async def grant_modal(interaction: discord.Interaction):
        p = PluginPayload(interaction, api)
        inputs = [
            api.text_input("Player ID", "Enter the Player ID"),
            api.text_input("Item IDs", "Comma-separated items")
        ]

        async def handle_modal(p2, values):
            await p2.interaction.response.defer(ephemeral=True)
            pid = values["Player ID"].strip()
            items = [i.strip() for i in values["Item IDs"].split(",") if i.strip()]

            results = []
            for item in items:
                payload = api.playfab.grant_item(pid, item)
                res = await p2.playfab(payload)
                try:
                    grant = res["data"]["ItemGrantResults"][0]
                    ok = grant.get("Result")
                    results.append(f"• **{item}** → {'✅ Success' if ok else '❌ Failed'}")
                except Exception:
                    results.append(f"• **{item}** → ❌ Error")

            embed = discord.Embed(
                title="📜 Grant Results",
                description="\n".join(results),
                color=discord.Color.green() if all("✅" in r for r in results) else discord.Color.red()
            )
            embed.set_footer(text=f"Granted {len(items)} item(s) to {pid}")
            await p2.interaction.followup.send(embed=embed, ephemeral=True)

        modal = api.create_modal("Grant Items", inputs, handle_modal)
        await p.send_modal(modal)
```

---

## FAQ

**Q:** Can one plugin work in multiple servers?
**A:** Yes. Just load it in each server using `/plugin load`.

**Q:** Can plugins be updated?
**A:** Yes. Just re-upload with a new version ID.

---

## License

MIT License © Troyy\_

---

## Publishing Plugins

1. Zip your folder (`plugin.json` + `main.py`)
2. Submit via the [Plugin Portal](#)
3. Once approved, install it using:

   ```
   /plugin load <plugin_id>
   ```
