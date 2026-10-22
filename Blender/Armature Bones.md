---
title: Blender Armature Bones Explained
---

**Article Type:** #article_type/tips
**Complexity:** #complexity/beginner
**Software:** #software/blender
**Topics Covered:** #topics/animation
___
## What is a Bone?
Bone is a general term for representing a "thing" that can move (translate/rotate/scale) individual model part, such as a Propeller of a Fan, or a Leg of the creature.
## Bone's Head and Tail
In Blender, the bone has a Head and a Tail, which can be determined by the bone width (if the shape is default "Octahedral"):
![](5b10d950cb3169b2a0dd674fd25e1483.png)

It can be easily determined by the bone width if the Armature bone display is set to Octahedral, as on image:
![](d3cdc6458b37f766504e2c66a60653b4.png)

But different display modes will display the bones in a different way, such as Stick:
![](1fdffd6f039e129f233e799a6122d69e.png)

### Head and Tail of a Bone in Context of Starfield Rig File
In creating Starfield `.rig` file, the Bone's Pivot = Bone's Head, and bone length (the length of the connection from Head to Tail) shouldn't matter (still, not recommended to change anything about it on existing rigs/rigs that already have animations baked), as the Starfield Animation IO plugin will recalculate it automatically on export, so the bone length may change on export - most likely, the length of all bones will stay exactly *0.07 m*.
## Bone Roll
The bone's roll matters more. Bone roll is the 'invisible'/'barely visible' rotation of the bone that affects how it may be manipulated in the pose mode.

Such as, consider the following example, with a first bone having roll of 0, and a second bone having a roll of 90. **They appear identical:**
![](495a52ebe43105bee171fb45a4f84be8.png)

However, if you attempt to rotate the bones on some **local** axis (for example, X), they will rotate in a different way, which is happening because the bones have different roll:
![](fb4e50b41e63effd8c5c8582e878f20c.png)
