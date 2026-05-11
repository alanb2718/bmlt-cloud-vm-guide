# BMLT Installation

This page covers installing the BMLT Server on your configured LAMP stack.

## Download and Extract the BMLT Server Zip File

Execute the following commands, replacing 4.0.2 in the GitHub URL with the version number for the latest stable release. You can find the version number for the latest stable release here: https://github.com/bmlt-enabled/bmlt-server/releases/.

```bash
# Navigate to web directory
cd /var/www/your-domain.com

# Download BMLT Server (current stable version)
sudo wget https://github.com/bmlt-enabled/bmlt-server/releases/download/4.0.2/bmlt-server.zip

# Extract the archive
sudo unzip bmlt-server.zip

# Remove the zip file
sudo rm bmlt-server.zip

# Set proper ownership
sudo chown -R www-data:www-data main_server

# Set proper permissions
sudo chmod -R 755 main_server
```

## Download and Update the auto-config File

```bash
# Navigate to web directory
cd /var/www/your-domain.com

# Download the file initial-auto-config.txt
sudo wget https://github.com/bmlt-enabled/bmlt-server/blob/main/installation/initial-auto-config.txt

# rename it and set proper ownership and permissions
sudo mv initial-auto-config.txt auto-config.inc.php
sudo chown www-data:www-data auto-config.inc.php
sudo chmod 0644 auto-config.inc.php
```

The permissions say that the owner of the file can read and write it, and the owning group and others can read it. Note that the file `auto-config.inc.php` is not inside `main_server`, but rather at the same level. This is a little weird, but does have the advantage that you can upload a new version of the server easily without touching the `auto-config.inc.php` file.  So your directory structure should look something like this:
```
public_html
   auto-config.inc.php
   main_server
      app
      bootstrap
      ......
```

Now edit the `auto-config.inc.php` file using nano or another editor, updating parameter values as needed. There are two parameter values you definitely need to update, namely for `$dbUser` and `$dbPassword` (the user and password for your server database). You also need to either update the parameter value for `$gkey` if you are using Google Maps, or else delete this parameter altogether if you are using OSM (Open Street Maps) for maps and nominatim for geocoding.

There are various other parameters in the file, but the default values may well be what you want.

## Configure Virtual Host (Recommended)

If using a domain name, create an Apache virtual host:

### Create Virtual Host Configuration
```bash
sudo nano /etc/apache2/sites-available/your-domain.com.conf
```

Add this configuration:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@your-domain.com
    ServerName your-domain.com
    DocumentRoot /var/www/your-domain.com
    
    ErrorLog ${APACHE_LOG_DIR}/your-domain.com_error.log
    CustomLog ${APACHE_LOG_DIR}/your-domain.com_access.log combined
    
    <Directory "/var/www/your-domain.com">
        AllowOverride All
        Require all granted
    </Directory>
    
    # Optional: Redirect to HTTPS (after SSL setup)
    # RewriteEngine on
    # RewriteCond %{SERVER_NAME} =your-domain.com
    # RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

### Enable the Site
```bash
# Enable your site
sudo a2ensite your-domain.com.conf

# Disable default site (optional)
sudo a2dissite 000-default.conf

# Test configuration
sudo apache2ctl configtest

# Reload Apache
sudo systemctl reload apache2
```

## Set File Permissions
Ensure proper security permissions:

```bash
# Set ownership
sudo chown -R www-data:www-data /var/www/your-domain.com/main_server

# Set directory permissions
sudo find /var/www/your-domain.com/main_server -type d -exec chmod 755 {} \;

# Set file permissions
sudo find /var/www/your-domain.com/main_server -type f -exec chmod 644 {} \;

# Secure configuration file
sudo chmod 600 /var/www/your-domain.com/auto-config.inc.php
```

## Setting Up the BMLT Database

The next steps vary depending on whether you are importing an existing BMLT database or initializing a fresh one.

**Note:** the directions in this section assume you haven't set up SSL yet, and so the URLs start with `http`.  After you set up SSL (which is described in a later section), use `https` instead. If you are feeling cautious, after you've set up SSL, change any passwwords that were transmitted using `http`.  Or - probably better - set up SSL first before logging in to BMLT.

### Importing an Existing BMLT Database

Download a mysql dump of your existing database to your server.  Run the following command to import it, replacing `mydata.sql` with the name of the sql dump file on your server:
```bash
sudo mysql rootserver < mydata.sql
```
Verify that the user `bmlt` can connect to the database and that it was imported correctly:

```bash
mysql -u bmlt -p

# List tables (should show BMLT tables)
USE rootserver;
SHOW TABLES;
EXIT;
```
You should now be able to visit the login page for your BMLT installation and log in as usual with your current credentials at this URL: `http://your-domain.com/main_server/`. (See the note at the beginning of this section regarding SSL and `http` versus `https`.)

### Initializing a Fresh BMLT Database

Instead, you can start with an empty BMLT database.  The database migrations in the server code will set up the needed tables.

Log in at `http://your-domain.com/main_server/` as user `serveradmin` password `change-this-password-first-thing`. As the initial password suggests (not very subtly), first go to the Account tab and change the password to something unique for your BMLT server.

Verify that the user `bmlt` can connect to the database and that it was initialized correctly by the migrations:

```bash
mysql -u bmlt -p

# List tables (should show BMLT tables)
USE rootserver;
SHOW TABLES;
EXIT;
```

At this point you can set up one or more Service Body Administrators and Service Bodies, and start adding meetings. We are now back to steps that are unchanged from the old tutorial for shared webhosting installations, so refer to that for details.

## Troubleshooting

### Common Installation Issues

**Database Connection Failed:**
```bash
# Verify database user
mysql -u bmlt -p

# Check database exists
mysql -u root -p
SHOW DATABASES;
```

**Permission Errors:**
```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/your-domain.com/main_server

# Check Apache error log
sudo tail -f /var/log/apache2/error.log
```

**White Screen/PHP Errors:**
```bash
# Check PHP error log
sudo tail -f /var/log/apache2/error.log

# Verify PHP extensions
php -m | grep -E "(mysql|curl|gd|zip|mbstring|xml)"
```

### Apache Configuration Issues

**Site Not Loading:**
```bash
# Check virtual host configuration
sudo apache2ctl -S

# Test configuration
sudo apache2ctl configtest

# Check if site is enabled
sudo a2ensite your-domain.com.conf
sudo systemctl reload apache2
```

### Performance Optimization

**Enable PHP OPcache (Optional):**
```bash
# Install OPcache
sudo apt install php-opcache -y

# Restart Apache
sudo systemctl restart apache2
```

## Security Hardening

### Hide PHP Version
Edit PHP configuration:
```bash
sudo nano /etc/php/8.1/apache2/php.ini
```

Set:
```ini
expose_php = Off
```

### Secure Auto-Config File
```bash
# Ensure only web server can read config
sudo chmod 600 /var/www/your-domain.com/auto-config.inc.php
sudo chown www-data:www-data /var/www/your-domain.com/auto-config.inc.php
```

### Add Security Headers
Create `.htaccess` file in BMLT directory:
```bash
sudo nano /var/www/your-domain.com/main_server/.htaccess
```

Add security headers:
```apache
# Security headers
Header always set X-Content-Type-Options nosniff
Header always set X-Frame-Options DENY
Header always set X-XSS-Protection "1; mode=block"

# Hide server information
ServerTokens Prod
```

## Backup Configuration

Create initial backup after successful installation:

```bash
# Backup database
sudo mysqldump bmlt > ~/bmlt-initial-backup.sql

# Backup configuration file
sudo cp /var/www/your-domain.com/auto-config.inc.php ~/auto-config-backup.php
```

## Next Steps

With BMLT successfully installed:

1. **Configure SSL**: [SSL Setup](ssl-setup) for HTTPS access
2. **Install YAP**: [YAP Installation](yap-installation) if you need phone services
3. **Set up backups**: Configure automated backups

## BMLT Resources

- **User Manual**: [bmlt.app/documentation](https://bmlt.app/documentation/)
- **Administrator Guide**: Available in BMLT admin panel
- **GitHub Repository**: [github.com/bmlt-enabled/bmlt-server](https://github.com/bmlt-enabled/bmlt-server)
- **Support Forums**: Community support available

:::tip
Document your admin credentials and database passwords in a secure location. You'll need them for maintenance and troubleshooting.
:::

:::warning
Always test BMLT functionality after installation. Verify meeting searches, admin access, and any integrations work properly before going live.
:::