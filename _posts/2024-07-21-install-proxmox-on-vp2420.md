---
title: Homelab Origins: Installing and Configuring Proxmox on a Protectli VP2420
date: 2024-07-21 13:12:13 -0500
categories: [featured,homelab]
tags: [server,hardware,setup]     # TAG names should always be lowercase
---

## Introduction

Security is a critical aspect of self-hosting. This guide covers key security practices to help protect your self-hosted services from various threats. By implementing these measures, you can significantly enhance the security of your infrastructure.

## Regular Updates and Backups

### Keeping Your System and Software Updated

Regular updates are crucial for patching vulnerabilities:

1. Enable automatic updates (e.g., `unattended-upgrades` on Debian-based systems)
2. Schedule manual checks weekly
3. Monitor security advisories
4. Test updates in a staging environment before production deployment

### Implementing a Robust Backup Strategy

Follow the 3-2-1 backup rule:

1. Maintain 3 copies of data
2. Store on 2 different storage types
3. Keep 1 copy off-site

Implement:
- Automated backups using tools like `rsync` or `rclone`
- Encryption for sensitive data (GPG for file-level, LUKS for full-disk)
- Regular restore tests (quarterly recommended)
- Versioned backups with tools like `restic` or `borg`

## Implementing Security Headers

Security headers instruct browsers to enhance web application security.

### Key Security Headers

1. Content-Security-Policy (CSP)
2. X-Frame-Options
3. X-XSS-Protection
4. Strict-Transport-Security (HSTS)
5. X-Content-Type-Options
6. Referrer-Policy

### Configuring Security Headers in Nginx

Add to your Nginx server block:

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

After configuration:

```bash
nginx -t
systemctl reload nginx
```

Verify using tools like Mozilla Observatory or Security Headers.

## Using Strong Passwords and SSH Keys

### Strong Passwords

- Minimum 12 characters
- Mix of uppercase, lowercase, numbers, and special characters
- Unique for each service
- Use a password manager

### SSH Keys

Set up SSH key authentication:

1. Generate key pair
```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
```

2. Copy to server:
```bash
   ssh-copy-id user@server_ip
```

3. Disable password authentication in `/etc/ssh/sshd_config`:
```
   PasswordAuthentication no
```

4. Restart SSH:
```bash
   systemctl restart sshd
```

## Additional Layers of Security

### Setting up a Firewall

Using UFW (Uncomplicated Firewall):

1. Install:
```bash
   sudo apt install ufw
```

2. Set default policies:
```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
```

3. Allow necessary services:
```bash
   sudo ufw allow ssh
   sudo ufw allow http
   sudo ufw allow https
```

4. Enable:
```bash
   sudo ufw enable
```

### Using fail2ban to Prevent Brute Force Attacks

1. Install:
```bash
   sudo apt install fail2ban
```

2. Create local config:
```bash
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

3. Edit `/etc/fail2ban/jail.local`:
```ini
   [sshd]
   enabled = true
   port = ssh
   filter = sshd
   logpath = /var/log/auth.log
   maxretry = 3
   bantime = 3600
```

4. Start and enable:
```bash
   sudo systemctl start fail2ban
   sudo systemctl enable fail2ban
```

## Monitoring and Maintenance

### Tools for Monitoring Your Server

1. Netdata:
```bash
   bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

2. Prometheus + Grafana (requires more setup)

3. Logwatch:
```bash
   sudo apt install logwatch
```

### Regular Maintenance Tasks

1. Log review:
```bash
   sudo journalctl -p err..emerg
```

2. Disk space management:
```bash
   df -h
   du -sh /var/log/*
```

3. User account audit:
```bash
   awk -F: '{ print $1}' /etc/passwd
```

4. Service audit:
```bash
   systemctl list-unit-files --type=service
```

5. Security scans (using Lynis):
```bash
   sudo lynis audit system
```

## Conclusion

Implementing these security practices provides a solid foundation for protecting your self-hosted services. Remember that security is an ongoing process:

- Regularly review and update your security measures
- Stay informed about new threats and vulnerabilities
- Continuously educate yourself on best practices

By remaining vigilant and proactive, you can significantly reduce the risk of security incidents in your self-hosted environment.


