# questdb Installation Guide

questdb is a free and open-source high-performance time-series database. QuestDB offers SQL with time-series extensions and exceptional performance, serving as an alternative to InfluxDB or TimescaleDB for demanding workloads

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores recommended
  - RAM: 4GB minimum (16GB+ recommended)
  - Storage: 10GB+ for data
  - Network: HTTP and PostgreSQL wire
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9000 (default questdb port)
  - Port 8812 for PostgreSQL
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install questdb
sudo dnf install -y questdb

# Enable and start service
sudo systemctl enable --now questdb

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
questdb --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install questdb
sudo apt install -y questdb

# Enable and start service
sudo systemctl enable --now questdb

# Configure firewall
sudo ufw allow 9000

# Verify installation
questdb --version
```

### Arch Linux

```bash
# Install questdb
sudo pacman -S questdb

# Enable and start service
sudo systemctl enable --now questdb

# Verify installation
questdb --version
```

### Alpine Linux

```bash
# Install questdb
apk add --no-cache questdb

# Enable and start service
rc-update add questdb default
rc-service questdb start

# Verify installation
questdb --version
```

### openSUSE/SLES

```bash
# Install questdb
sudo zypper install -y questdb

# Enable and start service
sudo systemctl enable --now questdb

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
questdb --version
```

### macOS

```bash
# Using Homebrew
brew install questdb

# Start service
brew services start questdb

# Verify installation
questdb --version
```

### FreeBSD

```bash
# Using pkg
pkg install questdb

# Enable in rc.conf
echo 'questdb_enable="YES"' >> /etc/rc.conf

# Start service
service questdb start

# Verify installation
questdb --version
```

### Windows

```bash
# Using Chocolatey
choco install questdb

# Or using Scoop
scoop install questdb

# Verify installation
questdb --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/questdb

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
questdb --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable questdb

# Start service
sudo systemctl start questdb

# Stop service
sudo systemctl stop questdb

# Restart service
sudo systemctl restart questdb

# Check status
sudo systemctl status questdb

# View logs
sudo journalctl -u questdb -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add questdb default

# Start service
rc-service questdb start

# Stop service
rc-service questdb stop

# Restart service
rc-service questdb restart

# Check status
rc-service questdb status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'questdb_enable="YES"' >> /etc/rc.conf

# Start service
service questdb start

# Stop service
service questdb stop

# Restart service
service questdb restart

# Check status
service questdb status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start questdb
brew services stop questdb
brew services restart questdb

# Check status
brew services list | grep questdb
```

### Windows Service Manager

```powershell
# Start service
net start questdb

# Stop service
net stop questdb

# Using PowerShell
Start-Service questdb
Stop-Service questdb
Restart-Service questdb

# Check status
Get-Service questdb
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream questdb_backend {
    server 127.0.0.1:9000;
}

server {
    listen 80;
    server_name questdb.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name questdb.example.com;

    ssl_certificate /etc/ssl/certs/questdb.example.com.crt;
    ssl_certificate_key /etc/ssl/private/questdb.example.com.key;

    location / {
        proxy_pass http://questdb_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName questdb.example.com
    Redirect permanent / https://questdb.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName questdb.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/questdb.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/questdb.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend questdb_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/questdb.pem
    redirect scheme https if !{ ssl_fc }
    default_backend questdb_backend

backend questdb_backend
    balance roundrobin
    server questdb1 127.0.0.1:9000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R questdb:questdb /etc/questdb
sudo chmod 750 /etc/questdb

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status questdb

# View logs
sudo journalctl -u questdb -f

# Monitor resource usage
top -p $(pgrep questdb)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/questdb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/questdb-backup-$DATE.tar.gz" /etc/questdb /var/lib/questdb

echo "Backup completed: $BACKUP_DIR/questdb-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop questdb

# Restore from backup
tar -xzf /backup/questdb/questdb-backup-*.tar.gz -C /

# Start service
sudo systemctl start questdb
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u questdb -n 100
sudo tail -f /var/log/questdb/questdb.log

# Check configuration
questdb --version

# Check permissions
ls -la /etc/questdb
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000

# Test connectivity
telnet localhost 9000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep questdb)

# Check disk I/O
iotop -p $(pgrep questdb)

# Check connections
ss -an | grep 9000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  questdb:
    image: questdb:latest
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/questdb
      - ./data:/var/lib/questdb
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update questdb

# Debian/Ubuntu
sudo apt update && sudo apt upgrade questdb

# Arch Linux
sudo pacman -Syu questdb

# Alpine Linux
apk update && apk upgrade questdb

# openSUSE
sudo zypper update questdb

# FreeBSD
pkg update && pkg upgrade questdb

# Always backup before updates
tar -czf /backup/questdb-pre-update-$(date +%Y%m%d).tar.gz /etc/questdb

# Restart after updates
sudo systemctl restart questdb
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/questdb

# Clean old logs
find /var/log/questdb -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/questdb
```

## Additional Resources

- Official Documentation: https://docs.questdb.org/
- GitHub Repository: https://github.com/questdb/questdb
- Community Forum: https://forum.questdb.org/
- Best Practices Guide: https://docs.questdb.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
