# kafka Installation Guide

kafka is a free and open-source distributed streaming platform. Kafka provides high-throughput distributed messaging system

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
  - CPU: 4+ cores
  - RAM: 6GB minimum (32GB+ for production)
  - Storage: 100GB+ for logs
  - Network: Kafka protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9092 (default kafka port)
  - Zookeeper on 2181
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

# Install kafka
sudo dnf install -y kafka

# Enable and start service
sudo systemctl enable --now kafka

# Configure firewall
sudo firewall-cmd --permanent --add-port=9092/tcp
sudo firewall-cmd --reload

# Verify installation
kafka --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install kafka
sudo apt install -y kafka

# Enable and start service
sudo systemctl enable --now kafka

# Configure firewall
sudo ufw allow 9092

# Verify installation
kafka --version
```

### Arch Linux

```bash
# Install kafka
sudo pacman -S kafka

# Enable and start service
sudo systemctl enable --now kafka

# Verify installation
kafka --version
```

### Alpine Linux

```bash
# Install kafka
apk add --no-cache kafka

# Enable and start service
rc-update add kafka default
rc-service kafka start

# Verify installation
kafka --version
```

### openSUSE/SLES

```bash
# Install kafka
sudo zypper install -y kafka

# Enable and start service
sudo systemctl enable --now kafka

# Configure firewall
sudo firewall-cmd --permanent --add-port=9092/tcp
sudo firewall-cmd --reload

# Verify installation
kafka --version
```

### macOS

```bash
# Using Homebrew
brew install kafka

# Start service
brew services start kafka

# Verify installation
kafka --version
```

### FreeBSD

```bash
# Using pkg
pkg install kafka

# Enable in rc.conf
echo 'kafka_enable="YES"' >> /etc/rc.conf

# Start service
service kafka start

# Verify installation
kafka --version
```

### Windows

```bash
# Using Chocolatey
choco install kafka

# Or using Scoop
scoop install kafka

# Verify installation
kafka --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/kafka

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
kafka --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable kafka

# Start service
sudo systemctl start kafka

# Stop service
sudo systemctl stop kafka

# Restart service
sudo systemctl restart kafka

# Check status
sudo systemctl status kafka

# View logs
sudo journalctl -u kafka -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add kafka default

# Start service
rc-service kafka start

# Stop service
rc-service kafka stop

# Restart service
rc-service kafka restart

# Check status
rc-service kafka status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'kafka_enable="YES"' >> /etc/rc.conf

# Start service
service kafka start

# Stop service
service kafka stop

# Restart service
service kafka restart

# Check status
service kafka status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start kafka
brew services stop kafka
brew services restart kafka

# Check status
brew services list | grep kafka
```

### Windows Service Manager

```powershell
# Start service
net start kafka

# Stop service
net stop kafka

# Using PowerShell
Start-Service kafka
Stop-Service kafka
Restart-Service kafka

# Check status
Get-Service kafka
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream kafka_backend {
    server 127.0.0.1:9092;
}

server {
    listen 80;
    server_name kafka.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name kafka.example.com;

    ssl_certificate /etc/ssl/certs/kafka.example.com.crt;
    ssl_certificate_key /etc/ssl/private/kafka.example.com.key;

    location / {
        proxy_pass http://kafka_backend;
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
    ServerName kafka.example.com
    Redirect permanent / https://kafka.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName kafka.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/kafka.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/kafka.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9092/
    ProxyPassReverse / http://127.0.0.1:9092/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend kafka_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/kafka.pem
    redirect scheme https if !{ ssl_fc }
    default_backend kafka_backend

backend kafka_backend
    balance roundrobin
    server kafka1 127.0.0.1:9092 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R kafka:kafka /etc/kafka
sudo chmod 750 /etc/kafka

# Configure firewall
sudo firewall-cmd --permanent --add-port=9092/tcp
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
sudo systemctl status kafka

# View logs
sudo journalctl -u kafka -f

# Monitor resource usage
top -p $(pgrep kafka)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/kafka"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/kafka-backup-$DATE.tar.gz" /etc/kafka /var/lib/kafka

echo "Backup completed: $BACKUP_DIR/kafka-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop kafka

# Restore from backup
tar -xzf /backup/kafka/kafka-backup-*.tar.gz -C /

# Start service
sudo systemctl start kafka
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u kafka -n 100
sudo tail -f /var/log/kafka/kafka.log

# Check configuration
kafka --version

# Check permissions
ls -la /etc/kafka
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9092

# Test connectivity
telnet localhost 9092

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep kafka)

# Check disk I/O
iotop -p $(pgrep kafka)

# Check connections
ss -an | grep 9092
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  kafka:
    image: kafka:latest
    ports:
      - "9092:9092"
    volumes:
      - ./config:/etc/kafka
      - ./data:/var/lib/kafka
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update kafka

# Debian/Ubuntu
sudo apt update && sudo apt upgrade kafka

# Arch Linux
sudo pacman -Syu kafka

# Alpine Linux
apk update && apk upgrade kafka

# openSUSE
sudo zypper update kafka

# FreeBSD
pkg update && pkg upgrade kafka

# Always backup before updates
tar -czf /backup/kafka-pre-update-$(date +%Y%m%d).tar.gz /etc/kafka

# Restart after updates
sudo systemctl restart kafka
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/kafka

# Clean old logs
find /var/log/kafka -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/kafka
```

## Additional Resources

- Official Documentation: https://docs.kafka.org/
- GitHub Repository: https://github.com/kafka/kafka
- Community Forum: https://forum.kafka.org/
- Best Practices Guide: https://docs.kafka.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
