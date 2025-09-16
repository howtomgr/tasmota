# tasmota Installation Guide

tasmota is a free and open-source ESP8266 firmware. Tasmota provides open source firmware for ESP devices

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
  - CPU: 1 core minimum
  - RAM: 64MB minimum
  - Storage: 100MB for OTA
  - Network: HTTP/MQTT
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default tasmota port)
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

# Install tasmota
sudo dnf install -y tasmota

# Enable and start service
sudo systemctl enable --now tasmota

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
tasmota --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install tasmota
sudo apt install -y tasmota

# Enable and start service
sudo systemctl enable --now tasmota

# Configure firewall
sudo ufw allow 80

# Verify installation
tasmota --version
```

### Arch Linux

```bash
# Install tasmota
sudo pacman -S tasmota

# Enable and start service
sudo systemctl enable --now tasmota

# Verify installation
tasmota --version
```

### Alpine Linux

```bash
# Install tasmota
apk add --no-cache tasmota

# Enable and start service
rc-update add tasmota default
rc-service tasmota start

# Verify installation
tasmota --version
```

### openSUSE/SLES

```bash
# Install tasmota
sudo zypper install -y tasmota

# Enable and start service
sudo systemctl enable --now tasmota

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
tasmota --version
```

### macOS

```bash
# Using Homebrew
brew install tasmota

# Start service
brew services start tasmota

# Verify installation
tasmota --version
```

### FreeBSD

```bash
# Using pkg
pkg install tasmota

# Enable in rc.conf
echo 'tasmota_enable="YES"' >> /etc/rc.conf

# Start service
service tasmota start

# Verify installation
tasmota --version
```

### Windows

```bash
# Using Chocolatey
choco install tasmota

# Or using Scoop
scoop install tasmota

# Verify installation
tasmota --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/tasmota

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
tasmota --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable tasmota

# Start service
sudo systemctl start tasmota

# Stop service
sudo systemctl stop tasmota

# Restart service
sudo systemctl restart tasmota

# Check status
sudo systemctl status tasmota

# View logs
sudo journalctl -u tasmota -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add tasmota default

# Start service
rc-service tasmota start

# Stop service
rc-service tasmota stop

# Restart service
rc-service tasmota restart

# Check status
rc-service tasmota status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'tasmota_enable="YES"' >> /etc/rc.conf

# Start service
service tasmota start

# Stop service
service tasmota stop

# Restart service
service tasmota restart

# Check status
service tasmota status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start tasmota
brew services stop tasmota
brew services restart tasmota

# Check status
brew services list | grep tasmota
```

### Windows Service Manager

```powershell
# Start service
net start tasmota

# Stop service
net stop tasmota

# Using PowerShell
Start-Service tasmota
Stop-Service tasmota
Restart-Service tasmota

# Check status
Get-Service tasmota
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream tasmota_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name tasmota.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tasmota.example.com;

    ssl_certificate /etc/ssl/certs/tasmota.example.com.crt;
    ssl_certificate_key /etc/ssl/private/tasmota.example.com.key;

    location / {
        proxy_pass http://tasmota_backend;
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
    ServerName tasmota.example.com
    Redirect permanent / https://tasmota.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName tasmota.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/tasmota.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/tasmota.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend tasmota_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/tasmota.pem
    redirect scheme https if !{ ssl_fc }
    default_backend tasmota_backend

backend tasmota_backend
    balance roundrobin
    server tasmota1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R tasmota:tasmota /etc/tasmota
sudo chmod 750 /etc/tasmota

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
sudo systemctl status tasmota

# View logs
sudo journalctl -u tasmota -f

# Monitor resource usage
top -p $(pgrep tasmota)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/tasmota"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/tasmota-backup-$DATE.tar.gz" /etc/tasmota /var/lib/tasmota

echo "Backup completed: $BACKUP_DIR/tasmota-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop tasmota

# Restore from backup
tar -xzf /backup/tasmota/tasmota-backup-*.tar.gz -C /

# Start service
sudo systemctl start tasmota
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u tasmota -n 100
sudo tail -f /var/log/tasmota/tasmota.log

# Check configuration
tasmota --version

# Check permissions
ls -la /etc/tasmota
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
top -p $(pgrep tasmota)

# Check disk I/O
iotop -p $(pgrep tasmota)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  tasmota:
    image: tasmota:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/tasmota
      - ./data:/var/lib/tasmota
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update tasmota

# Debian/Ubuntu
sudo apt update && sudo apt upgrade tasmota

# Arch Linux
sudo pacman -Syu tasmota

# Alpine Linux
apk update && apk upgrade tasmota

# openSUSE
sudo zypper update tasmota

# FreeBSD
pkg update && pkg upgrade tasmota

# Always backup before updates
tar -czf /backup/tasmota-pre-update-$(date +%Y%m%d).tar.gz /etc/tasmota

# Restart after updates
sudo systemctl restart tasmota
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/tasmota

# Clean old logs
find /var/log/tasmota -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/tasmota
```

## Additional Resources

- Official Documentation: https://docs.tasmota.org/
- GitHub Repository: https://github.com/tasmota/tasmota
- Community Forum: https://forum.tasmota.org/
- Best Practices Guide: https://docs.tasmota.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
