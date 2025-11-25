# L4D2 Predicaments Plugin

This SourceMod plugin enhances survivor gameplay by allowing players to help themselves and each other in predicaments. Based on Pan Xiaohai's original plugin, it includes bot support, incapacitated item interactions, a struggling system for pinned survivors, and developer API hooks for extensibility.

## Features

### Core Mechanics
- **Revive Yourself**: Survivors can revive themselves from incapacitation by holding CROUCH and consuming pills, adrenaline, or first-aid kits
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
- **Bot Support**: Bots can revive themselves with configurable delay and chance
- **Progress Bars**: Visual indicators for revival and struggle progress
- **Adrenaline Rush**: Proper adrenaline effects when reviving with adrenaline

### Developer API
- **Forward Hooks**: Allow external plugins to control healing and struggle permissions
- **Native Functions**: Query whether a client can heal others or struggle
- **Extensible Design**: Easy integration with custom game modes and mechanics

## Installation

### Pre-compiled Version

1. **Download** the latest compiled plugin from the [Releases](https://github.com/janiluuk/L4D2_Predicaments/releases) page
   - Or use the pre-compiled `l4d2_predicaments.smx` from this repository

2. **Install** the files:
   - Copy `l4d2_predicaments.smx` to `addons/sourcemod/plugins/`
   - Copy `gamedata/l4d2_predicaments.txt` to `addons/sourcemod/gamedata/`

3. **Restart** the server or change map

4. **Configure** (optional):
   - Edit `cfg/sourcemod/l4d2_predicaments.cfg` (auto-generated on first load)

### Building from Source

#### Requirements
- SourceMod 1.10 or higher
- SourceMod compiler (spcomp)

#### Manual Compilation

1. **Download SourceMod**:
   ```bash
   # For Linux
   wget https://sm.alliedmods.net/smdrop/1.12/sourcemod-1.12.0-git7219-linux.tar.gz
   tar -xzf sourcemod-1.12.0-git7219-linux.tar.gz
   
   # For Windows
   # Download from https://www.sourcemod.net/downloads.php
   ```

2. **Compile the plugin**:
   ```bash
   # Linux/Mac
   ./addons/sourcemod/scripting/spcomp l4d2_predicaments.sp -o l4d2_predicaments.smx
   
   # Windows
   addons\sourcemod\scripting\spcomp.exe l4d2_predicaments.sp -o l4d2_predicaments.smx
   ```

3. **Install** as described in the Pre-compiled Version section above

#### Automated Build (GitHub Actions)

This repository includes a GitHub Actions workflow that automatically compiles the plugin:

- **Triggers**: Push, Pull Request, Manual Dispatch, Quarterly Schedule
- **Platforms**: Ubuntu (Linux) and Windows
- **Artifacts**: Compiled plugins are available as workflow artifacts

To use the automated build:
1. Fork this repository
2. Push your changes or manually trigger the workflow
3. Download the compiled plugin from the workflow artifacts

## Configuration

### ConVar Table

| ConVar | Default | Description |
|--------|---------|-------------|
| `l4d2_predicaments_version` | `0.4` | Plugin version (read-only) |
| `l4d2_predicament_enable` | `1` | Master switch: 0=disabled, 1=enabled |
| `l4d2_predicament_use` | `3` | Items allowed: 0=none, 1=pills/adrenaline, 2=medkits, 3=both |
| `l4d2_predicament_incap_pickup` | `1` | Allow item pickup while incapped: 0=no, 1=yes |
| `l4d2_predicament_delay` | `1.0` | Delay (seconds) before plugin mechanics activate |
| `l4d2_predicament_kill_attacker` | `2` | Pin behavior: 0=unpin only, 1=unpin+kill, 2=disable unpin (use struggle) |
| `l4d2_predicament_struggle_mode` | `2` | Struggle system: 0=disabled, 1=hold crouch, 2=mash crouch |
| `l4d2_predicament_struggle_gain` | `5.0` | Percent gained per crouch press (0.0-100.0) |
| `l4d2_predicament_struggle_pushback` | `2.5` | Percent lost when attacker presses sprint (0.0-100.0) |
| `l4d2_predicament_struggle_effect` | `0` | On escape: 0=nothing, 1=kill attacker, 2=knockback both |
| `l4d2_predicament_bot` | `1` | Bot revival: 0=disabled, 1=enabled |
| `l4d2_predicament_bot_chance` | `4` | Bot chance: 1=sometimes, 2=often, 3=seldom, 4=always |
| `l4d2_predicament_hard_hp` | `50` | Permanent health given after revival (1-100) |
| `l4d2_predicament_temp_hp` | `50.0` | Temporary health given after revival (1.0-100.0) |

### Configuration Examples

**Competitive Settings** (harder):
```
l4d2_predicament_use "1"                  // Pills/adrenaline only
l4d2_predicament_struggle_mode "2"        // Must struggle to escape pins
l4d2_predicament_struggle_gain "3.0"      // Slower struggle progress
l4d2_predicament_bot "0"                  // Disable bot revival
l4d2_predicament_hard_hp "30"             // Less health on revival
l4d2_predicament_temp_hp "20.0"
```

**Casual Settings** (easier):
```
l4d2_predicament_use "3"                  // All items work
l4d2_predicament_struggle_mode "1"        // Just hold crouch
l4d2_predicament_kill_attacker "1"        // Kill attacker on escape
l4d2_predicament_bot "1"                  // Enable bot revival
l4d2_predicament_bot_chance "4"           // Bots always revive themselves
```

**Struggle-Only Mode** (no item usage while pinned):
```
l4d2_predicament_kill_attacker "2"        // Disable item-based unpin
l4d2_predicament_struggle_mode "2"        // Enable struggle system
l4d2_predicament_struggle_effect "1"      // Kill attacker on successful struggle
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

## Performance Considerations

This plugin has been optimized for performance with the following improvements:

### Optimization Strategies
- **Cached Property Lookups**: Entity properties are cached to avoid redundant GetEntProp calls
- **Early Exit Conditions**: Functions return early when conditions aren't met to minimize processing
- **Efficient Loop Logic**: Player iteration loops use optimized validation checks
- **Reduced Timer Overhead**: Timer callbacks include early validation to skip unnecessary work

### Performance Impact
The plugin uses several mechanisms that run frequently:
- `OnPlayerRunCmd`: Called every frame for player input handling (crawling, bot behavior)
- `RecordLastPosition`: Runs every 0.1 seconds to track survivor positions
- `AnalyzePlayerState`: Per-player timer (0.1s) for monitoring incapacitation state

### Recommendations for High-Performance Servers
- Disable crawling if not needed: `l4d2_predicament_crawl_enable "0"`
- Limit bot self-help if server has many bots: `l4d2_predicament_bot "0"`
- Increase timer intervals if experiencing performance issues (requires source modification)

## Compatibility

- **Left 4 Dead**: Fully supported
- **Left 4 Dead 2**: Fully supported
- **Game Modes**: Works in all game modes (Coop, Versus, Survival, etc.)
- **SourceMod**: Requires SourceMod 1.10 or higher

The plugin automatically detects the game version and adjusts features accordingly.

## Troubleshooting

**Q: Plugin fails to load with "Game Data Missing" error**  
A: Ensure `gamedata/l4d2_predicaments.txt` is in the correct directory

**Q: Revival doesn't work**  
A: Check that `l4d2_predicament_enable` is set to `1` and `l4d2_predicament_use` is not `0`

**Q: Bots never revive themselves**  
A: Verify `l4d2_predicament_bot "1"` and adjust `l4d2_predicament_bot_chance` (4 = always)

**Q: Struggle system doesn't work when pinned**  
A: Make sure `l4d2_predicament_struggle_mode` is set to `1` or `2`, and `l4d2_predicament_kill_attacker` is set to `2`

**Q: Players can use items to escape pins even with struggle mode enabled**  
A: Set `l4d2_predicament_kill_attacker "2"` to disable item-based escapes and force struggle system

**Q: Server experiencing performance issues with many players**  
A: Try disabling crawling and bot self-help features, or reduce the number of active features

## Credits

- **Pan Xiaohai** - Original plugin
- **cravenge** - Improvements  
- **Yani** - Enhancements and performance optimizations
- **Community** - Testing and feedback

## Links

- **Original Thread**: https://forums.alliedmods.net/showthread.php?t=281620
- **Repository**: https://github.com/janiluuk/L4D2_Predicaments
- **Issues**: Report bugs via GitHub Issues

## License

This plugin is based on the original work by Pan Xiaohai and subsequent contributors. Please refer to the original AlliedModders thread for licensing information.
