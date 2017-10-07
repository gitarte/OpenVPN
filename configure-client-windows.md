### The goal
* OpenVPN server  ```centos.artgaw.pl```
* virtual network ```10.8.0.0/24```
* server          ```IP:10.8.0.1  OS:CentOS7```
* client1         ```IP:10.8.0.10 OS:Ubuntu 16.04```
* client2         ```IP:10.8.0.20 OS:Ubuntu 16.04```
* client3         ```IP:10.8.0.30 OS:Windows 10```


### Build the certs for the client and ensure that the client will use fixed IP address
This steps have to be done on properly configured server side. Follow this [recipe]. 
### Install OpenVPN 
```bash
TODO
```
### Download cert files from the server
Pay attention for the actual client you are working on. Here is an example for ```client3```
```bash
TODO
```
### Generate client's config
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
key  client3.key
cert client3.crt
EOF
```
### Deal with the service and enjoy
```bash
TODO
```
[recipe]: <https://github.com/gitarte/OpenVPN/blob/master/configure-server.md>
