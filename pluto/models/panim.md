---
title: PAnim
layout: home
parent: Models
nav_order: 2
---

# .panim File Format

`.panim` is Pluto's binary animation format. It extends the SM64 DMA animation layout (values + indices) with a metadata header and optional per-bone translation and scale tracks stored in tagged sections appended after the core data.

All multi-byte integers are **big-endian** unless otherwise noted.

---

## High-Level Layout

```
[Header]          - fixed 103 bytes
[values section]  - tagged: "values" + raw s16 stream
[indices section] - tagged: "indices" + raw s16 stream
[translations]    - optional tagged section
[scales]          - optional tagged section
```

---

## Header (bytes 0x00 – 0x66)

| Offset | Size | Type | Field | Notes |
|--------|------|------|-------|-------|
| `0x00` | 64 | `char[64]` | **Name** | UTF-8, null-padded to 64 bytes |
| `0x40` | 32 | `char[32]` | **Author** | UTF-8, null-padded to 32 bytes |
| `0x60` | 1 | `u8` | **Looping** | `0xA6` = looping, `0x00` = one-shot |
| `0x61` | 2 | `u16 BE` | **Length** | Frame count (exclusive end), i.e. `loopEnd` |
| `0x63` | 1 | `u8` | **Node count** | Number of animatable bones, capped at 255 |

Total header size: **100 bytes** (`0x64`). The values section tag begins immediately after at offset `0x64`.

> `BoneCount` is derived at load time from the indices section: `indices.size() / 6 - 1`.

---

## Values Section

```
"values"  <-- 6 ASCII bytes, no length prefix
<s16 stream, big-endian>
```

Begins at offset `0x64`. The literal ASCII string `values` (6 bytes) acts as the section tag. It is followed immediately by the raw rotation value table - a flat array of `s16` big-endian shorts, identical in layout to the SM64 DMA animation values array.

The section ends when the loader finds the literal string `indices` scanning forward two bytes at a time.

---

## Indices Section

```
"indices"  <-- 7 ASCII bytes, no length prefix
<s16 stream, big-endian>
```

Following the values section is the 7-byte tag `indices`. The raw index table follows - also a flat array of `s16` big-endian shorts matching the SM64 DMA animation index array.

Each animatable bone occupies **6 shorts** (3 axes * 2 shorts each: `numFrames` + `startOffset`). The root bone's 6 shorts come first, followed by one entry per non-root bone. Therefore:

```
BoneCount = (indices.size() / sizeof(s16)) / 6 - 1
```

The section ends when either:
- The 12-byte tag `translations` is found, or
- EOF is reached.

---

## Translations Section (optional)

Stores per-bone, per-frame local translation offsets for all non-root animatable bones. Only present when at least one bone has a non-zero translation track.

```
"translations"       <-- 12 ASCII bytes
[2] frame_count      u16 BE  - number of frames in all tracks
[2] bone_count       u16 BE  - number of non-root animatable bones tracked
[ceil(bone_count/8)] bitmask - one bit per bone; 1 = data present for that bone
[data]               s16 x 3 x frame_count per present bone, big-endian
```

### Bitmask

`ceil(bone_count / 8)` bytes immediately follow the header. Bit `bi % 8` of byte `bi / 8` is `1` if bone `bi` has translation data; bones with all-zero translations are omitted and their bit is `0`.

### Data layout

Only bones whose bitmask bit is `1` have data. They are written in ascending bone index order. For each present bone, `frame_count` frames of `(tx, ty, tz)` are written:

```
for each present bone (in ascending index order):
    for each frame in [0, frame_count):
        s16 tx   (big-endian)
        s16 ty
        s16 tz
```

Total data size for one present bone: `frame_count x 6` bytes.

### Addressing

In memory (`PlutoAnim::Translations`), the flat array is indexed bone-major:

```
index = (bone_idx * frame_count + frame) * 3 + axis
```

where `bone_idx` is 0-based from the first non-root bone (bone 1 in the skeleton), `frame` is the current animation frame, and `axis` is 0=X, 1=Y, 2=Z.

### Units

Raw `s16` game units, matching the existing SM64 translation scale. Values are read directly into `Vec3s` and added to the bone's animated translation each frame. Clamped to `[-32768, 32767]` at export.

---

## Scales Section (optional)

Stores per-bone, per-frame scale factors for all non-root animatable bones. Only present when at least one bone has a non-identity scale track.

```
"scales"             <-- 6 ASCII bytes
[2] frame_count      u16 BE
[2] bone_count       u16 BE
[ceil(bone_count/8)] bitmask
[data]               s16 x 3 x frame_count per present bone, big-endian
```

The layout is basically identical to the translations section. Bones with unchanged scale `(1.0, 1.0, 1.0)` on all frames are omitted.

### Encoding

Scale is stored as a fixed-point `s16`:

```
stored = round(scale_float * 1024)
scale_float = stored / 1024.0
```

Identity `1.0` -> `1024`. The default value for entries not present in the file is `1024` (identity).

Clamped to `s16` range at export: `max(-32768, min(32767, round(scale * 1024)))`.

### Addressing

```
index = (bone_idx * frame_count + frame) * 3 + axis
```

Same bone-major layout as translations. `CustomBoneCountScale` tracks the bone count separately from the translations section.

---