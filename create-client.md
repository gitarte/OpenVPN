### Build the certs for the client
This has to be done on properly configured server side. Folloe this [recipe]. Here the client is called simply ```client```. Consider a meaningfull name instead.
```bash
cd /etc/openvpn/easy-rsa
./build-key client
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
[recipe]: <https://github.com/gitarte/OpenVPN/blob/master/configure-server.md>
