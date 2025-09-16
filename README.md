# opencve Installation Guide

opencve is a free and open-source CVE monitoring. OpenCVE provides CVE monitoring and alerting platform

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
  - RAM: 1GB minimum
  - Storage: 5GB for data
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8000 (default opencve port)
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

# Install opencve
sudo dnf install -y opencve

# Enable and start service
sudo systemctl enable --now opencve

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
opencve --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install opencve
sudo apt install -y opencve

# Enable and start service
sudo systemctl enable --now opencve

# Configure firewall
sudo ufw allow 8000

# Verify installation
opencve --version
```

### Arch Linux

```bash
# Install opencve
sudo pacman -S opencve

# Enable and start service
sudo systemctl enable --now opencve

# Verify installation
opencve --version
```

### Alpine Linux

```bash
# Install opencve
apk add --no-cache opencve

# Enable and start service
rc-update add opencve default
rc-service opencve start

# Verify installation
opencve --version
```

### openSUSE/SLES

```bash
# Install opencve
sudo zypper install -y opencve

# Enable and start service
sudo systemctl enable --now opencve

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
opencve --version
```

### macOS

```bash
# Using Homebrew
brew install opencve

# Start service
brew services start opencve

# Verify installation
opencve --version
```

### FreeBSD

```bash
# Using pkg
pkg install opencve

# Enable in rc.conf
echo 'opencve_enable="YES"' >> /etc/rc.conf

# Start service
service opencve start

# Verify installation
opencve --version
```

### Windows

```bash
# Using Chocolatey
choco install opencve

# Or using Scoop
scoop install opencve

# Verify installation
opencve --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/opencve

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
opencve --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable opencve

# Start service
sudo systemctl start opencve

# Stop service
sudo systemctl stop opencve

# Restart service
sudo systemctl restart opencve

# Check status
sudo systemctl status opencve

# View logs
sudo journalctl -u opencve -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add opencve default

# Start service
rc-service opencve start

# Stop service
rc-service opencve stop

# Restart service
rc-service opencve restart

# Check status
rc-service opencve status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'opencve_enable="YES"' >> /etc/rc.conf

# Start service
service opencve start

# Stop service
service opencve stop

# Restart service
service opencve restart

# Check status
service opencve status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start opencve
brew services stop opencve
brew services restart opencve

# Check status
brew services list | grep opencve
```

### Windows Service Manager

```powershell
# Start service
net start opencve

# Stop service
net stop opencve

# Using PowerShell
Start-Service opencve
Stop-Service opencve
Restart-Service opencve

# Check status
Get-Service opencve
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream opencve_backend {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name opencve.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name opencve.example.com;

    ssl_certificate /etc/ssl/certs/opencve.example.com.crt;
    ssl_certificate_key /etc/ssl/private/opencve.example.com.key;

    location / {
        proxy_pass http://opencve_backend;
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
    ServerName opencve.example.com
    Redirect permanent / https://opencve.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName opencve.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/opencve.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/opencve.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend opencve_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/opencve.pem
    redirect scheme https if !{ ssl_fc }
    default_backend opencve_backend

backend opencve_backend
    balance roundrobin
    server opencve1 127.0.0.1:8000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R opencve:opencve /etc/opencve
sudo chmod 750 /etc/opencve

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
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
sudo systemctl status opencve

# View logs
sudo journalctl -u opencve -f

# Monitor resource usage
top -p $(pgrep opencve)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/opencve"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/opencve-backup-$DATE.tar.gz" /etc/opencve /var/lib/opencve

echo "Backup completed: $BACKUP_DIR/opencve-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop opencve

# Restore from backup
tar -xzf /backup/opencve/opencve-backup-*.tar.gz -C /

# Start service
sudo systemctl start opencve
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u opencve -n 100
sudo tail -f /var/log/opencve/opencve.log

# Check configuration
opencve --version

# Check permissions
ls -la /etc/opencve
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8000

# Test connectivity
telnet localhost 8000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep opencve)

# Check disk I/O
iotop -p $(pgrep opencve)

# Check connections
ss -an | grep 8000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  opencve:
    image: opencve:latest
    ports:
      - "8000:8000"
    volumes:
      - ./config:/etc/opencve
      - ./data:/var/lib/opencve
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update opencve

# Debian/Ubuntu
sudo apt update && sudo apt upgrade opencve

# Arch Linux
sudo pacman -Syu opencve

# Alpine Linux
apk update && apk upgrade opencve

# openSUSE
sudo zypper update opencve

# FreeBSD
pkg update && pkg upgrade opencve

# Always backup before updates
tar -czf /backup/opencve-pre-update-$(date +%Y%m%d).tar.gz /etc/opencve

# Restart after updates
sudo systemctl restart opencve
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/opencve

# Clean old logs
find /var/log/opencve -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/opencve
```

## Additional Resources

- Official Documentation: https://docs.opencve.org/
- GitHub Repository: https://github.com/opencve/opencve
- Community Forum: https://forum.opencve.org/
- Best Practices Guide: https://docs.opencve.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
