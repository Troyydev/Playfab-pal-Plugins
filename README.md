# PlayFabPal SDK Guide for Plugin Authors

Welcome to the **PlayFabPal SDK**, designed for **plugin authors** to extend your bot via `/plugin load` and `/plugin remove`. This document explains everything you need—**no access to `bot.py` required**.

---

## Table of Contents

1. [Overview](#overview)
2. [Installation & Loading](#installation--loading)
3. [Plugin Directory Structure](#plugin-directory-structure)
4. [plugin.json Metadata](#pluginjson-metadata)
5. [Writing Your Plugin (main.py)](#writing-your-plugin-mainpy)

   * [The `register(api)` Function](#the-registerapi-function)
   * [Slash Commands](#slash-commands)
   * [Using PluginPayload](#using-pluginpayload)
   * [SafePlayFab Calls](#safeplayfab-calls)
6. [Adding & Removing Plugins via Slash](#adding--removing-plugins-via-slash)
7. [Modals & UI Interaction](#modals--ui-interaction)
8. [Permissions](#permissions)
9. [Examples](#examples)
10. [FAQ](#faq)
11. [License](#license)

---

## Overview

Plugin authors can build standalone folders containing metadata and code—**no edits to the main bot**. Use the `/plugin load` command to upload your plugin, `/plugin remove` to uninstall.

## Installation & Loading

1. **Prepare your plugin folder** locally:

   ```
   my_plugin/
     ├── plugin.json
     └── main.py
   ```
2. In Discord, run:

   ```
   /plugin load path_or_url_to/my_plugin.zip
   ```
3. Bot responds with ✅ on success. Your commands are now registered.
4. To uninstall:

   ```
   /plugin remove my_plugin
   ```

---

## Plugin Directory Structure

Every plugin **must** have:

```
<plugin_name>/
  plugin.json
  main.py
```

## plugin.json Metadata

```json
{
  "name": "MyPlugin",         // Unique plugin name
  "permissions": [            // PlayFab permissions required
    "grant_item",
    "get_inventory"
  ]
}
```

* `name`: identifier used by `/plugin remove`.
* `permissions`: limiting PlayFab calls available.

---

## Writing Your Plugin (main.py)

All code lives in `main.py`. You import only from `playfabpal_sdk`.

### The `register(api)` Function

```python
from playfabpal_sdk import PluginAPI, PluginPayload

def register(api: PluginAPI):
    # your registration logic here
```

* Called automatically after `/plugin load`.
* `api` instance is tied to the guild.

### Slash Commands

Use `@api.command("name", "description")`:

```python
@api.command("ping", "Replies pong")
async def ping(interaction: discord.Interaction):
    p = PluginPayload(interaction, api)
    await p.send("Pong!")
```

* The function’s signature determines slash-options.
* First parameter must be `interaction`.

### Using PluginPayload

```python
p = PluginPayload(interaction, api)
p.log("Debug message")
await p.send("Hello!")
```

Methods:

* `send(msg, ephemeral=True)`
* `send_modal(modal)`
* `playfab(payload)`
* `log(msg)`

### SafePlayFab Calls

Construct API payloads:

```python
payload = api.playfab.grant_item(player_id, item_id)
result = await p.playfab(payload)
await p.send(f"Result: {result}")
```

---

## Adding & Removing Plugins via Slash

* **Load**: `/plugin load url_or_attachment`
* **Remove**: `/plugin remove <plugin_name>`
* **List**: `/plugin list`

---

## Modals & UI Interaction

Create modals by subclassing `api.Modal`:

```python
class MyModal(api.Modal):
    def __init__(self):
        super().__init__(title="Form")
        self.input = api.TextInput("Label", "Placeholder")
        self.add_item(self.input)

    async def on_submit(self, interaction):
        p = PluginPayload(interaction, api)
        await p.send(f"You entered: {self.input.value}")

@api.command("show_form")
async def show_form(interaction: discord.Interaction):
    p = PluginPayload(interaction, api)
    await p.send_modal(MyModal())
```

---

## Permissions

List only the SafePlayFab permissions your plugin needs:

```json
"permissions": ["grant_item","get_inventory"]
```

---

## Examples

**GrantItem Plugin**

```python
# plugin.json
{ "name": "grantitem", "permissions": ["grant_item"] }

# main.py
def register(api):
    @api.command("grantitem","Grant item")
    async def grant(interaction, player_id: str, item_id: str):
        p = PluginPayload(interaction,api)
        payload=api.playfab.grant_item(player_id,item_id)
        r=await p.playfab(payload)
        await p.send(f"✅ {r}")
```

---

## FAQ

**Q:** Can one plugin serve multiple guilds?
**A:** Yes—run `/plugin load` in each server.

---

## License

MIT © YourName
