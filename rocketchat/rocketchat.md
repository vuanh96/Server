### Cloning Rocket.chat
    $ cd /opt
    $ git clone https://github.com/RocketChat/Rocket.Chat.git

### Edit `/opt/Rocket.Chat/docker-compose.yml`
    version: '2'

    services:
      rocketchat:
        image: rocketchat/rocket.chat:latest
        ...
        ports:
          - 1010:3000
    
### Create file `/etc/systemd/system/rocketchat.service`
    [Unit]
    Description=RocketChat
    Requires=docker.service
    After=docker.service
    [Service]
    Restart=always
    User=root
    Group=docker
    # Shutdown container (if running) when unit is stopped
    ExecStartPre=/usr/local/bin/docker-compose -f /opt/Rocket.Chat/docker-compose.yml down -v
    # Start container when unit is started
    ExecStart=/usr/local/bin/docker-compose -f /opt/Rocket.Chat/docker-compose.yml up
    # Stop container when unit is stopped
    ExecStop=/usr/local/bin/docker-compose -f /opt/Rocket.Chat/docker-compose.yml down -v
    [Install]
    WantedBy=multi-user.target

### Enable and start the gitlab service
    $ sudo systemctl enable rocketchat
    $ sudo systemctl start rocketchat
