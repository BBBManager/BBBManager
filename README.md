BBBManager
==========

Opensource BigBlueButton room manager

Setup (Ubuntu 14.04)

Step 1: Update your system
        sudo apt-get update
        sudo apt-get dist-upgrade
        sudo reboot

Step 2: Install web server
        cd /var/www/
        sudo apt-get install nginx apache2 php5 php5-mysql php5-ldap php5-gd git lynx-cur wget

Step 3: Configure apache
        Apache and nginx listen in port 80 by default. It would cause a conflict.
    
        Replace the contents of the file /etc/apache2/ports.conf to the following lines:
            Listen 127.0.0.1:81
            Listen 127.0.0.1:82
    
Step 4: Get the source code
        cd /var/www/
        sudo git clone https://github.com/BBBManager/web-api.git
        sudo git clone https://github.com/BBBManager/web-ui.git

Step 5 (Optional): Go to a specific version
        cd /var/www/bbbmanager-ui/
        sudo git checkout v1.0

        cd /var/www/bbbmanager-api/
        sudo git checkout v1.0
    
Step 6: Configure permissions
        sudo chown `id -un`.www-data /var/www/web-api/ -R
        sudo chown `id -un`.www-data /var/www/web-ui/ -R
        chmod 775 /var/www/web-ui/private/tmp/cache/
        chmod 775 /var/www/web-ui/private/tmp/session/
        chmod 775 /var/www/web-api/private/tmp/cache/
        chmod 775 /var/www/web-api/private/tmp/session/

Step 7: Configure virtualhosts
        sudo cp /var/www/web-api/conf-template/apache2/bbbmanager-api.conf /etc/apache2/sites-available/
        sudo cp /var/www/web-ui/conf-template/apache2/bbbmanager-ui.conf /etc/apache2/sites-available/
        sudo ln -s /etc/apache2/sites-available/bbbmanager-api.conf /etc/apache2/sites-enabled/ -s
        sudo ln -s /etc/apache2/sites-available/bbbmanager-ui.conf /etc/apache2/sites-enabled/ -s
