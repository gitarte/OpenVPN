### The goal
* client OS: Ubuntu 16.04
* OpenVPN server at ```centos.artgaw.pl```
* virtual network ```10.8.0.0/24```
* client IP ```10.8.0.10```


### Build the certs for the client
This has to be done on properly configured server side. Follow this [recipe]. Here the client is called simply ```client```. Consider a meaningfull name instead.
```bash
cd /etc/openvpn/easy-rsa
./build-key client
```
### Install OpenVPN on your client
```bash
apt -y install openvpn
```
### Download cert files from the server
```bash
 scp root@centos.artgaw.pl:/etc/openvpn/easy-rsa/keys/client* /etc/openvpn/
```
### Generate client's config
```bash
cat <<EOT >> /etc/openvpn/client.conf
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
key  client.key
cert client.crt
```
### Deal with the service and enjoy
```bash
systemctl enable openvpn@client.service
systemctl start  openvpn@client.service
```
[recipe]: <https://github.com/gitarte/OpenVPN/blob/master/configure-server.md>
