# Project Memory

## User Preferences

- **Never assume/placeholder when uncertain** — always ask the user instead of guessing and using a "reasonable placeholder". If a value (e.g. texture name, spell ID) is unknown, stop and ask.

## Adding a New Spell/Action to a Class Addon

Every new `lazyXxx.actions.foo = lazyXxx.Action:New(...)` requires a matching locale entry, otherwise `obj.name` will be `nil` and the addon will crash with "attempt to concatenate field 'name' (a nil value)" when the action is used.

**Always add both:**
1. `Addons/Lazy<Class>/Parse<Class>.lua` — the `Action:New` entry
2. `Addons/Lazy<Class>/Localization.lua` — `lazyXxxLocale.enUS.ACTION_TTS.<code> = "<In-game spell name>"`

The in-game spell name must match exactly what appears in the spellbook tooltip (used for spell lookup).
