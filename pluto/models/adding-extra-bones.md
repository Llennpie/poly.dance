---
title: Adding Extra Bones
layout: home
parent: Models
nav_order: 1
---

Pluto models can have extra animatable bones. This guide will demonstrate how to set them up in PFast64, and basic usage at runtime.

---

## **Overview**

By default, the vanilla model's armature has 20 animated bones - named **000-offset.[index]** in Blender. These bones are called **Animated Parts** and can be influenced by vanilla or custom animations.

Other bones in the blend, such as **000-displaylist** or **002-rotate** cannot be animated, so for now let's pretend they don't exist.

Pluto supports two kinds of extra bones beyond the standard 20 vanilla Mario bones:

| Bone Type | SFast64 Label | Geo Command | Purpose |
|-----------|--------------|-------------|---------|
| **Extra Bone** | `Extra Bone (Saturn)` | `GEO_MCOMP_EXTRA` | Extra animatable bone |
| **Wiggle Bone** | `Wiggle Bone (Pluto)` | `GEO_EXTRA_WIGGLE` | Extra animatable bone with per-bone spring/lag physics and wind sway |

Just like Animated Parts, these bones can be moved, rotated or scaled within Blender - but are designed to **animate in-game only when in use** to avoid disrupting the **bone order**. This is ideal when model authors want to animate extra limbs or items, **without breaking compatibility with vanilla animations**.

**Wiggle bones** are identical to extra bones, but run simple gravity physics in and outside custom animations. They can also be affected by wind (ideal for hair or loose clothing).

---

## **Setting Up Bones in SFast64**

In Blender, Bones can be created by selecting an Armature, entering **Edit Mode**, and selecting **Add -> Single Bone**. Then in the **Bone Properties** panel, assign a parent (such as `000-offset.002` for the head) and enable **Deform**.

### Armature bone type

With your model's armature in **Pose Mode**, select your added bone, then in the **Bone Properties** panel change the `Geolayout Command` dropdown to either:

- **`Extra Bone (Saturn)`** - MComp bone, geo command `0x22`
- **`Wiggle Bone (Pluto)`** - Wiggle bone, geo command `0x23`

{: .warning }
The parent of a Wiggle Bone must be an **Animated Part (0x13)**, another **Extra Bone**, or another **Wiggle Bone**. If the parent is anything else the wiggle physics will not activate at runtime.

### Wiggle Bone parameters

When `Wiggle Bone (Pluto)` is selected, an extra panel of physics parameters appears:

| Property | Internal name | Default | Description |
|----------|--------------|---------|-------------|
| **Smoothness** | `wiggle_smooth` | `10` | Lag speed - `0` = frozen, `100` = instant snap |
| **Max Distance** | `wiggle_max_dist` | `25` | Maximum distance in units the bone can drift from its parent |
| **Snap Smooth** | `wiggle_snap_smooth` | `80` | Catch-up speed used when changing directions or sudden animation changes |
| **Spring Stiffness** | `wiggle_spring_k` | `30` | How "bouncy" the bone is, like those rubber door stoppers - `0` = disabled |
| **Spring Damping** | `wiggle_spring_damp` | `65` | Bounce duration - `0` = bounces forever, `100` = disabled |

All five parameters are stored as integers x100 in the binary geo layout. The engine divides by 100 at load time to produce the actual floats used in physics. The in-game defaults (from `geo_commands.h`) are:

```c
#define WIGGLE_SMOOTH_DEFAULT      10   /* 0.10f */
#define WIGGLE_MAX_DIST_DEFAULT    25   /* 25.0f */
#define WIGGLE_SNAP_SMOOTH_DEFAULT 80   /* 0.80f */
#define WIGGLE_SPRING_K_DEFAULT    30   /* 0.30f */
#define WIGGLE_SPRING_DAMP_DEFAULT 65   /* 0.65f */
```

## Animating Extra Bones (PAnim)

The standard **20-set (vanilla armature) animations** will work seamlessly on complex armatures, as wiggle/extra bones are skipped if the animation data fails to cover all Animated Parts.

**However,** animations that were animated on these complex armatures **will look crumpled or broken on other armatures** - so keep that in mind when you're designing an animation set.

A model's bone count can also change when switch states (such as hand poses or cap on/off) are involved, so using dummy/unused extra bones to balance out each potential state is highly recommended.