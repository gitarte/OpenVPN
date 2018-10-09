### The goal
* OpenVPN server  ```centos.artgaw.pl```
* virtual network ```10.8.0.0/24```
* server          ```IP:10.8.0.1  OS:CentOS7```
* client1         ```IP:10.8.0.10 OS:Ubuntu 16.04```
* client2         ```IP:10.8.0.20 OS:Ubuntu 16.04```
* client3         ```IP:10.8.0.30 OS:Windows 10```

### Install all necessary software
```bash
yum -y install openvpn easy-rsa iptables-services
```
### Create certs
First prepare the tools
```sh
mkdir -p /etc/openvpn/easy-rsa/keys
mkdir    /etc/openvpn/ccd
cp -rf   /usr/share/easy-rsa/2.0/*               /etc/openvpn/easy-rsa
cp       /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
```
Then customize a vars file ```/etc/openvpn/easy-rsa/vars``` Pay attention for two fildes:
* KEY_NAME: enter a word ```server``` here. Otherwise the rest of this recipe wont work, because it references to this
* KEY_CN: enter the domain name that resolves to your server's IP address
```bash
export KEY_COUNTRY="Your 2 Letters Country Code"
export KEY_PROVINCE="Your Province Name"
export KEY_CITY="Your City Name"
export KEY_ORG="Your Organization Name"
export KEY_EMAIL="you@yourmail.org"
export KEY_OU="Your Organization Unit"
export KEY_NAME="server"
export KEY_CN="centos.artgaw.pl"
```
### Build the certs for the server
```bash
cd /etc/openvpn/easy-rsa
source ./vars
./easyrsa  build-ca
./easyrsa  build-server-full server
./easyrsa  gen-dh
cd /etc/openvpn/easy-rsa/keys
cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
```
### Build the certs for clients
Here the clients are called simply ```client1```, ```client2``` and ```client3```. Consider a meaningfull names instead.
To generate cert files for clients you just do the following. Leave defaults and agree with all questions.
```bash
cd /etc/openvpn/easy-rsa
./easyrsa build-client-full client1
./easyrsa build-client-full client2
./easyrsa build-client-full client3
```
You may need to load variables before issuing ```build-key``` command:
```bash
source ./vars
```
Do not issue ```./clean-all``` unless your intention is to start all over again.

### Ensure that clients will use fixed IP addresses
Pay attention for the name of files created bellow. It must match the name of Common Name given while creation of client cert files
```bash
echo "ifconfig-push 10.8.0.10 255.255.255.0" > /etc/openvpn/ccd/client1
echo "ifconfig-push 10.8.0.20 255.255.255.0" > /etc/openvpn/ccd/client2
echo "ifconfig-push 10.8.0.30 255.255.255.0" > /etc/openvpn/ccd/client3
```
You need also to prevent any possible IP conflicts. To do so ensure, that every client has its own IP assigned in ipp.txt
```bash
echo "client1,10.8.0.10" >> /etc/openvpn/ipp.txt
echo "client2,10.8.0.20" >> /etc/openvpn/ipp.txt
echo "client3,10.8.0.30" >> /etc/openvpn/ipp.txt
```
### Prepare server's config
```sh
cat <<EOT >> /etc/openvpn/server.conf
port   2193
proto  udp
dev    tap
ca     ca.crt
key    server.key
cert   server.crt
dh     dh2048.pem
server 10.8.0.0 255.255.255.0
push   "bypass-dhcp"
push   "dhcp-option DNS 8.8.8.8"
push   "dhcp-option DNS 8.8.4.4"
ifconfig-pool-persist ipp.txt
client-config-dir     /etc/openvpn/ccd
keepalive 10 120
comp-lzo
user  nobody
group nobody
persist-key
persist-tun
status     openvpn-status.log
log-append openvpn.log
verb 3
EOT
```
### Final touch
```bash
systemctl disable firewalld
systemctl stop    firewalld
systemctl enable  iptables
systemctl start   iptables
systemctl enable  openvpn@server.service

echo "net.ipv4.ip_forward = 1" > /etc/sysctl.conf

iptables --flush
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables-config 
iptables-save > /etc/sysconfig/iptables

reboot
```
