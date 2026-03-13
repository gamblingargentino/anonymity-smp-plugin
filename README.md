# Anonymous Havoc - PaperMC Plugin Configuration

## Overview
Anonymous Havoc is a sophisticated social-deduction SMP plugin for PaperMC 1.21+ that implements identity masking and a staggered release schedule for limited items.

## Building the Plugin

### Prerequisites
- Java 21 JDK
- Maven 3.8+
- PaperMC server running version 1.21+

### Build Steps

1. Navigate to the plugin directory:
```bash
cd plugins/Anonimity\ plugin
```

2. Build with Maven:
```bash
mvn clean package
```

3. The compiled JAR will be in `target/AnonymousHavoc-1.0-SNAPSHOT.jar`

4. Copy the JAR to your server's `plugins/` directory:
```bash
cp target/AnonymousHavoc-1.0-SNAPSHOT.jar /path/to/server/plugins/
```

5. **Ensure ProtocolLib is installed** - Download from [here](https://www.spigotmc.org/resources/protocollib.1997/)

6. Restart your server

## Features

### Phase 1: Identity & Death (Day 1+)
- **Identity Masking**: All players appear as Steve with the name "Player" in tab list and chat
- **Death Ban**: 24-hour ban on death
- **Soul-Binder Execution**: 48-hour ban + name broadcast on death by Soul-Binder Dagger

### Phase 2: Staggered Release (Days 1-16)
- **Day 1**: Hunter's Compass (Limit: 10) - Tracks random player for 5 mins
- **Day 4**: Sight Glass (Limit: 8) - Reveals HP and Enchants of zoomed target
- **Day 7**: Bell of Truth (Limit: 4) - 8-block reveal pulse reveals all players
- **Day 10**: Totem of Scrambling (Limit: 5) - Offhand; redirects compass tracking to sheep
- **Day 13**: Shard of Anonymity (Limit: 2) - Offhand; 15% lifesteal + Bell immunity
- **Day 16**: Soul-Binder Dagger (Limit: 1) - Kills broadcast true identity

### Phase 3: Anti-Hoarding & Admin
- **48h Offline Drop**: Limited items drop if owner is offline/banned >48h
- **Coordinates Leak**: Every 4 hours, broadcast 150-block radius hint of Shard/Dagger
- **Admin Command**: `/havoc give <item>` (OP only) - Bypass all restrictions for testing

## Configuration

Edit `plugins/AnonymousHavoc/config.yml`:

```yaml
death-ban:
  normal-duration-hours: 24
  soul-binder-duration-hours: 48

coordinates-leak:
  interval-hours: 4
  reveal-radius: 150

server-start-time: <auto-set>
```

## Commands

### `/havoc give <item>`
**Permission**: `havoc.admin`

Available items:
- `hunters_compass`
- `sight_glass`
- `bell_of_truth`
- `totem_of_scrambling`
- `shard_of_anonymity`
- `soul_binder_dagger`

Example:
```
/havoc give hunters_compass
```

## Resource Pack Setup (Optional)

To use custom item textures, create a resource pack with CustomModelData:
- 1001: Hunter's Compass
- 1002: Sight Glass
- 1003: Bell of Truth
- 1004: Totem of Scrambling
- 1005: Shard of Anonymity
- 1006: Soul-Binder Dagger

Place your resource pack in `server.properties` as:
```
resource-pack=<your_pack_url>
resource-pack-sha1=<sha1_hash>
```

## Database

The plugin uses SQLite for persistence:
- Location: `plugins/AnonymousHavoc/anonymous_havoc.db`
- Contains: Item ownership, death bans, last player locations
- Automatic initialization on first load

## Troubleshooting

### Items not crafting
- Ensure current server day has reached item unlock day
- Verify global limit not reached: `/havoc give <item>` for testing

### Death ban not working
- Check database connectivity: `plugins/AnonymousHavoc/anonymous_havoc.db` exists
- Restart server if needed

### Identity masking not visible
- Ensure ProtocolLib is installed and enabled
- Check server logs for ProtocolLib initialization

## File Structure

```
AnonymousHavoc/
├── src/main/java/com/anonymoushavoc/
│   ├── AnonymousHavoc.java          # Main plugin class
│   ├── commands/
│   │   └── HavocCommand.java         # /havoc command handler
│   ├── listeners/
│   │   ├── DeathListener.java
│   │   ├── CraftingListener.java
│   │   ├── ItemCraftBroadcastListener.java
│   │   ├── LogoutListener.java
│   │   └── JoinListener.java
│   ├── managers/
│   │   ├── DatabaseManager.java
│   │   ├── ConfigManager.java
│   │   └── ItemManager.java
│   └── packets/
│       └── PacketHandler.java        # ProtocolLib packet interception
├── src/main/resources/
│   ├── plugin.yml
│   └── havoc_items.json
└── pom.xml
```

## Support

For issues, check server logs in `logs/latest.log` for errors related to AnonymousHavoc.

## License

This plugin is custom-made for your server.
