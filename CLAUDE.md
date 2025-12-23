# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sandbox

This is an experimental sandbox environment for VFXLand5 game engine development. Contains prototype code for testing new concepts and systems.

**Engine Submodule**: `vfxland5/` - Full VFXLand5 engine. See `vfxland5/CLAUDE.md` for comprehensive engine documentation.

## Development Commands

```bash
# Run the sandbox project (Windows)
dev.bat

# Compile a release build
vfxland5/engineer/turnkey.bat <projectname>
```

## Current Experiments

- **Microcodes** (`microcodes.vfx`): Godot-like node system with tiny modular workers that perform simple actions continuously. Part of a "physicality" project for tangible game programming.
- **Tiled Integration** (`tiled.vfx`): TMJ/TSJ file parsing for Tiled map editor integration with scene and tilemap loading.
- **Dungeon/Town**: Tiled-based level experiments using the tilemap3 system.

## Project Structure

```
main.vfx           # Entry point - includes active experiment
microcodes.vfx     # Micro-node system experiment
tiled.vfx          # Tiled JSON integration
colors.vfx         # Named color definitions (rgba8 format)
shapes.vfx         # Drawing primitives
platformer.vfx     # Platformer prototype
scrolling.vfx      # Scrolling system test
dungeon/           # Dungeon level experiment (Tiled maps)
town/              # Town level experiment (Tiled maps)
obj/               # Object templates
vfxland5/          # Engine submodule
```

## Key Patterns

### Path Macros
- `%vfxland5%` - Engine root directory
- `%idir%` - Project include directory (sandbox root)

### Require System
Uses `require` for conditional includes (only loads once):
```forth
require %vfxland5%/supershow/tilemap3.vfx
require %idir%/colors.vfx
```

### NIBS Object System
Classes use trait composition. Example from microcodes:
```forth
trait: %micro-like
    is-a %element
    :: action  ( - ) ;   \ Protocol method
    prop val :fixed      \ Property with type hint
    static icon-color    \ Class-level storage
trait;

class: %micro
    is-a %micro-like
    template { 1.0 val ! }   \ Default values
class;

c: %move %micro ang acc ;    \ Compact class syntax
```

### Tiled Map Loading
```forth
JSONFILE dungeon/dungeon.tmj   \ Load TMJ file
dungeon/dungeon.tmj z" Tiles" *tilemap-from  \ Create tilemap from layer
```

## Engine References

For detailed documentation on:
- VFX Forth dialect and conventions
- NIBS object system
- Actor/sprite systems
- Tilemap rendering
- Fixed-point math

See `vfxland5/CLAUDE.md`
