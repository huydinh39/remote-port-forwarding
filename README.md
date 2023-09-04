## SSH Tunneling - Remote Port Forwarding
### Server Site (Centos7)

**1. Edit sshd_config**
- Run Command `sudo vim /etc/ssh/sshd_config`
- Then edit 2 following lines
```
# Enable Remote Port Forwarding
GatewayPorts yes

# Disable SSH by Password
PasswordAuthentication no
```

**2. Restart SSH Service after change**
```
sudo systemctl restart sshd
```
**3. Open the ports you want to assign for your sevices**

- Open port (2001-2005)
```
firewall-cmd --zone=public --add-port=2001/tcp --permanent
firewall-cmd --zone=public --add-port=2002/tcp --permanent
firewall-cmd --zone=public --add-port=2003/tcp --permanent
firewall-cmd --zone=public --add-port=2004/tcp --permanent
firewall-cmd --zone=public --add-port=2005/tcp --permanent
```

- Reload filewall
```
firewall-cmd --reload
```

- Check port opened
```
firewall-cmd --list-ports
```

***
### Raspberry Pi Site

**1. Create SSH Key on Pi and Copy to SSH Server**

- Create SSH Key, press Enter to the end by command `ssh-keygen`

- Copy Public Key after run command `cat ~/.ssh/id_rsa.pub`

**2. Access SSH Server to paste the Public Key was copied above into authorized_keys file**

- Run following command and paste the Public Key into this file

***Note: Please paste carefully, only add, not clear old content !!!***
```
sudo vim ~/.ssh/authorized_keys
```
- Restart SSH Service after change
```
sudo systemctl restart sshd
```

**3. Create Service in systemd**
- Let's change **HOST**, **HOST_PORT**, **USER**, **LOCAL_PORT** to suit your information before run following command
  + HOST: _IP Address your SSH Server (**192.168.100.100**)_
  + HOST_PORT: _Port of your SSH Server (**2001**) (Unique)_
  + USER: _User you want to use to login to SSH Server (**root**)_
  + LOCAL_PORT: _Port of servive on your Local Server, you want to mapping to Port of SSH Server (In this case, **Node-red** running on port **1880**)_

- Run following command (**after changed to suit your information**)
```
sudo cat << EOF | sudo tee -a /etc/systemd/system/ssh_tunnel.service
[Unit]
Description=SSH Tunneling - Remote Port Forwarding
After=network-online.target
Wants=network-online.target

[Service]
Environment="HOST=192.168.100.100"
Environment="HOST_PORT=2001"
Environment="USER=root"
Environment="LOCAL_PORT=1880"
ExecStart=/usr/bin/ssh -tt -o ExitOnForwardFailure=yes -R ${HOST_PORT}:localhost:${LOCAL_PORT} ${USER}@${HOST} -i /home/pi/.ssh/id_rsa -o StrictHostKeyChecking=no
StandardOutput=null
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

- Start Service
```
sudo systemctl start ssh_tunnel
```

- Check Status Service
```
sudo systemctl status ssh_tunnel
```

- Enable Service to Run On Boot
```
sudo systemctl enable ssh_tunnel
```

***
### Test
Access http://\<HOST>:\<PORT> to check each of your services

Example: http://192.168.100.100:2001