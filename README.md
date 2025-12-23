
Overview of ToleHitbox
ToleHitbox is a powerful, performant hitbox system for Roblox games (especially fighting/melee/shooter games). It detects when something (like a sword swing or projectile) "hits" characters or objects without relying on Roblox's built-in Touched events (which are unreliable due to lag and tunneling).

Key features:

Works on server or client (great for reducing lag in fast-paced actions)
Supports velocity prediction (compensates for fast-moving hitboxes)
Debug visuals (shows the hitbox as a red part/sphere)
Debounce (prevents hitting the same target multiple times too quickly)
Blacklisting (ignore specific characters/parts)
Lifetime/Debris (auto-destroy after a set time)
Directional checks via dot product (e.g., only hit in front of the player)
Multiple shapes: Box, Sphere (radius), Magnitude (simple distance), or custom Part
It uses spatial queries like GetPartBoundsInBox, GetPartsInPart, etc., which are much faster and more reliable than raycasting or Touched.

Setup Requirements:

Place the module in ReplicatedStorage.
Set the Values of all Settings in the "Settings" Folder.

Modules Used:

Types (ModuleScript with type definitions)
Signal (sleitnick's Signal module)
Timer (a simple timer module)
ToleHitboxLocal (LocalScript that handles client-side hit detection)
