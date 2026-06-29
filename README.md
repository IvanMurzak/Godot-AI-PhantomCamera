<h1 align="center">Godot AI PhantomCamera</h1>

<p align="center">
  AI <b>MCP tools</b> for the <b>Phantom Camera</b> Godot addon (Cinemachine-style virtual cameras) —
  an extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-PhantomCamera` adds a focused MCP tool family for the community
[**Phantom Camera**](https://github.com/ramokz/phantom-camera) addon — virtual cameras with
follow/look-at behaviours, priorities, and damping (the Godot analog of Unity's Cinemachine). The tools
are authored in C# with `[AiToolType]` / `[AiTool]` (the same model as Unity-MCP and the core Godot-MCP
addon) and shipped as a **source-only NuGet package** that compiles inside any consumer's Godot project
against the consumer's own GodotSharp — no bundled Godot, no version lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

This is an **addon-dependent** extension: it drives Phantom Camera's nodes **by name at runtime**
(resolved from Godot's global script-class list) and never takes a compile-time dependency on the addon.
Every editor tool is **presence-gated** — if Phantom Camera is not installed, the tool returns a clean,
structured `installed: false` result (with an install hint) instead of crashing.

## Required prerequisite — Phantom Camera (install it yourself)

This extension **does NOT include** the Phantom Camera addon and does not download or vendor it. You must
install Phantom Camera into your own Godot project separately:

- Install **Phantom Camera** from the **Godot Asset Library**, or from
  **https://github.com/ramokz/phantom-camera** (tested against **v0.11.0.2**).
- Enable it under **Project Settings → Plugins**.

Without Phantom Camera installed and enabled, the editor tools below report `installed: false` and take no
action (by design).

> Phantom Camera is © its authors and distributed under the **MIT License**. This extension is **not
> affiliated with, endorsed by, or sponsored by** the Phantom Camera project — it merely provides AI tools
> that drive it. See the addon's own repository for its licence and terms.

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `phantomcamera-defaults` | pure-managed | Return the recommended follow-mode + damping starter config (no addon needed). |
| `phantomcamera-host-create` | editor (`#if TOOLS`) | Ensure/create the required `PhantomCameraHost` on a scene `Camera3D` (the host is what actually drives the real camera). |
| `phantomcamera-create` | editor (`#if TOOLS`) | Instantiate a `PhantomCamera3D` node in the edited scene; optional parent, name, priority. |
| `phantomcamera-set-follow` | editor (`#if TOOLS`) | Set a PhantomCamera's `follow_mode` + `follow_target` (node path). |
| `phantomcamera-set-look-at` | editor (`#if TOOLS`) | Set a PhantomCamera's `look_at_mode` + `look_at_target`. |
| `phantomcamera-set-priority` | editor (`#if TOOLS`) | Set a PhantomCamera's `priority` (higher wins). |
| `phantomcamera-get` | editor (`#if TOOLS`) | Read a PhantomCamera's scalar config (read-only). |

The exact editor tool set is finalized against the installed Phantom Camera **v0.11** API. Pure-managed
tools (no Godot native API — the `*-defaults` tool plus the addon class/member name + enum-int constants)
live under `src/Godot-AI-PhantomCamera/Runtime/` and are CI-unit-tested; editor-driving tools live under
`Editor/` behind `#if TOOLS`, marshal every Godot call onto the editor main thread via
`MainThread.Instance.Run(...)`, and resolve Phantom Camera's classes dynamically through
`Runtime/Interop/AddonInterop.cs`.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon **and** the Phantom Camera
addon (see the prerequisite above). Then either:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.PhantomCamera`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.PhantomCamera" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary, addon absent)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless. Because the
package references Phantom Camera **only by string name**, it compiles cleanly with the addon **absent**:

```bash
dotnet build src/Godot-AI-PhantomCamera/Godot-AI-PhantomCamera.csproj            # compiles tools (Godot API resolves; addon NOT needed)
dotnet test  tests/Godot-AI-PhantomCamera.Tests/Godot-AI-PhantomCamera.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-PhantomCamera/Godot-AI-PhantomCamera.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/PhantomCamera-Testbed.csproj                                # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon **and the pinned Phantom Camera addon**, then drives each tool and asserts the presence-gated
results.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `CLAUDE.md` — maintainer notes (incl. the addon-dependent / presence-gate model).

## Publish

Source-only, version-gated release (see `docs/ci.md`): configure NuGet Trusted Publishing (OIDC) + the
`NUGET_USER` variable, bump `<Version>` (`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`;
`release.yml` runs the full matrix, publishes the package to NuGet, and cuts an atomic GitHub Release.

License: **Apache-2.0** (this extension). The Phantom Camera addon it drives is MIT, © its authors —
install it yourself (see the prerequisite above); it is never bundled here.
