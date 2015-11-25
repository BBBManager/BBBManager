# BBBManager

Opensource BigBlueButton room manager

## Setup (Ubuntu 14.04)

### Update your system and install components

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get install nginx apache2 php5 php5-mysql php5-ldap php5-gd git lynx-cur wget
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

    chmod 775 /var/bbbmanager/web-ui/private/tmp/cache/
    chmod 775 /var/bbbmanager/web-ui/private/tmp/session/

    chmod 775 /var/bbbmanager/web-api/private/tmp/cache/
    chmod 775 /var/bbbmanager/web-api/private/tmp/session/

### Configure virtualhosts

    sudo cp /var/bbbmanager/web-api/conf-template/apache2/bbbmanager-api.conf /etc/apache2/sites-available/
    sudo cp /var/bbbmanager/web-ui/conf-template/apache2/bbbmanager-ui.conf /etc/apache2/sites-available/
    sudo ln -s /etc/apache2/sites-available/bbbmanager-api.conf /etc/apache2/sites-enabled/ -s
    sudo ln -s /etc/apache2/sites-available/bbbmanager-ui.conf /etc/apache2/sites-enabled/ -s

### Install and load initial database

The following command will install MySQL server and will ask a root password for you. On this example we will use password rootbbb

    sudo apt-get install mysql-server-5.6 mysql-client-core-5.6
    
Verify if server is running using the following command

    echo "select 1+1" | mysql -uroot -prootbbb

The output must be similar to:

    Warning: Using a password on the command line interface can be insecure.
    1+1
    2

In this example we are using user bbbmanager and password bbbmanagerpwd. You should use a different password in production.

Create a user and database for bbbmanager on mysql:

    echo "create user bbbmanager identified by 'bbbmanagerpwd';" | mysql -uroot -prootbbb
    echo "create database bbbmanager;" | mysql -uroot -prootbbb
    echo "grant all privileges on bbbmanager.* to bbbmanager;" | mysql -uroot -prootbbb
    echo "flush privileges;" | mysql -uroot -prootbbb

Create schema:

    cat /var/bbbmanager/web-api/conf-template/db/schema.sql | mysql -ubbbmanager -pbbbmanagerpwd bbbmanager

Load default data (user admin, pass bbbmanager):

    cat /var/bbbmanager/web-api/conf-template/db/data.sql | mysql -ubbbmanager -pbbbmanagerpwd bbbmanager

### Create PHP config files (API)
    cd /var/bbbmanager/web-api/
    rsync -av conf-template/php/ private/application/configs/
    
If you used a different user and password for database, you must to change it in file private/application/configs/application.ini

### Create PHP config files (UI)
    echo "10.30.10.101" > /var/bbbmanager/parameters/external_hostname
    
    cd /var/bbbmanager/web-ui/
    cat conf-template/php/application.ini  | sed 's/<<BBBMANAGER_HOSTNAME>>/'`cat /var/bbbmanager/parameters/external_hostname | head -n 1 | xargs -n 1 echo -n `'/g' > private/application/configs/application.ini

### Configure nginx to route traffic to BBBManager internal ports (81 and 82)
    