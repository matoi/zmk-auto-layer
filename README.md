# ZMK-AUTO-LAYER

This module adds an `auto-layer` behavior to ZMK. A layer activated by the behavior continues to be
active for as long as keys in a configurable `continue-list` are pressed, and deactivates
automatically on any other key press.

This is a re-implementation of [PR #1451](https://github.com/zmkfirmware/zmk/pull/1451), separating
the `auto-layer` behavior from `caps-word` and making the layer index a parameter.

> **Note:** This branch targets **ZMK v0.3** and is not compatible with other versions.

## Usage

To load the module, add the following entries to `remotes` and `projects` in `config/west.yml`.

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: matoi
      url-base: https://github.com/matoi
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3
      import: app/west.yml
    - name: zmk-auto-layer
      remote: matoi
      revision: feat/strict-modifiers
  self:
    path: config
```

## Configuration

There are five configuration properties for the behavior (the original implementation has four;
`strict-modifiers` is the new addition):

- **`continue-list`** (required): An array of keycodes that will keep the layer active.
- **`ignore-alphas`** (optional): If set, the layer will not be deactivated by any alphabetic key.
- **`ignore-numbers`** (optional): If set, the layer will not be deactivated by any numeric key.
- **`ignore-modifiers`** (optional): If set, the layer will not be deactivated by any modifier key.
- **`strict-modifiers`** (optional, requires ZMK v0.3): Implicitly enables `ignore-modifiers` (standalone modifier
  presses will not deactivate the layer) and ensures that modifier+key combos are always checked
  against the `continue-list` — bypassing `ignore-alphas` and `ignore-numbers` when explicit
  modifiers are held. This can be used with or without `ignore-alphas`/`ignore-numbers`:
  - **With `ignore-alphas`**: Pressing `H` alone continues the layer (via `ignore-alphas`), but
    `Ctrl+H` only continues if `LC(H)` or `RC(H)` is in the `continue-list`.
  - **Without `ignore-alphas`**: Pressing `H` alone is checked against the `continue-list` as
    usual, while `Ctrl+H` is also checked against the `continue-list`. Standalone modifier presses
    (e.g. pressing `Ctrl` before a key) do not deactivate the layer.

Behavior instances take one mandatory argument that specifies the index of the layer to be
activated.

## Example: Num-word

The module pre-configures a `num-word` behavior instance that activates a layer for as long as only
numbers and a few other keys are pressed. To use it, source the definition at the top of your keymap:

```c
#include <behaviors/num_word.dtsi>
```

Then, add `&num_word NUM` anywhere to your keymap where `NUM` is the index of your numbers layer.

**Customization**: By default, `num_word` continues on number keys as well as on `BSPC`, `DEL`,
`DOT`, `COMMA`, `PLUS`, `MINUS`, `STAR`, `FSLH`, and `EQUAL`. To customize the `continue-list`,
overwrite it in the keymap. For instance:

```c
&num_word {
  continue-list = <BSPC DEL DOT COMMA>;
};
```

## Example: General case

Custom behavior instances can be defined using the general `auto-layer` behavior. The following
illustrates how to define a `nav-word` behavior that continues on arrow keys, `PG_UP`, `PG_DOWN`,
and all modifiers.

```c
/ {
  behaviors {
    nav_word: nav_word {
      compatible = "zmk,behavior-auto-layer";
      #binding-cells = <1>;
      continue-list = <LEFT DOWN UP RIGHT PG_DN PG_UP>;
      ignore-modifiers;
    };
  };
};
```

## Example: Shift-word with strict modifiers

This example uses the newly added `strict-modifiers` property.
The following defines a `shft-word` behavior that continues on all alphabetic keys and
shift keys, but deactivates only on _specific_ modifier+key combos in the `continue-list` (e.g. Emacs-style
`Ctrl+H`, `Ctrl+D`, `Ctrl+F`, `Ctrl+B`) and arrow keys. Other combos like `Ctrl+Z` will deactivate the layer.

```c
/ {
  behaviors {
    shft_word: shft_word {
      compatible = "zmk,behavior-auto-layer";
      #binding-cells = <1>;
      continue-list = <BSPC DEL LC(H) RC(H) LC(D) RC(D) LC(F) RC(F) LC(B) RC(B) LEFT RIGHT LSHFT RSHFT>;
      ignore-alphas;
      strict-modifiers;
    };
  };

  keymap {
    compatible = "zmk,keymap";
    default_layer {
      bindings = <
        // ...
        &shft_word 1 // Use layer number 1 (shift_layer)
        // ...
      >;
    };
    shift_layer {
      bindings = <
        &kp LS(A) &kp LS(B) &kp LS(C) /* ... */
      >;
    };
  };
};
```

## References

- The behavior is inspired by Jonas Hietala's
  [Numword](https://www.jonashietala.se/blog/2021/06/03/the-t-34-keyboard-layout/#where-are-the-digits)
  for QMK
- A zero-parameter version where layers are part of the behavior definition
  is available [here](https://github.com/urob/zmk-auto-layer/tree/zero-param)
- My personal [zmk-config](https://github.com/urob/zmk-config) contains a more advanced example
