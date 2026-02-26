# Project Memory

## User Preferences

- **Never assume/placeholder when uncertain** — always ask the user instead of guessing and using a "reasonable placeholder". If a value (e.g. texture name, spell ID) is unknown, stop and ask.

## Version Bumping

Every change to any addon requires bumping `## Version` in the corresponding `.toc` file so the addon manager detects the update. Bump the `.toc` of every addon whose files were modified (e.g. change in `Addons/LazyScript/` → bump `LazyScript.toc`, change in `Addons/LazyDruid/` → bump `LazyDruid.toc`).

This includes **reverting code** — reverting a commit is still a change and must also bump the version.

## Adding a New Spell/Action to a Class Addon

Every new `lazyXxx.actions.foo = lazyXxx.Action:New(...)` requires a matching locale entry, otherwise `obj.name` will be `nil` and the addon will crash with "attempt to concatenate field 'name' (a nil value)" when the action is used.

**Always add both:**
1. `Addons/Lazy<Class>/Parse<Class>.lua` — the `Action:New` entry
2. `Addons/Lazy<Class>/Localization.lua` — `lazyXxxLocale.enUS.ACTION_TTS.<code> = "<In-game spell name>"`

The in-game spell name must match exactly what appears in the spellbook tooltip (used for spell lookup).
