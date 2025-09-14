# WardenGG Unlocker - Lua API Documentation

## System Information

### WGG_GetHWID()
Returns the system hardware ID.
```lua
local hwid = WGG_GetHWID()
```

### WGG_IsInGame()
Checks if player is in-game.
```lua
if WGG_IsInGame() then
    -- Player is in-game
end
```

## File System

### WGG_FileExists(path)
```lua
if WGG_FileExists("C:\\config.txt") then
    -- File exists
end
```

### WGG_FileRead(path)
```lua
local content = WGG_FileRead("C:\\data.txt")
```

### WGG_FileWrite(path, content, append)
```lua
WGG_FileWrite("C:\\log.txt", "Hello", true) -- append
WGG_FileWrite("C:\\log.txt", "Hello", false) -- overwrite
```

### WGG_DirExists(path) / WGG_CreateDir(path)
```lua
if not WGG_DirExists("C:\\MyPlugin") then
    WGG_CreateDir("C:\\MyPlugin")
end
```

## HTTP/Network

### WGG_HttpRequest(jsonRequest)
Sends HTTP requests asynchronously.
```lua
-- Setup callback
_G.__WGG_Callbacks = _G.__WGG_Callbacks or {}
_G.__WGG_Callbacks["myCallback"] = function(status, response)
    if status == 200 then
        print("Success: " .. response)
    end
end

-- Make request
local request = '{"url":"https://api.example.com","method":"GET","callback":"myCallback"}'
WGG_HttpRequest(request)
```

### WGG_UrlEncode(input)
```lua
local encoded = WGG_UrlEncode("hello world") -- "hello%20world"
```

## Authentication

### WGG_SetSessionToken(token) / WGG_GetSessionToken()
```lua
WGG_SetSessionToken("your_jwt_token")
local token = WGG_GetSessionToken()
```

## Encryption

### WGG_AESEncrypt(data, key, iv) / WGG_AESDecrypt(data, key, iv)
```lua
local encrypted = WGG_AESEncrypt("secret", "my32characterkey")
local decrypted = WGG_AESDecrypt(encrypted, "my32characterkey")
```

## JSON

### WGG_JsonEncode(data) / WGG_JsonDecode(jsonString)
```lua
local json = WGG_JsonEncode('{"name":"test"}')
local luaCode = WGG_JsonDecode('{"name":"test"}')
local data = loadstring(luaCode)()
```

## Position/Player

### WGG_GetPlayerPosition()
```lua
local x, y, z, valid = WGG_GetPlayerPosition()
if valid then
    print("Position: " .. x .. ", " .. y .. ", " .. z)
end
```

### WGG_GetTargetPosition()
```lua
local x, y, z, name, valid = WGG_GetTargetPosition()
```

### WGG_GetPlayerTargetDistance()
```lua
local distance = WGG_GetPlayerTargetDistance() -- -1 if no target
```

## Object Management

### Object Counts
```lua
local totalObjects = WGG_GetAllObjects()
local totalUnits = WGG_GetAllUnits()
```

### Object Access
```lua
-- Get object by index
local objPtr = WGG_GetObjectWithIndex(0)

-- Get object info
local typeId = WGG_ObjectType(objPtr)
local guid = WGG_ObjectGUID(objPtr)

-- Get objects by type
local players = WGG_Objects(4) -- Type 4 = Players

-- Get common objects
local playerPtr = WGG_Object("player")
local targetPtr = WGG_Object("target")
```

### Object Types
- `0`: Object, `1`: Item, `2`: Container, `3`: Unit
- `4`: Player, `5`: GameObject, `6`: DynamicObject, `7`: Corpse

### NPC Storage
```lua
WGG_SetNPCObject(objPtr) -- Store NPC
local npcPtr = WGG_Object("npc") -- Retrieve stored NPC
WGG_ClearNPCObject() -- Clear stored NPC
```

## Protected Functions

### WGG_CallProtected(functionName, arg1, arg2, ...)
Calls protected WoW functions bypassing taint restrictions.
```lua
WGG_CallProtected("TargetUnit", "player")
WGG_CallProtected("CastSpellByName", "Fireball", "target")
```

## Quick Example
```lua
-- Basic plugin structure
if not WGG_IsInGame() then return end

-- Load config
local config = "{}"
if WGG_FileExists("C:\\config.json") then
    config = WGG_FileRead("C:\\config.json")
end

-- Get player position
local x, y, z, valid = WGG_GetPlayerPosition()
if valid then
    print("Player at: " .. x .. ", " .. y .. ", " .. z)
end

-- Make API call
_G.__WGG_Callbacks = _G.__WGG_Callbacks or {}
_G.__WGG_Callbacks["test"] = function(status, response)
    print("API Response: " .. status)
end

WGG_HttpRequest('{"url":"https://api.test.com","callback":"test"}')
```

## Error Handling
Functions return safe defaults on error:
- Strings: `""`
- Booleans: `false` 
- Numbers: `0` or `-1`

Always check return values before use.
