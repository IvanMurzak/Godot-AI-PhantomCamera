# CLAUDE.md — Godot-AI-PhantomCamera

A **Godot-MCP extension** that wraps the third-party [**Phantom Camera**](https://github.com/ramokz/phantom-camera)
addon (Cinemachine-style virtual cameras), shipped as a **source-only NuGet package**
(`com.IvanMurzak.Godot.MCP.PhantomCamera`) that compiles inside a consumer's Godot project against the
consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe is
the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

This is an **addon-dependent ("Class B")** extension: Phantom Camera's classes are **not** in GodotSharp,
and the package **must not depend on the addon**. So the tools reference Phantom Camera's classes **only by
string name**, resolved + invoked at runtime, and **presence-gate** every editor tool so a missing addon
returns a clean structured `installed: false` result instead of crashing. The addon is **never** vendored,
submoduled, or downloaded by this repo — installing Phantom Camera is the consumer's own responsibility
(CI downloads a pinned copy only to exercise the e2e leg).

## Layout

- `src/Godot-AI-PhantomCamera/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_PhantomCamera.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_PhantomCamera.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Tools/Tool_PhantomCamera.Defaults.cs` — `phantomcamera-defaults` (pure-managed tool).
  - `Runtime/Interop/AddonInterop.cs` — dynamic name-resolution helper (global script-class list →
    `GD.Load` → `New()`); pure-managed resolution/result-shaping, the `Node`-constructing calls stay in
    `#if TOOLS` editor tools.
  - `Runtime/Interop/AddonGate.cs` — the shared `AddonGateResult` shape + `NotInstalled(...)` factory
    (pure-managed, unit-tested).
  - `Runtime/PhantomCamera/PhantomCameraEnums.cs` — the addon's class/member **snake_case** name constants
    + enum-int values (no compile-time enum types exist, so the constants ARE the contract — unit-tested).
  - `Editor/Tools/Tool_PhantomCamera.{HostCreate,Create,SetFollow,SetLookAt,SetPriority,Get}.cs` — editor
    tools behind `#if TOOLS` (touch `EditorInterface`/live nodes; main-thread-marshalled; presence-gated
    FIRST line; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.PhantomCamera.props` — the source-injection props (auto-imported by
    NuGet in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-PhantomCamera.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary):
  the tool-id consts, the `AddonGateResult` shape + hint text, the snake_case name + enum-int constants.
- `testbed/PhantomCamera-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the
  local-packed package; `dotnet build` of it is the source-injection proof.

## Tools

| Tool | Kind | File |
| --- | --- | --- |
| `phantomcamera-defaults` | pure-managed | `Runtime/Tools/Tool_PhantomCamera.Defaults.cs` |
| `phantomcamera-host-create` | editor | `Editor/Tools/Tool_PhantomCamera.HostCreate.cs` |
| `phantomcamera-create` | editor | `Editor/Tools/Tool_PhantomCamera.Create.cs` |
| `phantomcamera-set-follow` | editor | `Editor/Tools/Tool_PhantomCamera.SetFollow.cs` |
| `phantomcamera-set-look-at` | editor | `Editor/Tools/Tool_PhantomCamera.SetLookAt.cs` |
| `phantomcamera-set-priority` | editor | `Editor/Tools/Tool_PhantomCamera.SetPriority.cs` |
| `phantomcamera-get` | editor | `Editor/Tools/Tool_PhantomCamera.Get.cs` |

The editor tool set is confirmed/adjusted against the installed Phantom Camera **v0.11** API in the
implement step. The `PhantomCameraHost` companion is **required** by the addon or a PhantomCamera does
nothing — the `host-create` tool ensures one exists.

## Build / test (no Godot binary, addon absent)

```bash
dotnet build src/Godot-AI-PhantomCamera/Godot-AI-PhantomCamera.csproj   # source-only package compiles tools (addon NOT needed)
dotnet test  tests/Godot-AI-PhantomCamera.Tests/Godot-AI-PhantomCamera.Tests.csproj
dotnet pack  src/Godot-AI-PhantomCamera/Godot-AI-PhantomCamera.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/PhantomCamera-Testbed.csproj                       # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
**`dotnet build -c Debug` MUST exit 0 with the Phantom Camera addon ABSENT** — the Class-B no-dependency
gate: the package compiles on a machine that never installed the addon, because it never names an addon
type (only string names). When proving locally, note `dotnet pack` re-uses the **global NuGet cache** for
an already-cached version: if you re-pack the same `Version`, clear
`~/.nuget/packages/com.ivanmurzak.godot.mcp.phantomcamera/<ver>` (or pack a unique version) before
re-restoring the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.PhantomCamera`. Every `.cs` starts with the Apache-2.0 header.
- Pure-managed tools + the `AddonInterop` resolution/`AddonGate` result shape + the name/enum-int constants
  → `Runtime/` (outside `#if TOOLS`, unit-testable); editor-driving tools → `Editor/` (behind `#if TOOLS`,
  every Godot call via `MainThread.Instance.Run(...)`, the presence gate as the FIRST line, E2E-verified).
- **No GodotSharp and no addon dependency** — the package declares ONLY the `com.IvanMurzak.McpPlugin` /
  `com.IvanMurzak.ReflectorNet` min-version pins; Phantom Camera is referenced **by string name only** (CI
  asserts the nuspec). Keep the MCP pins in lockstep with the core Godot-MCP addon; bump with
  `commands/update-core.ps1`.
- Phantom Camera member names are **GDScript `snake_case`** (`follow_target`, `follow_mode`, `priority`),
  **not** C# PascalCase; enums are plain ints. Centralize + unit-test them (there are no compile-time enum
  types to lean on).
- One `[AiToolType] partial class Tool_PhantomCamera`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, the publish secrets.
