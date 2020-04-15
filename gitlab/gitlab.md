### Create file /opt/gitlab/docker-compose.yml
    web:
      image: 'gitlab/gitlab-ee:latest'
      container_name: 'gitlab'
      restart: always
      environment:
        GITLAB_OMNIBUS_CONFIG: |
          gitlab_rails['smtp_enable'] = true
          gitlab_rails['smtp_address'] = "smtp.gmail.com"
          gitlab_rails['smtp_port'] = 587
          gitlab_rails['smtp_user_name'] = "user@gmail.com"
          gitlab_rails['smtp_password'] = "app-password"
          gitlab_rails['smtp_domain'] = "smtp.gmail.com"
          gitlab_rails['smtp_authentication'] = "login"
          gitlab_rails['smtp_enable_starttls_auto'] = true
          gitlab_rails['smtp_tls'] = false
          gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
          # Add any other gitlab.rb configuration here, each on its own line
      ports:
        - '1000:80'
        - '1001:443'
        - '1002:22'
        - '1003:587'
      volumes:
        - './config:/etc/gitlab'
        - './logs:/var/log/gitlab'
        - './data:/var/opt/gitlab'    

### Create file /etc/systemd/system/gitlab.service
    [Unit]
    Description=Gitlab
    Requires=docker.service
    After=docker.service
    [Service]
    Restart=always
    User=root
    Group=docker
    # Shutdown container (if running) when unit is stopped
    ExecStartPre=/usr/local/bin/docker-compose -f /opt/gitlab/docker-compose.yml down -v
    # Start container when unit is started
    ExecStart=/usr/local/bin/docker-compose -f /opt/gitlab/docker-compose.yml up
    # Stop container when unit is stopped
    ExecStop=/usr/local/bin/docker-compose -f /opt/gitlab/docker-compose.yml down -v
    [Install]
    WantedBy=multi-user.target

### Enable and start the gitlab service
    $ sudo systemctl enable gitlab
    $ sudo systemctl start gitlab
    

