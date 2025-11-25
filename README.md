# L4D2 Predicaments Plugin

This SourceMod plugin enhances survivor gameplay by allowing players to help themselves and each other in predicaments. Based on Pan Xiaohai's original "Self Help" plugin, it includes bot support, incapacitated item interactions, a struggling system for pinned survivors, and developer API hooks for extensibility.

## Features

### Core Mechanics
- **Self-Revival**: Survivors can revive themselves from incapacitation by holding CROUCH and consuming pills, adrenaline, or first-aid kits
- **Ledge Rescue**: Pull yourself up from ledges using available medical items
- **Pin Escape**: Break free from Special Infected (Smoker, Hunter, Jockey, Charger) grabs by using items or struggling
- **Teammate Revival**: Incapacitated survivors can revive other incapacitated teammates by pressing RELOAD

### Struggle System
- **Mash to Escape**: When pinned, rapidly press CROUCH to build up struggle progress
- **Counter-Struggle**: Infected players can press SPRINT to push back survivor progress
- **Visual Feedback**: On-screen progress bar shows struggle status
- **Escape Effects**: Configurable outcome when survivors break free (stagger, kill attacker, or knockback)

### Quality of Life
- **Item Pickup While Down**: Grab nearby medical supplies while incapacitated
- **Smart Healing**: Plugin intelligently chooses between pills and medkits based on situation
- **Bot Support**: Bots can self-revive with configurable delay and chance
- **Progress Bars**: Visual indicators for revival and struggle progress
- **Adrenaline Rush**: Proper adrenaline effects when self-reviving with adrenaline

### Developer API
- **Forward Hooks**: Allow external plugins to control healing and struggle permissions
- **Native Functions**: Query whether a client can heal others or struggle
- **Extensible Design**: Easy integration with custom game modes and mechanics

## Installation

1. **Compile** the plugin using the SourceMod compiler:
   ```bash
   spcomp l4d2_predicaments.sp
   ```

2. **Install** the compiled plugin:
   - Copy `l4d2_predicaments.smx` to `addons/sourcemod/plugins/`
   - Copy `gamedata/l4d2_predicaments.txt` to `addons/sourcemod/gamedata/`

3. **Restart** the server or change map

4. **Configure** (optional):
   - Edit `cfg/sourcemod/l4d2_predicaments.cfg` (auto-generated on first load)

## Configuration

### ConVar Table

| ConVar | Default | Description |
|--------|---------|-------------|
| `l4d2_predicaments_version` | `0.4` | Plugin version (read-only) |
| `self_help_enable` | `1` | Master switch: 0=disabled, 1=enabled |
| `self_help_use` | `3` | Items allowed: 0=none, 1=pills/adrenaline, 2=medkits, 3=both |
| `self_help_incap_pickup` | `1` | Allow item pickup while incapped: 0=no, 1=yes |
| `self_help_delay` | `1.0` | Delay (seconds) before plugin mechanics activate |
| `self_help_kill_attacker` | `2` | Pin behavior: 0=unpin only, 1=unpin+kill, 2=disable unpin (use struggle) |
| `self_help_struggle_mode` | `2` | Struggle system: 0=disabled, 1=hold crouch, 2=mash crouch |
| `self_help_struggle_gain` | `5.0` | Percent gained per crouch press (0.0-100.0) |
| `self_help_struggle_pushback` | `2.5` | Percent lost when attacker presses sprint (0.0-100.0) |
| `self_help_struggle_effect` | `0` | On escape: 0=nothing, 1=kill attacker, 2=knockback both |
| `self_help_bot` | `1` | Bot self-help: 0=disabled, 1=enabled |
| `self_help_bot_chance` | `4` | Bot chance: 1=sometimes, 2=often, 3=seldom, 4=always |
| `self_help_hard_hp` | `50` | Permanent health given after self-help (1-100) |
| `self_help_temp_hp` | `50.0` | Temporary health given after self-help (1.0-100.0) |

### Configuration Examples

**Competitive Settings** (harder self-help):
```
self_help_use "1"                  // Pills/adrenaline only
self_help_struggle_mode "2"        // Must struggle to escape pins
self_help_struggle_gain "3.0"      // Slower struggle progress
self_help_bot "0"                  // Disable bot self-help
self_help_hard_hp "30"             // Less health on revival
self_help_temp_hp "20.0"
```

**Casual Settings** (easier self-help):
```
self_help_use "3"                  // All items work
self_help_struggle_mode "1"        // Just hold crouch
self_help_kill_attacker "1"        // Kill attacker on self-help
self_help_bot "1"                  // Enable bot self-help
self_help_bot_chance "4"           // Bots always help themselves
```

**Struggle-Only Mode** (no item usage while pinned):
```
self_help_kill_attacker "2"        // Disable item-based unpin
self_help_struggle_mode "2"        // Enable struggle system
self_help_struggle_effect "1"      // Kill attacker on successful struggle
```

## Developer API

The plugin provides forward hooks and natives for external plugins to control game mechanics.

### Forwards

#### `Predicaments_OnCanHealOthers`
Called when a survivor attempts to heal another incapacitated survivor.

```sourcepawn
forward Action Predicaments_OnCanHealOthers(int client, int target, bool &canHeal);
```

**Parameters:**
- `client` - The player attempting to heal
- `target` - The incapacitated player being healed  
- `canHeal` - Reference to bool; set to `false` to prevent healing

**Returns:** `Plugin_Handled` or `Plugin_Stop` to apply your decision, `Plugin_Continue` to allow default behavior

#### `Predicaments_OnCanStruggle`
Called when a pinned survivor attempts to struggle.

```sourcepawn
forward Action Predicaments_OnCanStruggle(int client, bool &canStruggle);
```

**Parameters:**
- `client` - The pinned player
- `canStruggle` - Reference to bool; set to `false` to prevent struggling

**Returns:** `Plugin_Handled` or `Plugin_Stop` to apply your decision, `Plugin_Continue` to allow default behavior

### Natives

#### `Predicaments_CanHealOthers`
Check if a client can heal another client.

```sourcepawn
native bool Predicaments_CanHealOthers(int client, int target);
```

**Returns:** `true` if allowed, `false` otherwise

#### `Predicaments_CanStruggle`
Check if a client can struggle when pinned by an infected.

```sourcepawn
native bool Predicaments_CanStruggle(int client);
```

**Returns:** `true` if allowed, `false` otherwise

### Example Plugin

```sourcepawn
#include <sourcemod>
#include <l4d2_predicaments>

public Plugin myinfo = 
{
    name = "Predicaments Example",
    author = "YourName",
    description = "Example usage of Predicaments API",
    version = "1.0",
    url = ""
};

// Prevent healing if players are too far apart
public Action Predicaments_OnCanHealOthers(int client, int target, bool &canHeal)
{
    float clientPos[3], targetPos[3];
    GetClientAbsOrigin(client, clientPos);
    GetClientAbsOrigin(target, targetPos);
    
    float distance = GetVectorDistance(clientPos, targetPos);
    
    if (distance > 100.0)
    {
        canHeal = false;
        PrintToChat(client, "[Predicaments] Too far to help!");
        return Plugin_Handled;
    }
    
    return Plugin_Continue;
}

// Prevent struggling if client has very low health
public Action Predicaments_OnCanStruggle(int client, bool &canStruggle)
{
    int health = GetClientHealth(client);
    
    if (health < 10)
    {
        canStruggle = false;
        PrintToChat(client, "[Predicaments] Too weak to struggle!");
        return Plugin_Handled;
    }
    
    return Plugin_Continue;
}

// Check if a specific player can heal others (using native)
public void OnClientPutInServer(int client)
{
    if (!IsFakeClient(client))
    {
        CreateTimer(5.0, WelcomeMessage, GetClientUserId(client));
    }
}

public Action WelcomeMessage(Handle timer, int userid)
{
    int client = GetClientOfUserId(userid);
    
    if (client && IsClientInGame(client))
    {
        // Check if they can help others
        bool canHelp = Predicaments_CanHealOthers(client, client);
        PrintToChat(client, "[Predicaments] Can help others: %s", canHelp ? "Yes" : "No");
    }
    
    return Plugin_Stop;
}
```

### Include File

To use the API in your plugin, include the header file:

```sourcepawn
#include <l4d2_predicaments>
```

The include file (`l4d2_predicaments.inc`) should be placed in `addons/sourcemod/scripting/include/`

## Compatibility

- **Left 4 Dead**: Fully supported
- **Left 4 Dead 2**: Fully supported
- **Game Modes**: Works in all game modes (Coop, Versus, Survival, etc.)
- **SourceMod**: Requires SourceMod 1.10 or higher

The plugin automatically detects the game version and adjusts features accordingly.

## Troubleshooting

**Q: Plugin fails to load with "Game Data Missing" error**  
A: Ensure `gamedata/l4d2_predicaments.txt` is in the correct directory

**Q: Self-help doesn't work**  
A: Check that `self_help_enable` is set to `1` and `self_help_use` is not `0`

**Q: Bots never self-help**  
A: Verify `self_help_bot "1"` and adjust `self_help_bot_chance` (4 = always)

**Q: Struggle system doesn't work when pinned**  
A: Make sure `self_help_struggle_mode` is set to `1` or `2`, and `self_help_kill_attacker` is set to `2`

**Q: Players can use items to escape pins even with struggle mode enabled**  
A: Set `self_help_kill_attacker "2"` to disable item-based escapes and force struggle system

## Credits

- **Pan Xiaohai** - Original Self Help plugin
- **cravenge** - Self-Help Reloaded improvements  
- **Yani** - DLR edition enhancements
- **Community** - Testing and feedback

## Links

- **Original Thread**: https://forums.alliedmods.net/showthread.php?t=281620
- **Repository**: https://github.com/janiluuk/L4D2_Self_Help
- **Issues**: Report bugs via GitHub Issues

## License

This plugin is based on the original work by Pan Xiaohai and subsequent contributors. Please refer to the original AlliedModders thread for licensing information.
