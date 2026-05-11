# Server Setup

This page covers the complete installation and configuration of the LAMP (Linux, Apache, MySQL, PHP) stack on your Ubuntu server.

## Connecting to Your Server

### SSH Connection
Connect to your server using SSH (this assumes your username is `ubuntu` -- change as needed):

```bash
# Using SSH key (recommended)
ssh -i ~/.ssh/your-key ubuntu@your-server-ip

# Using password (if keys not configured)
ssh ubuntu@your-server-ip
```

### First Time Connection
On first connection, you may see a security warning. Type `yes` to continue:
```
The authenticity of host 'your-server-ip' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
```

## System Updates

Always start with updating your system:

```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y

# Reboot if kernel updates were installed
sudo reboot
```

After reboot, reconnect to your server.

## Install Required Packages

### Core LAMP Stack Components

Install Apache web server:
```bash
sudo apt install apache2 -y
```

Install MySQL database server:
```bash
sudo apt install mysql-server mysql-client -y
```

Install PHP and required extensions:
```bash
sudo apt install php php-mysql php-curl php-gd  php-intl php-zip php-mbstring php-xml libapache2-mod-php -y
```

### Additional Useful Packages

Install supporting tools:
```bash
sudo apt install software-properties-common unzip htop python3-pip -y
```

Install Certbot for SSL certificates:
```bash
sudo apt install certbot python3-certbot-apache -y
```

## Verify Installations

### Check Service Status
Verify that services are running:

```bash
# Check Apache status
sudo systemctl status apache2

# Check MySQL status
sudo systemctl status mysql

# Check PHP version
php --version
```

### Test Web Server
Visit your server's IP address in a web browser to see the Apache default page:
```
http://your-server-ip
```

You should see the "Apache2 Ubuntu Default Page".

## Configure Services

### Enable Services at Boot
Ensure services start automatically on server restart:

```bash
sudo systemctl enable apache2
sudo systemctl enable mysql
```

### Configure Apache Modules
Enable required Apache modules:

```bash
# Enable URL rewriting (required for BMLT)
sudo a2enmod rewrite

# Enable expires headers (improves performance)
sudo a2enmod expires

# Restart Apache to apply changes
sudo systemctl restart apache2
```

### Test Apache Configuration
Verify Apache configuration is valid:

```bash
sudo apache2ctl configtest
```

You should see: `Syntax OK`

## Configure Firewall

### Option 1: Cloud Provider Security Groups (Recommended)
Most cloud providers use security groups instead of local firewalls. Configure these in your cloud provider's dashboard:

- **SSH (Port 22)**: Allow from your IP address
- **HTTP (Port 80)**: Allow from anywhere (0.0.0.0/0)
- **HTTPS (Port 443)**: Allow from anywhere (0.0.0.0/0)

### Option 2: Local UFW Firewall
If your cloud provider doesn't use security groups:

```bash
# Enable UFW firewall
sudo ufw enable

# Allow SSH (replace with your IP for better security)
sudo ufw allow ssh

# Allow HTTP and HTTPS
sudo ufw allow 'Apache Full'

# Check firewall status
sudo ufw status
```

### Option 3: Disable Local Firewall (Cloud Security Groups Only)
If using cloud provider security groups exclusively:

```bash
# Disable UFW
sudo ufw disable

# Configure iptables rules for HTTP/HTTPS (if needed)
sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
```









## Secure MySQL Installation

Run the MySQL security script to improve database security:

```bash
sudo mysql_secure_installation
```

Follow these recommendations:
1. **Password validation component**: `Y` (choose MEDIUM or STRONG policy)
2. **Root password**: Modern MySQL uses `auth_socket` authentication by default, so no password is set. Root access requires `sudo mysql` instead (more secure)
3. **Remove anonymous users**: `Y`
4. **Disallow root login remotely**: `Y`
5. **Remove test database**: `Y` (or `N` if needed for testing)
6. **Reload privilege tables**: `Y`

### Accessing MySQL as Root

With `auth_socket` authentication (default), connect using:

```bash
sudo mysql
```

No password is required. This is more secure as it ties MySQL root access to system-level authentication.

## Create MySQL Databases and Users

### Connect to MySQL
```bash
sudo mysql
```

### Create Databases and Users
```sql
-- Create BMLT server database and user
CREATE DATABASE rootserver;
CREATE USER 'bmlt'@'localhost' IDENTIFIED BY 'your_secure_bmlt_password';
GRANT ALL PRIVILEGES ON rootserver.* TO 'bmlt'@'localhost';

-- Create YAP database and user (if planning to use YAP)
CREATE DATABASE yap;
CREATE USER 'yap'@'localhost' IDENTIFIED BY 'your_secure_yap_password';
GRANT ALL PRIVILEGES ON yap.* TO 'yap'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

-- Exit MySQL
EXIT;
```

:::warning Password Security
Replace `your_secure_bmlt_password` and `your_secure_yap_password` with strong, unique passwords. Save these passwords securely as you'll need them during application installation.
:::

## Test Database Connection

Verify database users can connect:

```bash
# Test BMLT user connection
mysql -u bmlt -p

# If successful, you'll see MySQL prompt. Type SHOW DATABASES; and verify that you can see the rootserver and yap databases.  Then type EXIT; to quit.
```

## Configure PHP (Optional Optimizations)

### Check Current PHP Configuration
```bash
# View PHP configuration file location
php --ini

# Check important PHP settings
php -m | grep -E "(mysql|curl|gd|zip|mbstring|xml)"
```

### Optimize PHP Settings (Optional)
Edit PHP configuration for better performance:

```bash
sudo nano /etc/php/8.3/apache2/php.ini
```

Key settings to consider:
```ini
# Increase memory limit
memory_limit = 256M

# Increase maximum execution time
max_execution_time = 300

# Set timezone (replace with your timezone)
date.timezone = America/New_York
```

Restart Apache after changes:
```bash
sudo systemctl restart apache2
```

## Set Up Web Directory Structure

### Using Domain Name (Recommended)
If you have a domain name:

```bash
# Create directory for your domain
sudo mkdir -p /var/www/your-domain.com

# Set proper ownership
sudo chown -R www-data:www-data /var/www/your-domain.com

# Set proper permissions
sudo chmod -R 755 /var/www/your-domain.com
```

## Verification Checklist

Before proceeding to application installation:

- [ ] All packages installed successfully
- [ ] Apache serving default page
- [ ] MySQL secured and databases created
- [ ] PHP working with required extensions
- [ ] Firewall configured properly
- [ ] Services enabled for auto-start

## System Information

Document your setup for future reference:

```bash
# Check system information
uname -a
lsb_release -a

# Check service versions
apache2 -v
mysql --version
php --version

# Check disk space
df -h

# Check memory
free -h
```

## Troubleshooting

### Common Issues

**Apache won't start:**
```bash
# Check Apache error log
sudo tail -f /var/log/apache2/error.log

# Check configuration
sudo apache2ctl configtest
```

**MySQL connection issues:**
```bash
# Check MySQL status
sudo systemctl status mysql

# Check MySQL error log
sudo tail -f /var/log/mysql/error.log
```

**PHP not working:**
```bash
# Verify PHP module is loaded
apache2ctl -M | grep php

# Check PHP error log
sudo tail -f /var/log/apache2/error.log
```

### Performance Monitoring

```bash
# Monitor system resources
htop

# Check disk usage
df -h

# Check memory usage
free -h

# Monitor Apache processes
sudo systemctl status apache2
```

## Next Steps

With your LAMP stack configured, you can now proceed to:
1. [BMLT Installation](bmlt-installation) - Install the BMLT Server
2. [YAP Installation](yap-installation) - Install YAP (if needed)
3. [SSL Setup](ssl-setup) - Configure HTTPS certificates

:::tip Performance Tip
Consider creating a server snapshot/backup at this point. This gives you a clean LAMP stack baseline to restore if needed.
:::

:::info
Your server is now ready for application installation. The next steps will install and configure BMLT and YAP applications on this foundation.
:::