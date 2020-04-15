### Install prerequisites
    $ sudo apt-get update
    $ sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
### Add docker's package signing key
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
### Add repository
    $ sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
### Install latest stable docker stable version
    $ sudo apt-get update
    $ sudo apt-get -y install docker-ce
### Install docker-compose
    $ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    $ sudo chmod a+x /usr/local/bin/docker-compose
### Add current user to the docker group
    $ sudo usermod -a -G docker $USER
### Enable & start docker service
    $ sudo systemctl enable docker
    $ sudo systemctl start docker
### Create directories
    $ sudo mkdir /var/lib/redmine
    $ sudo mkdir -p /var/lib/redmine/redmine_data /var/lib/redmine/mariadb_data
### Set correct permissions for the directories
    $ sudo chown -R 1001:1001 /var/lib/redmine/redmine_data /var/lib/redmine/mariadb_data
    $ sudo chown -R $USER:docker /var/lib/redmine

### Create file /var/lib/redmine/docker-compose.yml
    version: '2'
    services:
      mariadb:
        image: 'bitnami/mariadb:latest'
        environment:
          - ALLOW_EMPTY_PASSWORD=yes
        volumes:
          - '/var/lib/redmine/mariadb_data:/bitnami'
   
## Backup
### Dump old database to sql and copy this file to local
    $ mysqldump -u bn_redmine –p bitnami_redmine > dump.sql
### Run maria_db container and access to database
    $ docker exec -it <bitnami/maria_db-container-id> bash
    $ mysql -u root
### Create required user and table
    $ CREATE USER 'bn_redmine'@'%' IDENTIFIED BY '';
    $ GRANT ALL PRIVILEGES ON *.* TO 'bn_redmine'@'%' WITH GRANT OPTION;
    $ FLUSH PRIVILEGES;
    $ CREATE DATABASE bitnami_redmine;
### Update backup data to database
    $ exit
    $ mysql -u root -p bitnami_redmine < dump.sql
### Edit /var/lib/redmine/docker-compose.yml
    version: '2'
    services:
      mariadb:
        image: 'bitnami/mariadb:latest'
        environment:
          - ALLOW_EMPTY_PASSWORD=yes
        volumes:
          - '/var/lib/redmine/mariadb_data:/bitnami'
      redmine:
        image: 'bitnami/redmine:latest'
        environment:
          - REDMINE_USERNAME=admin
          - REDMINE_PASSWORD=redmineadmin
          - REDMINE_EMAIL=me@gmail.com
          - SMTP_HOST=smtp.gmail.com
          - SMTP_PORT=25
          - SMTP_USER=me@gmail.com
          - SMTP_PASSWORD=yourGmailPassword
        ports:
          - '3718:3000'
        volumes:
          - '/var/lib/redmine/redmine_data:/bitnami'
        depends_on:
          - mariadb
          
### Create file /etc/systemd/system/redmine.service
    [Unit]
    Description=Redmine
    Requires=docker.service
    After=docker.service
    [Service]
    Restart=always
    User=root
    Group=docker
    # Shutdown container (if running) when unit is stopped
    ExecStartPre=/usr/local/bin/docker-compose -f /var/lib/redmine/docker-compose.yml down -v
    # Start container when unit is started
    ExecStart=/usr/local/bin/docker-compose -f /var/lib/redmine/docker-compose.yml up
    # Stop container when unit is stopped
    ExecStop=/usr/local/bin/docker-compose -f /var/lib/redmine/docker-compose.yml down -v
    [Install]
    WantedBy=multi-user.target

### Enable and start the redmine service
    $ sudo systemctl enable redmine
    $ sudo systemctl start redmine
    
### Open browser and access http://localhost:3718
