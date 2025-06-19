# WhatsApp MCP - Docker Setup

This document explains how to build and run the WhatsApp MCP project using Docker Compose.

## Prerequisites

- Docker
- Docker Compose

## Project Structure

The project consists of two main services:

1. **whatsapp-bridge**: A Go service that connects to WhatsApp and provides a REST API
2. **whatsapp-mcp-server**: A Python MCP server that provides WhatsApp functionality to MCP clients

## Quick Start

1. **Build and start the services**:
   ```bash
   docker-compose up --build
   ```

2. **For the first run, you'll need to authenticate with WhatsApp**:
   - The whatsapp-bridge service will display a QR code in the logs
   - Scan this QR code with your WhatsApp mobile app
   - The authentication data will be stored in `./whatsapp-bridge/store/`

3. **Access the services**:
   - WhatsApp Bridge API: http://localhost:8080
   - MCP Server: Available via stdio transport

## Service Details

### WhatsApp Bridge (Port 8080)

- **Purpose**: Connects to WhatsApp and provides REST API endpoints
- **Authentication**: Requires QR code scan on first run
- **Data Storage**: Stores session and message data in `./whatsapp-bridge/store/`
- **Health Check**: Available at `http://localhost:8080/api/health`

### WhatsApp MCP Server

- **Purpose**: Provides WhatsApp functionality as MCP tools
- **Dependencies**: Requires whatsapp-bridge to be healthy
- **Configuration**: Uses environment variables for API URL and database path
- **Transport**: Uses stdio for MCP communication

## Environment Variables

### WhatsApp Bridge
- `TZ`: Timezone (default: UTC)

### WhatsApp MCP Server
- `WHATSAPP_API_BASE_URL`: URL of the WhatsApp Bridge API (default: http://whatsapp-bridge:8080/api)
- `MESSAGES_DB_PATH`: Path to the messages database (default: /app/whatsapp-bridge/store/messages.db)
- `TZ`: Timezone (default: UTC)

## Volumes

- `./whatsapp-bridge/store`: Persistent storage for WhatsApp session and message data
- `./media`: Shared media directory for file uploads/downloads

## Commands

### Start services
```bash
docker-compose up
```

### Start services in background
```bash
docker-compose up -d
```

### View logs
```bash
docker-compose logs -f
```

### View logs for specific service
```bash
docker-compose logs -f whatsapp-bridge
docker-compose logs -f whatsapp-mcp-server
```

### Stop services
```bash
docker-compose down
```

### Rebuild and start
```bash
docker-compose up --build
```

### Access container shell
```bash
docker-compose exec whatsapp-bridge sh
docker-compose exec whatsapp-mcp-server bash
```

## Troubleshooting

### QR Code Authentication
- If you need to re-authenticate, delete the `./whatsapp-bridge/store/whatsapp.db` file
- Restart the services: `docker-compose restart`

### Service Health Issues
- Check if the bridge is healthy: `docker-compose ps`
- View bridge logs: `docker-compose logs whatsapp-bridge`
- The MCP server waits for the bridge to be healthy before starting

### Port Conflicts
- If port 8080 is already in use, modify the port mapping in `docker-compose.yml`:
  ```yaml
  ports:
    - "8081:8080"  # Change 8081 to your preferred port
  ```

### Database Issues
- Ensure the `./whatsapp-bridge/store` directory exists and has proper permissions
- The database files are automatically created on first run

## Development

### Rebuilding after code changes
```bash
docker-compose up --build
```

### Running in development mode
For development, you might want to mount the source code as volumes:

```yaml
# Add to docker-compose.yml for development
volumes:
  - ./whatsapp-bridge:/app:ro  # For bridge development
  - ./whatsapp-mcp-server:/app:ro  # For MCP server development
```

## Security Notes

- Both services run as non-root users
- The MCP server has read-only access to the bridge's store directory
- Consider using Docker secrets for sensitive configuration in production
- The WhatsApp session data should be kept secure as it provides access to your WhatsApp account 