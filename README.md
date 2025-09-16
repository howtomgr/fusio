# fusio Installation Guide

fusio is a free and open-source API management. Fusio provides open source API management platform to build REST APIs

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 5GB for data
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default fusio port)
  - None
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

# Install fusio
sudo dnf install -y fusio

# Enable and start service
sudo systemctl enable --now fusio

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
fusio --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install fusio
sudo apt install -y fusio

# Enable and start service
sudo systemctl enable --now fusio

# Configure firewall
sudo ufw allow 80

# Verify installation
fusio --version
```

### Arch Linux

```bash
# Install fusio
sudo pacman -S fusio

# Enable and start service
sudo systemctl enable --now fusio

# Verify installation
fusio --version
```

### Alpine Linux

```bash
# Install fusio
apk add --no-cache fusio

# Enable and start service
rc-update add fusio default
rc-service fusio start

# Verify installation
fusio --version
```

### openSUSE/SLES

```bash
# Install fusio
sudo zypper install -y fusio

# Enable and start service
sudo systemctl enable --now fusio

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
fusio --version
```

### macOS

```bash
# Using Homebrew
brew install fusio

# Start service
brew services start fusio

# Verify installation
fusio --version
```

### FreeBSD

```bash
# Using pkg
pkg install fusio

# Enable in rc.conf
echo 'fusio_enable="YES"' >> /etc/rc.conf

# Start service
service fusio start

# Verify installation
fusio --version
```

### Windows

```bash
# Using Chocolatey
choco install fusio

# Or using Scoop
scoop install fusio

# Verify installation
fusio --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/fusio

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
fusio --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable fusio

# Start service
sudo systemctl start fusio

# Stop service
sudo systemctl stop fusio

# Restart service
sudo systemctl restart fusio

# Check status
sudo systemctl status fusio

# View logs
sudo journalctl -u fusio -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add fusio default

# Start service
rc-service fusio start

# Stop service
rc-service fusio stop

# Restart service
rc-service fusio restart

# Check status
rc-service fusio status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'fusio_enable="YES"' >> /etc/rc.conf

# Start service
service fusio start

# Stop service
service fusio stop

# Restart service
service fusio restart

# Check status
service fusio status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start fusio
brew services stop fusio
brew services restart fusio

# Check status
brew services list | grep fusio
```

### Windows Service Manager

```powershell
# Start service
net start fusio

# Stop service
net stop fusio

# Using PowerShell
Start-Service fusio
Stop-Service fusio
Restart-Service fusio

# Check status
Get-Service fusio
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream fusio_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name fusio.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name fusio.example.com;

    ssl_certificate /etc/ssl/certs/fusio.example.com.crt;
    ssl_certificate_key /etc/ssl/private/fusio.example.com.key;

    location / {
        proxy_pass http://fusio_backend;
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
    ServerName fusio.example.com
    Redirect permanent / https://fusio.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName fusio.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/fusio.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/fusio.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend fusio_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/fusio.pem
    redirect scheme https if !{ ssl_fc }
    default_backend fusio_backend

backend fusio_backend
    balance roundrobin
    server fusio1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R fusio:fusio /etc/fusio
sudo chmod 750 /etc/fusio

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status fusio

# View logs
sudo journalctl -u fusio -f

# Monitor resource usage
top -p $(pgrep fusio)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/fusio"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/fusio-backup-$DATE.tar.gz" /etc/fusio /var/lib/fusio

echo "Backup completed: $BACKUP_DIR/fusio-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop fusio

# Restore from backup
tar -xzf /backup/fusio/fusio-backup-*.tar.gz -C /

# Start service
sudo systemctl start fusio
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u fusio -n 100
sudo tail -f /var/log/fusio/fusio.log

# Check configuration
fusio --version

# Check permissions
ls -la /etc/fusio
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep fusio)

# Check disk I/O
iotop -p $(pgrep fusio)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  fusio:
    image: fusio:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/fusio
      - ./data:/var/lib/fusio
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update fusio

# Debian/Ubuntu
sudo apt update && sudo apt upgrade fusio

# Arch Linux
sudo pacman -Syu fusio

# Alpine Linux
apk update && apk upgrade fusio

# openSUSE
sudo zypper update fusio

# FreeBSD
pkg update && pkg upgrade fusio

# Always backup before updates
tar -czf /backup/fusio-pre-update-$(date +%Y%m%d).tar.gz /etc/fusio

# Restart after updates
sudo systemctl restart fusio
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/fusio

# Clean old logs
find /var/log/fusio -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/fusio
```

## Additional Resources

- Official Documentation: https://docs.fusio.org/
- GitHub Repository: https://github.com/fusio/fusio
- Community Forum: https://forum.fusio.org/
- Best Practices Guide: https://docs.fusio.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
