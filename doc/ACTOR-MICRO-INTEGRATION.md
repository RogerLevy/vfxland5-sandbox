# Actor-Micro Integration Design

## Overview

This document describes the integration between the **Actor System** (Supershow game objects) and **Microcodes** (GUI-based visual scripting system).

## The Problem

**Actors** and **Micros** exist in separate spatial systems:
- **Actors** live in game/stage space (traditional game world coordinates)
- **Micros** live in GUI space (infinite desktop/canvas)

They cannot be directly parent-child related via dltree because they're in different coordinate systems.

## Architecture

### Actor Mind Reference

Each actor can point to a "mind" - the root of a micro network that controls its behavior:

```forth
class: %actor
    prop mind :ref %element          \ points to micro network root
    prop mind-source :lstring <save  \ template file: "enemy-ai.json"
```

The `mind` lives in GUI space (typically under a stage's desktop), while the actor lives in game space.

### Stage Desktop

Each stage has its own desktop for scene-specific GUI elements:

```forth
class: %stage
    prop desktop :ref %desktop  \ scene GUI - minds, notes, debug viz
```

**Structure:**
```
%stage
  ├─ actors (game objects, flat display list)
  └─ desktop (GUI space)
      ├─ player-mind (%origin micro network)
      ├─ enemy-mind (%origin micro network)
      ├─ debug-panel (visualizations)
      └─ notes-panel (dev notes, drawings)
```

## Lifetime Management

**Coupled Deletion:**
```forth
%actor :: unload
    mind @ ?dup if unload then  \ unload mind tree when actor dies
    unloaded on ;
```

When an actor is deleted, its associated mind (and all children) are also deleted.

## Save/Load Strategy

### Template vs Instance Problem

**Challenge:** Support two use cases:
1. **Scene saves** - Template-based minds (loaded from files at runtime, don't save with scene)
2. **Save states** - Full snapshot including runtime-created minds

**Solution: `mind-source` Property**

Actors track where their mind came from:

```forth
%enemy :: init
    mind @ -exit  \ mind already exists (from save state), skip
    mind-source @ ?dup if
        lcount load-tree desktop @ push dup me bind-mind
    then ;
```

### Save Modes

#### 1. Scene Save (Development)

**Purpose:** Save scene for editing, with all dev tools

**Saves:**
- Entire desktop tree (minds + notes + debug panels + everything)
- All actors with mind references

**Format:**
```json
{
    "desktop": {
        "class": "%desktop",
        "children": [
            {"id": 0, "class": "%origin", ...},  // player mind
            {"id": 1, "class": "%panel", ...},   // debug panel
            {"id": 2, "class": "%label", "txt": "TODO", ...}
        ]
    },
    "actors": [
        {"class": "%player", "mind": 0, "x": 100, ...}
    ]
}
```

#### 2. Scene Save (Game-Ready)

**Purpose:** Save scene for game builds, filtered

**Saves:**
- Only minds referenced by actors (and their children)
- All actors

**Excludes:**
- Notes, debug panels, dev harnesses

**Format:**
```json
{
    "elements": [
        {"id": 0, "class": "%origin", ...}  // only minds
    ],
    "actors": [
        {"class": "%player", "mind": 0, "x": 100, ...}
    ]
}
```

#### 3. Save State (Runtime Snapshot)

**Purpose:** Save/restore exact game state

**Saves:**
- All minds (including runtime-instantiated template-based ones)
- All actors with full state

**Format:** Same as game-ready, but includes runtime-created minds

**Key difference:** On load, actors check if mind exists:
```forth
mind @ -exit  \ mind loaded from save state, skip recreation
```

### Implementation

```forth
\ Mark elements as dev-only
class: %element
    prop exclude <save  \ exclude from game-ready saves

\ Save full scene (development)
: save-stage-dev ( stage path len - )
    \ Save entire desktop + actors

\ Save game-ready scene (filtered)
: save-stage-game ( stage path len - )
    \ Save only minds + actors (skip exclude elements)

\ Save runtime state
: save-stage-state ( stage path len - )
    \ Save all minds (even template-based) + actors
```

## Coordinate System Integration

### Option A: Actor Pushes to Micro

Actor updates micro's remote position each frame:

```forth
%actor :: behave
    micro @ ?dup if
        x 2@ over >rx 2!  \ sync actor pos to origin
        me swap { ... process micros ... }
    then ;
```

### Option B: Micro Pulls from Actor

Micro checks for actor parent and uses its position:

```forth
%move :: _gather
    parent @ ?dup if
        dup %actor is? if
            >x 2@ ... use actor position
        then
    then ;
```

### Option C: Micro Controls Actor

Micro directly modifies actor position:

```forth
%origin :: _act
    parent @ ?dup if
        dup %actor is? if
            ang @ val @ vec rot >x 2+!  \ move actor
            exit
        then
    then
    \ Otherwise use normal origin rx/ry behavior
    ang @ val @ vec rx 2+! ;
```

## Usage Examples

### Creating Actor with Mind

```forth
%player :: init
    mind @ -exit  \ already have mind from load
    s" player-ai.json" mind-source !
    mind-source @ lcount load-tree
    stage @ >desktop @ push  \ add to stage desktop
    dup me bind-mind ;       \ bind to actor
```

### Manual Binding

```forth
\ Create actor
%player make

\ Load mind from template
s" player-ai.json" load-tree stage @ >desktop @ push

\ Bind them
player-actor bind-mind
```

### Saving Development Scene

```forth
stage s" level1-dev.json" save-stage-dev
```

### Saving Game Build

```forth
stage s" level1.json" save-stage-game
```

### Saving Runtime State

```forth
stage s" savegame.json" save-stage-state
```

## Future: Workspaces

The ultimate vision is saving entire development sessions:

```
%workspace
  ├─ %container (stage 1)
  │   ├─ actors
  │   └─ desktop (minds + dev tools)
  ├─ %container (stage 2)
  │   ├─ actors
  │   └─ desktop
  └─ %notes-panel (global notes)
```

This integration design doesn't preclude this - stages can be nested, desktops can be saved recursively.

## Open Questions

1. **Coordinate sync timing** - When does actor position sync to micro?
2. **Bi-directional references** - Should micros have back-reference to actor?
3. **Multiple minds** - Should actors support multiple micro networks, or just one root?
4. **Rendering integration** - How do micro sprites interact with actor sprites?

## Implementation Status

- [ ] Add `mind` and `mind-source` properties to %actor
- [ ] Add `desktop` property to %stage
- [ ] Implement `save-stage-dev`
- [ ] Implement `save-stage-game`
- [ ] Implement `save-stage-state`
- [ ] Implement `load-stage-scene` with ID resolution
- [ ] Add coupled lifetime (unload mind when actor unloads)
- [ ] Test template-based mind loading
- [ ] Test save state round-trip
