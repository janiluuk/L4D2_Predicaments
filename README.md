# L4D2 Predicaments Plugin

This repository provides a SourceMod plugin that lets survivors rescue themselves in tough situations. The plugin is based on Pan Xiaohai's original "Self Help" plugin, with extensions for bot support, item interactions while incapacitated, and a struggling system for pinned survivors.

## Features at a Glance

- Self-unpin from grabs, pounces, rides, and ledge hangs by consuming pills or first-aid kits.
- Optional attacker punishment when breaking free.
- Bots can self-revive after a configurable delay.
- Adrenaline rush support with configurable duration.
- On-screen prompts show what aid you carry and remind you to hold crouch to self-revive.
- Teammate revives display progress to both players so the downed survivor sees who is reviving them.
- When pinned by a special infected, survivors get a visible struggle meter that builds by mashing crouch while the attacker can fight back.
- Item pickup and healing actions while incapacitated.
- **Forward hooks** for external plugins to control healing and struggle permissions.

## Installation

1. Compile the plugin (`l4d2_predicaments.sp`) with the SourceMod compiler.
2. Copy the resulting `.smx` file into your server's `addons/sourcemod/plugins/` directory.
3. Place the provided game data file from `gamedata/l4d2_predicaments.txt` into `addons/sourcemod/gamedata/`.
4. Restart the server or change the map.

## Key ConVars

- `l4d2_predicaments_version` – version display (read-only).
- `self_help_enable` – master enable/disable switch.
- `self_help_use` – choose allowed items: 1=pills/adrenaline, 2=first-aid kits, 3=both.
- `self_help_incap_pickup` – allow incapacitated players to pick up healable items.
- `self_help_kill_attacker` – 0=unpin only, 1=unpin and kill attacker, 2=disable unpin.
- `self_help_struggle_mode` – pinned behaviour: 0=disabled, 1=hold crouch to self-help, 2=mash crouch to struggle.
- `self_help_struggle_gain` / `self_help_struggle_pushback` – percent gained per survivor crouch press vs lost when the attacker taps sprint.
- `self_help_struggle_effect` – on escape: 0=none, 1=kill attacker, 2=knock both back.
- `self_help_bot` / `self_help_bot_chance` – toggle and configure bot self-help behaviour.
- `self_help_hard_hp` / `self_help_temp_hp` – health granted after successful self-help.

After changing any ConVar, use `sm_cvar <name> <value>` or place them in `cfg/sourcemod/l4d2_predicaments.cfg` (auto-generated) to persist settings.

## Developer API

The plugin provides forward hooks that allow other plugins to control healing and struggle permissions:

### Forwards

**`Predicaments_OnCanHealOthers(int client, int target, bool &canHeal)`**
- Called when a survivor attempts to heal another incapacitated survivor
- `client` - The player attempting to heal
- `target` - The incapacitated player being healed
- `canHeal` - Set to `false` to prevent the heal action
- Return `Plugin_Handled` or `Plugin_Stop` to apply your decision

**`Predicaments_OnCanStruggle(int client, bool &canStruggle)`**
- Called when a pinned survivor attempts to struggle
- `client` - The pinned player
- `canStruggle` - Set to `false` to prevent struggling
- Return `Plugin_Handled` or `Plugin_Stop` to apply your decision

### Natives

**`bool Predicaments_CanHealOthers(int client, int target)`**
- Check if a client can heal another client
- Returns `true` if allowed, `false` otherwise

**`bool Predicaments_CanStruggle(int client)`**
- Check if a client can struggle when pinned
- Returns `true` if allowed, `false` otherwise

### Example Usage

```sourcepawn
#include <sourcemod>
#include <l4d2_predicaments>

public void OnPluginStart()
{
    // Your plugin initialization
}

public Action Predicaments_OnCanHealOthers(int client, int target, bool &canHeal)
{
    // Example: Prevent healing if client is too far from target
    float clientPos[3], targetPos[3];
    GetClientAbsOrigin(client, clientPos);
    GetClientAbsOrigin(target, targetPos);
    
    if (GetVectorDistance(clientPos, targetPos) > 100.0)
    {
        canHeal = false;
        return Plugin_Handled;
    }
    
    return Plugin_Continue;
}

public Action Predicaments_OnCanStruggle(int client, bool &canStruggle)
{
    // Example: Prevent struggling if client has low health
    if (GetClientHealth(client) < 10)
    {
        canStruggle = false;
        PrintToChat(client, "You're too weak to struggle!");
        return Plugin_Handled;
    }
    
    return Plugin_Continue;
}
```

## Notes

- The plugin automatically detects Left 4 Dead vs Left 4 Dead 2 and adjusts available features.
- For detailed discussion and community feedback, see the original AlliedModders thread: https://forums.alliedmods.net/showthread.php?t=281620
