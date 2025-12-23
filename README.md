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
