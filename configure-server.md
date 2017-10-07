### Install and prepare all necessary software
```bash
yum -y install openvpn easy-rsa iptables-services
```
### Create certs
First prepare the tools
```sh
mkdir -p /etc/openvpn/easy-rsa/keys
cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa
cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
```
Then customize a vars file ```/etc/openvpn/easy-rsa/vars``` Pay attention for two fildes:
* KEY_NAME: enter a word ```server``` here. Otherwise the rest of this recipe wont work, because it references to this
* KEY_CN: enter the domain name that resolves to your server
```bash
export KEY_COUNTRY="PL"
export KEY_PROVINCE="Silesia"
export KEY_CITY="Katowice"
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
./clean-all
./build-ca
./build-key-server server
./build-dh
cd /etc/openvpn/easy-rsa/keys
cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
```
### Build the certs for the client
```bash
cd /etc/openvpn/easy-rsa
./build-key client
```
### Prepare server's config
```sh
cat <<EOT >> /etc/openvpn/server.conf
port 2193
proto udp
dev tun
ca ca.crt
cert server.crt
dh dh2048.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
comp-lzo
user nobody
group nobody
persist-key
persist-tun
status openvpn-status.log
log-append  openvpn.log
verb 3
EOT
```
### Final touch
```bash
systemctl disable firewalld
systemctl stop    firewalld
systemctl enable  iptables
systemctl start   iptables

echo "net.ipv4.ip_forward = 1" > /etc/sysctl.conf

iptables --flush
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables-config 
iptables-save > /etc/sysconfig/iptables

reboot
```
### Generate client's config
```sh
cd /root/keys
cat <<EOT >> arga-1-zfs.ovpn
client
port 2193
remote 86.105.51.161
comp-lzo yes
dev tun
proto udp
nobind
auth-nocache
persist-key
persist-tun
verb 2
key-direction 1
<ca>
EOT
cat ca.crt >> arga-1-zfs.ovpn 
cat <<EOT >> arga-1-zfs.ovpn
</ca>
<cert>
EOT
cat arga-1-zfs.crt >> arga-1-zfs.ovpn 
cat <<EOT >> arga-1-zfs.ovpn
</cert>
<key>
EOT
cat arga-1-zfs.key >> arga-1-zfs.ovpn 
echo '</key>' >> arga-1-zfs.ovpn 
systemctl restart openvpn@server.service
```
