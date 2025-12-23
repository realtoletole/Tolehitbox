# ToleHitbox v1.0.0

A powerful and performant hitbox system for Roblox games, perfect for fighting, melee, or shooter mechanics. It provides reliable hit detection without using the unreliable `.Touched` event, avoiding issues like lag and tunneling.

## Key Features

- **Server or Client-side detection** – Run on the client for low-lag fast actions (anti-lag compensation)
- **Velocity prediction** – Compensates for fast-moving hitboxes/projectiles
- **Debug visuals** – Shows the active hitbox as a red part or sphere
- **Debounce system** – Prevents hitting the same target too quickly
- **Blacklisting** – Easily ignore specific characters or parts
- **Lifetime/Debris** – Automatic cleanup after a set time
- **Directional checks** – Use dot product to limit hits to a cone (e.g., only in front of the player)
- **Multiple shapes supported**:
  - Box (`InBox`)
  - Sphere (`InRadius`)
  - Simple distance check (`Magnitude`)
  - Custom part shape (`InPart`)
- Uses modern spatial queries (`GetPartBoundsInBox`, `GetPartsInPart`, etc.) for speed and accuracy

## Setup Requirements

1. Place the **ToleHitbox** ModuleScript inside the `ReplicatedStorage`.


## Modules Used
   - `Types` (ModuleScript with type definitions)
   - `Signal` (sleitnick's Signal module)
   - `Timer` (simple timer module)

## Basic Usage Example

```lua
local ToleHitbox = require(game.ReplicatedStorage.ToleHitbox) -- Adjust path as needed

-- Example: Sword swing hitbox
local hitboxParams = {
    SizeOrPart     = Vector3.new(4, 4, 8),           -- Box size
    SpatialOption  = "InBox",                       -- Use box overlap detection
    InitialPosition = tool.Handle.CFrame * CFrame.new(0, 0, -4),
    DebounceTime   = 0.3,                           -- Prevent rapid re-hits
    Debris         = 0.2,                           -- Auto-destroy after 0.2 seconds
    Debug          = true,                          -- Show red visual
    Blacklist      = {character},                   -- Ignore self
    LookingFor     = "Humanoid",                    -- Detect characters (use "Object" for parts)
}

local hitbox = ToleHitbox.new(hitboxParams)

-- Handle hits
hitbox.HitSomeone:Connect(function(hitCharacters)
    for _, character in hitCharacters do
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:TakeDamage(20)
            print("Hit", character.Name)
        end
    end
end)

-- Start detection
hitbox:Start()

-- Optional: Update position mid-animation
task.delay(0.1, function()
    hitbox:SetPosition(tool.Handle.CFrame * CFrame.new(0, 0, -6))
end)
```

## ToleHitbox.new(params) Parameters

| Parameter             | Type                          | Default       | Description                                                                                  |
|-----------------------|-------------------------------|---------------|----------------------------------------------------------------------------------------------|
| SizeOrPart            | Vector3 / number / BasePart   | -             | Shape definition. `Vector3` = box, `number` = radius/magnitude, `BasePart` = custom.        |
| SpatialOption         | string                        | Varies        | "InBox", "InRadius", "InPart", or "Magnitude" (must match `SizeOrPart`).                     |
| InitialPosition       | CFrame                        | `CFrame.new()`| Starting position and orientation.                                                          |
| DebounceTime          | number                        | 0             | Seconds before the same target can be hit again.                                            |
| Debris (or Lifetime)  | number                        | 0             | Auto-destroy after X seconds (0 = no auto-destroy).                                         |
| Debug                 | boolean                       | false         | Show red debug visual.                                                                       |
| Blacklist             | table of Instances            | nil           | Characters/parts to ignore.                                                                 |
| LookingFor            | string                        | "Humanoid"    | "Humanoid" (characters) or "Object" (raw parts).                                           |
| VelocityPrediction    | boolean                       | true          | Enable velocity compensation for fast movement.                                             |
| DotProductRequirement | table or nil                  | nil           | Directional cone restriction (see example below).                                           |
| UseClient             | Player or nil                 | nil           | Run detection on a specific client's machine (recommended for melee).                       |
| ID                    | string / number or nil        | nil           | Identifier for bulk clearing hitboxes.                                                      |

# Advanced Examples (Client Sided)
```lua
local hitboxParams = {
    SizeOrPart      = Vector3.new(5, 5, 10),
    SpatialOption   = "InBox",
    InitialPosition = someCFrame,
    UseClient       = player,     -- Runs detection on the attacker's client
    Debug           = true,
}

local hitbox, success = ToleHitbox.new(hitboxParams)
if success then
    hitbox.HitSomeone:Connect(function(hitCharacters)
        -- Server-authoritative damage logic here
    end)
    hitbox:Start()
end
```

# Directional Cone Hit (Only Hits in Front)

```lua
local hitboxParams = {
    -- ... other params
    DotProductRequirement = {
        PartForVector = tool.Handle,      -- Part used for direction reference
        VectorType    = "LookVector",     -- Options: "LookVector", "UpVector", "RightVector"
        DotProduct    = 0.7,              -- 0 = wide cone, 1 = very narrow (exact forward)
        Negative      = false,            -- Set true to check behind the vector
    },
}
```
# Attach to Moving Object (e.g., Projectile)

```lua
local hitbox = ToleHitbox.new(hitboxParams)
hitbox:Start()

-- Attach hitbox to a moving part (it will follow automatically)
hitbox:WeldTo(projectile.PrimaryPart, CFrame.new(0, 0, 0))  -- Optional offset CFrame

-- Later, if needed:
hitbox:Unweld()                    -- Detach from the part
hitbox:ChangeWeldOffset(CFrame.new(0, 1, 0))  -- Adjust attachment offset
```

## Hitbox Methods

| Method                           | Description                                                      |
|---------------------------------|------------------------------------------------------------------|
| `hitbox:Start()`                 | Begins hit detection.                                            |
| `hitbox:Stop()`                  | Pauses hit detection.                                            |
| `hitbox:SetPosition(cframe)`     | Manually moves the hitbox to a new `CFrame`.                    |
| `hitbox:SetDebug(bool)`          | Toggles the red debug visual on/off.                             |
| `hitbox:WeldTo(part, offset?)`   | Attaches the hitbox to a moving `BasePart` (follows automatically). |
| `hitbox:Unweld()`                | Detaches from any welded part.                                   |
| `hitbox:ChangeWeldOffset(cframe)`| Changes the offset when welded.                                   |
| `hitbox:ClearTaggedChars()`      | Manually resets debounce tags.                                   |
| `hitbox:Destroy()`               | Fully cleans up the hitbox (connections, parts, timers).        |

## ToleHitbox Static Functions

| Function                                | Description                                                      |
|----------------------------------------|------------------------------------------------------------------|
| `ToleHitbox.ClearHitboxesWithID(id)`    | Destroys all active hitboxes with the matching ID.              |
| `ToleHitbox.ClearClientHitboxes(player)`| Destroys all hitboxes running on the specified player's client. |
| `ToleHitbox.GetHitboxCache()`           | Returns a table containing all currently active hitboxes.       |



