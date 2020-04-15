
### 1. Download and install XAMPP
    $ cd /opt
    $ sudo wget https://www.apachefriends.org/xampp-files/7.2.29/xampp-linux-x64-7.2.29-0-installer.run -P /opt
    $ sudo chmod 755 xampp-linux-x64-7.2.29-0-installer.run
    $ sudo ./xampp-linux-x64-7.2.29-0-installer.run

### 2. XAMPP auto start
#### Create `/etc/init.d/lampp` file
    #!/bin/bash
    ### BEGIN INIT INFO
    # Provides: lampp
    # Required-Start:    $local_fs $syslog $remote_fs dbus
    # Required-Stop:     $local_fs $syslog $remote_fs
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: Start lampp
    ### END INIT INFO   
    /opt/lampp/lampp start
         
#### Update the relevant file
    $ sudo chmod +x /etc/init.d/lampp
    $ sudo update-rc.d lampp defaults

### 3. Config port for Website
#### Edit file `/opt/lampp/etc/httpd.conf`
    Listen <port>
    Include Include etc/extra/httpd-vhosts.conf;    
#### Edit file `/opt/lampp/etc/extra/httpd-vhosts.conf`
    <VirtualHost *:80>
        DocumentRoot "/opt/lampp/htdocs/attendance"
        ServerName 192.168.18.10:80
        <Directory "/opt/lampp/htdocs/attendance">
            Options Indexes FollowSymLinks MultiViews
            AllowOverride all
            Order Deny,Allow
            Allow from all
            Require all granted
        </Directory>
    </VirtualHost>
    
    <VirtualHost *:443>
    DocumentRoot "/opt/lampp/htdocs/attendance"
    ServerName 192.168.18.10:443
    SSLEngine on
    SSLCertificateFile /opt/lampp/etc/ssl.crt/server.crt
    SSLCertificateKeyFile /opt/lampp/etc/ssl.key/server.key
        <Directory "/opt/lampp/htdocs/attendance">
            Options Indexes FollowSymLinks MultiViews
            AllowOverride all
            Order Deny,Allow
            Allow from all
            Require all granted
        </Directory>
    </VirtualHost>

##### Redirect http to https
    <VirtualHost *:80>
        DocumentRoot "/opt/lampp/htdocs/attendance"
        ServerName 192.168.18.10:80
        Redirect permanent / https://192.168.18.10:443
        <Directory "/opt/lampp/htdocs/attendance">
            Options Indexes FollowSymLinks MultiViews
            AllowOverride all
            Order Deny,Allow
            Allow from all
            Require all granted
        </Directory>
    </VirtualHost>

### 4. Note
#### Create the SSL Certificate
    $ sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /opt/lampp/etc/ssl.key/server.key -out /opt/lampp/etc/ssl.crt/server.crt

#### Error when build Laravel
    $ sudo chmod -R 777 /opt/lampp/htdocs/<project>/storage
    
#### Login as root remotely
    Login, and edit this file: $ sudo nano /etc/ssh/sshd_config
    Find this line 'PermitRootLogin without-password' and Edit 'PermitRootLogin yes'
    Restart sshd service: $ /etc/init.d/ssh restart
