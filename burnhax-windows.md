# BURNHax Premium - Lua API Documentation
*Version: 1.2.5 Premium | Compatible with GentaHax | GTProxy*

Welcome to the **BURNHax Premium Lua API** documentation. This guide lists all available Lua functions you can use to create powerful automation scripts, UI overlays, and packet manipulators within the executor.

## 📌 Networking & Packets

### `sendPacket(type, text)` | `SendPacket(type, text)`
Sends a primitive string network packet to the server.
- **`type`** *(number)*: Packet Type (e.g., `2` for `NET_MESSAGE_GENERIC_TEXT`, `3` for `NET_MESSAGE_GAME_MESSAGE`).
- **`text`** *(string)*: Packet contents.
```lua
sendPacket(2, "action|input\n|text|Hello World!")
```

### `sendPacketRaw(to_client, data)` | `SendPacketRaw(to_client, data)`
Sends a raw `GameUpdatePacket` (`TankPacketStruct`).
- **`to_client`** *(boolean)*: `true` to send to the local client, `false` to send to the server.
- **`data`** *(table / GameUpdatePacket)*: The packet payload as a table or struct.

**Data Table Properties for Raw Packets:**
```lua
local pkt = { 
    type = 3,         -- Packet type
    value = 18,       -- Item ID or Value
    state = 0,        -- Packet flag/state
    x = 120.0,        -- Float X
    y = 140.0,        -- Float Y
    punchx = 5,       -- Integer X
    punchy = 5        -- Integer Y
}
sendPacketRaw(false, pkt)
```

### `sendVariant(table, net_id, delay)` | `SendVariant(function_name, arg1, arg2...)`
Constructs & sends a `VariantList` (`OnVarlist` / `OnTextOverlay`) packet. You can pass arguments individually or use a table.
```lua
-- Table Method (Compatible with GentaHax):
local var = {}
var[0] = "OnTextOverlay"
var[1] = "`9[BURNHax] `7Hello"
sendVariant(var)

-- Variadic Method:
SendVariant("OnConsoleMessage", "Hello from BURNHax!")
```

## 🪝 Event Hooks

### `AddHook(event_name, label, callback)`
Registers a function to intercept incoming and outgoing packets.
- **`event_name`**: `"OnVarlist"`, `"OnTextPacket"`, `"OnRawPacket"`.
- **`callback(v, netid)`**: The callback function.
  - For `OnVarlist`, `v` works like a 0-indexed array (`v[0]`, `v[1]`, etc.) returning the string or number values in the variant.

```lua
AddHook("OnVarlist", "my_hook", function(v, netid)
    -- Important: Safely check items. v[0] might be a string.
    if tostring(v[0]) == "OnTalkBubble" then
        OnConsoleLog("Bubble: ", v[2])
    end
end)
```
### `RemoveHook(label)` / `RemoveHooks()`
Removes a specific hook by its registered `label` or removes all hooks at once to keep the proxy clean.

## 🏃 Player & Automation Actions

### `punch(x, y)`
Punches a tile at the absolute pixel coordinate (TileX * 32, TileY * 32).
### `place(id, x, y)`
Places the specified `id` at the target pixel coordinate.
### `requestTileChange(dx, dy, fg)`
Alias for placing/punching tiles, fully compatible with GentaHax tile change logic for automations (e.g., auto-fishing, auto-planting).
### `drop(id, amount)`
Drops an item from your inventory into the world.
### `wear(id)`
Equips an item by its ID.
### `findPath(x, y)`
Teleports locally to the target Tile X, Y. (Server-side teleportation may require further exploits).
### `collect(radius)`
Automatically collects floating world objects within the specified pixel radius.

## 🌎 World & Entities

### `getWorld()`
Returns a `World` object (contains `.name`, `.width`, `.height`).
### `getLocal()` | `GetLocal()`
Returns a `NetAvatar` (Player) object of your own player.
- Properties: `.name`, `.netid`, `.posX`, `.posY` (Tile Pos), `.pos.x`, `.pos.y` (Pixel Pos).
### `getPlayerByNetID(id)`
Gets another player's `NetAvatar` data by their NetID.
```lua
local ply = getPlayerByNetID(1)
if ply then OnConsoleLog("Found: ", ply.name) end
```
### `getPlayerlist()`
Returns a Lua array/table of all active players in the current world.
### `getObjects()` / `getWorldObject()`
Returns a Lua array/table of all dropped items in the world. Each item has `.id`, `.x`, `.y`, `.count`, and `.oid`.
### `getInventory()`
Returns a Lua array of all items in your backpack (`.id`, `.count`).
### `getTile()`
Returns a Lua array containing all tiles in the current map. Each entry has `.fg`, `.bg`, and `.pos.x`, `.pos.y`.
### `checkTile(x, y)`
Gets information about a specific tile X, Y. **Note:** Returns `nil` if x or y is out of bounds!
```lua
local tile = checkTile(15, 20)
if tile then OnConsoleLog("Tile Foreground: ", tile.fg) end
```
### `getItemByID(id)`
Performs a lookup on the game's items.dat to fetch data. Returns a table containing `.name` and `.id`.
```lua
local itemInfo = getItemByID(7188)
OnConsoleLog("Item is: ", itemInfo.name)
```

## 🎭 Spoofing & Device Info

### `getMac()` / `getGid()`
Returns the proxy's current MAC Address and GrowID hash.
### `setMac(mac)` / `SetMAC(mac)` 
Overrides the MAC Address.
### `setGid(rid)` / `SetRID(rid)` 
Overrides the RID / Machine ID.
### `randomMac()` / `randomGid()`
Automatically regenerates a random hardware signature to evade bans.
```lua
randomMac()
randomGid()
OnConsoleLog("Device info spoofed!")
```

## 📝 Logging & UI

### `OnConsoleLog(...)` / `Log(...)` / `log(...)`
Prints a custom message to the GTProxy Executor UI Log and the in-game console. It automatically converts numbers, booleans, and tables to strings. You can pass multiple arguments separated by commas.
```lua
OnConsoleLog("Bot Name: ", getLocal().name, " Pos: ", getLocal().pos.x)
```
### `callToast(text, type)` / `doToast(text, type)`
Sends a visual toast banner to your local game client (Visible on screen).
```lua
callToast("ModFly Enabled!", 0)
```

## ⏱️ Threading & Utilities

### `runThread(function)`
Runs a piece of Lua code asynchronously without freezing the game UI, commonly used for while loops (e.g., Auto Farm).
```lua
runThread(function()
    while true do
        punch(getLocal().pos.x + 32, getLocal().pos.y)
        sleep(200) -- Sleep is REQUIRED in loops
    end
end)
```
### `sleep(milliseconds)`
Pauses the current thread execution. Only use inside `runThread()`.
### `getCurrentTimeInternal()`
Returns the system's current Epoch runtime in milliseconds.
