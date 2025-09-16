# milvus Installation Guide

milvus is a free and open-source vector database for AI applications. Milvus is designed to store, index, and search embedding vectors for AI applications, serving as an open-source alternative to Pinecone or Weaviate cloud

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
  - CPU: 4+ cores recommended
  - RAM: 8GB minimum (16GB+ recommended)
  - Storage: 10GB+ for vectors
  - Network: gRPC and HTTP access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 19530 (default milvus port)
  - Port 9091 for metrics
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

# Install milvus
sudo dnf install -y milvus

# Enable and start service
sudo systemctl enable --now milvus

# Configure firewall
sudo firewall-cmd --permanent --add-port=19530/tcp
sudo firewall-cmd --reload

# Verify installation
milvus --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install milvus
sudo apt install -y milvus

# Enable and start service
sudo systemctl enable --now milvus

# Configure firewall
sudo ufw allow 19530

# Verify installation
milvus --version
```

### Arch Linux

```bash
# Install milvus
sudo pacman -S milvus

# Enable and start service
sudo systemctl enable --now milvus

# Verify installation
milvus --version
```

### Alpine Linux

```bash
# Install milvus
apk add --no-cache milvus

# Enable and start service
rc-update add milvus default
rc-service milvus start

# Verify installation
milvus --version
```

### openSUSE/SLES

```bash
# Install milvus
sudo zypper install -y milvus

# Enable and start service
sudo systemctl enable --now milvus

# Configure firewall
sudo firewall-cmd --permanent --add-port=19530/tcp
sudo firewall-cmd --reload

# Verify installation
milvus --version
```

### macOS

```bash
# Using Homebrew
brew install milvus

# Start service
brew services start milvus

# Verify installation
milvus --version
```

### FreeBSD

```bash
# Using pkg
pkg install milvus

# Enable in rc.conf
echo 'milvus_enable="YES"' >> /etc/rc.conf

# Start service
service milvus start

# Verify installation
milvus --version
```

### Windows

```bash
# Using Chocolatey
choco install milvus

# Or using Scoop
scoop install milvus

# Verify installation
milvus --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/milvus

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
milvus --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable milvus

# Start service
sudo systemctl start milvus

# Stop service
sudo systemctl stop milvus

# Restart service
sudo systemctl restart milvus

# Check status
sudo systemctl status milvus

# View logs
sudo journalctl -u milvus -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add milvus default

# Start service
rc-service milvus start

# Stop service
rc-service milvus stop

# Restart service
rc-service milvus restart

# Check status
rc-service milvus status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'milvus_enable="YES"' >> /etc/rc.conf

# Start service
service milvus start

# Stop service
service milvus stop

# Restart service
service milvus restart

# Check status
service milvus status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start milvus
brew services stop milvus
brew services restart milvus

# Check status
brew services list | grep milvus
```

### Windows Service Manager

```powershell
# Start service
net start milvus

# Stop service
net stop milvus

# Using PowerShell
Start-Service milvus
Stop-Service milvus
Restart-Service milvus

# Check status
Get-Service milvus
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream milvus_backend {
    server 127.0.0.1:19530;
}

server {
    listen 80;
    server_name milvus.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name milvus.example.com;

    ssl_certificate /etc/ssl/certs/milvus.example.com.crt;
    ssl_certificate_key /etc/ssl/private/milvus.example.com.key;

    location / {
        proxy_pass http://milvus_backend;
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
    ServerName milvus.example.com
    Redirect permanent / https://milvus.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName milvus.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/milvus.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/milvus.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:19530/
    ProxyPassReverse / http://127.0.0.1:19530/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend milvus_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/milvus.pem
    redirect scheme https if !{ ssl_fc }
    default_backend milvus_backend

backend milvus_backend
    balance roundrobin
    server milvus1 127.0.0.1:19530 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R milvus:milvus /etc/milvus
sudo chmod 750 /etc/milvus

# Configure firewall
sudo firewall-cmd --permanent --add-port=19530/tcp
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
sudo systemctl status milvus

# View logs
sudo journalctl -u milvus -f

# Monitor resource usage
top -p $(pgrep milvus)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/milvus"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/milvus-backup-$DATE.tar.gz" /etc/milvus /var/lib/milvus

echo "Backup completed: $BACKUP_DIR/milvus-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop milvus

# Restore from backup
tar -xzf /backup/milvus/milvus-backup-*.tar.gz -C /

# Start service
sudo systemctl start milvus
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u milvus -n 100
sudo tail -f /var/log/milvus/milvus.log

# Check configuration
milvus --version

# Check permissions
ls -la /etc/milvus
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 19530

# Test connectivity
telnet localhost 19530

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep milvus)

# Check disk I/O
iotop -p $(pgrep milvus)

# Check connections
ss -an | grep 19530
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  milvus:
    image: milvus:latest
    ports:
      - "19530:19530"
    volumes:
      - ./config:/etc/milvus
      - ./data:/var/lib/milvus
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update milvus

# Debian/Ubuntu
sudo apt update && sudo apt upgrade milvus

# Arch Linux
sudo pacman -Syu milvus

# Alpine Linux
apk update && apk upgrade milvus

# openSUSE
sudo zypper update milvus

# FreeBSD
pkg update && pkg upgrade milvus

# Always backup before updates
tar -czf /backup/milvus-pre-update-$(date +%Y%m%d).tar.gz /etc/milvus

# Restart after updates
sudo systemctl restart milvus
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/milvus

# Clean old logs
find /var/log/milvus -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/milvus
```

## Additional Resources

- Official Documentation: https://docs.milvus.org/
- GitHub Repository: https://github.com/milvus/milvus
- Community Forum: https://forum.milvus.org/
- Best Practices Guide: https://docs.milvus.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
