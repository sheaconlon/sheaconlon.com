# Deployment Instructions
If an option is not specified, then accept the default.
1. Go to your AWS Management Console.
2. Switch to region `us-west (N. California)`.
3. Go to AWS EC2.
4. Create a VPC.
    1. Name it `sheaconlon.com-vpc`.
    2. Choose CIDR block `10.0.0.0/16`
    3. Set `DNS hostnames` to `yes`.
5. Create a subnet.
    1. Name it `sheaconlon.com-only-subnet`.
    2. Put it in `sheaconlon.com-vpc`.
    3. Choose the CIDR block `10.0.0.0/16`.
6. Create a security group.
    1. Name it `sheaconlon.com-security-group`.
    2. Allow SSH, HTTP, and HTTPS from anywhere.
7. Launch an instance.
    1. Choose the image `Ubuntu Server 16.04 LTS`.
    2. Choose the instance type `t2.nano`.
    3. Choose as the network `sheaconlon.com-vpc`.
    4. Choose as the subnet `sheaconlon.com-only-subnet`.
    5. Enable termination protection.
    6. Enable detailed monitoring.
    7. Add tag with key `Name` and value `sheaconlon.com-instance`.
    8. Give it security group `sheaconlon.com-security-group`.
    9. Create a new key pair named `sheaconlon.com-keypair`. Do not lose or release the key file. Put it in `~/.ssh` locally.
8. Go to the network interfaces pane of the EC2 console.
    1. Edit the name of the primary network interface of `sheaconlon.com-instance` to be `sheaconlon.com-interface`.
9. Go to the internet gateway pane of the VPC console.
    1. Create an internet gateway named `sheaconlon.com-gateway`.
    2. Attach it to `sheaconlon.com-vpc`.
10. Go to the elastic IP pane of the VPC console.
    1. Allocate an elastic IP named `sheaconlon.com-ip`.
    2. Associate it with `sheaconlon.com-interface`.
    3. Retain the public IP address.
11. Go to the route tables pane of the VPC console.
    1. Edit the route table associated with `sheaconlon.com-vpc`.
    2. Add a rule routing `0.0.0.0/0` to `sheaconlon.com-gateway`.
12. Go to Route 53.
13. Create a hosted zone.
    1. Enter your domain name.
    2. Create a record set.
          1. Leave the subdomain blank
          2. Set the type to `A`.
          3. Set the value to the public IP address of `sheaconlon.com-ip`.
    3. Create another record set.
          1. Set the subdomain to `www`.
          2. Set the type to `CNAME`.
          3. Set the value to your domain name.
    4. Create another record set.
          1. Set the subdomain to `dev`.
          2. Set the type to `CNAME`.
          3. Set the value to your domain name.
    5. Create another record set.
          1. Set the subdomain to `www.dev`.
          2. Set the type to `CNAME`.
          3. Set the value to your domain name.
    6. Go to your domain name provider and set the nameservers to the lines of the `NS` record.
14. Go to the SNS console.
15. Switch your region to `us-east (N Virginia)`.
16. Create a topic.
    1. Name it `sheaconlon-com-topic`.
    2. Give it display name `Site Stat`.
    3. Create a subscription to this topic.
        1. Choose protocol SMS.
        2. Enter your phone number.
    4. Create another subscription to this topic.
        1. Choose protocol Email.
        2. Enter your email address.
        3. Check your email for a message requesting confirmation.
17. Go to the health check pane of Route 53.
    1. Create a new health check.
    2. Name it `sheaconlon.com-check`.
    3. Choose endpoint as what to monitor.
    4. Specify endpoint by domain name.
    5. Set the domain name to your domain name.
    6. Set the path to `/health-check`.
    7. Check for box for latency graphs.
18. Switch your region to `us-east (N Virginia)`.
19. Go to the alarms pane of Cloudwatch.
    1. Create an alarm using the metric `HealthCheckStatus` of `sheaconlon.com-check`.
    2. Name the alarm `sheaconlon.com-up-alarm`.
    3. Change the trigger to `< 1`.
    4. Change the number of periods to `1`.
    5. Treat missing data as `bad (breaching threshold)`.
    6. Set the period to `1 minute`.
    7. Set the statistic to `Minimum`.
20. Connect to the server using SSH (`ssh ubuntu@<yourdomainhere>` on macOS). If connecting using your domain name doesn't work, you can reset your DNS cache (`sudo killall -HUP mDNSResponder` on the most recent version of Mac OS) or connect directly to the public address of `sheaconlon.com-ip`.
    1. Run `sudo apt-get update`.
    2. Run `sudo apt-get install apache2`.
    3. Navigate to your domain name in a browser to check that you get the Apache2 Ubuntu Default Page.
    4. Run `sudo a2enmod rewrite`.
    5. Run `sudo service apache2 restart`.
    6. Edit `/etc/apache2/apache2.conf`.
        1. Change `Timeout 300` to `Timeout 30`.
        2. Change all instances of `Options... Indexes`, `Options... FollowSymLinks`, `Options... MultiViews` by adding dashes in front of the option names.
    7. Edit `/etc/apache2/sites-available/sheaconloncom.conf` to be
        <VirtualHost *:80>
                # The files for this site are in /var/sheaconloncom.
                DocumentRoot /var/sheaconloncom
                # This configuration serves sheaconlon.com.
                ServerName sheaconlon.com
                # Serve www.sheaconlon.com with this configuration as well.
                ServerAlias www.sheaconlon.com
                <Directory /var/sheaconloncom/>
                        # Disallow .htaccess files.
                        AllowOverride None
                        # Allow serving from this directory.
                        Require all granted
                        Order allow,deny
                        Allow from all
                        # If the server gets a request for <somepath>/<somedirectory>, then it should serve <somepath>/<somedirectory>/index.html.
                        DirectoryIndex index.html
                        # Allow us to rewrite requests.
                        RewriteEngine On
                        Options FollowSymLinks
                        # If the server gets a request for www.<somedomain>/<somepath>, then it should respond with 301 Permanently Moved and a redirection to <somedomain>/<somepath>.
                        RewriteBase /
                        RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
                        RewriteRule ^(.*)$ http://%1/$1 [R=301,L]
                        # If the server gets a request for <somepath>/something and <somepath>/something.html exists, then it should serve that.
                        RewriteCond %{REQUEST_FILENAME}.html -f
                        RewriteRule !.*\.html$ %{REQUEST_FILENAME}.html [L]
                </Directory>
        </VirtualHost>
    8. Run `sudo a2ensite sheaconloncom`.
    9. Edit `/etc/apache2/sites-available/devsheaconloncom.conf` to be
        <VirtualHost *:80>
                # The files for this site are in /var/devsheaconloncom.
                DocumentRoot /var/devsheaconloncom
                # This configuration serves dev.sheaconlon.com.
                ServerName dev.sheaconlon.com
                # Serve www.dev.sheaconlon.com with this configuration as well.
                ServerAlias www.dev.sheaconlon.com
                <Directory /var/devsheaconloncom/>
                        # Disallow .htaccess files.
                        AllowOverride None
                        # Allow serving from this directory.
                        Require all granted
                        Order allow,deny
                        Allow from all
                        # If the server gets a request for <somepath>/<somedirectory>, then it should serve <somepath>/<somedirectory>/index.html.
                        DirectoryIndex index.html
                        # Allow us to rewrite requests.
                        RewriteEngine On
                        Options FollowSymLinks
                        # If the server gets a request for www.<somedomain>/<somepath>, then it should respond with 301 Permanently Moved and a redirection to <somedomain>/<somepath>.
                        RewriteBase /
                        RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
                        RewriteRule ^(.*)$ http://%1/$1 [R=301,L]
                        # If the server gets a request for <somepath>/something and <somepath>/something.html exists, then it should serve that.
                        RewriteCond %{REQUEST_FILENAME}.html -f
                        RewriteRule !.*\.html$ %{REQUEST_FILENAME}.html [L]
                </Directory>
        </VirtualHost>
    10. Run `sudo a2ensite devsheaconloncom`.
    11. Run `sudo service apache2 reload`.
    13. Run `sudo mkdir /var/sheaconloncom/`.
    14. Run `cd /var/sheaconloncom/`.
    15. Run `sudo git init`.
    16. Run `sudo git remote add origin https://github.com/sheaconlon/sheaconlon.com.git`.
    17. Run `sudo git pull origin master`.
    18. Run `sudo mkdir /var/devsheaconloncom/`.
    19. Run `cd /var/devsheaconloncom/`.
    20. Run `sudo git init`.
    21. Run `sudo git remote add origin https://github.com/sheaconlon/sheaconlon.com.git`.
    22. Run `sudo git pull origin dev`.