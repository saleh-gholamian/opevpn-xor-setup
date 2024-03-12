## opevpn-xor-setup
Openvpn XOR Patch installation guide on Ubuntu server



**#0 Change ssh port**
```bash
nano /etc/ssh/sshd_config
systemctl restart sshd
```



**#1 Update the ubuntu server**
```bash
apt update && apt upgrade -y
```



**#2 Enable IPv4 forwadring**
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
```



**#3 Download the OpenVPN tarball for your version**
```bash
wget https://github.com/TheHavlok/Openvpn-XOR-Patch/releases/download/files/openvpn-2.6.8.tar.gz
```
```bash
tar -xvf openvpn-2.6.8.tar.gz
```

Download and extract the Tunnelblick repository:

**#4 Download and extract the Tunnelblick repository:**
```bash
wget https://github.com/TheHavlok/Openvpn-XOR-Patch/releases/download/files/Tunnelblick-master.zip
```
```bash
apt install unzip -y
```
```bash
unzip Tunnelblick-master.zip
```



**#5 Apply Patches**
```bash
cp Tunnelblick-master/third_party/sources/openvpn/openvpn-2.6.8/patches/*.diff openvpn-2.6.8
```
```bash
cd openvpn-2.6.8
apt install patch -y
patch -p1 < 02-tunnelblick-openvpn_xorpatch-a.diff
patch -p1 < 03-tunnelblick-openvpn_xorpatch-b.diff
patch -p1 < 04-tunnelblick-openvpn_xorpatch-c.diff
patch -p1 < 05-tunnelblick-openvpn_xorpatch-d.diff
patch -p1 < 06-tunnelblick-openvpn_xorpatch-e.diff
patch -p1 < 10-route-gateway-dhcp.diff
```



**#6 Build OpenVPN with XOR Patch**
Install the prerequisites for the build:
```bash
apt install build-essential libssl-dev iproute2 liblz4-dev liblzo2-dev libpam0g-dev libpkcs11-helper1-dev libsystemd-dev resolvconf pkg-config autoconf automake libtool libcap-ng-dev liblz4-dev libsystemd-dev liblzo2-dev libpam0g libpam0g-dev -y
```
Compile and install OpenVPN with the XOR patch:
```bash
autoreconf -i -v -f
./configure --enable-systemd --enable-async-push --enable-iproute2
make
make install
```



**#7 Create Configuration Directories**
```bash
mkdir -p /etc/openvpn/{server,client}
```



**#8 Create Keys and Certificates with EasyRSA**
Install the EasyRSA package:
```bash
apt install easy-rsa -y
```

Make a copy of the EasyRSA scripts and configuration files:
```bash
cp -r /usr/share/easy-rsa ~
cd ~/easy-rsa
```

Make a copy of the example variables:
```bash
cp vars.example vars
```

You can edit the vars file if you wish, but we will just use the defaults. Initialize the public key infrastructure:
```bash
./easyrsa init-pki
```

Build Certificate Authority (CA):
```bash
./easyrsa build-ca nopass
```

Generate and sign your server key and certificate:
```bash
./easyrsa gen-req server nopass
./easyrsa sign-req server server
```

Generate and sign your client key and certificate:
```bash
./easyrsa gen-req client nopass
./easyrsa sign-req client client
```

Generate the Diffie-Hellman parameters:
```bash
./easyrsa gen-dh
```

Generate a preshared key to encrypt the initial exchange:
```bash
openvpn --genkey secret pki/tls-crypt.key
```

Copy all the keys and certificates into OpenVPN directory:
```bash
cp pki/ca.crt /etc/openvpn
cp pki/private/server.key /etc/openvpn/server
cp pki/issued/server.crt /etc/openvpn/server
cp pki/private/client_name.key /etc/openvpn/client_name.key
cp pki/issued/client_name.crt /etc/openvpn/client_name.crt
cp pki/tls-crypt.key /etc/openvpn
cp pki/dh.pem /etc/openvpn
```


**#9 Generate Scramble Obfuscation Code**
```bash
openssl rand -base64 24
```
*Save generated code (We using this code in server.conf && client.ovpn)*



**#10 Configure OpenVPN Server**
Edit the OpenVPN configuration file:
```bash
nano /etc/openvpn/server.conf
```

Adding config and save file:
```bash
port 443
proto tcp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server/server.crt
key /etc/openvpn/server/server.key
dh /etc/openvpn/dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /etc/openvpn/ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 1.0.0.1"
push "dhcp-option DNS 1.1.1.1"
keepalive 10 120
topology subnet
cipher AES-128-GCM
tls-crypt /etc/openvpn/tls-crypt.key
persist-key
persist-tun
status openvpn-status.log
verb 3
scramble obfuscate <Code>
```



**#**
```bash

```



**#**
```bash

```



**#**
```bash

```



**#**
```bash

```



**#**
```bash

```





