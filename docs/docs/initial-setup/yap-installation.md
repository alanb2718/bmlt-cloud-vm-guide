# YAP Installation

This guide covers installing YAP (Yet Another Phonebot) to provide telephone helpline services integrated with your BMLT server.

## What is YAP?

YAP is a telephone helpline system that integrates with BMLT and Twilio to provide:
- **Meeting search by phone** - Callers can find meetings by location
- **Helpline routing** - Route calls to volunteers
- **Conference calling** - Connect callers with support
- **SMS responses** - Send meeting information via text

:::info Requirements
YAP requires a Twilio account for phone services. You'll need Twilio credentials before starting installation.
:::

## Prerequisites

Before installing YAP, ensure you have:
- BMLT Server installed and configured
- MySQL database server running
- Twilio account with phone number
- YAP database and user created (from server setup)

## Download and Install YAP

### Get Latest YAP Release

Execute the following commands, replacing 4.5.0 in the GitHub URL with the version number for the latest stable release (two occurrences). You can find the version number for the latest stable release here: https://github.com/bmlt-enabled/yap/releases/.

```bash
# Navigate to web directory
cd /var/www/your-domain.com
# Or for IP setup: cd /var/www/html

# Download YAP (current stable version)
sudo wget https://github.com/bmlt-enabled/yap/releases/download/4.5.0/yap-4.5.0.zip

# Extract the archive
sudo unzip yap-4.5.0.zip

# Remove the zip file
sudo rm yap-4.5.0.zip

# Rename to standard directory
sudo mv yap-4.5.0 yap

# Set proper ownership
sudo chown -R www-data:www-data yap

# Set proper permissions
sudo chmod -R 755 yap
```

## Configure YAP

### Create Configuration File

YAP configuration is stored in a PHP file:

```bash
# Navigate to YAP directory
cd /var/www/your-domain.com/yap

# Copy example configuration
sudo cp config.php.example config.php

# Edit configuration
sudo nano config.php
```

### Basic YAP Configuration

Edit the configuration file with your settings:

```php
<?php
// Database Configuration
static $mysql_hostname = "localhost";
static $mysql_username = "yap";
static $mysql_password = "your_secure_yap_password";
static $mysql_database = "yap";

// BMLT Integration
static $bmlt_root_server = "https://your-domain.com/main_server";
static $bmlt_username = "your_bmlt_admin_username";
static $bmlt_password = "your_bmlt_admin_password";

// Twilio Configuration
static $twilio_account_sid = "your_twilio_account_sid";
static $twilio_auth_token = "your_twilio_auth_token";
static $caller_id = "+1234567890"; // Your Twilio phone number

// Service Configuration
static $service_body_id = "1"; // Your BMLT service body ID
static $helpline_service_body_id = "1"; // Service body for helpline
static $timezone = "America/New_York"; // Your timezone

// Optional: SMS Configuration
static $sms_enable = true;
static $sms_ask = "Would you like the results sent to you via text message?";

// Optional: Title and Branding
static $title = "Your Area Helpline";
```

### Secure Configuration File

Set proper permissions for security:

```bash
# Secure the configuration file
sudo chmod 600 /var/www/your-domain.com/yap/config.php
sudo chown www-data:www-data /var/www/your-domain.com/yap/config.php
```

## Database Setup

### Initialize YAP Database

YAP will automatically create its database tables on first use. To verify:

```bash
# Connect to MySQL
mysql -u yap -p

# Use YAP database
USE yap;
SHOW TABLES;

# Exit MySQL
EXIT;
```

If you are migrating an existing yap server, copy the `config.php` file from that server, and use this command instead to initialize the database:

```bash
sudo mysql yap < my_yap.sql
```

## Twilio Configuration

### Get Twilio Credentials

1. **Create Twilio Account**: Sign up at [twilio.com](https://www.twilio.com)
2. **Get Account SID**: Found in Twilio Console dashboard
3. **Get Auth Token**: Found in Twilio Console dashboard
4. **Purchase Phone Number**: Buy a phone number for your helpline

### Configure Twilio Webhooks

In your Twilio Console, configure your phone number webhooks:

**Voice Webhook URL:**
```
https://your-domain.com/yap/index.php
```

**SMS Webhook URL (if using SMS):**
```
https://your-domain.com/yap/sms-gateway.php
```

**HTTP Method:** POST

### Test Twilio Connection

Create a test script to verify Twilio integration:

```bash
# Create test file
sudo nano /var/www/your-domain.com/yap/test-twilio.php
```

Add test content:
```php
<?php
require_once 'config.php';
require_once 'vendor/autoload.php';

use Twilio\Rest\Client;

$twilio = new Client(static::$twilio_account_sid, static::$twilio_auth_token);

try {
    $account = $twilio->api->v2010->accounts(static::$twilio_account_sid)->fetch();
    echo "Twilio connection successful!\n";
    echo "Account: " . $account->friendlyName . "\n";
} catch (Exception $e) {
    echo "Twilio connection failed: " . $e->getMessage() . "\n";
}
?>
```

Run the test:
```bash
cd /var/www/your-domain.com/yap
php test-twilio.php
```

## Apache Configuration for YAP

### Virtual Host Update

If using a domain, ensure your virtual host includes the yap directory:

```bash
sudo nano /etc/apache2/sites-available/your-domain.com.conf
```

Your virtual host should already include the DocumentRoot that contains YAP:

```apache
<VirtualHost *:80>
    ServerName your-domain.com
    DocumentRoot /var/www/your-domain.com
    
    <Directory "/var/www/your-domain.com">
        AllowOverride All
        Require all granted
    </Directory>
    
    # YAP should be accessible at /yap/
</VirtualHost>
```

### URL Rewriting for YAP

YAP includes its own `.htaccess` file for URL rewriting. Ensure mod_rewrite is enabled:

```bash
# Enable rewrite module (should already be done)
sudo a2enmod rewrite

# Restart Apache
sudo systemctl restart apache2
```

## Test YAP Installation

### Web Interface Test

Visit YAP in your browser:

**With Domain:**
```
https://your-domain.com/yap/
```

**With IP:**
```
http://your-server-ip/yap/
```

You should see a YAP status page showing configuration details.

### Phone Test

1. **Call your Twilio number**
2. **Listen for YAP greeting**
3. **Try meeting search** by entering a zip code
4. **Verify results** are read back

### SMS Test (if enabled)

1. **Send SMS to your Twilio number**
2. **Send a zip code** (e.g., "10001")
3. **Verify response** with meeting information

## YAP Administration

### Access YAP Admin

YAP includes an administration interface:

```
https://your-domain.com/yap/admin/
```

Log in with BMLT administrator credentials.

### Configure Service Bodies

In YAP admin, you can:
- Set up service body routing
- Configure helpline volunteers
- Set up meeting search parameters
- Configure SMS responses
- View call logs and statistics

### Volunteer Management

Set up volunteers for helpline calls:

1. **Access YAP Admin**
2. **Go to Volunteers section**
3. **Add volunteer phone numbers**
4. **Set availability schedules**
5. **Configure call routing**

## Customization Options

### Custom Greetings

Edit greeting messages in YAP configuration:

```php
// Custom greeting options
static $voice_greeting = "Welcome to the area helpline. Press 1 for meeting information or stay on the line to speak with someone.";
static $meeting_search_greeting = "Please enter your zip code to find meetings near you.";
```

### Meeting Search Options

Configure meeting search behavior:

```php
// Search configuration
static $meeting_search_radius = 25; // miles
static $meeting_search_max_results = 5;
static $include_service_body_names = true;
static $include_meeting_formats = true;
```

### SMS Configuration

Customize SMS responses:

```php
// SMS settings
static $sms_enable = true;
static $sms_ask = "Would you like the results sent to you via text message?";
static $sms_summary_page = true;
static $sms_combine = true;
```

## Monitoring and Logs

### YAP Logs

Monitor YAP activity:

```bash
# View YAP logs
sudo tail -f /var/www/your-domain.com/yap/storage/logs/laravel.log

# Search for errors
sudo grep "ERROR" /var/www/your-domain.com/yap/storage/logs/laravel.log

# Monitor call activity
sudo grep "call" /var/www/your-domain.com/yap/storage/logs/laravel.log
```

### Twilio Logs

Monitor Twilio activity in the Twilio Console:
- Call logs and recordings
- SMS delivery status
- Error messages and debugging
- Usage and billing information

## Troubleshooting

### Common YAP Issues

**YAP not answering calls:**
```bash
# Check Twilio webhook configuration
curl -X POST https://your-domain.com/yap/index.php

# Check Apache error log
sudo tail -f /var/log/apache2/error.log

# Verify YAP configuration
php -l /var/www/your-domain.com/yap/config.php
```

**Database connection errors:**
```bash
# Test database connection
mysql -u yap -p -e "SELECT 1;"

# Check YAP configuration
sudo grep mysql /var/www/your-domain.com/yap/config.php
```

**BMLT integration issues:**
```bash
# Test BMLT connection
curl https://your-domain.com/main_server/client_interface/json/?switcher=GetServerInfo

# Check BMLT credentials in YAP config
sudo grep bmlt /var/www/your-domain.com/yap/config.php
```

### Performance Issues

**Slow response times:**
- Check server resources (CPU, memory)
- Optimize MySQL queries
- Enable PHP OPcache
- Consider CDN for static assets

**Call quality issues:**
- Check network connectivity
- Monitor Twilio status page
- Verify webhook response times
- Check server load during peak times

## Security Considerations

### Secure Configuration

```bash
# Ensure config file is secure
sudo chmod 600 /var/www/your-domain.com/yap/config.php

# Set proper ownership
sudo chown www-data:www-data /var/www/your-domain.com/yap/config.php
```

### Webhook Security

Consider adding Twilio signature validation for enhanced security.

### Volunteer Data Protection

- Limit volunteer information exposure
- Use secure connections (HTTPS) only
- Regularly rotate API credentials
- Monitor access logs

## Backup YAP Configuration

```bash
# Backup YAP database
sudo mysqldump yap > ~/yap-backup-$(date +%Y%m%d).sql

# Backup YAP configuration
sudo cp /var/www/your-domain.com/yap/config.php ~/yap-config-backup.php

# Backup custom files (if any)
sudo tar -czf ~/yap-custom-backup.tar.gz /var/www/your-domain.com/yap/custom/
```

## Next Steps

With YAP installed and configured:

1. **Set up SSL**: [SSL Setup](ssl-setup) for secure HTTPS connections
2. **Configure monitoring**: Set up log monitoring and alerting
3. **Train volunteers**: Provide volunteer training materials
4. **Test thoroughly**: Conduct end-to-end testing
5. **Go live**: Announce your helpline number

## YAP Resources

- **YAP Documentation**: [github.com/bmlt-enabled/yap/wiki](https://github.com/bmlt-enabled/yap/wiki)
- **Twilio Documentation**: [twilio.com/docs](https://www.twilio.com/docs)
- **BMLT Integration**: [bmlt.app](https://bmlt.app)
- **YAP Support**: GitHub issues and community forums

:::tip Testing Recommendation
Always test YAP thoroughly before going live. Make test calls, verify meeting searches, and ensure SMS functionality works as expected.
:::

:::warning Cost Awareness
YAP uses Twilio services which incur costs based on usage. Monitor your Twilio usage and set up billing alerts to avoid unexpected charges.
:::