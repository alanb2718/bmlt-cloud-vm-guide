# Prerequisites

Before setting up your BMLT server, ensure you have the necessary resources and access in place.

## Cloud Server Requirements

### Minimum System Specifications
- **OS**: Ubuntu 24.04 LTS (recommended)
- **RAM**: 2GB minimum, 4GB+ recommended for production
- **CPU**: 1 vCPU minimum, 2vCPUs recommended
- **Storage (SSD) **: 30GB minimum, 60GB recommended
- **Network**: Public IP address

### Supported Cloud Providers
This guide works with any Ubuntu cloud server, including:
- **Digital Ocean** (Droplets)
- **Amazon Web Services** (EC2)
- **Google Cloud Platform** (Compute Engine)
- **Microsoft Azure** (Virtual Machines)
- **Linode** (Linodes)
- **Vultr** (Cloud Compute)

## Access Requirements

### SSH Access
You'll need SSH access to your server:
- SSH key pair (*[Guide for Generating SSH Key pair](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)*)
- Terminal application (Terminal on macOS/Linux, PuTTY on Windows)
- Basic command line familiarity

### Domain Name
A domain name is required for:
- SSL certificate installation
- Professional appearance
- Email functionality (if using YAP)

**Domain Setup Checklist:**
- [ ] Domain purchased and accessible
- [ ] DNS A record pointing to server IP
- [ ] DNS propagation completed (can take 24-48 hours)

**Recommended Providers**
- [Namecheap](https://www.namecheap.com)

## Required Information

Before starting, gather the following information:

### Server Access
- **Server IP Address**: `_____._____._____.____`
- **SSH Username**: Usually `ubuntu` for Ubuntu servers
- **SSH Key Location**: Path to your private key file

### Domain Configuration (if applicable)
- **Domain Name**: `your-domain.com`

### Service Configuration
- **BMLT Admin Email**: For BMLT administrator account
- **BMLT Admin Password**: Choose a strong password
- **Time Zone**: Your local timezone for meeting displays

## Local Tools Setup

### SSH Client
**macOS/Linux:**
SSH is pre-installed. Test with:
```bash
ssh -V
```

**Windows:**
- Use built-in SSH client (Windows 10/11)
- Or install PuTTY from [putty.org](https://www.putty.org/)

### Text Editor (Optional)
For editing configuration files locally:
- **Visual Studio Code** with SSH extensions
- **Sublime Text**
- **Vim** or **Nano** (command line editors)

## Security Considerations

### SSH Key Best Practices
- Use RSA 4096-bit or Ed25519 keys
- Never share private keys
- Use different keys for different servers

### Password Requirements
Plan strong passwords for:
- Server user account (if not using keys)
- MySQL root user
- BMLT admin account
- Database users (bmlt, yap)

**Strong Password Characteristics:**
- At least 12 characters
- Mix of uppercase, lowercase, numbers, symbols
- Unique for each service
- Consider using a password manager

## Network Requirements

### Firewall Ports
Your cloud provider should allow these ports:
- **Port 22**: SSH access
- **Port 80**: HTTP (web traffic)
- **Port 443**: HTTPS (secure web traffic)

### Outbound Internet Access
Your server needs internet access for:
- Package updates and installation
- SSL certificate renewal
- BMLT/YAP downloads
- External API calls (Google Maps, Twilio if using YAP)

## Knowledge Prerequisites

### Basic Linux Skills
You should be comfortable with:
- Connecting to servers via SSH
- Running commands in terminal
- Basic file operations (ls, cd, cp, mv)
- Text editing with nano, vi, or emacs
- Understanding file permissions

### Helpful But Not Required
- Apache web server concepts
- MySQL database basics
- DNS and domain management
- SSL certificate concepts

## Pre-Installation Checklist

Before proceeding to server setup:

- [ ] Cloud server created and accessible
- [ ] SSH access confirmed
- [ ] Domain DNS configured (if using)
- [ ] Strong passwords planned
- [ ] Backup contact information ready
- [ ] Time zone determined
- [ ] This guide bookmarked for reference

## Support and Documentation

Keep these resources handy:
- **Ubuntu Documentation**: [help.ubuntu.com](https://help.ubuntu.com/)
- **Apache Documentation**: [httpd.apache.org/docs](https://httpd.apache.org/docs/)
- **MySQL Documentation**: [dev.mysql.com/doc](https://dev.mysql.com/doc/)
- **BMLT Documentation**: [bmlt.app](https://bmlt.app/)

:::tip
If this is your first time managing a Linux server, consider practicing these commands on a test server first.
:::

:::warning
Always backup important data before making system changes. Cloud providers typically offer snapshot features for easy server backups.
:::

## Next Steps

Once you have all prerequisites in place, proceed to [Server Setup](server-setup) to begin installing the LAMP stack.