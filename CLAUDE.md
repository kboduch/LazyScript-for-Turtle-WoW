# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LazyScript is a WoW 1.12 addon (Turtle WoW private server) that provides a scripting language for automating class ability rotations. Users write "forms" — prioritized action lists with conditional criteria — and bind them to a macro (`/lazyscript` or `/ls`). The engine evaluates each line top-to-bottom and executes the first action whose conditions are all true.

This is an **Enhanced Edition** fork based on Fengshen's edit, adding decimal time support (`-every0.5s`), `-ifTargetInMeleeRange` for Paladin/Shaman, `-ifSwinged` (requires SP_SwingTimer addon), and `targettarget` health support.

## Addon Structure

There are two layers of addons:

**Core (`Addons/LazyScript/`)** — Loaded for all classes:
- `LazyScript.lua` — Global `lazyScript` table, state initialization, slash command handlers
- `Parse.lua` — `ParseLine()` and `ParseForm()`: the form parser and `parsedFormCache`
- `ParseGeneral.lua` — General actions (racials, pet commands, items) and `bitParsers` / `masks` tables shared across all classes
- `ParseBuffs.lua` — Buff/debuff checking criteria (`ifPlayerHasBuff`, `ifTargetHasDebuff`, etc.)
- `Actions.lua` — Base `Action`, `PetAction`, `ShapeshiftForm`, `CastSpellByRankAction` class definitions
- `Util.lua` — `split()`, `re()`, `rebit()`, `relax()` regex helpers used everywhere
- `Deathstimator.lua` — Time-to-death (`-ifTimeToDeath`) estimation logic
- `ImmunityTypeTracking.lua` — Tracks target immunity types
- `AutoAttack.lua` — Auto-attack management
- `Interrupt.lua` — Interrupt tracking

**Class addons (`Addons/Lazy<Class>/`)** — `LoadOnDemand: 1`, only loaded for matching class:
- `Lazy<Class>.lua` — Sets `lazy<Class> = lazyScript` (aliasing), calls load functions, registers events
- `Parse<Class>.lua` — Registers class-specific actions and `bitParsers`/`masks` (e.g. stances, combo points, energy)
- `Localization.lua` — Class-specific locale strings
- `<Class>.toc` — Declares `RequiredDeps: LazyScript`

The class addon aliases itself to the core (`lazyWarrior = lazyScript`), so all core functions/tables are accessible under the class namespace.

## Form Syntax

Forms are stored in `lsConf.forms` (per-character SavedVariable). Each line follows:
```
action[-criteria][-criteria]...
```

- Actions are defined by short-code keys in `lazyScript.actions` (e.g. `ss`, `heroicStrike`)
- Criteria are parsed by `lazyScript.bitParsers` functions; `Parse.lua:ParseLine()` splits on `-` and dispatches each token
- Dashes within known spell names (e.g. `Mind-numbing Poison`) must be escaped with `\-`
- Comments: `--`, `//`, or `#` at start of line
- `includeForm=<name>` — inline another form's lines
- `callForm=<name>-[criteria]` — call another form only if criteria pass
- Multiple non-GCD actions can chain in one line; at most one GCD-triggering action per line

## Key Patterns

**Adding a new action** (e.g. new spell for a class):
1. Add an entry in `Parse<Class>.lua` inside `LoadParse<Class>()`:
   ```lua
   lazyWarrior.actions.newSpell = lazyWarrior.Action:New("newSpell", "TextureName")
   ```
2. If the action needs implicit conditions (e.g. requires a stance), add a `bitParser` function instead of a plain table entry.

**Adding a new criterion/mask**:
1. Add a `bitParser` function in `ParseGeneral.lua` or the class `Parse<Class>.lua`
2. The function signature is `function(bit, actions, masks)` — return `false` if not matched, `nil` on syntax error, `true` on success (appending to `masks`)

**Action constructor signature**:
```lua
lazyScript.Action:New(code, texture, interruptsAutoAttack, triggersGlobal, problemTexture)
```
- `triggersGlobal = false` means action does NOT trigger the GCD (can chain multiple per line)

## SavedVariables

- `lsConfGlobal` — Global settings (minimap position, etc.)
- `lsConf` — Per-character settings including `lsConf.forms` (table of form name → lines array)

## Scripts Directory

`Scripts/` contains example `.txt` form scripts for Paladin (various specs/roles). These are reference scripts to copy into the in-game form editor, not loaded by WoW directly.

## Optional Dependencies

**Cursive** (`https://github.com/pepopo978/Cursive`) — requires SuperWoW. When loaded, `ifNotTargetHasDebuff` and `ifNotTargetHasDebuffTitle` use `Cursive.curses:HasCurse(lowercaseName, guid, 0)` to detect only player-cast debuffs on the target, ignoring debuffs from other players with the same spell. Implemented in `ParseBuffs.lua:CheckBuffOrDebuff` and `CheckBuffOrDebuffTitle`. Category-based checks (`ifTargetIs=CCd/Stunned` etc.) intentionally bypass Cursive and always use `UnitDebuff`.

## No Build System

This is pure Lua/XML for WoW 1.12. There is no compilation, linting, or test runner. Development workflow is:
1. Edit `.lua` files directly
2. Bump `## Version` in the relevant `.toc` file(s) — required so the addon manager detects the change
3. Copy the `Addons/` directory into the WoW client's `Interface/AddOns/` folder
4. Reload the UI in-game (`/reload`) to test changes
5. Use the in-game "Test" button in the LazyScript form editor to validate form syntax
