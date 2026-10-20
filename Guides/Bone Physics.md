---
title: Bone Physics
---

How Starfield moves bones with springs and pose correctives (BMOD records), read from the game executable
(Starfield 1.16.244). The same system drives hair, capes, pouches and any other jiggle or sway.

## The records
Bone modifiers are `BMOD` records. Two kinds matter here:
- **SpringBone:** moves a target bone like a damped spring that follows a driver bone.
- **PoseDeformer:** offsets a target bone by a fixed amount depending on how a driver bone is turned.

The `DATA` field starts with the kind and two bone names. Strings are a u32 length (counting the closing zero), the
characters, then a zero.
- **SpringBone:** `SpringBone`, driver, target, then 17 floats: max animation distance; strength X, Y, Z, W; damping
  X, Y, Z, W; scale X, Y, Z, W; max distance X, Y, Z, W; then one byte (0).
- **PoseDeformer:** `PoseDeformer`, driver, target, then 15 floats: max animation distance; vector X, Y, Z; position
  X, Y, Z (m); angle X, Y, Z (degrees); scale X, Y, Z; inner angle; outer angle (degrees); then one byte for the
  driver axis (0 = X, 1 = Y, 2 = Z).

## Where the list lives
- A race or an armor addon lists its modifiers (`BNAM`).
- **A list only builds modifiers whose BMOD records are in the same plugin as the list** (confirmed in game). A list
  in one plugin that points at BMODs in another gets nothing.
- An armor addon's modifiers only drive nodes inside that addon's own mesh. Modifiers for the body's bones under
  clothes belong on the race.
- A modifier listed on an armor addon runs while that addon is worn, PoseDeformers included (confirmed in game).
- A plugin that copies a record from the localized `Starfield.esm` (a race, for example) holds string IDs, not text.
  It needs the localized flag (0x80) and its own `<plugin>_<language>.strings/.dlstrings/.ilstrings`, or the names
  show blank in game.

## How a spring runs
Every per-axis value works **in the driver bone's frame**. An empty driver means the bone's parent. Each frame:
1. The offset (where the bone should be minus where it is) grows by how far the driver moved since the last frame.
2. It's solved as a damped spring with RK4: x'' = -Strength * x - 2 * Damping * sqrt(Strength) * x', in
   n = max(1, ceil(frame time * 0.63662 * sqrt(largest Strength))) sub-steps. After each sub-step the velocity is
   multiplied by Scale (1 = no extra loss).
3. The offset is clamped per axis to **max distance x sub-step time** (metres).
4. The bone sits at its rest position minus the offset. A spring at rest (every axis within 0.5 mm and barely moving)
   is skipped.

What the numbers mean:
- **Strength** is stiffness. Bounce frequency = sqrt(Strength) / 2 pi Hz.
- **Damping** is a damping ratio: at 1 there's no overshoot.
- **Max animation distance** is how far from the camera an actor can be and still run the modifier.
- Game defaults: max animation distance 20, strength 500, damping 1, scale 0.25, max distance 1000.

## Hard risks
- **Frame rate changes the look.** The clamp is max distance x sub-step time, so max distance 1.0 allows 1.67 cm at
  60 fps, 3.3 cm at 30 fps and 0.7 cm at 144 fps. To tune for another frame rate, scale max distance by fps / 60.
- **Steady motion pins soft springs.** Damping works on absolute motion, so at a steady speed v a spring lags by
  2 * Damping * v / sqrt(Strength) and sits at its clamp while walking or running. Good for hair and capes trailing
  behind; keep side and front/back limits small on anything that should stay close to the body.
- **A missing bone means no spring.** If the driver or target doesn't exist on the actor, the modifier is never
  created.
- **Never put a SpringBone and a PoseDeformer on the same bone.** A PoseDeformer sets the local position outright.
- **Another mod overriding the same race** drops your list unless a patch merges both, with the list still pointing
  at BMODs in the patch's own plugin.

## PoseDeformer weights
- The weight comes from the angle between the driver's local axis (the axis byte) and the vector, given in the
  driver's **parent** frame: 1 inside the inner angle, falling linearly to 0 at the outer angle.
- The target's local position = rest + weight x position, rotation = rest + weight x angle, scale = 1 + weight x
  scale.
- Several PoseDeformers on one bone add up for position and rotation. For scale the last one wins.
