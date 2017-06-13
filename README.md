# BBBManager

Opensource BigBlueButton room manager

## Setup (Ubuntu 16.04)

### Update your system and install components

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get install nginx apache2 php php-mysql php-ldap php-gd git lynx-cur wget php-curl
    sudo reboot

### Configure apache

Apache and nginx listen in port 80 by default. It would cause a conflict.
Replace the contents of the file `/etc/apache2/ports.conf` to the following lines:

    Listen 127.0.0.1:81
    Listen 127.0.0.1:82

Enable mod_rewrite

    sudo ln /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/ -s

### Restart apache and nginx

    sudo service nginx restart
    sudo service apache2 restart

    
### Get the source code

    sudo mkdir /var/bbbmanager/
    sudo mkdir /var/bbbmanager/parameters/
    sudo chown `id -un`.www-data /var/bbbmanager/ -R
    
    cd /var/bbbmanager/
    git clone https://github.com/BBBManager/web-api.git
    git clone https://github.com/BBBManager/web-ui.git

### (Optional): Go to a specific version

    cd /var/bbbmanager/web-ui/
    git checkout v1.0
    cd /var/bbbmanager/web-api/
    git checkout v1.0
    
### Configure permissions

    sudo chown `id -un`.www-data /var/bbbmanager/web-api/ -R
    sudo chown `id -un`.www-data /var/bbbmanager/web-ui/ -R

    chmod 775 /var/bbbmanager/web-ui/private/tmp/cache/ -R
    chmod 775 /var/bbbmanager/web-ui/private/tmp/session/ -R
    chmod 775 /var/bbbmanager/web-ui/private/tmp/log/ -R
    chmod 775 /var/bbbmanager/web-ui/httpdocs/tmp/ -R
    chmod 775 /var/bbbmanager/web-ui/httpdocs/tmp/captcha/ -R
    
    chmod 775 /var/bbbmanager/web-api/private/tmp/cache/ -R
    chmod 775 /var/bbbmanager/web-api/private/tmp/session/ -R
    chmod 775 /var/bbbmanager/web-api/httpdocs/public/ -R
    chmod 775 /var/bbbmanager/web-api/httpdocs/public/export/ -R
    chmod 775 /var/bbbmanager/web-api/httpdocs/public/export/rooms-audience/ -R


### Configure virtualhosts

    sudo cp /var/bbbmanager/web-api/conf-template/apache2/bbbmanager-api.conf /etc/apache2/sites-available/
    sudo cp /var/bbbmanager/web-ui/conf-template/apache2/bbbmanager-ui.conf /etc/apache2/sites-available/
    sudo ln /etc/apache2/sites-available/bbbmanager-api.conf /etc/apache2/sites-enabled/ -s
    sudo ln /etc/apache2/sites-available/bbbmanager-ui.conf /etc/apache2/sites-enabled/ -s
    sudo service apache2 restart

### Install and load initial database

The following command will install MySQL server and will ask a root password for you. On this example we will use password rootbbb

    sudo apt-get install mysql-server mysql-client
    
Verify if server is running using the following command

    echo "select 1+1" | mysql -uroot -prootbbb

The output must be similar to:

    Warning: Using a password on the command line interface can be insecure.
    1+1
    2

In this example we are using user bbbmanager and password bbbmanagerpwd. You should use a different password in production.

Create a user and database for bbbmanager on mysql:

    echo "create user bbbmanager identified by 'bbbmanagerpwd';" | mysql -uroot -prootbbb
    echo "create database bbbmanager DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;" | mysql -uroot -prootbbb
    echo "grant all privileges on bbbmanager.* to bbbmanager;" | mysql -uroot -prootbbb
    echo "flush privileges;" | mysql -uroot -prootbbb

Create schema:

    cat /var/bbbmanager/web-api/conf-template/db/schema.sql | mysql -ubbbmanager -pbbbmanagerpwd bbbmanager

Load default data (user admin, pass bbbmanager):

    cat /var/bbbmanager/web-api/conf-template/db/data.sql | mysql -ubbbmanager -pbbbmanagerpwd bbbmanager

### Configure your external host name
    echo "192.168.0.5" > /var/bbbmanager/parameters/external_hostname

*Replace 192.168.0.5 with your external ip / hostname*

### Create PHP config files (API)
    cd /var/bbbmanager/web-api/
    rsync -av conf-template/php/ private/application/configs/
    cat conf-template/php/application.ini  | sed 's/<<BBBMANAGER_HOSTNAME>>/'`cat /var/bbbmanager/parameters/external_hostname | head -n 1 | xargs -n 1 echo -n `'/g' > private/application/configs/application.ini

If you used a different user and password for database, you must to change it in file private/application/configs/application.ini

### Create PHP config files (UI), 
    cd /var/bbbmanager/web-ui/
    rsync -av conf-template/php/ private/application/configs/
    cat conf-template/php/application.ini  | sed 's/<<BBBMANAGER_HOSTNAME>>/'`cat /var/bbbmanager/parameters/external_hostname | head -n 1 | xargs -n 1 echo -n `'/g' > private/application/configs/application.ini


### Configure nginx to route traffic to BBBManager - Option A (Without BigBlueButton)
    sudo rm /etc/nginx/sites-enabled/default
    sudo cp /var/bbbmanager/web-ui/conf-template/nginx/bbbmanager-web-ui.alone /etc/nginx/sites-available/
    sudo ln /etc/nginx/sites-available/bbbmanager-web-ui.alone /etc/nginx/sites-enabled/ -s
    sudo service nginx restart

Now you can open your browser in the external hostname (in this example 192.168.0.5)

    http://192.168.0.5/
    User: admin
    Password: bbbmanager

### Configure nginx to route traffic to BBBManager - Option B (With BigBlueButton)
If you executed the previous step, you need to remove the following files:

    sudo rm -f /etc/nginx/sites-enabled/bbbmanager-web-ui.alone
    sudo rm -f /etc/nginx/sites-available/bbbmanager-web-ui.alone

Install the BigBlueButton in your server: http://docs.bigbluebutton.org/1.0/10install.html#installing-bigbluebutton-1-0-beta

Generate a key for the BBBManager Agent:

    uuidgen | sudo tee /var/bbbmanager/parameters/agent_key

Edit file /etc/nginx/sites-available/bigbluebutton and replace:

Replace the following text (original):

    # BigBlueButton landing page.
    location / {
      root   /var/www/bigbluebutton-default;
      index  index.html index.htm;
      expires 1m;
    }

by the following text (commented):

    # BigBlueButton landing page.
    #location / {
    #  root   /var/www/bigbluebutton-default;
    #  index  index.html index.htm;
    #  expires 1m;
    #}

Execute the following command to expose BBBManager in nginx:

    sudo cp /var/bbbmanager/web-ui/conf-template/nginx/bbbmanager-web-ui.nginx /etc/bigbluebutton/nginx/

Execute the following command to download the BBBManager agent:
    
    `sudo wget "https://github.com/BBBManager/standalone-agent/raw/master/dist/release/bbbmanager-standalone-agent.war" -O /var/lib/tomcat7/webapps/bbbmanager-standalone-agent.war`
    
Do a clean restart of BBB:

    sudo bbb-conf --clean



## Upgrade to new version

### How to upgrade
    TODO: describe how to upgrade

### After you upgrade
After you update, you need to clear all sessions and cache, to avoid inconsistency, using the following commands:

    rm -f /var/bbbmanager/web-ui/private/tmp/session/sess_*
    rm -f /var/bbbmanager/web-api/private/tmp/session/sess_*
    rm -f /var/bbbmanager/web-ui/private/tmp/cache/*
    rm -f /var/bbbmanager/web-api/private/tmp/cache/*

