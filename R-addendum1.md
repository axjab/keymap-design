Below is a **detailed errata/addendum** intended to sit alongside the existing README rather than silently rewriting it. It records the decisions made after the README's current design was written, including places where the older specification is now superseded.

 # README Addendum / Errata — QMK Semantic Layer Architecture

 > Status: Design clarification and superseding decisions\
>  Applies to: QMK firmware architecture, layer activation, modifier semantics, command input\
>  Relationship to README: This document supersedes conflicting statements in the original README.

---

 ## 1\. Purpose of This Addendum

 The original README established the physical philosophy and initial layer scheme. Subsequent design work clarified several important semantic requirements that the original specification did not capture correctly.

 This addendum records those decisions explicitly.

 The most important change is that the keymap is no longer treated as a collection of symmetrical "toggleable layers." Instead, different semantic domains have different activation semantics:

 - some are normal modifiers;
- some are momentary layers;
- some can be persistently toggled;
- some are momentary overlays;
- some may eventually be accessed through a dedicated command/Morse interface.

 The design priority is now:

 > **Every activation mechanism must be predictable, low-cognitive-load, and collision-free with legitimate keyboard operations.**

---

 # 2\. Superseding Layer Taxonomy

 The current semantic domains are:

 | Domain | Meaning | Activation model |
| --- | --- | --- |
| **A** | Alpha/default input | Always active |
| **B** | Numeric/symbol input | Momentary + persistent toggle |
| **NAV** | Navigation | Momentary + persistent toggle |
| **GUI** | Window/compositor control | Normal modifier + semantic overrides |
| **MOS** | Mouse/pointer control | TBD; must be collision-free |
| **SYS** | System/hardware controls | Momentary only |

 The previously considered GAME layer is **out of scope** and should not be incorporated into the present design.

---

 # 3\. Layer Activation Is Not Symmetrical

 Layers should not be forced into a common activation model merely for consistency.

 The current design intentionally distinguishes:

 ### A — Base

 A is not really a mode that needs activation.

 It is the default state:

```
A = normal keyboard
```

 A persistent mode can be cancelled by tapping `Esc`.

---

 ### B — Numeric/Symbol

 B is accessed through the **Space key**.

```
Hold Space
    → momentary B

Space + Morse command
    → toggle B
```

 Space therefore retains its ordinary base-layer meaning while held behavior provides access to B.

 The exact Morse sequence used for the toggle remains a firmware-design detail, but the conceptual mechanism is fixed:

 > **Hold the layer key, then issue an explicit command through the Morse key.**

 This deliberately replaces double-tap-based layer locking.

---

 ### NAV — Navigation

 NAV is accessed through the **Fn thumb key**.

```
Hold Fn
    → momentary NAV

Fn + Morse command
    → toggle NAV
```

 NAV therefore has both temporary and persistent access without requiring a second dedicated navigation-toggle key.

 The exact Morse command remains to be finalized.

---

 ### GUI — Window Control

 GUI is **not a toggleable layer**.

 GUI remains a legitimate OS modifier:

```
GUI → OS receives GUI/Super
```

 However, QMK provides GUI-specific semantic transformations so that common navigation operations do not require simultaneously holding GUI and NAV.

 For example:

```
GUI + W → GUI + Up
GUI + A → GUI + Left
GUI + S → GUI + Down
GUI + D → GUI + Right
```

 Thus NAV is effectively **absorbed by GUI** for these common operations.

 This provides:

```
GUI + W
```

 instead of:

```
GUI + NAV + W
```

 without preventing other legitimate combinations.

 ### Critical collision constraint

 `GUI + NAV` is a legitimate combination and **must remain available**.

 It must never be appropriated as the activation mechanism for MOS or another layer.

 This is a hard design constraint.

---

 ### MOS — Mouse

 MOS remains unresolved.

 No activation mechanism should be selected merely because a combination happens to be convenient.

 The activation mechanism must first be demonstrated to be collision-free with:

 - ordinary typing;
- Ctrl combinations;
- GUI combinations;
- NAV;
- GUI + NAV;
- B;
- SYS;
- Alt/Meta access;
- terminal conventions.

 A proposed mechanism that conflicts with an existing semantic operation is rejected regardless of its physical convenience.

 The intended model remains:

```
Hold MOS mechanism
    → momentary MOS

MOS mechanism + Morse command
    → persistent MOS
```

 but the physical MOS mechanism is currently **TBD**.

---

 ### SYS — System

 SYS is fundamentally different from the other layers.

```
Hold Esc
    → SYS
Release Esc
    → return to previous persistent layer
```

 SYS is **never toggled**.

 There is no SYS lock operation.

 The semantic meaning of `Esc` is intentionally twofold:

```
Esc tap
    → cancel persistent mode(s)

Esc hold
    → escape normal interaction and enter direct system controls
```

 The two behaviors are deliberately distinct.

 If B is persistent:

```
B active
Hold Esc
    → SYS
Release Esc
    → B
```

 If NAV is persistent:

```
NAV active
Hold Esc
    → SYS
Release Esc
    → NAV
```

 SYS therefore behaves as a **momentary system overlay on top of the current persistent state**.

---

 # 4\. ESC Semantics

 The previous README's use of `Esc` primarily as a normal key does not fully capture its new architectural role.

 `Esc` is now a particularly important control key.

 ## 4.1 Tap

```
ESC tap
    → cancel all persistent/toggled modes
    → return to A
```

 This should be treated as a universal **cancel/reset-to-base** operation.

 It should not be described as "toggle A."

 The distinction matters:

 > A is the default state; ESC cancels modes rather than toggling A.

---

 ## 4.2 Hold

```
ESC hold
    → SYS
```

 Release returns to whatever persistent layer was active immediately before SYS was invoked.

 This means SYS is an overlay, not a destination.

---

 # 5\. Double-Tap Layer Toggling Is Rejected

 The previous README proposed double-tap mechanisms for layer toggling.

 That mechanism is now **rejected as a general layer-toggle strategy**.

 Reasons:

 1. **False-positive risk**\
    A double tap can accidentally become a toggle when two ordinary taps happen close enough together.
2. **Timing dependence**\
    The behavior depends on tapping-term configuration.
3. **Cognitive load**\
    The user must remember timing rules rather than a simple physical gesture.
4. **Poor semantic separation**\
    A normal key action and a mode-changing action are being distinguished primarily by timing.
5. **Bad fit for high-frequency keys**\
    Space in particular is too frequently used to make accidental persistent-mode changes desirable.

 Triple-tap and quadruple-tap are technically possible through QMK Tap Dance/custom logic, but they are **not considered a superior solution**. Increasing the tap count does not solve the fundamental timing/cognitive problem.

 Therefore:

 > **Persistent layer activation should use an explicit command gesture rather than ordinary multi-tap timing.**

---

 # 6\. Morse / Command Key

 A dedicated physical **Morse/Command key** is now considered a serious architectural component.

 Its purpose is not primarily to type Morse text.

 Its primary purpose is to provide a **low-frequency command namespace** without consuming numerous physical keys.

 Conceptually:

```
Layer key held
       +
Morse/Command key
       ↓
Morse sequence
       ↓
command
```

 For example:

```
Hold Space + Morse "."
    → toggle B

Hold Fn + Morse "."
    → toggle NAV
```

 The actual sequences should be selected for mnemonic value and reliability.

---

 # 7\. Why Morse Is Attractive

 A single physical key can represent a very large command namespace.

 Potential uses include:

 - layer toggling;
- rarely used keycodes;
- Alt/Meta;
- unusual symbols;
- macros;
- system commands;
- QMK firmware functions;
- other low-frequency operations.

 This is preferable to dedicating prime physical real estate to functions that are rarely used.

 However:

 > **Morse should not be used for high-frequency operations.**

 Typing common commands through Morse would increase cognitive and temporal overhead.

 Morse is appropriate precisely because the commands it represents are rare enough that memorizing a small command vocabulary is acceptable.

---

 # 8\. QMK Capability: Morse

 QMK does not provide a native general-purpose "Morse input layer."

 However, QMK's custom key processing and timer facilities are sufficient to implement one.

 A custom Morse implementation can distinguish, for example:

```
short activation → dot
long activation  → dash
pause             → command boundary
```

 and translate the resulting sequence into arbitrary QMK behavior.

 The decoded result does not have to be text.

 It can invoke:

 - a keycode;
- a modifier;
- a layer operation;
- a macro;
- a custom function;
- a key sequence.

 Thus Morse is better understood here as a **command encoding system** than as a typing method.

---

 # 9\. Morse Feedback

 If the keyboard hardware supports an appropriate buzzer/piezo/speaker, QMK can potentially provide local audio feedback.

 A useful implementation would be:

```
dot → short beep
dash → long beep
```

 This could make Morse entry substantially more reliable because the user receives immediate confirmation of how the firmware interpreted the input.

 If no suitable audio hardware exists, feedback could instead use:

 - RGB/LED state;
- another visual firmware indicator;
- terminal BEL where appropriate.

---

 # 10\. Unix Terminal BEL

 The Unix/terminal "bell" should not be confused with a physical keyboard buzzer.

 The ASCII BEL character is:

```
0x07
Ctrl-G
```

 A terminal emulator/application may interpret BEL as:

 - an audible bell;
- a visual bell;
- no visible/audible action.

 Therefore:

```
QMK → BEL
```

 does **not** guarantee that the keyboard or computer produces a sound.

 It is application/terminal dependent.

 Physical keyboard sound requires appropriate keyboard audio hardware.

 For a Linux/terminal-oriented workflow, BEL remains useful as a software-level feedback mechanism, but it should not be treated as a universal sound API.

---

 # 11\. Modifier Semantics

 The physical modifier set is:

```
Ctrl
Super
Fn
```

 with Alt deliberately omitted from the thumb cluster.

 ## Ctrl

 Ctrl remains a normal OS modifier.

 It is not currently assigned its own layer.

 The earlier idea of a dedicated CTRL layer restricted to non-alphanumeric modifications is **not adopted** at this stage. A normal modifier already provides the required semantics without introducing another mode.

---

 ## Super / GUI

 Super is both:

 1. a genuine OS-level modifier; and
2. a QMK semantic modifier with GUI-specific key overrides.

 QMK does not need to "hide" Super from the OS when implementing GUI-specific behaviors.

 For example:

```
GUI + W
```

 can result in the OS receiving:

```
GUI + Up
```

 while the GUI modifier remains active.

 This distinction is fundamental:

 > QMK can transform the non-modifier key while preserving the modifier that the operating system expects.

---

 ## Alt / Meta

 Alt has no dedicated physical key because its expected frequency is low.

 Unix terminology commonly represents the modifier as:

```
C- = Ctrl
M- = Meta / Alt
```

 Examples:

```
C-Left
M-Left
C-M-Left
```

 The `M-` terminology makes Meta a natural candidate for representation through the future Morse/Command mechanism.

 A physical Alt key should not be added merely for completeness.

---

 # 12\. Collision-Free Design Rule

 The strongest architectural lesson from subsequent design work is:

 > **Convenience is not sufficient justification for an activation combination.**

 Before assigning any combo as a layer activation mechanism, its collision surface must be audited.

 For example, `GUI + NAV` cannot be used for MOS because GUI + NAV has an independent legitimate semantic meaning.

 Likewise, a common typing chord should not become a hidden layer toggle merely because QMK makes it technically possible.

 The design therefore prefers:

 1. dedicated physical controls where their priority justifies the key;
2. contextual layer-key + command-key gestures;
3. deliberately reserved combos;
4. Morse/command encoding for low-frequency functions.

---

 # 13\. Current Thumb Architecture

 The current intended six-thumb arrangement remains:

```
LEFT THUMB                    RIGHT THUMB

[ CTRL ] [ GUI ] [ NAV ]      [ SPACE ] [ BSPC ] [ ENTER ]
```

 This preserves the original README's primary decision.

 The right cluster remains the Editing Cluster:

```
Space + Backspace + Enter
```

 The left cluster remains the Modifier/Layer Control Cluster:

```
Ctrl + GUI + NAV
```

 `Fn` in the original README is therefore conceptually superseded by **NAV** as the semantic layer-control key in this architecture.

---

 # 14\. The Vacant Physical Key

 The previously vacant physical key is **not automatically assigned to B**.

 B already has a physical activation mechanism:

```
Space hold → B
```

 The vacant key therefore remains available.

 Possible future uses include:

 - dedicated Morse/Command key;
- another high-value function discovered through actual usage;
- deliberate reserve capacity.

 The current decision is:

 > **Do not consume the vacant key until its use has been justified by the complete interaction architecture.**

 A dedicated Morse/Command key is currently the most compelling candidate, but this remains a design decision to finalize rather than an automatic reassignment.

---

 # 15\. Current Layer-Control Specification

 The consolidated design is therefore:

```
A
    default
    ESC tap → cancel persistent modes → A

B
    hold SPACE → momentary B
    hold SPACE + Morse command → toggle B

NAV
    hold NAV/Fn → momentary NAV
    hold NAV/Fn + Morse command → toggle NAV

GUI
    normal GUI/Super modifier
    GUI-specific overrides absorb common NAV operations
    not toggleable

MOS
    activation mechanism TBD
    must be collision-free
    intended model:
        hold MOS mechanism → momentary MOS
        MOS mechanism + Morse command → toggle MOS

SYS
    hold ESC → SYS
    release ESC → previous persistent layer
    never toggle
```

---

 # 16\. Navigation Semantics

 The physical keyboard has no dedicated arrow keys.

 NAV therefore provides:

```
W → Up
A → Left
S → Down
D → Right
```

 with secondary navigation functions arranged around the same semantic area.

 Potential mappings include:

```
Y → PgUp
U → PgDn
I → Home
O → End
P → Delete
```

 Modifier combinations should remain meaningful:

```
Ctrl + W → Ctrl + Up
Ctrl + A → Ctrl + Left
Ctrl + S → Ctrl + Down
Ctrl + D → Ctrl + Right
```

 This is particularly valuable in terminal/editor workflows.

---

 # 17\. GUI + NAV Must Remain Orthogonal

 Although GUI absorbs NAV for common GUI-navigation operations:

```
GUI + W → GUI + Up
```

 NAV itself must remain independently usable.

 Likewise:

```
GUI + NAV
```

 must remain available for any semantic function assigned to that combination.

 Therefore the design must not implement GUI by globally converting or consuming NAV.

 GUI-specific overrides should target the relevant key behaviors while preserving the independent existence of the GUI and NAV controls.

---

 # 18\. Macros

 QMK macros are considered appropriate when:

```
one physical action
    ↓
multiple output events
```

 are required.

 Examples include:

```
Ctrl+A
Backspace
text sequence
Enter
```

 or other repetitive command sequences.

 A simple one-key-to-one-key transformation is generally better represented as a key behavior or override than as a macro.

 The Morse/Command key is particularly well suited to invoking macros because it provides a large low-frequency command namespace.

---

 # 19\. Design Principles Added to the Original Four Laws

 The original four laws remain:

 1. Topology Conservation
2. Priority
3. Grouping by Function
4. Frequency

 The subsequent QMK design introduces two additional operational principles.

 ### 19.1 Collision Freedom

 > No activation mechanism may steal a combination that has a legitimate semantic meaning elsewhere.

 This is a hard constraint, not merely a preference.

 ### 19.2 Explicit State Change

 > Persistent mode changes should require an intentional, distinguishable action rather than relying on ambiguous timing patterns.

 This is why double-tap/triple-tap layer locking is rejected in favor of explicit layer-key + command mechanisms.

---

 # 20\. Current Architectural Direction

 The keyboard is evolving toward a **semantic input language**, rather than merely a collection of conventional keyboard layers.

 The conceptual structure is:

```
                     KEYBOARD
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       INPUT          CONTROL         SYSTEM
          │              │              │
        A / B       NAV / GUI / MOS     SYS
```

 with an orthogonal command channel:

```
             MORSE / COMMAND
                    │
       ┌────────────┼────────────┐
       │            │            │
    layer ops     Alt/Meta      macros
```

 This approach allows a 42-key keyboard to expose a substantially larger functional vocabulary without assigning dedicated physical keys to every low-frequency operation.

 The objective is not to maximize the number of things the firmware can do.

 The objective is:

 > **Make the common operations physically obvious, make state changes deliberate, and provide a compact command mechanism for everything else.**

---

 ## 21\. Decisions Superseding the Original README

 For clarity, the following original README decisions are explicitly superseded:

 | Original README | Current decision |
| --- | --- |
| Double-tap `Fn` toggles NUM | **Rejected as general toggle model** |
| Double-tap for persistent layers | **Rejected** |
| Fn + Ctrl toggles NAV | **Replaced by NAV/Fn + Morse command** |
| Fn + Super momentary SYS | **Replaced by ESC hold → SYS** |
| SYS entered through Fn + Super | **Superseded** |
| SYS toggle semantics | **Rejected; SYS is momentary only** |
| ESC merely normal Escape | **Expanded: tap = cancel modes, hold = SYS** |
| GUI as ordinary modifier only | **Expanded: GUI-specific NAV-absorbing overrides** |
| GUI + NAV potentially used for another layer | **Explicitly prohibited; GUI + NAV is reserved** |
| Alt potentially via mod-tap | **Deferred; Morse/Meta is preferred candidate** |
| B/NUM activation via Fn | **B is accessed through Space hold** |
| Vacant key automatically available for layer | **Remain vacant pending architectural justification** |
| MOUSE activation via `SYS + M` | **Rejected/deferred pending collision audit** |

---

 ## 22\. Remaining Open Decisions

 The following are still genuinely unresolved:

 1. Exact Morse/Command physical key.
2. Exact Morse timing and feedback behavior.
3. Exact Morse sequences for B/NAV toggles.
4. Collision-free MOS activation mechanism.
5. Whether Morse should provide Alt/Meta directly.
6. Final B numeric/symbol arrangement.
7. Final NAV secondary functions.
8. Whether keyboard audio hardware is available/useful.
9. Host-side behavior of brightness/media/system keycodes under Wayland/Niri.
10. Complete collision audit of all modifier and combo combinations.

 These should be resolved empirically rather than by adding complexity prematurely.

---

 ## 23\. Current Non-Negotiable Constraints

 The following should be treated as hard requirements for future revisions:

 - The keyboard has 42 physical keys.
- A is always the default/base domain.
- B is accessed momentarily by holding Space.
- B already has a physical switch; the vacant key is **not** the B switch.
- NAV is accessed momentarily by holding the NAV/Fn thumb key.
- GUI remains a genuine OS-level Super/GUI modifier.
- GUI + W/A/S/D must provide GUI-modified navigation without requiring simultaneous NAV.
- GUI + NAV must remain available for legitimate semantic use.
- GUI + NAV must not be appropriated for MOS.
- SYS is entered by holding ESC.
- SYS cannot be toggled.
- Releasing ESC from SYS returns to the previously active persistent layer.
- ESC tap cancels persistent layers and returns to A.
- Alt has no dedicated physical key.
- Persistent layer activation should not rely on double-tap/triple-tap timing.
- Any combo-based activation must be demonstrably collision-free.
- GAME is out of scope.
- The vacant physical key remains unassigned until a compelling use is established.
- Morse/Command is a serious candidate for a dedicated low-frequency command channel.
- Common/high-frequency operations should not be routed through Morse.
- The design should optimize for desired steady-state behavior, not novice typing errors.

---

 ## 24\. Design Status

 The design is now best understood as a **semantic, stateful QMK keymap** with three kinds of interaction:

```
1. NORMAL INPUT
   A / B

2. MODIFIER / MOMENTARY CONTROL
   Ctrl / GUI / NAV / SYS / MOS

3. EXPLICIT COMMAND
   Morse / Command
```

 The resulting architecture deliberately avoids making every layer behave identically.

 That asymmetry is intentional.

 A good 42-key keyboard does not need every function to have equal physical representation. It needs a small number of extremely predictable interaction primitives from which the complete command vocabulary can be constructed.
