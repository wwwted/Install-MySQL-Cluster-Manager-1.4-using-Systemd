# Install MySQL Cluster Manager "MCM" using Systemd

For more information on how to create a service look at ```man systemd.service```

### Install

Directory for MCM installation is /opt
Download latest tar package of MCM from edelivery.oracle.com or support.oracle.com
(today this is package: MySQL Cluster Manager 1.4.8 TAR for Generic Linux x86 (64bit))

Run commands below once you have downloaded the MCM package:
```
sudo su -
cd /opt
tar xzf /tmp/mcm-1.4.8-linux-glibc2.12-x86-64bit.tar.gz
mv mcm-1.4.8-linux-glibc2.12-x86-64bit mcm
cp /opt/mcm/mcm1.4.8/etc/mcmd.ini /opt/mcm/
mkdir /opt/mcm/mcm_data
```
Set manager-directory to folder /opt/mcm/mcm_data in configuration file:
```
cat /opt/mcm/mcmd.ini | sed 's/^##manager-directory.*/manager-directory = \/opt\/mcm\/mcm_data/' >> /opt/mcm/mcmd.ini
```

Structure should be:
```
 /opt/mcm
         mcmd.ini
         mcm1.4.8
         mcm_data

 Folder mcm_data is where all data on disk will be stored by default.
 Folder mcmN.N.N contains installed version of MCM.
```

### Systemd configuration

##### Create user to run the MCM service:
```
sudo useradd --no-create-home -s /bin/false mcm
sudo chown -R mcm:mcm /opt/mcm
```

##### Create systemd file:

Make sure to update path to correct version of mcm binaries in
mcm service file below.

Create file /etc/systemd/system/mcm.service like below.
```
[Unit]
Description=MySQL Cluster Manager
Documentation=https://dev.mysql.com/doc/mysql-cluster-manager/en/
After=network-online.target

[Service]
User=mcm
Group=mcm
Restart=always
Type=simple

ExecStart=/opt/mcm/mcm1.4.8/bin/mcmd --defaults-file=/opt/mcm/mcmd.ini

[Install]
WantedBy=multi-user.target
```

##### Start and enable service:
```
sudo systemctl daemon-reload
sudo systemctl start mcm
sudo systemctl enable mcm
sudo systemctl status mcm
```
If the service is not started correctly, look in messages file: ```sudo tail -150f /var/log/messages```

Remember also to set option StopOnError to 0 when creating your cluster so mcmd can restart failed data nodes ```set StopOnError:ndbmtd=0 mycluster;```.

