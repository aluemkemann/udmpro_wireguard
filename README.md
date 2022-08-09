# udmpro_wireguard
How to install and use Wireguard VPN on the Ubiquiti Dream Machine Pro

Getestet mit UDM Pro mit UniFi OS UDM Pro 1.12.22
Die UDM Pro SE kann hiermit _nicht_ ohne Anpassung eingerichtet werden, hierfür werde ich noch eine extra Anleitung erstellen
Befehle Zeile für Zeiler per SSH auf der UDM Pro copy und pasten

```
unifi-os shell
curl -L https://udm-boot.boostchicken.dev -o udm-boot.deb
dpkg -i udm-boot.deb
exit
```

```
alias ll='ls -la --color=auto'
```

```
curl -LJo wireguard-kmod.tar.Z  https://github.com/tusc/wireguard-kmod/releases/download/v06-01-22/wireguard-kmod-06-01-22.tar.Z
tar -C /mnt/data -xvzf wireguard-kmod.tar.Z
grep "^LOAD_BUILTIN" /mnt/data/wireguard/setup_wireguard.sh
sed -i 's/LOAD_BUILTIN=1/LOAD_BUILTIN=0/g' /mnt/data/wireguard/setup_wireguard.sh
grep "^LOAD_BUILTIN" /mnt/data/wireguard/setup_wireguard.sh
/mnt/data/wireguard/setup_wireguard.sh
dmesg | grep wireguard
```

```
cp /mnt/data/wireguard/setup_wireguard.sh /mnt/data/on_boot.d/
echo "wg-quick up wg0" >> /mnt/data/on_boot.d/setup_wireguard.sh
```

```
cd /etc/wireguard
wg genkey | tee privatekey.wg0 | wg pubkey > publickey.wg0
```

# Den folgenden Block bis zur Zeile mit dem einzelnen EOF kopieren und pasten!
```
cat << EOF > wg0.conf
[Interface]
Address = 10.9.8.1/24
PrivateKey = priv.priv.priv.priv
ListenPort = 1820
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables ! -o lo -t nat -A POSTROUTING -j MASQUERADE; iptables -A FORWARD -i br0 -m state --state RELATED,ESTABLISHED -j ACCEPT
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables ! -o lo -t nat -D POSTROUTING -j MASQUERADE; iptables -D FORWARD -i br0 -m state --state RELATED,ESTABLISHED -j ACCEPT

[Peer]
PublicKey = 
AllowedIPs = 10.9.8.2/32
EOF
```

```
awk 'BEGIN{getline l < "privatekey.wg0"}/priv\.priv\.priv\.priv/{gsub("priv\.priv\.priv\.priv",l)}1' wg0.conf
```

