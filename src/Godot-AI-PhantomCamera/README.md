# PhantomCamera Tools

AI MCP tools for Godot PhantomCamera.

A **source-only** MCP tool extension for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP)
that adds AI tools for the community [**Phantom Camera**](https://github.com/ramokz/phantom-camera) addon
(Cinemachine-style virtual cameras). The package ships C# source (no compiled DLL, no bundled Godot) that
compiles inside your Godot project against your own GodotSharp, so it never locks you to a Godot version.
It drives Phantom Camera **by string name at runtime** and never depends on the addon at compile time, so
every tool is **presence-gated** (a missing addon returns a clean `installed: false` result).

## Required prerequisite — Phantom Camera (install it yourself)

This extension **does NOT include** the Phantom Camera addon. Install it separately into your Godot project
from the **Godot Asset Library** or **https://github.com/ramokz/phantom-camera** (tested against
**v0.11.0.2**), and enable it under **Project Settings → Plugins**. Phantom Camera is © its authors under
the **MIT License**; this extension is **not affiliated with or endorsed by** it.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.PhantomCamera

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.PhantomCamera" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool families are auto-discovered — no registry edit.

## Tools

Every editor tool is **presence-gated**: when the Phantom Camera addon is not installed it returns a
structured `installed: false` result with an install hint instead of crashing.

| Tool | Kind | Description |
| --- | --- | --- |
| `phantomcamera-defaults` | pure-managed | Recommended starter config (priority, follow/look-at mode, damping). No addon needed. |
| `phantomcamera-host-create` | editor | Ensure a `PhantomCameraHost` on a `Camera3D` (required by the addon to drive the real camera). |
| `phantomcamera-create` | editor | Create a `PhantomCamera3D` virtual camera (optional name / parent / priority). |
| `phantomcamera-set-follow` | editor | Set a PhantomCamera's follow mode and/or follow target. |
| `phantomcamera-set-look-at` | editor | Set a PhantomCamera's look-at mode and/or look-at target. |
| `phantomcamera-set-priority` | editor | Set a PhantomCamera's priority (higher wins). |
| `phantomcamera-get` | editor | Read a PhantomCamera's scalar config (read-only). |

License: Apache-2.0.
