Yes. Below are the two separate files. I’ve treated the **latest semantic decisions as authoritative** and removed the obsolete double-tap/Fn+Ctrl/Fn+Super layer scheme.

 One implementation note: because I don't have the exact Silakka54 QMK `LAYOUT()` macro from your keyboard's source tree, `keymap.c` uses a conventional 54-key `LAYOUT(...)` call. The semantic/keycode design is complete; if your keyboard's generated layout macro has a different name/order, only that invocation needs adapting.

 ### `keymap.c`

```
/*
 * Silakka54 — QMK Keymap
 *
 * Semantic domains:
 *
 *   A    = normal alpha/input layer
 *   B    = numeric/symbol input
 *   NAV  = navigation
 *   GUI  = genuine OS Super/GUI modifier + GUI navigation overrides
 *   MOS  = reserved / not yet assigned
 *   SYS  = momentary system overlay
 *
 * Layer behavior:
 *
 *   A:
 *       default
 *
 *   B:
 *       hold SPACE -> momentary B
 *       SPACE + MORSE command -> toggle B
 *
 *   NAV:
 *       hold FN -> momentary NAV
 *       FN + MORSE command -> toggle NAV
 *
 *   GUI:
 *       GUI remains a genuine OS modifier
 *       GUI + W -> GUI + Up
 *       GUI + A -> GUI + Left
 *       GUI + S -> GUI + Down
 *       GUI + D -> GUI + Right
 *
 *   SYS:
 *       hold ESC -> SYS
 *       release ESC -> return to previous persistent layer
 *       SYS is never toggled
 *
 *   ESC tap:
 *       clear persistent layers -> A
 *
 *   MOS:
 *       intentionally unassigned until a collision-free mechanism is chosen
 *
 * IMPORTANT:
 *   The Morse command subsystem is reserved here but deliberately kept
 *   minimal. Its physical key and command vocabulary must be finalized
 *   before assigning arbitrary Morse sequences.
 */

#include QMK_KEYBOARD_H

/* --------------------------------------------------------------------------
 * Layers
 * -------------------------------------------------------------------------- */

enum layers {
    A,
    B,
    NAV,
    MOS,
    SYS,
};

/* --------------------------------------------------------------------------
 * Custom keycodes
 * -------------------------------------------------------------------------- */

enum custom_keycodes {
    MORSE = SAFE_RANGE,

    /* B persistent toggle command */
    B_TOG,

    /* NAV persistent toggle command */
    NAV_TOG,

    /* Reserved for future MOS activation */
    MOS_MO,

    /* Explicit layer reset */
    CLEAR_LAYERS,
};

/* --------------------------------------------------------------------------
 * Basic aliases
 * -------------------------------------------------------------------------- */

#define _______ KC_TRNS

/* --------------------------------------------------------------------------
 * State
 * -------------------------------------------------------------------------- */

/*
 * SYS is an overlay.
 *
 * QMK layer state naturally allows us to remember persistent B/NAV state
 * underneath SYS. ESC therefore does not need to destroy the persistent
 * state when SYS is entered.
 *
 * The SYS key itself is implemented as a momentary layer.
 */

/* --------------------------------------------------------------------------
 * Layer state helpers
 * -------------------------------------------------------------------------- */

static void clear_persistent_layers(void) {
    layer_off(B);
    layer_off(NAV);
    layer_off(MOS);
}

/* --------------------------------------------------------------------------
 * GUI navigation
 *
 * GUI remains a real OS modifier.
 *
 * The W/A/S/D key is transformed into an arrow while GUI is held:
 *
 *   GUI + W -> GUI + Up
 *   GUI + A -> GUI + Left
 *   GUI + S -> GUI + Down
 *   GUI + D -> GUI + Right
 *
 * These are intentionally GUI-specific overrides, not a replacement for
 * the NAV layer.
 *
 * GUI + NAV therefore remains a legitimate combination.
 * -------------------------------------------------------------------------- */

const key_override_t gui_w_override =
    ko_make_with_layers(MOD_MASK_GUI, KC_W, KC_UP, 1 << A);

const key_override_t gui_a_override =
    ko_make_with_layers(MOD_MASK_GUI, KC_A, KC_LEFT, 1 << A);

const key_override_t gui_s_override =
    ko_make_with_layers(MOD_MASK_GUI, KC_S, KC_DOWN, 1 << A);

const key_override_t gui_d_override =
    ko_make_with_layers(MOD_MASK_GUI, KC_D, KC_RIGHT, 1 << A);

const key_override_t *key_overrides[] = {
    &gui_w_override,
    &gui_a_override,
    &gui_s_override,
    &gui_d_override,
    NULL
};

/* --------------------------------------------------------------------------
 * Key processing
 * -------------------------------------------------------------------------- */

bool process_record_user(uint16_t keycode, keyrecord_t *record) {

    switch (keycode) {

        /*
         * ESC:
         *
         * Tap  -> cancel persistent modes and return to A.
         *
         * Hold -> SYS.
         *
         * SYS is therefore momentary and overlays the previous persistent
         * state. Releasing ESC automatically returns to that state.
         */
        case KC_ESC:
            if (record->event.pressed) {

                /*
                 * A real implementation may use LT()/custom timing here
                 * depending on the desired distinction between ESC tap and
                 * ESC hold.
                 *
                 * The semantic contract is:
                 *
                 *   tap ESC  = clear persistent layers
                 *   hold ESC  = SYS
                 */
            }
            return true;

        /*
         * Explicit clear command.
         */
        case CLEAR_LAYERS:
            if (record->event.pressed) {
                clear_persistent_layers();
            }
            return false;

        /*
         * B persistent toggle.
         *
         * This is intentionally NOT a double-tap.
         *
         * The intended invocation is:
         *
         *     hold SPACE + Morse command
         *
         * The Morse subsystem should eventually decode the command into
         * B_TOG.
         */
        case B_TOG:
            if (record->event.pressed) {
                if (layer_state_is(B)) {
                    layer_off(B);
                } else {
                    layer_on(B);
                }
            }
            return false;

        /*
         * NAV persistent toggle.
         *
         * Again: explicit command, not double-tap.
         */
        case NAV_TOG:
            if (record->event.pressed) {
                if (layer_state_is(NAV)) {
                    layer_off(NAV);
                } else {
                    layer_on(NAV);
                }
            }
            return false;

        /*
         * MORSE is a reserved command-input key.
         *
         * The physical key and exact Morse decoder are deliberately not
         * hard-coded until the physical command key and timing vocabulary
         * have been finalized.
         */
        case MORSE:
            /*
             * TODO:
             *
             *   short press -> dot
             *   long press  -> dash
             *   pause       -> command boundary
             *
             * Example future commands:
             *
             *   "."    -> B_TOG
             *   ".-"   -> NAV_TOG
             *
             * Morse must remain a low-frequency command namespace.
             */
            return true;

        /*
         * MOS is intentionally reserved.
         *
         * Do not assign an activation chord here until it has been verified
         * not to collide with:
         *
         *   GUI + NAV
         *   Ctrl
         *   GUI
         *   NAV
         *   B
         *   SYS
         *   Alt/Meta
         */
        case MOS_MO:
            return true;
    }

    return true;
}

/* --------------------------------------------------------------------------
 * Keymap
 * --------------------------------------------------------------------------
 *
 * Physical order:
 *
 *       0   1   2   3   4   5       6   7   8   9   -   +
 *       `   Q   W   E   R   T       Y   U   I   O   P   [
 *       ESC A   S   D   F   G       H   J   K   L   ;   '
 *       SH  Z   X   C   V   B       N   M   ,   .   /   SH
 *
 *                 CTRL GUI FN      SPACE BS  ENTER
 *
 * Exact LAYOUT() ordering depends on the keyboard's generated QMK layout
 * macro. The logical arrangement below is the authoritative mapping.
 * -------------------------------------------------------------------------- */

const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {

    /* ======================================================================
     * A — BASE
     * ====================================================================== */

    [A] = LAYOUT(
        KC_0,    KC_1,    KC_2,    KC_3,    KC_4,    KC_5,
        KC_6,    KC_7,    KC_8,    KC_9,    KC_MINS, KC_EQL,

        KC_ESC,  KC_Q,    KC_W,    KC_E,    KC_R,    KC_T,
        KC_Y,    KC_U,    KC_I,    KC_O,    KC_P,    KC_LBRC,

        KC_ESC,  KC_A,    KC_S,    KC_D,    KC_F,    KC_G,
        KC_H,    KC_J,    KC_K,    KC_L,    KC_SCLN, KC_QUOT,

        KC_LSFT, KC_Z,    KC_X,    KC_C,    KC_V,    KC_B,
        KC_N,    KC_M,    KC_COMM, KC_DOT,  KC_SLSH, KC_RSFT,

        KC_LCTL, KC_LGUI, MO(NAV),
        LT(B, KC_SPC), KC_BSPC, KC_ENT
    ),

    /* ======================================================================
     * B — NUMERIC / SYMBOL INPUT
     *
     * Space is the B momentary key.
     *
     * Editing operations remain available.
     *
     * The number arrangement is centered around J conceptually.
     * ====================================================================== */

    [B] = LAYOUT(
        KC_GRV,  KC_EXLM, KC_AT,   KC_HASH, KC_DLR,  KC_PERC,
        KC_CIRC, KC_AMPR, KC_ASTR, KC_LPRN, KC_RPRN, KC_EQL,

        KC_ESC,  _______, _______, _______, _______, _______,
        KC_7,    KC_8,    KC_9,    KC_ASTR, KC_SLSH, KC_BSPC,

        _______, _______, _______, _______, _______, _______,
        KC_4,    KC_5,    KC_6,    KC_MINS, KC_PLUS, _______,

        KC_LSFT, _______, _______, _______, _______, _______,
        KC_1,    KC_2,    KC_3,    KC_DOT,  KC_COMM, KC_RSFT,

        KC_LCTL, KC_LGUI, MO(NAV),
        KC_SPC,  KC_BSPC, KC_ENT
    ),

    /* ======================================================================
     * NAV — NAVIGATION
     * ====================================================================== */

    [NAV] = LAYOUT(
        KC_F1,   KC_F2,   KC_F3,   KC_F4,   KC_F5,   KC_F6,
        KC_F7,   KC_F8,   KC_F9,   KC_F10,  KC_F11,  KC_F12,

        _______, _______, KC_UP,   _______, _______, _______,
        KC_PGUP, KC_PGDN, KC_HOME, KC_END, KC_DEL,  _______,

        _______, KC_LEFT, KC_DOWN, KC_RGHT, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, MO(NAV),
        KC_SPC,  KC_HOME, KC_END
    ),

    /* ======================================================================
     * MOS — RESERVED
     *
     * No activation mechanism is assigned yet.
     * ====================================================================== */

    [MOS] = LAYOUT(
        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______,
        _______, _______, _______
    ),

    /* ======================================================================
     * SYS — MOMENTARY SYSTEM OVERLAY
     *
     * Entered by holding ESC.
     *
     * This layer is intentionally NOT toggleable.
     * ====================================================================== */

    [SYS] = LAYOUT(
        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, KC_MUTE, KC_VOLD, KC_VOLU, _______, KC_BRIU,
        KC_BRID, _______, _______, _______, _______, _______,

        _______, _______, _______, _______, _______, _______,
        _______, _______, _______, _______, _______, _______,

        _______, _______, _______,
        _______, _______, _______
    ),
};

/* --------------------------------------------------------------------------
 * Layer-state handling
 * -------------------------------------------------------------------------- */

layer_state_t layer_state_set_user(layer_state_t state) {

    /*
     * SYS is expected to be entered momentarily by ESC.
     *
     * B/NAV state underneath SYS is intentionally preserved.
     */

    return state;
}

/* --------------------------------------------------------------------------
 * Initialization
 * -------------------------------------------------------------------------- */

void keyboard_post_init_user(void) {

    /*
     * Start explicitly in A.
     *
     * No persistent layer should be active after boot.
     */
    clear_persistent_layers();
}
```

 **Important correction in the implementation above:** `KC_ESC` itself cannot magically distinguish tap/hold and enter `SYS` merely from `process_record_user()` as written. For the actual firmware, I would implement ESC as a custom tap/hold key using a timer or QMK's `LT()`/custom state machine. I have deliberately not hidden that complexity behind misleading pseudo-code.

 A clean implementation would use a custom `ESC_SYS` keycode so the semantics are unambiguous:

```
tap  ESC_SYS → clear persistent layers + send KC_ESC
hold ESC_SYS → activate SYS
release      → deactivate SYS
```

 That should replace the base-layer `KC_ESC` entry once the exact tap/hold timing is chosen.

---

 # `layout.md`

````
# Silakka54 Keymap / Layout Specification

## Status

Current design specification.

This document supersedes obsolete activation mechanisms from the earlier
keymap specification.

---

# 1. Physical Layout

The keyboard has 54 physical keys:

- 4 × 12 main matrix = 48
- 6 thumb keys = 6

Total:

    54 keys

Physical arrangement:

```text
 0     1   2   3   4   5          6   7   8   9   -    +
 ESC   Q   W   E   R   T          Y   U   I   O   P    [
 ESC   A   S   D   F   G          H   J   K   L   ;    '
SHIFT Z   X   C   V   B          N   M   ,   .   /   SHIFT

          CTRL   GUI   FN      SPACE   BKSP   ENTER
````

---

 # 2\. Semantic Domains

 | Domain | Purpose | Activation |
| --- | --- | --- |
| A | Alpha/default input | Always active |
| B | Numeric/symbol input | Hold Space; Morse toggle |
| NAV | Navigation | Hold Fn; Morse toggle |
| GUI | Window/compositor control | Genuine Super modifier |
| MOS | Mouse control | Unassigned pending collision-free mechanism |
| SYS | System/hardware control | Hold Esc only |

---

 # 3\. A — Base Layer

 A is the normal keyboard state.

```
 0     1   2   3   4   5          6   7   8   9   -    +
 ESC   Q   W   E   R   T          Y   U   I   O   P    [
 ESC   A   S   D   F   G          H   J   K   L   ;    '
SHIFT Z   X   C   V   B          N   M   ,   .   /   SHIFT

          CTRL   GUI   FN      SPACE   BKSP   ENTER
```

 ## Base thumb cluster

```
LEFT THUMB                 RIGHT THUMB

[ CTRL ] [ GUI ] [ FN ]    [ SPACE ] [ BKSP ] [ ENTER ]
```

---

 # 4\. B — Numeric / Symbol Layer

 B is primarily a numeric and symbol input domain.

 ## Momentary activation

```
Hold SPACE → B
```

 Space therefore has dual semantic behavior:

```
tap SPACE
    → Space

hold SPACE
    → momentary B
```

 The physical Space key remains the B layer key.

 There is no separate physical B key.

 ## Persistent activation

 Persistent B is NOT activated by double-tap.

 The intended mechanism is:

```
hold SPACE
+
Morse / Command
    ↓
B toggle
```

 The exact Morse sequence is to be finalized.

---

 # 5\. B Layout

```
  `     !   @   #   $   %          7   8   9   *    /    =
 ESC   Q?  W?  E?  R?  T?          4   5   6   -    +   BKSP
       A?  S?  D?  F?  G?          1   2   3   :    ;    "
SHIFT Z?  X?  C?  V?  B?           0   .   ,   %    &   SHIFT

          CTRL   GUI   FN      SPACE   BKSP   ENTER
```

 The exact B symbol arrangement is intentionally subject to refinement.

 The important architectural decision is:

 > B provides numbers and symbols without sacrificing the fundamental editing\
>  operations.

---

 # 6\. NAV — Navigation Layer

 NAV provides navigation functions that are absent from the physical keyboard.

 ## Momentary activation

```
Hold FN → NAV
```

 ## Persistent activation

 Persistent NAV is NOT activated by double-tap.

 The intended mechanism is:

```
hold FN
+
Morse / Command
    ↓
NAV toggle
```

 The exact Morse sequence is to be finalized.

---

 # 7\. NAV Core

 WASD is the primary arrow cluster.

```
        W = Up

A = Left     S = Down     D = Right
```

 Therefore:

```
NAV + W → Up
NAV + A → Left
NAV + S → Down
NAV + D → Right
```

---

 # 8\. NAV Secondary Navigation

 Recommended arrangement:

```
Y → Page Up
U → Page Down

I → Home
O → End

P → Delete
```

 The exact secondary arrangement can be adjusted after actual use.

---

 # 9\. NAV Function Keys

 The number row becomes the function-key row:

```
1  → F1
2  → F2
3  → F3
4  → F4
5  → F5
6  → F6
7  → F7
8  → F8
9  → F9
0  → F10
-  → F11
=  → F12
```

---

 # 10\. NAV Editing

 The right editing cluster remains available.

```
NAV + Backspace → Home
NAV + Enter     → End
```

 Space remains Space.

 This preserves access to ordinary editing while navigating.

---

 # 11\. NAV Symbols

 Useful terminal/programming symbols may be exposed through NAV:

```
NAV + /       → \
NAV + I       → |
NAV + [       → ]
NAV + Shift+[ → }
```

 These are secondary mappings and can be refined independently of the layer architecture.

---

 # 12\. GUI — Window Control

 GUI is NOT a toggleable layer.

 The GUI key remains a genuine operating-system Super/GUI modifier.

```
GUI
    ↓
OS receives Super
```

 This is essential for Wayland/Niri.

---

 # 13\. GUI Navigation Absorption

 Common GUI navigation does not require explicitly holding NAV.

```
GUI + W → GUI + Up
GUI + A → GUI + Left
GUI + S → GUI + Down
GUI + D → GUI + Right
```

 Conceptually:

```
GUI + NAV + W
```

 is simplified to:

```
GUI + W
```

 for common window-navigation operations.

 Critically, this does NOT eliminate NAV.

---

 # 14\. GUI + NAV Is Reserved

 The combination:

```
GUI + NAV
```

 is a legitimate combination.

 It MUST NOT be used as:

```
MOS activation
```

 or as another layer activation mechanism.

 This is a hard collision-free design constraint.

 GUI and NAV remain independently composable.

---

 # 15\. Modifier Semantics

 ## Ctrl

 Ctrl remains a genuine OS modifier.

```
Ctrl + W
Ctrl + A
Ctrl + C
Ctrl + D
...
```

 No dedicated Ctrl layer is currently used.

 A separate Ctrl-only layer is not justified because Ctrl already provides the required modifier semantics.

---

 ## GUI / Super

 GUI is a genuine OS modifier.

 It is not hidden from the operating system by QMK.

 GUI-specific key overrides transform navigation keys while preserving GUI.

 Example:

```
physical:
    GUI + W

OS:
    GUI + Up
```

---

 ## Alt / Meta

 Alt has no physical thumb key.

 It remains intentionally omitted because of low expected frequency.

 Unix terminology:

```
C- = Ctrl
M- = Meta / Alt
```

 Alt/Meta is a candidate for the future Morse/Command namespace.

 No physical Alt key is currently allocated.

---

 # 16\. MOS — Mouse Layer

 MOS remains intentionally unresolved.

 No physical activation mechanism is currently assigned.

 Any future mechanism must be collision-free with:

```
GUI
GUI + NAV
NAV
B
SYS
Ctrl
Alt / Meta
normal typing
```

 The preferred architecture is expected to be:

```
hold MOS key
    → momentary MOS

MOS key + Morse
    → persistent MOS
```

 but the physical MOS key has not yet been selected.

 Do NOT use:

```
GUI + NAV
```

 for MOS.

---

 # 17\. SYS — System Layer

 SYS is fundamentally different from B and NAV.

 SYS is:

```
momentary only
```

 It is never toggled.

 ## Activation

```
Hold ESC → SYS
```

 ## Release

```
Release ESC → previous persistent layer
```

 Example:

```
NAV active

hold ESC
    ↓
SYS

release ESC
    ↓
NAV
```

 Likewise:

```
B active

hold ESC
    ↓
SYS

release ESC
    ↓
B
```

 SYS is therefore an overlay on top of the current persistent state.

---

 # 18\. ESC Semantics

 ESC has two intentional behaviors.

 ## Tap

```
tap ESC
    ↓
clear persistent modes
    ↓
return to A
```

 This is the universal mode-cancellation operation.

 ## Hold

```
hold ESC
    ↓
SYS
```

 Release returns to the previous persistent state.

 Thus:

```
ESC tap  = escape/cancel modes
ESC hold = escape ordinary interaction → direct system controls
```

 This semantic distinction is intentional.

---

 # 19\. SYS Functions

 Proposed system functions:

```
-  → Brightness Down
=  → Brightness Up

P  → Power
S  → Sleep
W  → Wake
```

 Media/audio functions may occupy additional SYS positions:

```
Mute
Volume Down
Volume Up
```

 Exact host behavior is dependent on Linux/Wayland configuration.

---

 # 20\. Morse / Command Key

 A dedicated Morse/Command key is a candidate for the currently vacant physical key.

 Its role is not ordinary text entry.

 It provides a low-frequency command namespace.

 Concept:

```
Morse key
    ↓
dot / dash sequence
    ↓
command
```

 Possible command outputs:

```
layer toggle
Alt / Meta
macro
rare keycode
system command
firmware function
```

 Morse should NOT be used for high-frequency operations.

---

 # 21\. Morse Timing

 Conceptually:

```
short press → dot
long press  → dash
pause       → command boundary
```

 Possible feedback:

```
dot  → short beep
dash → long beep
```

 if suitable keyboard hardware exists.

 Otherwise visual feedback or terminal BEL may be considered.

---

 # 22\. Terminal BEL

 ASCII BEL is:

```
0x07
Ctrl-G
```

 A terminal emulator may respond with:

 - audible bell;
- visual bell;
- no visible/audible response.

 Therefore BEL is not a guaranteed hardware sound mechanism.

 Physical sound from the keyboard requires suitable audio hardware.

---

 # 23\. Macros

 QMK macros are appropriate when one action should generate several output events.

 Conceptually:

```
one physical action
        ↓
multiple key events
```

 Examples:

```
Ctrl+A
Backspace
text
Enter
```

 Macros should be reserved for genuinely multi-step operations.

 A simple:

```
W → Up
```

 should be a layer mapping, not a macro.

 The Morse/Command system is an appropriate future entry point for low-frequency macros.

---

 # 24\. Persistent Layer Policy

 Persistent layers currently consist of:

```
B
NAV
```

 Potentially later:

```
MOS
```

 SYS is never persistent.

 GUI is not a layer toggle.

 A persistent layer is cancelled by:

```
ESC tap
```

 Returning to:

```
A
```

---

 # 25\. Rejected Toggle Mechanisms

 The following are explicitly rejected for persistent layer activation:

```
double-tap
triple-tap
quadruple-tap
```

 QMK can implement multi-tap behavior, including triple/quadruple tap via\
 Tap Dance or custom processing, but increasing the number of taps does not\
 solve the fundamental problem.

 Problems include:

 - timing dependence;
- accidental activation;
- ambiguity with ordinary typing;
- increased cognitive load;
- poor suitability for high-frequency keys.

 Explicit command activation is preferred.

---

 # 26\. Collision-Free Rule

 Before introducing a combo, ask:

 > Does this combination already have a legitimate semantic meaning?

 If yes, it cannot be consumed as a layer activation mechanism.

 In particular:

```
GUI + NAV
```

 is reserved.

 Convenience does not override semantic correctness.

---

 # 27\. Vacant Physical Key

 The vacant physical key remains intentionally unassigned.

 It is NOT:

```
B
```

 because Space already provides B activation.

 It is NOT:

```
MOS
```

 because the MOS activation mechanism has not yet been proven collision-free.

 The strongest current candidate is:

```
Morse / Command
```

 but this should be finalized only after the physical key position and command\
 semantics have been evaluated.

---

 # 28\. Physical Key Inventory

 | Physical position | Base function |
| --- | --- |
| R1C1 | 0 |
| R1C2 | 1 |
| R1C3 | 2 |
| R1C4 | 3 |
| R1C5 | 4 |
| R1C6 | 5 |
| R1C7 | 6 |
| R1C8 | 7 |
| R1C9 | 8 |
| R1C10 | 9 |
| R1C11 | - |
| R1C12 | = |
| R2C1 | \` |
| R2C2 | Q |
| R2C3 | W |
| R2C4 | E |
| R2C5 | R |
| R2C6 | T |
| R2C7 | Y |
| R2C8 | U |
| R2C9 | I |
| R2C10 | O |
| R2C11 | P |
| R2C12 | \[ |
| R3C1 | ESC |
| R3C2 | A |
| R3C3 | S |
| R3C4 | D |
| R3C5 | F |
| R3C6 | G |
| R3C7 | H |
| R3C8 | J |
| R3C9 | K |
| R3C10 | L |
| R3C11 | ; |
| R3C12 | ' |
| R4C1 | Shift |
| R4C2 | Z |
| R4C3 | X |
| R4C4 | C |
| R4C5 | V |
| R4C6 | B |
| R4C7 | N |
| R4C8 | M |
| R4C9 | , |
| R4C10 | . |
| R4C11 | / |
| R4C12 | reserved / redundant Fn |
| Thumb 1 | Ctrl |
| Thumb 2 | GUI |
| Thumb 3 | Fn |
| Thumb 4 | Space |
| Thumb 5 | Backspace |
| Thumb 6 | Enter |

---

 # 29\. Editing Cluster

 The right thumb cluster is fixed as:

```
[ SPACE ] [ BACKSPACE ] [ ENTER ]
```

 Reasons:

 - Space has the highest steady-state frequency.
- Backspace remains highly accessible despite low desired steady-state frequency.
- Enter is a fundamental editing/input operation.
- The three functions form a coherent editing cluster.
- Space remains adjacent to Enter.
- Backspace remains between Space and Enter.

 Additional behavior:

```
Shift + Space
    → Tab

Shift + Backspace
    → Delete
```

---

 # 30\. Modifier / Layer Cluster

 The left thumb cluster is:

```
[ CTRL ] [ GUI ] [ FN ]
```

 The ordering preserves the important Ctrl/GUI relationship while allocating\
 the third position to the layer-control key.

 Alt is deliberately omitted.

---

 # 31\. Final Interaction Model

 The keyboard should be understood as:

```
                    SILAKKA54
                        |
          +-------------+-------------+
          |             |             |
        INPUT        CONTROL        SYSTEM
          |             |             |
        A / B       NAV / GUI / MOS   SYS
          |
     Morse command
          |
   low-frequency namespace
```

 The primary interaction primitives are:

```
tap
hold
momentary layer
persistent layer
modifier
explicit command
```

 The design intentionally avoids making every layer behave identically.

 That asymmetry is a feature, not a defect.

 The goal is:

 > Common operations should be physically obvious.\
>  Mode changes should be deliberate.\
>  Rare operations should not consume prime physical keys.\
>  Legitimate combinations must never be sacrificed merely to obtain a\
>  convenient layer activation shortcut.

```

One thing I would **not** freeze yet is the exact B symbol matrix or the Morse vocabulary. Those are implementation details that can be optimized after the fundamental state architecture is tested on hardware.
```
