# BBBManager

Opensource BigBlueButton room manager

## Setup (Ubuntu 14.04)

### Step 1: Update your system

    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo reboot

### Step 2: Install web server

    sudo apt-get install nginx apache2 php5 php5-mysql php5-ldap php5-gd git lynx-cur wget

### Step 3: Configure apache
Apache and nginx listen in port 80 by default. It would cause a conflict.
Replace the contents of the file `/etc/apache2/ports.conf` to the following lines:

    Listen 127.0.0.1:81
    Listen 127.0.0.1:82
    
### Step 4: Get the source code

    sudo mkdir /var/bbbmanager/
    sudo chown `id -un`.www-data /var/bbbmanager/
    
    cd /var/bbbmanager/
    git clone https://github.com/BBBManager/web-api.git
    git clone https://github.com/BBBManager/web-ui.git

### Step 5 (Optional): Go to a specific version

    cd /var/bbbmanager/web-ui/
    git checkout v1.0
    cd /var/bbbmanager/web-api/
    git checkout v1.0
    
### Step 6: Configure permissions

    sudo chown `id -un`.www-data /var/bbbmanager/web-api/ -R
    sudo chown `id -un`.www-data /var/bbbmanager/web-ui/ -R

    chmod 775 /var/bbbmanager/web-ui/private/tmp/cache/
    chmod 775 /var/bbbmanager/web-ui/private/tmp/session/

    chmod 775 /var/bbbmanager/web-api/private/tmp/cache/
    chmod 775 /var/bbbmanager/web-api/private/tmp/session/

### Step 7: Configure virtualhosts

    sudo cp /var/bbbmanager/web-api/conf-template/apache2/bbbmanager-api.conf /etc/apache2/sites-available/
    sudo cp /var/bbbmanager/web-ui/conf-template/apache2/bbbmanager-ui.conf /etc/apache2/sites-available/
    sudo ln -s /etc/apache2/sites-available/bbbmanager-api.conf /etc/apache2/sites-enabled/ -s
    sudo ln -s /etc/apache2/sites-available/bbbmanager-ui.conf /etc/apache2/sites-enabled/ -s
