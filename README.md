<h1 align="center">
	<img
		width="300"
		alt="Zubr"
		src="https://raw.githubusercontent.com/micr0-dev/zubr-web/master/client/img/zubr-logo-vertical.svg">
</h1>

<h3 align="center">
		Instance-based decentralized federated chat
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

Complete Docker setup for Zubr, an Instance-based decentralized federated chat.

## Components

- **zubr-server**: Go backend API server that manages users and InspIRCd (port 3000)
- **zubr-web**: Node.js frontend web interface (port 9000)
- **InspIRCd**: IRC daemon (port 6697 SSL) - managed automatically by zubr-server

## Quick Start

1. Clone this repository:
```bash
   git clone https://github.com/micr0-dev/zubr-docker.git
   cd zubr-docker
```

2. Create and edit the .env file:
```bash
   cp .env.example .env
   nano .env
```

Set your instance domain.

3. Start the services:
```bash
   docker compose up -d
```

4. Access the application:
   - Web interface: http://localhost:9000
   - API server: http://localhost:3000
   - IRC server (SSL): localhost:6697

## Production Deployment with Apache/httpd & SSL

### Prerequisites

#### Install Docker
Follow the official Docker installation guide for your operating system:
- **Docker Installation**: https://docs.docker.com/engine/install/

After installation, verify Docker is working:
```bash
docker --version
docker compose version
```

#### Install Apache HTTP Server

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install apache2
sudo a2enmod proxy proxy_http proxy_wstunnel rewrite ssl headers
sudo systemctl restart apache2
```

**CentOS/RHEL:**
```bash
sudo yum install httpd mod_ssl
sudo systemctl enable httpd
sudo systemctl start httpd
```

**macOS (Homebrew):**
```bash
brew install httpd
```

#### Install Certbot for SSL Certificates

Certbot is a free, automated tool for obtaining and renewing SSL/TLS certificates from Let's Encrypt.

**Project Page**: https://certbot.eff.org/

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install certbot python3-certbot-apache
```

**CentOS/RHEL:**
```bash
sudo yum install certbot python3-certbot-apache
```

**Other systems**: Visit https://certbot.eff.org/instructions for detailed instructions.

### Setup Steps

#### 1. Configure DNS
Point your domain name to your server's IP address:
```
A Record: your-domain.com → YOUR_SERVER_IP
```

#### 2. Deploy Zubr with Docker
```bash
git clone https://github.com/micr0-dev/zubr-docker.git
cd zubr-docker
```

Make the .env file:
```bash
cp .env.example .env
nano .env
```
Make sure to set your instance domain.

Start the services:
```bash
docker compose up -d
```

Verify services are running:
```bash
docker compose ps
curl http://localhost:9000
```

#### 3a. Configure Apache (Ubuntu/Debian)

Copy the Apache configuration file:
```bash
sudo cp zubr.conf /etc/apache2/sites-available/zubr.conf
```

Edit the configuration with your domain:
```bash
sudo nano /etc/apache2/sites-available/zubr.conf
```

Replace `your-domain.com` with your actual domain name.

Enable the site:
```bash
sudo a2ensite zubr.conf
sudo systemctl reload apache2
```

#### 3b. Configure Apache/httpd (CentOS/RHEL)

Copy the Apache configuration file:
```bash
sudo cp zubr.conf /etc/httpd/conf.d/zubr.conf
```

Edit the configuration with your domain:
```bash
sudo nano /etc/httpd/conf.d/zubr.conf
```

Replace `your-domain.com` with your actual domain name.

Restart Apache:
```bash
sudo systemctl restart httpd
```

#### 4. Obtain SSL Certificate with Certbot

Run Certbot to automatically obtain and configure SSL:
```bash
sudo certbot --apache -d your-domain.com
```

Follow the prompts:
- Enter your email address
- Agree to the Terms of Service
- Choose whether to redirect HTTP to HTTPS (recommended: Yes)

Certbot will automatically:
- Obtain the SSL certificate
- Update your Apache configuration
- Set up automatic renewal

#### 5. Configure stunnel for IRC TLS Termination (Optional but Recommended)

Instead of managing TLS certificates separately for InspIRCd, you can use stunnel to handle TLS termination for IRC connections using the same Let's Encrypt certificates as Apache.

**Benefits:**
- Uses the same certificates as your web server
- Automatic certificate renewal with Certbot
- Simpler IRC server configuration
- Centralized TLS management

**Install stunnel:**

**Ubuntu/Debian:**
```bash
sudo apt install stunnel4
```

**CentOS/RHEL:**
```bash
sudo yum install stunnel
```

**Configure stunnel for IRC:**

Create the stunnel configuration file:
```bash
sudo nano /etc/stunnel/irc.conf
```

Add the following configuration:
```ini
[irc]
; Accept TLS connections on port 6697
accept = 6697
; Forward to plain IRC on localhost:6667
connect = 127.0.0.1:6667

; Use the same Let's Encrypt certificates as Apache
cert = /etc/letsencrypt/live/your-domain.com/fullchain.pem
key = /etc/letsencrypt/live/your-domain.com/privkey.pem

; TLS settings
sslVersion = TLSv1.2
ciphers = HIGH:!aNULL:!MD5
```

Replace `your-domain.com` with your actual domain name.  (Or you can copy the paths from the Apache SSL configuration.)

**Enable stunnel:**

**Ubuntu/Debian:**
```bash
sudo systemctl enable stunnel4
sudo systemctl start stunnel4
```

**CentOS/RHEL:**
```bash
sudo systemctl enable stunnel
sudo systemctl start stunnel
```

#### 6. Verify Auto-Renewal

Certbot sets up automatic renewal. Test it with:
```bash
sudo certbot renew --dry-run
```

### Firewall Configuration

Open required ports:

**UFW (Ubuntu/Debian):**
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 6697/tcp  # IRC SSL (if you are running a public instance)
sudo ufw enable
```

**firewalld (CentOS/RHEL):**
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=6697/tcp  # IRC SSL (if you are running a public instance)
sudo firewall-cmd --reload
```

## Configuration

### Ports

Default ports can be changed in `docker-compose.yml`:
- `9000:9000` - Web interface
- `3000:3000` - API server
- `6667:6667` - IRC server (SSL only)

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



## Architecture
```
┌─────────────┐         ┌──────────────────┐
│  zubr-web   │────────▶│  zubr-server     │
│  (port 9000)│         │  (port 3000)     │
└─────────────┘         │                  │
                        │  ┌─────────────┐ │
                        │  │  InspIRCd   │ │
                        │  │ (port 6697) │ │
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

### Contact
For support, feel free to reachout on https://zubr.chat instance in the #support channel.

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
