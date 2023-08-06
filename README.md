# config-service

local start-up cmd
-Dspring.profiles.active=local

dev params
sudo nano /etc/systemd/system/dvdtheque-server-config.service


[Unit]
Description=Manage Dvdtheque Spring Server Config service

[Service]
WorkingDirectory=/opt/dvdtheque_server_config_service
ExecStart=/usr/lib/jvm/java-11-openjdk-armhf/bin/java -jar dvdtheque-config-server.jar --spring.profiles.active=dev1
User=dvdtheque-user
Type=simple
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target



sudo chown -R dvdtheque-user:java-app-gr /opt/dvdtheque_server_config_service

sudo usermod -a -G java-app-gr jenkins

sudo chmod -R g+w /opt/dvdtheque_server_config_service/

sudo systemctl daemon-reload

sudo systemctl start dvdtheque-server-config.service

sudo systemctl status dvdtheque-server-config.service