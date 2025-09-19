# WardenGG Unlocker - Lua API Documentation

This document describes the public Lua functions exposed by WardenGG. All functions are safe to call; they return sensible defaults on error. Unless otherwise stated, numeric angles are in radians within [0, 2π).

Note: Web requests, files, and other operations are performed from the host process. Always validate return values.

## System Information

### WGG_GetHWID()
Returns the system hardware ID.
```lua
local hwid = WGG_GetHWID()
```

### WGG_IsInGame()
Checks if the player is currently in-game.
```lua
if WGG_IsInGame() then
    -- Player is in-game
end
```

## File System

### WGG_FileExists(path)
Checks if a file exists.
```lua
if WGG_FileExists("C:\\config.txt") then
    -- File exists
end
```

### WGG_FileRead(path)
Reads an entire file as a string. Returns "" if not found or on error.
```lua
local content = WGG_FileRead("C:\\data.txt")
```

### WGG_FileWrite(path, content, append)
Writes text to a file.
- append = true appends
- append = false overwrites
```lua
WGG_FileWrite("C:\\log.txt", "Hello", true)  -- append
WGG_FileWrite("C:\\log.txt", "Hello", false) -- overwrite
```

### WGG_DirExists(path)
Checks if a directory exists.

### WGG_CreateDir(path)
Creates a directory (recursively).
```lua
if not WGG_DirExists("C:\\MyPlugin") then
    WGG_CreateDir("C:\\MyPlugin")
end
```

## HTTP/Network

### WGG_HttpRequest(jsonRequest)
Sends an HTTP request asynchronously. The request is a JSON string with optional callback.

Supported fields:
- url (string)
- method (string, e.g., "GET", "POST")
- body (string)
- timeout (number, ms)
- headers (array of "Key: Value" strings)
- fields (object; converted to application/x-www-form-urlencoded when body is empty)
- callback (string; callback id used below)

The callback will be invoked from the game’s Lua with: callbackId(statusCode, responseText).

Example:
```lua
-- Setup callback
_G.__WGG_Callbacks = _G.__WGG_Callbacks or {}
_G.__WGG_Callbacks["myCallback"] = function(status, response)
    if status == 200 then
        print("Success: " .. response)
    else
        print("HTTP error: " .. tostring(status))
    end
end

-- Make request
local request = '{"url":"https://api.example.com","method":"GET","callback":"myCallback"}'
WGG_HttpRequest(request)
```

### WGG_UrlEncode(input)
URL-encodes a string.
```lua
local encoded = WGG_UrlEncode("hello world") -- "hello%20world"
```

## Authentication

### WGG_SetSessionToken(token)
Sets a session token used for authenticated HTTP requests.

### WGG_GetSessionToken()
Returns the current session token (or "").
```lua
WGG_SetSessionToken("your_jwt_token")
local token = WGG_GetSessionToken()
```

## Encryption

### WGG_AESEncrypt(data, key, iv)
Encrypts data using AES-256 (Windows CryptoAPI). Returns hex string of ciphertext.
- key: arbitrary string; internally hashed to derive the AES key
- iv: optional; may be empty
```lua
local encrypted = WGG_AESEncrypt("secret", "my32characterkey")
```

### WGG_AESDecrypt(dataHex, key, iv)
Decrypts hex-encoded ciphertext to plaintext. Returns "" on failure.
```lua
local decrypted = WGG_AESDecrypt(encrypted, "my32characterkey")
```

## JSON

Note: Encoding/decoding is lightweight. JsonDecode returns a Lua chunk that, when executed, yields a table.

### WGG_JsonEncode(data)
Pass-through utility; if given a raw JSON string, returns it; otherwise quotes the string.

### WGG_JsonDecode(jsonString)
Parses a JSON string and returns a Lua chunk string:
```lua
local luaChunk = WGG_JsonDecode('{"name":"test"}')
local tbl = loadstring(luaChunk)()
print(tbl["name"]) -- test
```

## Position/Player

### WGG_GetPlayerPosition()
Returns player position and validity.
- Returns: x, y, z, valid
```lua
local x, y, z, valid = WGG_GetPlayerPosition()
if valid then
    print(("Player at: %.2f, %.2f, %.2f"):format(x, y, z))
end
```

### WGG_GetTargetPosition()
Returns target position, target name and validity.
- Returns: x, y, z, name, valid
```lua
local x, y, z, name, valid = WGG_GetTargetPosition()
if valid then
    print(("Target %s at: %.2f, %.2f, %.2f"):format(name, x, y, z))
end
```

### WGG_GetPlayerTargetDistance()
Returns the distance between player and current target, or -1 if invalid.
```lua
local distance = WGG_GetPlayerTargetDistance()
```

## Facing Control

Angles are yaw in radians within [0, 2π). The ActivePlayer is always the unit that rotates. You aim at another unit’s coordinates (e.g., target).

### WGG_GetFacing(unitToken)
Returns the yaw of a unit. Common tokens:
- "player": the active/local player
- "target": current target (if any)

Returns a number (radians); 0 on error.
```lua
local playerYaw = WGG_GetFacing("player")
local targetYaw = WGG_GetFacing("target")
```

### WGG_SetFacing(unitToken, x, y, z, away)
Rotates the ActivePlayer to face toward the provided coordinates. If x and y are not provided or are 0, the function resolves them from unitToken.

- unitToken: selector for the aim point if coordinates are omitted; commonly "target" or "player"
- x, y, z: world coordinates to aim at; can be nil/0 to auto-resolve via token
- away: optional boolean; when true, the player faces away from the aim point (adds π)

Behavior:
- The player is always rotated.
- If x,y are valid numbers, they are used as the aim point.
- Otherwise, the aim point is resolved from unitToken’s position.
- Angle is normalized into [0, 2π).
- Returns true on success, false on failure.

Examples:
```lua
-- Face the current target
local ok = WGG_SetFacing("target")  -- x,y,z auto-resolved from target
-- Face away from the target (opposite direction)
local okAway = WGG_SetFacing("target", nil, nil, nil, true)

-- Face toward explicit coordinates
local ok2 = WGG_SetFacing("target", 1234.5, 678.9, 100.0, false)

-- Validate result
local px,py,pz,pok = WGG_GetPlayerPosition()
local tx,ty,tz,tok = WGG_GetTargetPosition()
if pok and tok then
    local desired = math.atan2(ty - py, tx - px)
    if desired < 0 then desired = desired + 2 * math.pi end
    local after = WGG_GetFacing("player")
    local diff = (after - desired) % (2 * math.pi)
    if diff > math.pi then diff = diff - 2 * math.pi end
    print(("diff=%.2f°"):format(diff * 180 / math.pi))
end
```

Notes:
- away flips the resulting direction by π (180°).
- If player and aim point are the same (dx=dy=0), the function returns false and does not write a facing.

## Object Management

### Counts
```lua
local totalObjects = WGG_GetAllObjects()
local totalUnits = WGG_GetAllUnits()
```

### Get units in range
```lua
local countUnits = WGG_GetUnitsInRange(30)
local countPlayers = WGG_GetPlayersInRange(30)
local countHostiles = WGG_GetHostileUnitsInRange(30)
```

### Closest units
```lua
local name, x, y, z = WGG_GetClosestHostileUnit()
local name2, x2, y2, z2 = WGG_GetClosestUnit()
```

### Object pointers and types
```lua
-- Get object by index
local objPtr = WGG_GetObjectWithIndex(0)

-- Query type and GUID
local typeId = WGG_ObjectType(objPtr)
local guid = WGG_ObjectGUID(objPtr)

-- Get all objects of a given type
local players = WGG_Objects(4) -- Type 4 = Players

-- Get common objects by token
local playerPtr = WGG_Object("player")
local targetPtr = WGG_Object("target")
```

### Object Types
- 0: Object
- 1: Item
- 2: Container
- 3: Unit
- 4: Player
- 5: GameObject
- 6: DynamicObject
- 7: Corpse

### NPC Storage Helpers
Store and recall a custom NPC object pointer (from enumeration).
```lua
WGG_SetNPCObject(objPtr)      -- Store NPC
local npcPtr = WGG_Object("npc") -- Retrieve stored NPC pointer
WGG_ClearNPCObject()          -- Clear stored NPC
```

## Protected Functions

### WGG_CallProtected(functionName, arg1, arg2, ...)
Calls protected WoW Lua functions in a taint-safe manner.
```lua
WGG_CallProtected("TargetUnit", "player")
WGG_CallProtected("CastSpellByName", "Fireball", "target")
```

## Quick Example
```lua
if not WGG_IsInGame() then return end

-- Load config
local config = "{}"
if WGG_FileExists("C:\\config.json") then
    config = WGG_FileRead("C:\\config.json")
end

-- Player position
local x, y, z, valid = WGG_GetPlayerPosition()
if valid then
    print(("Player at: %.1f, %.1f, %.1f"):format(x, y, z))
end

-- HTTP example
_G.__WGG_Callbacks = _G.__WGG_Callbacks or {}
_G.__WGG_Callbacks["test"] = function(status, response)
    print("API Response: " .. tostring(status))
end

WGG_HttpRequest('{"url":"https://api.test.com","callback":"test"}')

-- Face the current target
local ok = WGG_SetFacing("target")
print("Facing set:", ok)
```

## Error Handling
Functions return safe defaults on error:
- Strings: ""
- Booleans: false
- Numbers: 0 (or -1 for specific cases like distance)
- Coordinate getters include a trailing boolean indicating validity

Always check return values before use.
