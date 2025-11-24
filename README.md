<h1 align="center">
	<img
		width="300"
		alt="Zubr"
		src="https://raw.githubusercontent.com/micr0-dev/zubr-web/master/client/img/zubr-logo-vertical.svg">
</h1>

<h3 align="center">
	Bringing IRC into the modern era
</h3>

<p align="center">
	<strong>
		<a href="https://github.com/micr0-dev/zubr-server">Zubr Server</a>
		•
		<a href="https://github.com/micr0-dev/zubr-web">Zubr Web</a>
		•
		<a href="https://github.com/micr0-dev/zubr-server/issues">Issues</a>
	</strong>
</p>

Complete Docker setup for the Zubr IRC-powered chat application.

## Components

- **zubr-server**: Go backend API server that manages users and InspIRCd (port 3000)
- **zubr-web**: Node.js frontend web interface (port 9000)
- **InspIRCd**: IRC daemon (port 6667) - managed automatically by zubr-server

## Quick Start

1. Clone this repository:
```bash
   git clone https://github.com/micr0-dev/zubr-docker.git
   cd zubr-docker
```

2. (Optional) Copy and customize environment variables:
```bash
   cp .env.example .env
```

3. Start the services:
```bash
   docker compose up -d
```

4. Access the application:
   - Web interface: http://localhost:9000
   - API server: http://localhost:3000
   - IRC server: localhost:6667

## Configuration

### Building Specific Versions

Edit `.env` to specify versions:
```bash
ZUBR_SERVER_VERSION=v0.3.0
ZUBR_WEB_VERSION=v1.2.0
```

### Ports

Default ports can be changed in `docker-compose.yml`:
- `9000:9000` - Web interface
- `3000:3000` - API server
- `6667:6667` - IRC server

### Data Persistence

Data is stored in Docker volumes:
- `zubr-data` - User data, settings, JWT secrets
- `inspircd-data` - IRC server data

## Management

### View logs
```bash
docker compose logs -f
```

### Restart services
```bash
docker compose restart
```

### Stop services
```bash
docker compose down
```

### Update to latest version
```bash
docker compose down
docker compose pull
docker compose build --no-cache
docker compose up -d
```

### Health Checks

Both services include health checks:
```bash
# Check zubr-server health
curl http://localhost:3000/api/health

# Check zubr-web health
curl http://localhost:9000
```

## Architecture
```
┌─────────────┐         ┌──────────────────┐
│  zubr-web   │───────▶│  zubr-server     │
│  (port 9000)│         │  (port 3000)     │
└─────────────┘         │                  │
                        │  ┌─────────────┐ │
                        │  │  InspIRCd   │ │
                        │  │ (port 6667) │ │
                        │  └─────────────┘ │
                        └──────────────────┘
```

- zubr-web provides the web UI and connects to zubr-server for user management
- zubr-server manages users, generates InspIRCd config, and controls the IRC daemon
- InspIRCd runs within the zubr-server container and is fully managed by it

## Troubleshooting

### Services won't start
Check logs: `docker compose logs`

### InspIRCd not responding
The zubr-server automatically manages InspIRCd. Check server logs:
```bash
docker compose logs zubr-server
```

### Reset everything
```bash
docker compose down -v
docker compose up -d
```

## Development

For development with local repos:

1. Clone the source repositories:
```bash
   git clone https://github.com/micr0-dev/zubr-web.git
   git clone https://github.com/micr0-dev/zubr-server.git
```

2. Modify `docker-compose.yml` to use local builds:
```yaml
   zubr-server:
     build:
       context: ./zubr-server
       dockerfile: Dockerfile
```

## License

See individual component repositories for license information.