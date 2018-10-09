http://www.startupcto.com/server-tech/centos/setting-up-openvpn-server-on-centos
https://community.openvpn.net/openvpn/wiki/Hardening

### The goal
* OpenVPN server  ```centos.artgaw.pl```
* virtual network ```10.8.0.0/24```
* server          ```IP:10.8.0.1  OS:CentOS7```
* client1         ```IP:10.8.0.10 OS:Ubuntu 18.04```
* client2         ```IP:10.8.0.20 OS:Ubuntu 18.04```
* client3         ```IP:10.8.0.30 OS:Windows 10```

### Install all necessary software
```bash
yum -y install openvpn easy-rsa iptables-services
```
### Create certs
First prepare the tools
```sh
mkdir -p /etc/openvpn/easy-rsa
mkdir    /etc/openvpn/ccd
cp -rf   /usr/share/easy-rsa/3.0/* /etc/openvpn/easy-rsa
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
export EASYRSA_KEY_SIZE="4096"
export EASYRSA_CRL_DAYS="3650"
export EASYRSA_DIGEST="sha512"
```
### Build the certs for the server
```bash
cd /etc/openvpn/easy-rsa
source ./vars
./easyrsa init-pki
./easyrsa build-ca [nopass]
./easyrsa gen-dh
./easyrsa build-server-full server [nopass]
./easyrsa gen-crl
openvpn --genkey --secret pki/ta.key

cp ./pki/ca.crt             /etc/openvpn/ca.crt
cp ./pki/dh.pem             /etc/openvpn/dh.pem
cp ./pki/issued/server.crt  /etc/openvpn/server.crt
cp ./pki/private/server.key /etc/openvpn/server.key
cp ./pki/ta.key             /etc/openvpn/ta.key
cp ./pki/crl.pem            /etc/openvpn/crl.pem
```
### Build the certs for clients
Here the clients are called simply ```client1```, ```client2``` and ```client3```. Consider a meaningfull names instead.
To generate cert files for clients you just do the following. Leave defaults and agree with all questions.
```bash
cd /etc/openvpn/easy-rsa
source ./vars

./easyrsa build-client-full client1 [nopass]
./easyrsa build-client-full client2 [nopass]
./easyrsa build-client-full client3 [nopass]

mkdir ./client1
cp ./pki/ca.crt              ./client1/ca.crt
cp ./pki/issued/client1.crt  ./client1/client.crt
cp ./pki/private/client1.key ./client1/client.key
cp ./pki/ta.key              ./client1/ta.key

mkdir ./client2
cp ./pki/ca.crt              ./client2/ca.crt
cp ./pki/issued/client2.crt  ./client2/client.crt
cp ./pki/private/client2.key ./client2/client.key
cp ./pki/ta.key              ./client2/ta.key

mkdir ./client3
cp ./pki/ca.crt              ./client3/ca.crt
cp ./pki/issued/client3.crt  ./client3/client.crt
cp ./pki/private/client3.key ./client3/client.key
cp ./pki/ta.key              ./client3/ta.key
```
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
# Basic
cat <<EOT >> /etc/openvpn/server.conf
persist-key
persist-tun
comp-lzo
port       2193
proto      udp
dev        tap
keepalive  10 120
user       nobody
group      nobody
status     openvpn-status.log
log-append openvpn.log
verb       3

# Certs
ca         /etc/openvpn/ca.crt
key        /etc/openvpn/server.key
cert       /etc/openvpn/server.crt
dh         /etc/openvpn/dh.pem
crl-verify /etc/openvpn/crl.pem
tls-auth   /etc/openvpn/ta.key 0

# Ciphers and Hardening
reneg-sec       0
remote-cert-tls client
tls-version-min 1.2
cipher          AES-256-CBC
auth            SHA512
tls-cipher TLS-DHE-RSA-WITH-AES-256-GCM-SHA384:TLS-DHE-RSA-WITH-AES-256-CBC-SHA256:TLS-DHE-RSA-WITH-AES-128-GCM-SHA256:TLS-DHE-RSA-WITH-AES-128-CBC-SHA256

# IP pool
server                10.8.0.0 255.255.255.0
topology              subnet
ifconfig-pool-persist ipp.txt
client-config-dir     /etc/openvpn/ccd

# DHCP Push options force all traffic through VPN and sets DNS servers
push "bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
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
