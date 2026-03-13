# IMPLEMENTATION GUIDE - Anonymous Havoc Plugin

## Quick Start

### Step 1: Build the Plugin
```bash
cd "plugins/Anonimity plugin"
mvn clean package
```

### Step 2: Install Dependencies
1. Download ProtocolLib from SpigotMC: https://www.spigotmc.org/resources/protocollib.1997/
2. Place the JAR in your `plugins/` directory
3. Copy the built AnonymousHavoc JAR to `plugins/`

### Step 3: Start Server
```bash
./start.sh  # or start.bat on Windows
```

### Step 4: Test Commands
```
/havoc give hunters_compass
/havoc give soul_binder_dagger
```

## Feature Implementation Status

### ✅ IMPLEMENTED

#### Core Systems
- [x] Plugin initialization with all managers
- [x] SQLite database for item ownership, death bans, locations
- [x] Configuration manager with staggered release schedule
- [x] Custom items with PDC tagging

#### Item System
- [x] 6 custom items with global limits (10, 8, 4, 5, 2, 1)
- [x] Shaped crafting recipes for each item
- [x] Global limit checking on craft
- [x] Crafting broadcast: "[Player] has crafted [Item]! ([Current]/[Max] in world)"
- [x] Item ownership tracking in database

#### Death & Ban System
- [x] 24-hour normal death ban
- [x] 48-hour Soul-Binder Dagger death ban
- [x] Ban check on player login (async pre-login)
- [x] Kick message with remaining ban time
- [x] Soul-Binder execution broadcast: "[RealName] was executed and erased"
- [x] Last location tracking for item drop-off

#### Staggered Release
- [x] Day-based release schedule (Days 1, 4, 7, 10, 13, 16)
- [x] Recipe activation based on server day
- [x] Release announcements "[RELEASE] [Item] is now available!"

#### Anti-Hoarding
- [x] 48-hour offline/ban threshold item drop-off
- [x] Drop location: last logout/death location
- [x] Coordinates leak: Every 4 hours, 150-block hint of Shard/Dagger
- [x] Automatic task checks every 30 minutes

#### Identity Masking
- [x] ProtocolLib packet handler setup
- [x] Packet listener registration
- [x] Foundation for Steve skin + "Player" name masking

#### Admin Tools
- [x] `/havoc give <item>` command (OP only)
- [x] Bypasses release date restrictions
- [x] Bypasses global limits
- [x] List of available items in help message

## File Descriptions

### Core Classes

**AnonymousHavoc.java** (Main Plugin)
- Initializes all managers and listeners
- Registers event listeners and packet handlers
- Manages scheduled tasks:
  - `startCoordinatesLeakTask()` - 4-hour leak broadcasts
  - `startItemReleaseTask()` - 1-hour release checks
  - `startOfflineItemDropTask()` - 30-minute offline checks

**DatabaseManager.java**
- SQLite connection and table initialization
- Item owner tracking: `addItemOwner()`, `getItemOwner()`
- Death ban management: `setBanForPlayer()`, `isPlayerBanned()`, `getRemainingBanTime()`
- Location tracking: `saveLastLocation()`, `getLastLocation()`

**ConfigManager.java**
- Loads/saves `config.yml`
- Stores release schedule with `ItemInfo` class:
  - Day unlock
  - Global limit
  - Command ID (1001-1006)
  - Display name
- Methods to get current server day, ban durations, leak interval

**ItemManager.java**
- Creates 6 custom items
- Sets PDC tags for item identification
- Registers shaped crafting recipes
- Methods: `getItemId()`, `setCustomItem()`, `registerRecipes()`

**PacketHandler.java**
- Uses ProtocolLib to intercept player info and add player packets
- Masks player UUIDs in identity system
- Ready for Steve skin and "Player" name masking via packet modification

### Event Listeners

**DeathListener.java**
- Handles `PlayerDeathEvent` for ban application
- Checks for Soul-Binder Dagger kills
- Broadcasts execution messages
- Checks ban status on `AsyncPlayerPreLoginEvent`

**CraftingListener.java**
- Handles `PrepareItemCraftEvent`
- Checks global limits before allowing craft
- Tracks item counts in memory

**ItemCraftBroadcastListener.java**
- Handles `CraftItemEvent`
- Broadcasts craft message: "[Player] has crafted [Item]! ([X]/[Y] in world)"

**JoinListener.java**
- Registers recipes on player join
- Welcome message

**LogoutListener.java**
- Saves last location on logout for drop-off system

### Commands

**HavocCommand.java**
- Handles `/havoc give <item>`
- Requires `havoc.admin` permission
- Gives testing items to players

### Packets

**PacketHandler.java**
- Registers ProtocolLib listeners for:
  - `ClientboundPlayerInfoUpdatePacket` - Tab list masking
  - `ClientboundAddPlayerPacket` - Name masking
- Foundation for Steve skin/name forcing

## How Each Feature Works

### 1. Identity Masking
**Current Session**: Packet handlers are registered but name/skin masking placeholder.
**Enhancement Needed**: Modify packets to force Steve skin and "Player" name using ProtocolLib's packet modification API.

### 2. Staggered Release
- `ConfigManager` stores schedule with unlock day for each item
- `AnonymousHavoc.startItemReleaseTask()` runs hourly check
- `ConfigManager.getCurrentDay()` calculates days since server start
- `ItemManager.registerRecipes()` enables recipes for items with reached unlock day

### 3. Global Limits
- `CraftingListener` tracks item count in `globalItemCount` map
- On `PrepareItemCraftEvent`, checks if count >= limit
- If limit reached, sets result to null (disables crafting)
- `/havoc give` bypasses limits entirely

### 4. Death Ban System
- `DeathListener.onPlayerDeath()` applies ban with duration
- Duration = 24h normal, 48h for Soul-Binder Dagger
- On join: `DeathListener.onPlayerPreLogin()` checks ban status
- If banned, kicks with remaining time message

### 5. Crafting Broadcast
- `ItemCraftBroadcastListener.onCraftItem()` triggers on craft
- Increments global count
- Broadcasts: "[Player] has crafted [Item]! ([Current]/[Max] in world)"

### 6. Coordinates Leak
- Runs every 4 hours (configurable)
- Finds item owners by UUID
- Broadcasts: "[LEAK] [Item] spotted within [Radius] blocks of X: [X] Z: [Z]"

### 7. Offline Item Drop
- Runs every 30 minutes
- Checks item owners' online status
- If offline AND banned >48h, drops item at last location
- Broadcasts drop notification

### 8. Admin Command
- `/havoc give <item>` finds player in inventory
- Clones custom item with PDC tag
- Bypasses all restrictions

## Testing Checklist

- [ ] Plugin compiles without errors
- [ ] Server starts with plugin enabled
- [ ] `/havoc give hunters_compass` works
- [ ] Item appears in inventory with gold name
- [ ] Crafting recipe shows in recipe book (Day 1+)
- [ ] Crafting item broadcasts message with count
- [ ] Player death applies 24h ban
- [ ] Knockout message for death-banned player
- [ ] `/give @s compass 1` works for normal compass
- [ ] Soul-Binder Dagger kill broadcasts true identity

## Next Steps for Enhancement

1. **Identity Masking Enhancement**
   - Modify `ClientboundPlayerInfoUpdatePacket` to set all names to "Player"
   - Modify skin data to force Steve texture
   - Create custom chat component handler for chat masking

2. **Item Abilities**
   - Hunter's Compass: Track random player task
   - Sight Glass: Right-click to reveal HP/enchants
   - Bell of Truth: Right-click to reveal all players
   - Totem of Scrambling: Offhand compass redirection
   - Shard of Anonymity: 15% lifesteal on damage, Bell immunity
   - Soul-Binder Dagger: Auto-tracks kills and broadcasts

3. **Polish**
   - Add custom model data references (1001-1006)
   - Create resource pack template
   - Add particle effects for items
   - Add custom sounds for item effects

## Key Variables & Constants

**Release Schedule** (ConfigManager.java)
```
Day 1: hunters_compass (limit 10)
Day 4: sight_glass (limit 8)
Day 7: bell_of_truth (limit 4)
Day 10: totem_of_scrambling (limit 5)
Day 13: shard_of_anonymity (limit 2)
Day 16: soul_binder_dagger (limit 1)
```

**Command IDs** (CustomModelData)
```
1001: Hunter's Compass
1002: Sight Glass
1003: Bell of Truth
1004: Totem of Scrambling
1005: Shard of Anonymity
1006: Soul-Binder Dagger
```

**Scheduled Tasks** (times in ticks, 20 ticks = 1 second)
```
Coordinates Leak: Every 10,800 seconds (configurable)
Item Release Check: Every 3,600 seconds (1 hour)
Offline Item Drop: Every 1,800 seconds (30 minutes)
```

## Permissions

```yaml
havoc.admin
  - Allows: /havoc give
  - Default: OP only
```

---

*Last Updated: Build Phase 1 Complete*
