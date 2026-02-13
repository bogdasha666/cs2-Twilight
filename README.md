# Lua Scripts (Twilight)

This document describes the Lua API exposed by Twilight.

## Load flow

1. Open `Menu -> Lua Scripts`.
2. Click `Load`.
3. Select a `.lua` file.
4. If the script defines categories/actions, they appear in the main menu list.

## Twilight API

Twilight injects a global table named `twilight`.

### `twilight.send_command(command, slot)`

- `command` (`string`): CS2 command text.
- `slot` (`number`, optional): config slot `1..12` used by `CommandsSender`.
- Default slot: `7`.

Example:

```lua
twilight.send_command("say hello from lua")
twilight.send_command("slot5; drop; lastinv", 7)
```

### `twilight.get_gsi_payload()`

Returns the last raw GSI payload as JSON string.

### `twilight.get_gsi_value(path)`

Returns a value by dot-path from latest GSI JSON.

- Object path example: `player.state.health`
- Array path example: `map_round_wins.0`

If path is invalid, returns `nil`.

### `twilight.add_category(name)`

Creates a category section in the main menu.

### `twilight.add_action(category, label, callback, options)`

Adds an action row in the selected category.

- `category` (`string`)
- `label` (`string`)
- `callback` (`function`)
- `options` can be:
  - `number`: hotkey virtual-key code (legacy short form)
  - `table`:
    - `hotkey` (`number`, optional)
    - `configurable` (`bool`, default `true`)
    - `always` (`bool`, default `false`)
    - `enabled` (`bool`, default `true`)

Modes:
- Configurable action: appears with checkbox + settings popup (hotkey can be changed in menu).
- Non-configurable + always action: no settings UI, callback runs every tick while script is loaded.

### `twilight.is_key_down(vk)`

Returns `true` while a key is pressed.

- `vk` is a Windows Virtual-Key code (for example `0x56` for `V`).

### `twilight.log(message)`

Writes message to Twilight console/log.

## Optional lifecycle callbacks

You can define any of these global functions:

- `on_load()` called once after script file executes.
- `on_tick()` called every render tick.
- `on_unload()` called when script is unloaded.

## Full example

```lua
twilight.add_category("Movement")
twilight.add_action("Movement", "Jump Throw", function()
    twilight.send_command("jump 1 1 0", 7)
    twilight.send_command("attack -999 1 0", 7)
    twilight.send_command("jump -999 1 0", 7)
end, { configurable = true }) -- set hotkey later in menu

twilight.add_category("Info")
twilight.add_action("Info", "Print Health", function()
    local hp = twilight.get_gsi_value("player.state.health")
    if hp == nil then
        twilight.log("health is not available yet")
        return
    end
    twilight.log("health = " .. tostring(hp))
end)

twilight.add_action("Info", "Always Tick Demo", function()
    -- runs every tick
end, { configurable = false, always = true })

function on_tick()
    local bomb = twilight.get_gsi_value("round.bomb")
    if bomb == "planted" then
        -- custom logic here
    end
end
```
