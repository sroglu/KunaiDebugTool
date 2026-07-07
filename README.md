# KunaiDebugTool

Immediate-mode in-game debug overlay for Unity: console, commander, inspector, profiler, system
info, and bug reporter. The whole UI renders in **one draw call** with **zero per-frame GC** via a
Burst-compiled vertex pipeline. Standalone leaf module — no dependency on other PFound modules.

Namespace: `Kunai`. Static entry point: `KUI.*`. Assemblies: `mehmetsrl.KunaiDebugTool` (runtime),
`mehmetsrl.KunaiDebugTool.Editor`. Full internals/architecture live in `MODULE.md`.

## Model

- **UI layer** — immediate-mode widgets (`KUI.Label`, `Button`, `Toggle`, `Slider`, `TextField`,
  `ChipStrip`, `BeginScroll`/`EndScroll`, `BeginCollapsible`/`EndCollapsible`, `BeginGroup`).
  Widget calls push `DrawCommand` structs into a deferred command buffer; GPU work happens once at
  flush.
- **Windows** — subclass `KuWindow`, override `Title` and `OnRenderUI()`, register with
  `KUI.RegisterWindow(...)`.
- **Tools** — prebuilt windows on top of the UI layer: `KuiConsoleWindow`, `KuiCommanderWindow`,
  `KuiInspectorWindow`, `KuiProfilerWindow`, `KuiSystemInfoWindow`, `KuiMasterWindow`, and the
  editor-side `KuiBugReporterWindow`.

## Public API (`KUI`)

**Lifecycle:** `KUI.Initialize(Texture2D fontAtlas, TextAsset fontMetrics)`, `KUI.Shutdown()`,
`KUI.IsVisible` (get/set), `KUI.Settings`, `KUI.LastTickDurationNs`.

**Windows:** `KUI.RegisterWindow(KuWindow)`, `KUI.UnregisterWindow(KuWindow)`.

**Widgets** (valid only inside a `KuWindow.OnRenderUI()`): `Label`, `Rect`, `Separator`, `Button`,
`Toggle`, `Slider`, `TextField`, `ChipStrip`, `BeginScroll`/`EndScroll`,
`BeginCollapsible`/`EndCollapsible`, `BeginGroup`.

**Console:** `KuiConsole.Initialize(capacity)` hooks Unity log events; then
`KUI.RegisterWindow(new KuiConsoleWindow())` mounts the UI.

## Setup / wiring

The overlay is **self-hosting** — you do NOT place any GameObject or prefab in a scene.
`KUI.Initialize(...)` builds the `KuiContext`, subscribes its tick to `Application.onBeforeRender`,
and spawns a hidden `DontDestroyOnLoad` GameObject (`[KunaiOverlayRunner]`) that issues the
overlay's `CommandBuffer` at `WaitForEndOfFrame` (so it draws on top of every uGUI / UI Toolkit
panel). Render-pipeline hookup (Built-in vs URP/HDRP/any SRP) is detected at runtime — no pipeline
package reference required.

```csharp
using Kunai;

// once, at startup (e.g. a bootstrap MonoBehaviour.Awake)
KUI.Initialize(fontAtlasTexture, fontMetricsTextAsset);   // your baked font atlas + metrics asset
KuiConsole.Initialize();                                  // optional: capture Unity logs
KUI.RegisterWindow(new KuiConsoleWindow());               // mount a tool window

// author your own window
public sealed class MyDebug : KuWindow
{
    public override string Title => "Debug";
    public override void OnRenderUI()
    {
        KUI.Label("Hello");
        if (KUI.Button("Reset")) ResetGame();
    }
}
KUI.RegisterWindow(new MyDebug());
```

### Required project configuration

1. **Shader in Always Included Shaders.** Add `Hidden/KUI-Combined` to
   *Project Settings → Graphics → Always Included Shaders*. It has no scene/material reference to
   survive player-build stripping otherwise; `KUI.Initialize` asserts it can be found.
2. **Legacy input must be enabled.** *Player → Active Input Handling* must be `Input Manager (Old)`
   or `Both`. Kunai reads keyboard/mouse/touch via the legacy `UnityEngine.Input` API; under
   `Input System Package (New)` **only**, the toggle keys (`` ` `` / F1), the top-left double-tap,
   and touch indicators all silently no-op (the overlay still renders if you set
   `KUI.IsVisible = true` programmatically). Changing this setting requires an Editor restart.
3. **A `Camera.main`.** Layout and the ortho projection are driven by `Camera.main.pixelWidth/Height`,
   so a camera tagged `MainCamera` must exist.

Toggle the overlay at runtime with the `` ` `` or F1 key (or the top-left double-tap on touch), or
programmatically via `KUI.IsVisible`. Call `KUI.Shutdown()` to tear it all down.

## Testing / Layout

- `Runtime/` — `Core/` (context, overlay runner), `Widgets/` (the `KUI` facade + widget impls),
  `Windows/`, `Tools/` (console/commander/inspector/profiler/system-info/master), `Pipeline/`
  (Burst vertex jobs), `Font/`, `Input/`, `Layout/`, `DPI/`, `Settings/`, `Shaders/`, `Styles/`,
  `Reflection/`.
- `Editor/` — bug reporter + editor tooling. `Tests/Runtime/` — NUnit suite.
