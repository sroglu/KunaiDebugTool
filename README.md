# KunaiDebugTool

Immediate-mode in-game debug overlay for Unity — the whole UI renders in **one draw call** with
**zero per-frame GC** via a Burst-compiled vertex pipeline. Runtime namespace `Kunai`, static entry
point `KUI.*`; sanctioned asmdef `PFound.KunaiDebugTool`.

## Quick reference

```csharp
using Kunai;

KUI.Initialize(fontAtlasTexture, fontMetricsTextAsset);   // once, at startup
KuiConsole.Initialize();                                  // optional: capture Unity logs
KUI.RegisterWindow(new KuiConsoleWindow());               // mount a tool window
// toggle at runtime with backtick (`) / F1, or KUI.IsVisible = true
```

## Dependencies

`Unity.Burst` + `Unity.Collections` + `Unity.Mathematics`, plus `Unity.InputSystem` when the
Input System package is present. Standalone leaf — no other PFound dependency.

## Setup at a glance

- Add `Hidden/KUI-Combined` to *Graphics → Always Included Shaders* (else it is stripped from builds).
- *Player → Active Input Handling* may be any of the three settings — `KuiInput` reads through the
  new Input System when `ENABLE_INPUT_SYSTEM` is defined, and the legacy `Input` manager otherwise.
- A `Camera.main` must exist (drives layout + the ortho projection).

Full detail in [MODULE.md](MODULE.md).

## Fonts & licensing

The overlay renders from a BMFont atlas baked out of **Iosevka Nerd Font Mono**
(SIL Open Font License 1.1). The licence text and attribution live in
[bake/IosevkaNerdFontMono-Regular.LICENSE.md](bake/IosevkaNerdFontMono-Regular.LICENSE.md), and
cover both the bundled `.ttf` and the generated atlas. `bake/bake.sh` regenerates the atlas.

## Docs

- Deep reference: [MODULE.md](MODULE.md)
- Zero-allocation render path — how the one-draw-call / no-GC guarantee actually works, and where
  it does not hold: [ZERO-ALLOCATION.md](ZERO-ALLOCATION.md)
- History: [CHANGELOG.md](CHANGELOG.md)
