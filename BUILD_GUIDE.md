# Anonymous Havoc - Build & Deployment Guide

## Prerequisites

- **Java**: JDK 21 or higher
- **Maven**: 3.8.1 or higher
- **PaperMC**: 1.21+ server with ProtocolLib plugin
- **Git**: (Optional) For version control

## Quick Build & Deploy

### 1. Build the Plugin JAR

```bash
# Navigate to plugin directory
cd plugins/Anonimity\ plugin

# Build with Maven
mvn clean package

# Output: target/AnonymousHavoc-1.0-SNAPSHOT.jar
```

### 2. Install ProtocolLib Dependency

ProtocolLib is **required** for the plugin to work.

1. Download from: https://www.spigotmc.org/resources/protocollib.1997/
2. Place JAR in `<your-server>/plugins/`
3. Restart server to initialize

### 3. Deploy AnonymousHavoc

```bash
# Copy compiled JAR to server plugins directory
cp target/AnonymousHavoc-1.0-SNAPSHOT.jar /path/to/server/plugins/

# Restart server
```

### 4. Verify Installation

On server startup, you should see:
```
[AnonymousHavoc] AnonymousHavoc plugin has been enabled!
[Server] Anonymous Havoc has awakened...
```

## Project Structure

```
plugins/Anonimity plugin/
├── pom.xml                              # Maven build config
├── README.md                            # User documentation
├── IMPLEMENTATION_GUIDE.md              # Technical guide
├── BUILD_GUIDE.md                       # This file
├── .gitignore                           # Git ignore rules
│
├── src/
│   └── main/
│       ├── java/com/anonymoushavoc/
│       │   ├── AnonymousHavoc.java      # Main entry point
│       │   │
│       │   ├── commands/
│       │   │   └── HavocCommand.java    # /havoc give command
│       │   │
│       │   ├── listeners/
│       │   │   ├── DeathListener.java
│       │   │   ├── CraftingListener.java
│       │   │   ├── ItemCraftBroadcastListener.java
│       │   │   ├── LogoutListener.java
│       │   │   └── JoinListener.java
│       │   │
│       │   ├── managers/
│       │   │   ├── DatabaseManager.java
│       │   │   ├── ConfigManager.java
│       │   │   └── ItemManager.java
│       │   │
│       │   └── packets/
│       │       └── PacketHandler.java
│       │
│       └── resources/
│           ├── plugin.yml               # Plugin manifest
│           ├── havoc_items.json         # Item definitions
│           └── config.yml               # Auto-generated
│
├── target/                              # Build output (ignore)
│   └── AnonymousHavoc-1.0-SNAPSHOT.jar # Final JAR
│
└── anonymous_havoc.db                   # SQLite database (auto-created)
```

## Maven Build Profile

**pom.xml** includes:
- **Java 21 target** compilation
- **Maven Shade Plugin** for SQLite bundling
- **ProtocolLib dependency** (provided scope - must be in server)
- **Paper API** (latest 1.21 build)

### Custom Build Options

```bash
# Skip tests (if any)
mvn clean package -DskipTests

# Build with verbose output
mvn clean package -X

# List dependencies
mvn dependency:tree
```

## Troubleshooting Build Issues

### Issue: "Cannot find symbol: class JavaPlugin"
**Fix**: Ensure Paper API dependency is downloaded:
```bash
mvn dependency:resolve
```

### Issue: Compilation error for ProtocolLib
**Fix**: ProtocolLib is marked as "provided" - it only needs to be in the server's plugins folder, not during compilation. Error usually means a version mismatch. Update pom.xml:
```xml
<dependency>
    <groupId>com.comphenix.protocol</groupId>
    <artifactId>ProtocolLib</artifactId>
    <version>5.1.0</version>
    <scope>provided</scope>
</dependency>
```

### Issue: JAR not created in target/
**Fix**: Ensure build succeeded:
```bash
mvn clean package
echo $?  # Should print 0 if successful
```

## Runtime Configuration

### Edit config.yml

Located at: `plugins/AnonymousHavoc/config.yml`

```yaml
# Death ban durations
death-ban:
  normal-duration-hours: 24
  soul-binder-duration-hours: 48

# Coordinates leak system
coordinates-leak:
  interval-hours: 4
  reveal-radius: 150

# Auto-set on first run
server-start-time: 1689360000000
```

### Reset Server Day

To reset the server day count:
1. Delete `plugins/AnonymousHavoc/config.yml`
2. Restart server (auto-regenerates with current time)

## Database Management

### Location
```
plugins/AnonymousHavoc/anonymous_havoc.db
```

### Tables
- `item_owners` - Tracks item ownership
- `death_bans` - Player ban records with expiration
- `last_locations` - Last known positions for drop-off system

### Manual Query (SQLite)
```bash
sqlite3 plugins/AnonymousHavoc/anonymous_havoc.db

# Check player bans
SELECT * FROM death_bans;

# Check item owners
SELECT * FROM item_owners;

# Check player locations
SELECT * FROM last_locations;
```

### Reset Database
```bash
rm plugins/AnonymousHavoc/anonymous_havoc.db
# Restart server to auto-regenerate
```

## Testing Commands

### Admin Commands (OP Only)

```
# Give items for testing
/havoc give hunters_compass
/havoc give soul_binder_dagger
/havoc give sight_glass
/havoc give bell_of_truth
/havoc give totem_of_scrambling
/havoc give shard_of_anonymity

# Test kill broadcasts
# Use Soul-Binder Dagger to kill a player
# Should broadcast: "[Playername] was executed and erased"

# Test ban on rejoin
# Die in-game, then rejoin before 24h
# Should kick with: "Eliminated. You may return in [time]"
```

## Performance Notes

- **Database**: SQLite is lightweight; suitable for small to medium servers
- **Scheduled Tasks**: Uses async-safe Bukkit scheduler for
  - Item release checks (hourly)
  - Offline item drops (every 30 mins)
  - Coordinates leaks (every 4 hours)
- **Memory**: ~5-10 MB typical usage with 20+ players

## Scaling to Production

For larger servers:
1. **Database Migration**: Consider MySQL for better concurrency
   - Update `DatabaseManager.java` connection string
   - Adjust SQL driver dependency
2. **Cache Layer**: Add Redis for ban checks (optional)
3. **Logging**: Increase verbosity in production environment

## Development Workflow

### Modify Source Code
1. Edit files in `src/main/java/com/anonymoushavoc/`
2. Recompile: `mvn clean package`
3. Copy JAR to server
4. Restart server for changes to take effect

### Add New Item
1. Update `ConfigManager.itemSchedule` Map
2. Add case in `ItemManager.createCustomItems()`
3. Add recipe in `ItemManager.registerRecipe()`
4. Rebuild and restart server

### Add New Command
1. Create `src/main/java/com/anonymoushavoc/commands/NewCommand.java`
2. Register in `AnonymousHavoc.onEnable()`
3. Add to `plugin.yml` commands section
4. Rebuild and restart

## Deployment Checklist

- [ ] Java 21 JDK installed
- [ ] Maven 3.8.1+ installed
- [ ] PaperMC 1.21+ server running
- [ ] ProtocolLib JAR in plugins folder
- [ ] Build succeeds: `mvn clean package`
- [ ] JAR copied to `plugins/`
- [ ] Server restart successful
- [ ] `/havoc give test_item` works
- [ ] No errors in `logs/latest.log`
- [ ] admin can execute /havoc commands
- [ ] Performance normal (TPS stable)

## Support & Debugging

### Enable Debug Logging
Add to `plugin.yml`:
```yaml
logging: INFO  # Change to DEBUG for verbose output
```

### Check Server Logs
```bash
tail -f logs/latest.log | grep AnonymousHavoc
```

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Plugin won't load | ProtocolLib missing | Download and install ProtocolLib |
| Commands don't work | Permission not granted | Use OP or grant `havoc.admin` permission |
| Items not crafting | Day not reached | Use `/havoc give` to test items |
| Ban system not working | Database corrupt | Delete `.db` file and restart |

---

**Version**: 1.0
**Last Updated**: Day 1
**Maintainer**: Adam
