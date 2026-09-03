
# Silakka54 Keymap Design Principles and Initial Specification

> Status: Initial design
> 
> Scope: Physical key placement, keycap semantics, and editing/modifier strategy
> 
> Target: Silakka54 split/columnar keyboard
> 
> Primary use case: Keyboard-driven software development, terminal-heavy workflow
> 
> Design philosophy: Preserve familiar topology where practical; minimize cognitive load; optimize physical access according to intended steady-state usage.

---

## 1. Motivation

The Silakka54 represents a transition from a conventional staggered-row US keyboard to a compact split/columnar keyboard.

The user is currently learning touch typing at approximately 15 WPM. Consequently, the present frequency of corrective operations—particularly Backspace—is substantially higher than the expected frequency in the desired steady state.

The keymap should therefore NOT optimize for current typing errors. Instead, it should optimize for the keyboard behavior we want to converge toward, while ensuring that currently frequent corrective operations remain sufficiently accessible during the learning period.

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

## 3. Text-Editing Model

For rough frequency reasoning, text can be modeled as a sequence of words separated by whitespace and terminated by carriage return.

A representative abstract line is:

    word space word backspace space word backspace backspace space
    word backspace backspace backspace space carriage-return

This model illustrates that Backspace can become disproportionately frequent during the learning phase.

The model is deliberately not treated as authoritative empirical data. It exists to distinguish:

1. normal text production, and
2. correction of errors made during text production.

The desired design optimizes primarily for (1).

---

## 4. Donor Keycap Strategy

The available keycaps originate from a conventional US staggered-row keyboard.

The original keycap-to-function mapping therefore cannot be preserved exactly. In particular, the conventional large Enter keycap does not physically fit the intended Silakka54 position and is consequently excluded from the design.

Rather than treating the remaining keycaps as arbitrary, the design uses semantic similarity to repurpose available keycaps.

The navigation cluster provides a useful donor set:

- Insert
- Home
- Page Up
- Delete
- End
- Page Down

Three of these can be reassigned to the three editing musketeers.

The resulting legends are intentionally treated as mnemonic donor labels, not authoritative descriptions of the underlying firmware behavior.

---

## 5. The Three Editing Musketeers

The current physical configuration is:

| Physical key | Size | Keycap legend | Actual function | Mnemonic |
|---|---:|---|---|---|
| Large key in right thumb cluster | 1.75u | Tab | Space | Tab represents grouped whitespace |
| Middle key | 1u | Delete | Backspace | Delete/remove text |
| Rightmost key | 1u | End | Enter / Return | End the line/statement |

The physical arrangement is therefore:

    +----------+------+------+
    |   TAB    | DEL  | END  |
    |  1.75u   |  1u  |  1u  |
    +----------+------+------+
    |  SPACE   |  BS  | ENTER|
    +----------+------+------+

---

### 5.1 Tab -> Space

The large 1.75u key cannot accept the conventional Spacebar keycap, which is physically much larger.

The Tab keycap is therefore repurposed as Space.

This is supported by a useful semantic relationship:

> Tab is whitespace expressed in grouped/incremental form.

Furthermore, Space can act as the primitive whitespace operation while modified Space can generate Tab.

This produces the following conceptual relationship:

    Space
        -> whitespace

    Shift/Ctrl/Alt/Super + Space
        -> Tab

This also avoids dedicating an additional prime physical key to Tab.

---

### 5.2 Delete -> Backspace

The Delete keycap is used for Backspace.

The semantic relationship is direct:

> Both operations remove text.

The distinction between forward Delete and backward Backspace is therefore considered less important at the keycap-label level than the functional grouping of text-removal operations.

This is particularly appropriate because Backspace is currently heavily used for correcting typing errors, while the long-term objective is to reduce that frequency.

---

### 5.3 End -> Enter

The three strongest donor candidates considered for Enter were:

#### Insert

Mnemonic: Insert a newline.

This is semantically plausible because pressing Enter in ordinary text input inserts a line terminator.

#### Home

Mnemonic: Return home.

This is a clever linguistic association, but technically Home is a navigation operation that moves the cursor toward the beginning of a line/document.

#### End

Mnemonic: End the statement/line.

This has particular relevance to programming and terminal usage. Pressing Enter commonly terminates a command, statement, or input line.

End was selected.

The mapping therefore intentionally uses a programming-oriented mnemonic:

> ENTER -> END -> terminate the current line/statement.

This is preferable to Home because Home's actual computational semantics are more strongly associated with navigation to the beginning of a region.

---

## 6. Current Right-Thumb Specification

The right-side three-key cluster is therefore provisionally canonical:

                 RIGHT THUMB CLUSTER

        +----------+------+------+
        |   TAB    | DEL  | END  |
        |  1.75u   |  1u  |  1u  |
        +----------+------+------+
             |        |      |
             v        v      v
           SPACE  BACKSPACE ENTER

The ordering follows multiple laws simultaneously:

1. Topology Conservation
   - Space remains adjacent to and left of Enter.

2. Priority
   - All three are high-value editing operations.

3. Functional Grouping
   - Editing operations are colocated.

4. Frequency
   - The largest/easiest position is assigned to Space.
   - Backspace remains highly accessible despite its low desired steady-state frequency.

5. Physical constraints
   - The 1.75u key accepts an appropriate donor keycap.
   - The conventional Spacebar does not.

6. Semantic mnemonic quality
   - Tab/Space, Delete/Backspace, and End/Enter each have defensible associations.

---

## 7. Modifier Strategy

The six thumb keys consist of three on each side.

The current architectural intention is:

    LEFT THUMB CLUSTER       RIGHT THUMB CLUSTER
    ------------------       ------------------
    Keyboard modifiers      Text editing

The left cluster is therefore intended to accommodate some combination of:

- Ctrl
- Shift
- Super
- Alt
- Fn
- potentially another modifier/function

The exact left-side allocation remains TBD.

This separation is intentional:

> Left thumb cluster = control of keyboard/application behavior.
>
> Right thumb cluster = manipulation of text.

This is especially appropriate for a keyboard-driven software-development workflow in which modifier combinations such as Ctrl-based terminal commands are common.

---

## 8. Open Questions

The following decisions remain deliberately unspecified:

- Exact physical ordering of Ctrl / Shift / Alt / Super / Fn.
- Whether any modifier should be duplicated.
- Whether conventional Shift positions should be retained in addition to thumb Shift.
- Exact location and behavior of Tab outside the modified-Space mechanism.
- Delete-forward behavior.
- Navigation layer design.
- Number/symbol layer design.
- Operating-system-specific Super/Alt behavior.
- Whether Fn is momentary, toggle-based, or chorded.
- Firmware implementation.
- Physical thumb-key orientation and ergonomic reach measurements.
- Empirical measurements after actual use.

These should be decided using the same four laws rather than by attempting to finalize the entire keyboard simultaneously.

---

## 9. Design Status

The current keymap has therefore established a firm principle:

                    RIGHT SIDE

            TAB       DELETE       END
             |           |          |
             v           v          v
           SPACE     BACKSPACE     ENTER

The keycap legends are not the keymap. They are donor artifacts selected for physical compatibility and mnemonic usefulness.

The underlying design objective is to produce a small, internally coherent keyboard whose physical topology, functional grouping, and accessibility remain predictable even though its physical key count differs substantially from a conventional staggered keyboard.
