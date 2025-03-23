# Traccar Installer with Mysql
Auto Install Traccar

This script will automatically install the latest version of Traccar along with the MySQL driver.

Log into your Linux server and run the following single line:

```bash
wget https://raw.githubusercontent.com/tonnybarros/instalar-traccar/main/instalador_traccar.sh && chmod +x instalador_traccar.sh && ./instalador_traccar.sh
```

At the end of the installation, you can manage Traccar by executing the following commands in the terminal:

start-traccar (Start Traccar)

stop-traccar (Stop Traccar)

status-traccar (View Traccar status)

restart-traccar (Restart Traccar)

log-traccar (View the log in real time)

same as:
```bash
sudo tail -f /opt/traccar/logs/tracker-server.log
```

log-traccar-search (View real-time log of specific term)
same as:
```bash
sudo tail -f /opt/traccar/logs/tracker-server.log | grep XXXX
```
Replace xxxx with the word you want to search for in the real-time log

edit-traccar (View the log in real time)
same as:
```bash
sudo nano /opt/traccar/conf/traccar.xml
```
Tests performed on Hetzner server with Ubuntu 20.04
