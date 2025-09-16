# crane Installation Guide

crane is a free and open-source Docker registry UI. Crane provides UI for Docker Registry with authentication

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
  - RAM: 256MB minimum
  - Storage: 1GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default crane port)
  - Registry access
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

# Install crane
sudo dnf install -y crane

# Enable and start service
sudo systemctl enable --now crane

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
crane --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install crane
sudo apt install -y crane

# Enable and start service
sudo systemctl enable --now crane

# Configure firewall
sudo ufw allow 80

# Verify installation
crane --version
```

### Arch Linux

```bash
# Install crane
sudo pacman -S crane

# Enable and start service
sudo systemctl enable --now crane

# Verify installation
crane --version
```

### Alpine Linux

```bash
# Install crane
apk add --no-cache crane

# Enable and start service
rc-update add crane default
rc-service crane start

# Verify installation
crane --version
```

### openSUSE/SLES

```bash
# Install crane
sudo zypper install -y crane

# Enable and start service
sudo systemctl enable --now crane

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
crane --version
```

### macOS

```bash
# Using Homebrew
brew install crane

# Start service
brew services start crane

# Verify installation
crane --version
```

### FreeBSD

```bash
# Using pkg
pkg install crane

# Enable in rc.conf
echo 'crane_enable="YES"' >> /etc/rc.conf

# Start service
service crane start

# Verify installation
crane --version
```

### Windows

```bash
# Using Chocolatey
choco install crane

# Or using Scoop
scoop install crane

# Verify installation
crane --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/crane

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
crane --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable crane

# Start service
sudo systemctl start crane

# Stop service
sudo systemctl stop crane

# Restart service
sudo systemctl restart crane

# Check status
sudo systemctl status crane

# View logs
sudo journalctl -u crane -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add crane default

# Start service
rc-service crane start

# Stop service
rc-service crane stop

# Restart service
rc-service crane restart

# Check status
rc-service crane status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'crane_enable="YES"' >> /etc/rc.conf

# Start service
service crane start

# Stop service
service crane stop

# Restart service
service crane restart

# Check status
service crane status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start crane
brew services stop crane
brew services restart crane

# Check status
brew services list | grep crane
```

### Windows Service Manager

```powershell
# Start service
net start crane

# Stop service
net stop crane

# Using PowerShell
Start-Service crane
Stop-Service crane
Restart-Service crane

# Check status
Get-Service crane
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream crane_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name crane.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name crane.example.com;

    ssl_certificate /etc/ssl/certs/crane.example.com.crt;
    ssl_certificate_key /etc/ssl/private/crane.example.com.key;

    location / {
        proxy_pass http://crane_backend;
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
    ServerName crane.example.com
    Redirect permanent / https://crane.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName crane.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/crane.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/crane.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend crane_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/crane.pem
    redirect scheme https if !{ ssl_fc }
    default_backend crane_backend

backend crane_backend
    balance roundrobin
    server crane1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R crane:crane /etc/crane
sudo chmod 750 /etc/crane

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
sudo systemctl status crane

# View logs
sudo journalctl -u crane -f

# Monitor resource usage
top -p $(pgrep crane)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/crane"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/crane-backup-$DATE.tar.gz" /etc/crane /var/lib/crane

echo "Backup completed: $BACKUP_DIR/crane-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop crane

# Restore from backup
tar -xzf /backup/crane/crane-backup-*.tar.gz -C /

# Start service
sudo systemctl start crane
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u crane -n 100
sudo tail -f /var/log/crane/crane.log

# Check configuration
crane --version

# Check permissions
ls -la /etc/crane
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
top -p $(pgrep crane)

# Check disk I/O
iotop -p $(pgrep crane)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  crane:
    image: crane:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/crane
      - ./data:/var/lib/crane
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update crane

# Debian/Ubuntu
sudo apt update && sudo apt upgrade crane

# Arch Linux
sudo pacman -Syu crane

# Alpine Linux
apk update && apk upgrade crane

# openSUSE
sudo zypper update crane

# FreeBSD
pkg update && pkg upgrade crane

# Always backup before updates
tar -czf /backup/crane-pre-update-$(date +%Y%m%d).tar.gz /etc/crane

# Restart after updates
sudo systemctl restart crane
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/crane

# Clean old logs
find /var/log/crane -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/crane
```

## Additional Resources

- Official Documentation: https://docs.crane.org/
- GitHub Repository: https://github.com/crane/crane
- Community Forum: https://forum.crane.org/
- Best Practices Guide: https://docs.crane.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
