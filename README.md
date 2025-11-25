# L4D2 Self-Help Plugins

This repository provides two SourceMod plugins that let survivors rescue themselves in tough situations. Both are based on Pan Xiaohai's original "Self Help" plugin, with extensions for bot support and item interactions while incapacitated.

- **`l4d_selfhelp_bot.sp`** – legacy plugin with optional bot self-revival and adrenaline rush support.
- **`self_help_dlr.sp`** – newer "Reloaded" version (DLR edition) that adds incap item pickup, self-healing while downed, and smarter bot behaviour.

## Which Plugin Should You Use?

**TL;DR: Use `self_help_dlr.sp` (DLR edition) - it's better and has more features.**

The **`self_help_dlr.sp`** (DLR Reloaded edition) is the **recommended and superior choice**. It includes all the functionality of the legacy plugin plus many additional features and improvements. The legacy `l4d_selfhelp_bot.sp` is only kept for backwards compatibility.

### Feature Comparison

| Feature | self_help_dlr.sp (DLR) | l4d_selfhelp_bot.sp (Legacy) |
|---------|:----------------------:|:----------------------------:|
| Self-revive with pills/adrenaline | ✅ | ✅ |
| Self-revive with medkit | ✅ | ✅ |
| Bot self-revival | ✅ | ✅ |
| Adrenaline rush support | ✅ | ✅ |
| Pick up items while incapacitated | ✅ | ❌ |
| Struggle mechanic when pinned | ✅ | ❌ |
| Attacker counter-struggle | ✅ | ❌ |
| Configurable struggle effects | ✅ | ❌ |
| Help other incapped players (R key) | ✅ | ✅ |
| Smart item usage (prioritize pills over medkit) | ✅ | ❌ |
| Better progress bar visuals | ✅ | ⚠️ Basic |
| Configurable attacker punishment | ✅ (3 options) | ⚠️ (2 options) |
| Colored chat messages | ✅ | ❌ |
| More granular ConVar options | ✅ | ⚠️ Limited |
| L4D1 and L4D2 support | ✅ | ✅ |

**Key advantages of DLR edition:**
- **Struggle mechanic**: Survivors can mash crouch to break free from Special Infected (SI), while attackers can fight back
- **Item pickup while downed**: Can grab nearby health items even when incapacitated
- **Smarter healing**: Automatically chooses the best item for the situation (pills vs medkit)
- **Better feedback**: Colored chat messages and improved progress indicators
- **More control**: Extensive ConVar options for fine-tuning gameplay

## Features at a Glance

- Self-unpin from grabs, pounces, rides, and ledge hangs by consuming pills or first-aid kits.
- Optional attacker punishment when breaking free.
- Bots can self-revive after a configurable delay.
- Adrenaline rush support with configurable duration (bug-fixed in this repo version).
- On-screen prompts show what aid you carry and remind you to hold crouch to self-revive.
- Teammate revives display progress to both players so the downed survivor sees who is reviving them.
- When pinned by a special infected (DLR), survivors get a visible struggle meter that builds by mashing crouch while the attacker can fight back.
- Item pickup and healing actions while incapacitated (DLR edition).

## Installation

**Recommended: Install the DLR edition for the best experience.**

1. Compile the desired plugin with the SourceMod compiler:
   - **Recommended**: `self_help_dlr.sp` (DLR edition)
   - Legacy alternative: `l4d_selfhelp_bot.sp` (if you need backwards compatibility)
2. Copy the resulting `.smx` file into your server's `addons/sourcemod/plugins/` directory.
3. Place the provided game data file from `gamedata/` into `addons/sourcemod/gamedata/`:
   - For DLR edition: `self_help.txt`
   - For legacy edition: `l4d_selfhelp_bot.txt`
4. Restart the server or change the map.

**Note**: Only install ONE of the two plugins, not both! They serve the same purpose and will conflict.

## Key ConVars

### Legacy bot edition (`l4d_selfhelp_bot`)

- `l4d_selfhelp_incap`, `l4d_selfhelp_grab`, `l4d_selfhelp_pounce`, `l4d_selfhelp_ride`, `l4d_selfhelp_pummel`, `l4d_selfhelp_edgegrab` – enable (1/2) pills or medkits, or both (3) for each situation.
- `l4d_selfhelp_bot_delay` – seconds before bots attempt to self-recover.
- `l4d_selfhelp_duration` – override self-help progress duration (seconds).
- `l4d_selfhelp_adrenaline_rush` – enable adrenaline rush when reviving with adrenaline.
- `l4d_selfhelp_adrenaline_duration` – length of the adrenaline rush (fixed name; previously mislabelled).

### Reloaded DLR edition (`self_help_dlr`)

- `self_help_enable` – master enable/disable switch.
- `self_help_use` – choose allowed items: 1=pills/adrenaline, 2=first-aid kits, 3=both.
- `self_help_incap_pickup` – allow incapacitated players to pick up healable items.
- `self_help_kill_attacker` – 0=unpin only, 1=unpin and kill attacker, 2=disable unpin.
- `self_help_struggle_mode` – pinned behaviour: 0=disabled, 1=hold crouch to self-help, 2=mash crouch to struggle.
- `self_help_struggle_gain` / `self_help_struggle_pushback` – percent gained per survivor crouch press vs lost when the attacker taps sprint.
- `self_help_struggle_effect` – on escape: 0=none, 1=kill attacker, 2=knock both back.
- `self_help_bot` / `self_help_bot_chance` – toggle and configure bot self-help behaviour.
- `self_help_hard_hp` / `self_help_temp_hp` – health granted after successful self-help.

After changing any ConVar, use `sm_cvar <name> <value>` or place them in `cfg/sourcemod/` auto-exec files to persist settings.

## Notes

- The DLR edition automatically detects Left 4 Dead vs Left 4 Dead 2 and adjusts available features.
- If you only need bot revival support without the newer mechanics, stick with `l4d_selfhelp_bot.smx`.
- For detailed discussion and community feedback, see the original AlliedModders thread: https://forums.alliedmods.net/showthread.php?t=281620

## Conclusion

**The `self_help_dlr.sp` (DLR Reloaded edition) is the superior plugin in every way.** It has more features, better configurability, and improved user experience. Unless you have a specific reason to use the legacy version (such as compatibility with existing server configurations), you should use the DLR edition.

The legacy `l4d_selfhelp_bot.sp` remains in this repository only for:
- Server administrators who need to maintain backwards compatibility
- Historical reference and comparison purposes
- Users who prefer the simpler, more minimal approach

For new installations, **always choose `self_help_dlr.sp`**.
