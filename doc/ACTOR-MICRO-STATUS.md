# Actor-Micro Integration Status

## Current State

The foundation is in place in `mind.vfx`. Actors can have minds (micro networks), stages can have flylofts (GUI containers for minds), and the binding/unbinding lifecycle works.

**What exists:**
- `%actor` has `mind` and `mindsrc` properties
- `%stage` has `flyloft` property (the GUI container for this stage's minds)
- `%element` has `bod` property (back-reference to actor)
- `it's-alive` binds actor↔element bidirectionally
- `init-mind` loads mind from template file
- `*mind` creates mind programmatically
- Actor unload triggers mind unload
- Element delete clears actor's mind reference (dangling pointer bug fixed)

## Architecture Summary

```
session (root element)
└─ desktop/container
    ├─ scene-viewport → %stage (reference)
    ├─ player-mind (%origin)
    ├─ enemy-mind (%origin)
    └─ debug-panel

%stage.flyloft → desktop (where minds live)
%actor.mind → element (the mind)
%element.bod → actor (the body)
```

Key insight: Stages are external/static (Supershow), elements are dynamic (GUI). The GUI scaffolds around Supershow, not the other way around.

## What's Not Implemented

### From ACTOR-MICRO-INTEGRATION.md

1. **Save modes** - `save-stage-dev`, `save-stage-game`, `save-stage-state`
2. **ID-based binding** - Minds get IDs in JSON, actors reference by ID, resolved on load
3. **Exclude flag** - Mark elements as dev-only for filtered saves
4. **Coordinate sync** - Actor position ↔ micro origin synchronization
5. **load-stage-scene** - Load with ID resolution for mind binding

### Open Questions

- Coordinate sync timing (when does actor pos sync to micro?)
- Multiple minds per actor?
- Micro sprites vs actor sprites rendering integration

## Priorities

### Essential (do first)
1. **Get one actor controlled by micros working** - Prove the concept end-to-end

### Can Defer
- Save mode distinctions (dev/game/state) - just save everything for now
- ID-based binding - only needed if reloading scenes with actor→mind links
- Exclude flag - only for shipping game builds
- Multiple save modes - one format works until you ship

### Future/Nice-to-have
- Full coordinate sync options
- Scene-panel for multiple stages
- Workspace persistence

## Minimal Working Loop

1. Create stage with flyloft
2. Create actors, call `init-mind` (or use `*mind`)
3. Micros control actor behavior
4. Save flyloft tree separately if needed
5. Actors recreated from level data (or saved separately)

## Files

- `mind.vfx` - Core implementation
- `doc/ACTOR-MICRO-INTEGRATION.md` - Full design document
- `doc/ACTOR-MICRO-STATUS.md` - This file

## Next Steps

When returning to this:
1. Create a simple test: one actor, one mind, micros move the actor
2. Verify the lifecycle (create, run, delete) works cleanly
3. Then consider save/load if needed
