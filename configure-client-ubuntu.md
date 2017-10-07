### The goal
* OpenVPN server  ```centos.artgaw.pl```
* virtual network ```10.8.0.0/24```
* server          ```IP:10.8.0.1  OS:CentOS7```
* client1         ```IP:10.8.0.10 OS:Ubuntu 16.04```
* client2         ```IP:10.8.0.20 OS:Ubuntu 16.04```
* client3         ```IP:10.8.0.30 OS:Windows 10```


### Build the certs for the client and ensure that the client will use fixed IP address
This steps have to be done on properly configured server side. Follow this [recipe]. 
### Install OpenVPN on each client
```bash
apt -y install openvpn
```
### Download cert files from the server
Pay attention for the actual client you are working on. Here is an example for ```client1```
```bash
 scp root@centos.artgaw.pl:/etc/openvpn/easy-rsa/keys/ca.cert     /etc/openvpn/
 scp root@centos.artgaw.pl:/etc/openvpn/easy-rsa/keys/client1.key /etc/openvpn/
 scp root@centos.artgaw.pl:/etc/openvpn/easy-rsa/keys/client1.crt /etc/openvpn/
```
### Generate client's config
Again here the example is for ```client1``` The name of config file ```/etc/openvpn/client.conf``` should however be the same on each client machine.
```bash
cat <<EOF >> /etc/openvpn/client.conf
client
port 2193
remote centos.artgaw.pl
comp-lzo yes
dev tun
proto udp
nobind
auth-nocache
persist-key
persist-tun
verb 2
key-direction 1
ca   ca.crt
key  client1.key
cert client1.crt
EOF
```
### Deal with the service and enjoy
```bash
systemctl enable openvpn@client.service
systemctl start  openvpn@client.service
```
[recipe]: <https://github.com/gitarte/OpenVPN/blob/master/configure-server.md>
