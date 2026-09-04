
# Silakka54 Keymap Design Principles and Specification

> Status: Current design specification (resolved thumb clusters, layer scheme, and special key behaviors)
>
> Scope: Physical key placement, keycap semantics, editing/modifier strategy, layer architecture
>
> Target: Silakka54 split/columnar keyboard
>
> Primary use case: Keyboard-driven software development, terminal-heavy workflow
>
> Design philosophy: Preserve familiar topology where practical; minimize cognitive load; optimize physical access according to intended steady-state usage.

---

## 1. Motivation

The Silakka54 represents a transition from a conventional staggered-row US keyboard
to a compact split/columnar keyboard.

The user is currently learning touch typing at approximately 15 WPM. Consequently, the present frequency of corrective operations—particularly
Backspace—is substantially higher than the expected frequency in the desired steady
state.

The keymap should therefore NOT optimize for current typing errors. Instead, it
should optimize for the keyboard behavior we want to converge toward, while ensuring
that currently frequent corrective operations remain sufficiently accessible during the learning period.

The design is guided by four principles, referred to here as laws.

---

## 2. The Four Laws

### 2.1 Topology Conservation

> Preserve familiar spatial relationships whenever practical.

A keyboard is not merely a set of independent functions. Users develop spatial and motor associations between keys. When transitioning to a new physical topology, preserving recognizable relationships reduces the number of simultaneously changing variables.

Topology conservation therefore has priority when the alternative would introduce unnecessary spatial novelty.

Example:

    SPACE -> ENTER

The relationship is retained:

    [ SPACE ] [ ENTER ]

rather than relocating Enter to an unrelated position merely because a different physical key is available.

This principle is not absolute. The Silakka54's reduced key count necessarily requires remapping and layering. "Conservation" therefore means preserving relationships whenever practical and when doing so does not conflict with higher-priority design considerations.

---

### 2.2 Priority

> Every physical key position must justify its existence.

The Silakka54 provides substantially fewer physical keys than a conventional full-size keyboard. Consequently, keys should be classified by functional necessity.

| Priority | Examples | Treatment |
|---|---|---|
| Essential | Space, Backspace, Enter | Dedicated physical positions |
| High | Ctrl, Shift, Alt, Super | Dedicated or highly accessible positions |
| Contextual | Fn/layer, Tab, Delete | Dedicated or layered depending on requirements |
| Occasional | Navigation, secondary editing | Prefer layers |
| Disposable | Menu/Application key, redundant functions | Candidate for removal |

The principle is not that low-priority functions are impossible to access. Rather, low-priority functions should not consume prime physical real estate when a layer can provide equivalent functionality.

---

### 2.3 Grouping by Function

> Physically group operations according to their functional role.

The keyboard should have recognizable functional neighborhoods.

For the present design, the major groups are:

- Editing: Space, Backspace, Enter, Delete
- Modifiers: Ctrl, Shift, Alt, Super
- Layer control: Fn
- Navigation: Home, End, Page Up, Page Down, etc.

This reduces the cognitive cost of remembering arbitrary mappings.

The three primary editing operations are consequently treated as a coherent unit:

    Space + Backspace + Enter

These are referred to informally as the "three editing musketeers."

---

### 2.4 Frequency

> Within a functional group, physical accessibility should reflect expected steady-state frequency rather than transient observed frequency.

Observed frequency is useful as a diagnostic measurement, but is not necessarily an appropriate optimization target.

For example, a novice typist may currently produce a very high number of Backspace operations. This does NOT imply that Backspace should outrank Space in the long-term design. The ideal typing state contains essentially no corrective Backspace operations.

Thus:

    Observed frequency != Desired frequency

The relevant target is the expected frequency after typing proficiency has improved.

| Function | Desired steady-state frequency |
|---|---|
| Space | Very high |
| Enter | Moderate/low |
| Backspace | Approximately zero during error-free typing |
| Delete | Low |
| Modifiers | Workflow-dependent |

Backspace nevertheless receives a highly accessible position because it is important despite its ideally low frequency.

This distinction prevents the keymap from accidentally optimizing around current typing deficiencies.

---

## 3. Cluster Terminology

The term **cluster** is adopted as the primary term for a physically contiguous group of keyboard keys that is treated as a coherent functional unit.

This terminology follows the established convention of describing groups of related keyboard keys as clusters, including the terminology used by IBM keyboard designs.

### Definition

> **Cluster:** A physically contiguous group of keys treated as a coherent functional unit. Cluster membership is determined by physical grouping and/or intended function; keycap legends do not determine membership.

This distinction is important for the Silakka54 because available keycaps are being repurposed. A keycap may retain a familiar legend while its physical key performs an entirely different operation.

The documentation distinguishes between:

1. **Physical cluster** — where keys are located.
2. **Functional cluster** — what related operations they provide.
3. **Keycap identity** — the legend and physical dimensions of the donor keycap.
4. **Logical key identity** — the function generated by the firmware.

For example:

    Physical cluster:   Editing Cluster
    Keycap:             DELETE
    Logical function:   BACKSPACE

These identities need not coincide. This separation is fundamental to the present design because the Silakka54 keymap intentionally repurposes conventional keyboard keycaps while preserving useful physical and semantic relationships.

---

## 4. Physical Layout

### 4.1 Full Grid

| Row | Col 1 | Col 2 | Col 3 | Col 4 | Col 5 | Col 6 | Col 7 | Col 8 | Col 9 | Col 10 | Col 11 | Col 12 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **R1** | `0` | `1` | `2` | `3` | `4` | `5` | `6` | `7` | `8` | `9` | `-` / `_` | `=` / `+` |
| **R2** | `` ` `` / `~` | `Q` | `W` | `E` | `R` | `T` | `Y` | `U` | `I` | `O` | `P` | `[` / `{` |
| **R3** | `Esc` | `A` | `S` | `D` | `F` | `G` | `H` | `J` | `K` | `L` | `;` / `:` | `'` / `"` |
| **R4** | `Shift` | `Z` | `X` | `C` | `V` | `B` | `N` | `M` | `,` / `<` | `.` / `>` | `/` / `?` | `Fn` |
| **Thumb** | | | | `Ctrl` | `Super` | `Fn` | `Space` | `Backspace` | `Enter` | | | |

## 5. The Editing Cluster (Right Thumb)

The right-side three-key cluster is the **Editing Cluster**. It is dedicated to fundamental text-entry and text-editing operations.

### 5.1 Base Layer Mapping

| Physical key | Size | Keycap legend | Actual function | Mnemonic |
|---|---:|---|---|---|
| Leftmost | 1.75u | Tab | Space | Tab represents grouped whitespace |
| Middle | 1u | Delete | Backspace | Delete/remove text |
| Rightmost | 1u | End | Enter / Return | End the line/statement |

The physical arrangement is:

```
+----------+------+------+
|   TAB    | DEL  | END  |
|  1.75u   |  1u  |  1u  |
+----------+------+------+
|  SPACE   |  BS  | ENTER|
+----------+------+------+
```

### 5.2 Shift Behaviors (Base Layer)

- `Shift` + `Space` = `Tab`  
  (Tab is expressed as modified Space; no dedicated Tab key.)
- `Shift` + `Backspace` = `Delete` (forward delete)  
  (The same key performs backward deletion normally and forward deletion when shifted.)

### 5.3 Layer Behavior (NAV Layer)

When the `NAV` layer is active (via `Fn`):

| Physical key | Function on `NAV` |
|---|---|
| `Space` | Transparent (remains Space) |
| `Backspace` | `Home` |
| `Enter` | `End` |

This mapping is mnemonic: Backspace is associated with moving left/back, Enter with ending a line/command. The left-to-right progression mirrors start-to-end navigation.

### 5.4 Justification

The ordering follows multiple laws simultaneously:

1. **Topology Conservation**  
   Space remains adjacent to and left of Enter.

2. **Priority**  
   All three are high-value editing operations.

3. **Functional Grouping**  
   Editing operations are colocated.

4. **Frequency**  
   The largest/easiest position is assigned to Space.  
   Backspace remains highly accessible despite its low desired steady-state frequency.

5. **Physical constraints**  
   The 1.75u key accepts an appropriate donor keycap; the conventional Spacebar does not.

6. **Semantic mnemonic quality**  
   Tab/Space, Delete/Backspace, and End/Enter each have defensible associations.

---

## 6. The Modifier Cluster (Left Thumb)

The left thumb cluster is the **Modifier Cluster**. Its intended role is control of keyboard/application behavior, as opposed to the right cluster's focus on text manipulation.

### 6.1 Final Allocation

| Position | Function |
|---|---|
| Leftmost | `Ctrl` |
| Middle | `Super` |
| Rightmost | `Fn` |

### 6.2 Justification

- **Priority**  
  `Ctrl` and `Super` are the two highest-frequency modifiers in a terminal-heavy, Super-driven window-manager workflow. `Fn` is the layer gateway and must be immediately thumb-accessible. `Alt` is lower priority and is deliberately not given a thumb slot.

- **Topology Conservation**  
  The standard bottom-row order is `Ctrl Super Alt Space`. We preserve `Ctrl Super` adjacency, but `Fn` displaces `Alt`. This is a conscious trade: `Fn` outranks `Alt` because losing a thumb-accessible layer key would degrade the whole navigation/layer scheme.

- **Frequency**  
  `Fn` will be used constantly for NAV/SYS access. `Alt` is not currently part of the primary workflow, so it should not consume a prime thumb position.

- **Grouping by Function**  
  Left cluster remains the Modifier Cluster: keyboard/application behavior control. `Fn` belongs there because it controls keyboard layers.

The conventional `Alt` key is omitted from the physical base layer. It can be introduced later on a layer or via a mod-tap if workflow needs change.

---

## 7. Special Key Behaviors

### 7.1 Shift (R4C1)

A single left-hand `Shift` key with stateful behavior replaces the conventional duplicate Shift and Caps Lock.

| Action | Behavior |
|---|---|
| **Tap** | One-shot Shift (next key is shifted) |
| **Hold** | Normal Shift while held |
| **Double-tap** | Toggle Caps Lock |
| **Tap while Caps Lock active** | Disable Caps Lock |

This provides Caps Lock functionality without a dedicated key, and eliminates the redundant second Shift. The stateful tap/hold/double-tap logic is implemented in QMK with tap-dance or custom key processing.

### 7.2 Escape (R3C1)

A dedicated physical `Esc` key is placed at R3C1 (previously an empty position). This avoids conflicts with modal editors (e.g., Vim) that rely heavily on a reliable, immediate Escape. `Esc` is not overloaded onto `Fn` tap.

### 7.3 Redundant Fn (R4C12)

A redundant `Fn` key is placed at R4C12 on the right side. This is essential for one-handed right-side layer access, particularly for volume/brightness adjustment on `NAV`/`SYS` layers. Both `Fn` keys behave identically:

- **Hold** = momentary `NAV` layer
- **Double-tap** = toggle `NUM` layer
- **Fn + Ctrl** = toggle `NAV` layer
- **Fn + Super** = momentary `SYS` layer

### 7.4 Dual-Purpose `-` and `=` Keys

The physical keys at R1C11 (`-` / `_`) and R1C12 (`=` / `+`) serve as increment/decrement operations on layers:

- On `NAV` layer: `-` = Volume Down, `=` = Volume Up
- On `SYS` layer: `-` = Brightness Down, `=` = Brightness Up (host-dependent)

This fulfills the requirement that `+`/`-` represent universal increment/decrement across layers.

---

## 8. Layer Scheme

### 8.1 Layer Summary

| Layer | Activation | Purpose |
|---|---|---|
| **Base** (Layer 0) | Default | Standard typing and editing |
| **NUM** (Layer 1) | Double-tap `Fn` | Redundant number/symbol pad (for future 40% compatibility) |
| **NAV** (Layer 2) | Hold `Fn`; toggle `Fn`+`Ctrl` | Navigation and secondary functions |
| **SYS** (Layer 3) | Hold `Fn`+`Super` | System and media controls |
| **MOUSE** (Layer 4) | Toggle `SYS`+`M` | Experimental mouse controls |

### 8.2 NAV Layer Mappings

| Physical Key | Function |
|---|---|
| `1`–`0` | `F1`–`F10` |
| `-` | `F11` |
| `=` | `F12` |
| `W` | `Up` |
| `A` | `Left` |
| `S` | `Down` |
| `D` | `Right` |
| `Backspace` | `Home` |
| `Enter` | `End` |
| `Shift`+`W` | `Page Up` |
| `Shift`+`S` | `Page Down` |
| `[` | `]` |
| `Shift`+`[` | `}` |
| `/` | `\` |
| `I` | `\|` |
| `-` | `Volume Down` |
| `=` | `Volume Up` |
| `0` | `Mute` |
| `B` | `XF86Back` |
| `F` | `XF86Forward` |
| `R` | `XF86Reload` |

### 8.3 SYS Layer Mappings

| Physical Key | Function |
|---|---|
| `-` | `Brightness Down` (host-dependent) |
| `=` | `Brightness Up` (host-dependent) |
| `P` | `XF86PowerOff` (hold) |
| `S` | `XF86Sleep` (hold) |
| `W` | `XF86WakeUp` |

Brightness controls are intentionally left unassigned until host/Wayland behavior is verified; they can be added without disturbing the core design.

### 8.4 NUM Layer

The `NUM` layer provides a redundant number/symbol arrangement mirroring the physical top row. It is accessed via double-tap on either `Fn`. This layer exists to preserve compatibility if the physical number row is ever removed (e.g., moving to a 40% layout). Toggle off by double-tapping `Fn` again or pressing `Esc`.

---

## 9. Remaining Open Questions

The following items remain deliberately unspecified, to be resolved after initial use and ergonomic measurement:

- Exact physical thumb-key orientation and ergonomic reach measurements.
- Empirical frequency measurements after actual use (to validate layer usage and modifier placement).
- Possible introduction of `Alt` via mod-tap or layer if workflow demands it.
- Mouse layer refinement (currently experimental).
- Brightness key implementation specifics (host-dependent).
- Firmware implementation details (QMK features like tap-dance, one-shot mods, etc.).

These will be decided using the same four laws and empirical feedback.

---

## 10. Design Status

The current keymap establishes the following firm principles:

```
LEFT MODIFIER CLUSTER        RIGHT EDITING CLUSTER

[ Ctrl ] [ Super ] [ Fn ]    [ Space ] [ Backspace ] [ Enter ]
```

The keycap legends are donor artifacts selected for physical compatibility and mnemonic usefulness. The underlying design objective is to produce a small, internally coherent keyboard whose physical topology, functional grouping, and accessibility remain predictable even though its physical key count differs substantially from a conventional staggered keyboard.

The full key mapping is specified in `keymap.csv`.
