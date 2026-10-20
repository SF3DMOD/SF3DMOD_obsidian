---
title: Female Skeleton
---

Starfield's human female skeleton, and how to add bones to it without breaking animations, outfits or other mods.
Read from Starfield 1.16.244.

## The two files
Both live in `meshes/actors/human/characterassets/female/`.

| File | Nodes | Used for |
|---|---|---|
| `skeleton.nif` | 117 | characters without face bones |
| `skeleton_facebones.nif` | 163 | characters with face bones |

- The two are not copies of each other with face bones added. Shared bones can sit up to 1.3 mm apart, and `L_Arm` is
  turned differently. A bone added to both files needs its own local transform in each file if it has to land on
  the same world position.
- `HumanExportRoot` carries the skeleton's `BSBound`, `BSXFlags` and `BSConnectPoint::Parents`.
- The game's own bone scales are 0.99999 to 1.00001, not exactly 1.

## Transforms
- A node's rotation is 9 floats, row-major. The **columns** are the bone's X, Y and Z axes in its parent's frame:
  a point `p` in the bone's space sits at `t + s * R * p` in the parent's.
- A bone points along its own local X axis.
- World space: metres, Z up, the character faces +Y, her left is -X.
- Blender can't hold a bone whose X axis points to -Y exactly. Four game bones come back up to 0.004 off in rotation
  after a round trip, so write game transforms from data, not from an edited Blender armature.

## Adding bones
- Append new nodes. Never rename, re-parent or move a vanilla bone: every animation, outfit and mod made for the
  vanilla skeleton then keeps working.
- Add the same bones to both files.
- A bone that no mesh is weighted to costs nothing.
- A bone modifier (spring or pose corrective) whose driver or target bone is missing is never created, so a plugin
  with springs is safe on a skeleton that lacks their bones. See [[Bone Physics]].

## Conflicts
- Every skeleton replacer replaces the same two files, so only one can win. Two mods that each add bones need one
  merged skeleton.
- Loose files in `Data` win over files in a BA2. A forgotten loose skeleton silently replaces the one in an archive.

## Hard risks
- A mesh weighted to a bone the installed skeleton doesn't have loses those weights in game.
- An outfit built against one bone-adding skeleton can break on another if the bone names or rest positions differ.
