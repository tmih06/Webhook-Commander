# Webhook Commander

A Minecraft 1.21.4 Fabric mod that opens an HTTP webhook server to receive and execute commands via JSON requests. Perfect for TikTok, Twitch, or any external integration!

## Features

- **HTTP Webhook Server**: Opens an HTTP server on a configurable port (default: 8080)
- **Command Execution**: Executes Minecraft commands received via POST requests
- **CORS Support**: Allows cross-origin requests for browser-based integrations
- **Optional Authentication**: Secure your webhook with a Bearer token
- **Health Check Endpoint**: Monitor server status at `/health`
- **In-Game Commands**: `/tiktok` command for debugging and status
- **Mod Menu Support**: Visual configuration via Mod Menu + Cloth Config

## Installation

### Requirements
- Minecraft 1.21.4
- Fabric Loader 0.16.10+
- Fabric API
- Java 21+
- (Optional) Mod Menu + Cloth Config for visual config screen

### Steps
1. Download the mod JAR from releases (or build with `./gradlew build`)
2. Copy `webhook-commander-1.0.0.jar` from `build/libs/` to your `mods/` folder
3. Install Fabric API for Minecraft 1.21.4
4. (Optional) Install Mod Menu and Cloth Config for visual configuration
5. Start your Minecraft server or singleplayer world

## Configuration

### Config File
On first run, the mod creates `config/webhook_commander.json`:

```json
{
  "port": 8080,
  "authToken": ""
}
```

| Option | Default | Description |
|--------|---------|-------------|
| `port` | `8080` | The port the webhook server listens on |
| `authToken` | `""` | Optional authentication token. If set, requests must include `Authorization: Bearer <token>` header |

### Mod Menu (Visual Config)
If you have Mod Menu and Cloth Config installed, you can configure the mod visually:
1. Open Mod Menu (default key: usually in the pause menu)
2. Find "Webhook Commander" and click the config button
3. Adjust settings and click "Save"

> **Note**: Changing the port requires a server restart to take effect.

## In-Game Commands

All commands require OP level 2 or higher.

| Command | Description |
|---------|-------------|
| `/tiktok debug true` | Enable verbose debug logging |
| `/tiktok debug false` | Disable debug logging |
| `/tiktok debug` | Check current debug status |
| `/tiktok status` | Show port, auth, and debug status |
| `/tiktok test` | Test if webhook server is running |

## API Usage

### Execute Command Endpoint

**POST** `/execute`

Send a command to be executed in the Minecraft world.

#### Request
```json
{
  "command": "/say Hello from TikTok!"
}
```

The leading slash in the command is optional.

#### Success Response (200)
```json
{
  "success": true,
  "message": "Command executed successfully",
  "result": 1
}
```

#### Error Response (400/500)
```json
{
  "success": false,
  "message": "Error description"
}
```

### Health Check Endpoint

**GET** `/health`

Returns the server status.

```json
{
  "status": "ok",
  "mod": "webhook_commander",
  "port": 8080
}
```

## Examples

### Basic Command (curl)

```bash
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/say Hello World!"}'
```

### PowerShell (Windows)

```powershell
Invoke-WebRequest -Uri "http://127.0.0.1:8080/execute" `
  -Method POST `
  -ContentType "application/json" `
  -Body '{"command": "/say Hello World!"}'
```

### With Authentication

```bash
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-secret-token" \
  -d '{"command": "/give @a diamond 1"}'
```

### Common Commands

```bash
# Give items to all players
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/give @a minecraft:diamond 64"}'

# Spawn an entity
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/summon minecraft:creeper ~ ~ ~"}'

# Set time
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/time set day"}'

# Play sound
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/playsound minecraft:entity.ender_dragon.growl master @a"}'

# Send title
curl -X POST http://127.0.0.1:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/title @a title {\"text\":\"Hello!\",\"color\":\"gold\"}"}'
```

## TikTok Integration Example

Here's an example Node.js handler for TikTok gift events:

```javascript
const fetch = require('node-fetch');

const WEBHOOK_URL = 'http://your-minecraft-server:8080/execute';
const AUTH_TOKEN = 'your-secret-token'; // Optional

async function handleTikTokGift(gift) {
  const command = `/say §d${gift.username}§r sent §e${gift.count}x ${gift.name}§r!`;
  
  const response = await fetch(WEBHOOK_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${AUTH_TOKEN}`
    },
    body: JSON.stringify({ command })
  });
  
  const result = await response.json();
  console.log('Command result:', result);
}

// Example: Give diamonds based on gift value
async function handleGiftReward(gift) {
  const diamonds = Math.floor(gift.diamondCount / 10);
  if (diamonds > 0) {
    await fetch(WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ 
        command: `/give @a minecraft:diamond ${diamonds}` 
      })
    });
  }
}
```

## Troubleshooting

### "Bad host name" error with curl
Use `127.0.0.1` instead of `localhost`:
```bash
curl http://127.0.0.1:8080/health
```

### Cannot connect to webhook
1. Check if the server started successfully (look for "Webhook Commander - Server Started!" in logs)
2. Ensure the port isn't blocked by firewall
3. Use `/tiktok status` in-game to verify the webhook is running

### Debug Mode
Enable debug mode to see all incoming requests:
```
/tiktok debug true
```

This will log every request, response, and command execution to the server console.

## Security Considerations

⚠️ **Warning**: This mod opens a network port that can execute arbitrary Minecraft commands with server-level permissions.

- **Use authentication**: Always set an `authToken` in production
- **Firewall**: Only expose the webhook port to trusted networks/IPs
- **HTTPS**: Consider using a reverse proxy with HTTPS for external access
- **Port Security**: Use a non-standard port and keep it private

## Building from Source

```bash
# Clone the repository
git clone <repository-url>
cd TiktokIntegration

# Build the mod
./gradlew build

# The JAR will be in build/libs/webhook-commander-1.0.0.jar
```

## License

MIT License
