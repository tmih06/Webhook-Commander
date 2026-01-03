# Webhook Commander

A Minecraft 1.21.4 Fabric mod that opens an HTTP webhook server to receive and execute commands via JSON requests.

## Features

-   **HTTP Webhook Server**: Opens an HTTP server on a configurable port (default: 8080)
-   **Command Execution**: Executes Minecraft commands received via POST requests
-   **CORS Support**: Allows cross-origin requests for browser-based integrations
-   **Optional Authentication**: Secure your webhook with a Bearer token
-   **Health Check Endpoint**: Monitor server status at `/health`

## Installation

1. Build the mod:

    ```bash
    ./gradlew build
    ```

2. Copy the generated JAR from `build/libs/webhook-commander-1.0.0.jar` to your Minecraft server's `mods` folder

3. Make sure you have Fabric Loader and Fabric API installed for Minecraft 1.21.4

4. Start your Minecraft server

## Configuration

On first run, the mod creates a config file at `config/webhook_commander.json`:

```json
{
    "port": 8080,
    "authToken": ""
}
```

| Option      | Default | Description                                                                                         |
| ----------- | ------- | --------------------------------------------------------------------------------------------------- |
| `port`      | `8080`  | The port the webhook server listens on                                                              |
| `authToken` | `""`    | Optional authentication token. If set, requests must include `Authorization: Bearer <token>` header |

## Usage

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
    "mod": "webhook_commander"
}
```

## Examples

### Basic command (curl)

```bash
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/say Hello World!"}'
```

### With authentication

```bash
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-secret-token" \
  -d '{"command": "/give @a diamond 1"}'
```

### Give items to all players

```bash
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/give @a minecraft:diamond 64"}'
```

### Spawn an entity

```bash
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/summon minecraft:creeper ~ ~ ~"}'
```

### Set time

```bash
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/time set day"}'
```

### Play sound to all players

```bash
curl -X POST http://localhost:8080/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "/playsound minecraft:entity.ender_dragon.growl master @a"}'
```

## TikTok Integration Example

Here's an example of how you might integrate this with a TikTok webhook handler (Node.js):

```javascript
const fetch = require("node-fetch");

async function handleTikTokGift(gift) {
    const command = `/say ${gift.username} sent ${gift.count}x ${gift.name}!`;

    const response = await fetch("http://your-minecraft-server:8080/execute", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            Authorization: "Bearer your-secret-token",
        },
        body: JSON.stringify({ command }),
    });

    const result = await response.json();
    console.log("Command result:", result);
}
```

## Security Considerations

⚠️ **Warning**: This mod opens a network port that can execute arbitrary Minecraft commands with server-level permissions.

-   **Use authentication**: Always set an `authToken` in production
-   **Firewall**: Only expose the webhook port to trusted networks/IPs
-   **HTTPS**: Consider putting the webhook behind a reverse proxy with HTTPS for external access

## Requirements

-   Minecraft 1.21.4
-   Fabric Loader 0.16.10+
-   Fabric API
-   Java 21+

## License

MIT License
