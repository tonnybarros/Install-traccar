#!/bin/bash

# Auto Install Traccar with MySQL

clear

echo ""
echo "████████╗██████╗  █████╗  ██████╗ ██████╗ █████╗ ██████╗ "
echo "╚══██╔══╝██╔══██╗██╔══██╗██╔════╝██╔════╝██╔══██╗██╔══██╗"
echo "   ██║   ██████╔╝███████║██║     ██║     ███████║██████╔╝ "
echo "   ██║   ██╔══██╗██╔══██║██║     ██║     ██╔══██║██╔══██╗"
echo "   ██║   ██║  ██║██║  ██║╚██████╗╚██████╗██║  ██║██║  ██║"
echo "   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝ ╚═════╝╚═════╝ ╚═╝  ╚═╝ v1.6"
echo ""
echo "Auto Install Traccar - Latest available version"
echo "Support: tectonny@gmail.com"
read -p "Press ENTER to start"

# User input requests
read -p "Enter the name of the database for Traccar: " DB_NAME
read -p "Enter the MySQL user to be created for Traccar: " DB_USER
read -sp "Enter the password for the MySQL user: " DB_PASS
echo ""
read -p "Enter your domain (e.g., tracking.mydomain.com): " DOMAIN

# Update the system
sudo apt update && sudo apt upgrade -y

# Install dependencies
echo "Installing dependencies..."
sudo apt install unzip mysql-server nginx certbot python3-certbot-nginx curl -y

# MySQL Configuration
echo "Configuring MySQL..."
sudo mysql -e "CREATE DATABASE $DB_NAME CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
sudo mysql -e "CREATE USER '$DB_USER'@'localhost' IDENTIFIED BY '$DB_PASS';"
sudo mysql -e "GRANT ALL PRIVILEGES ON $DB_NAME.* TO '$DB_USER'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Download the latest version of Traccar
echo "Detecting and downloading the latest version of Traccar..."
LATEST_VERSION=$(curl -s https://api.github.com/repos/traccar/traccar/releases/latest | grep tag_name | awk -F '"' '{print $4}')
wget https://github.com/traccar/traccar/releases/download/$LATEST_VERSION/traccar-linux-64-${LATEST_VERSION:1}.zip
unzip traccar-linux-64-${LATEST_VERSION:1}.zip
sudo ./traccar.run

# Download MySQL driver
echo "Downloading MySQL driver..."
sudo wget -P /opt/traccar/lib https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.3.0/mysql-connector-j-8.3.0.jar

# Configure the database connection in Traccar
echo "Configuring database connection in Traccar..."
sudo tee /opt/traccar/conf/traccar.xml > /dev/null <<EOL
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <entry key='database.driver'>com.mysql.cj.jdbc.Driver</entry>
    <entry key='database.url'>jdbc:mysql://localhost:3306/$DB_NAME?serverTimezone=UTC&amp;useSSL=false&amp;allowPublicKeyRetrieval=true</entry>
    <entry key='database.user'>$DB_USER</entry>
    <entry key='database.password'>$DB_PASS</entry>
</properties>
EOL

# Configure domain and SSL with Nginx
echo "Configuring domain and SSL with Nginx..."
sudo tee /etc/nginx/sites-available/traccar > /dev/null <<EOL
server {
    listen 80;
    server_name $DOMAIN;

    location / {
        proxy_pass http://localhost:8082;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOL
sudo ln -s /etc/nginx/sites-available/traccar /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx

# Automatic SSL via Certbot
sudo certbot --nginx -d $DOMAIN --non-interactive --agree-tos --register-unsafely-without-email --redirect

# Create the systemd service
echo "Creating Traccar service..."
sudo tee /etc/systemd/system/traccar.service > /dev/null <<EOL
[Unit]
Description=Traccar GPS Tracking System
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/traccar
ExecStart=/usr/bin/java -jar tracker-server.jar conf/traccar.xml
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
EOL

sudo systemctl daemon-reload
sudo systemctl enable traccar
sudo systemctl start traccar

# Create friendly commands
sudo tee /usr/local/bin/start-traccar > /dev/null <<EOL
#!/bin/bash
sudo systemctl start traccar
EOL
sudo tee /usr/local/bin/stop-traccar > /dev/null <<EOL
#!/bin/bash
sudo systemctl stop traccar
EOL
sudo tee /usr/local/bin/status-traccar > /dev/null <<EOL
#!/bin/bash
sudo systemctl status traccar
EOL
sudo tee /usr/local/bin/restart-traccar > /dev/null <<EOL
#!/bin/bash
sudo systemctl restart traccar
EOL
sudo tee /usr/local/bin/log-traccar > /dev/null <<EOL
#!/bin/bash
sudo tail -f /opt/traccar/logs/tracker-server.log
EOL
sudo tee /usr/local/bin/edit-traccar > /dev/null <<EOL
#!/bin/bash
sudo nano /opt/traccar/conf/traccar.xml
EOL
sudo tee /usr/local/bin/log-traccar-search > /dev/null <<EOL
#!/bin/bash
if [ -z "\$1" ]; then
    echo "Please provide a search term."
    echo "Usage: log-traccar-search <search-term>"
    exit 1
fi
sudo tail -f /opt/traccar/logs/tracker-server.log | grep --color=auto "\$1"
EOL
sudo chmod +x /usr/local/bin/start-traccar /usr/local/bin/stop-traccar /usr/local/bin/status-traccar /usr/local/bin/restart-traccar /usr/local/bin/log-traccar /usr/local/bin/edit-traccar /usr/local/bin/log-traccar-search

# Finalizing installation
echo "Installation completed successfully!"
echo "Manage Traccar with:"
echo "start-traccar | stop-traccar | status-traccar | restart-traccar | log-traccar | edit-traccar | log-traccar-search"
echo "Access via: https://$DOMAIN"
