### The goal
* OpenVPN server  ```centos.artgaw.pl```
* virtual network ```10.8.0.0/24```
* server          ```IP:10.8.0.1  OS:CentOS7```
* client1         ```IP:10.8.0.10 OS:Ubuntu 18.04```
* client2         ```IP:10.8.0.20 OS:Ubuntu 18.04```
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
 scp root@centos.artgaw.pl:/etc/openvpn/easy-rsa/client1/* /etc/openvpn/
```
### Generate client's config
Again here the example is for ```client1``` The name of config file ```/etc/openvpn/client.conf``` should however be the same on each client machine.
```bash
cat <<EOF >> /etc/openvpn/client.conf
client
tls-client
pull
nobind
persist-key
persist-tun
dev              tun
proto            udp
port             2193
remote           centos.artgaw.pl
#redirect-gateway def1
comp-lzo         yes
verb             3
ca               ca.crt
key              client1.key
cert             client1.crt
tls-auth         ta.key 1
remote-cert-tls  server
ns-cert-type     server
key-direction    1
cipher           AES-256-CBC
tls-version-min  1.2
auth             SHA512
tls-cipher       TLS-DHE-RSA-WITH-AES-256-GCM-SHA384:TLS-DHE-RSA-WITH-AES-256-CBC-SHA256:TLS-DHE-RSA-WITH-AES-128-GCM-SHA256:TLS-DHE-RSA-WITH-AES-128-CBC-SHA256
status           openvpn-status.log
log-append       openvpn.log
EOF
```
### Deal with the service and enjoy
```bash
systemctl enable openvpn@client.service
systemctl start  openvpn@client.service
```
[recipe]: <https://github.com/gitarte/OpenVPN/blob/master/configure-server.md>
