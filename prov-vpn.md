### Open VPN Settings

#### Server

``$ wget https://git.io/vpn -O openvpn-install.sh``

``$ sudo bash openvpn-install.sh``

Use UDP protocol, as for TCP the OpenVpn doesn't run out of the box.

#### Start / Stop

``$ sudo systemctl stop openvpn@server``

``$ sudo systemctl start openvpn@server``

``$ sudo systemctl restart openvpn@server``

``$ sudo systemctl status openvpn@server``

#### Boot

``$ cp /etc/openvpn/client.ovpn /etc/openvpn/client.conf``

Add ``AUTOSTART="all"`` to ``/etc/default/openvpn``

``$ sudo systemctl daemon-reload``

``$ sudo service openvpn restart``

#### Client

``$ sudo apt-get install openvpn -y``

``$ openvpn –-version``

From the Public server get the ``client.conf / client.ovpn`` file and (i.e.: /root/client.ovpn)

put it on the clien's location ``/etc/openvpn/client.ovpn``.

Copy client's certificates and key to /etc/openvpn/ directory on your Raspberry Pi

Run the client with the config file.

``$ openvpn /etc/openvpn/client.ovpn``

#### Statistics

Check logged clients:

``sudo cat /var/log/openvpn.log | grep "Peer Connection"``


### VPS Bastioning

#### OpenVPN Forwarding


``iptables -t nat -A PREROUTING -p tcp -i eth0 --dport DESIRED_PORT -j DNAT --to-destination VPN_CONNECTED_CLIENT_IP:APP_PORT``

``iptables -A FORWARD -p tcp -d VPN_CONNECTED_CLIENT_IP --dport APP_PORT -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT``

Auto IP Configuration:
```
#!/bin/bash
#### Adding preconfifured IP Addresses from file to nat-> PREROUTING and filter-> FORWARD chains

input="/root/scripts/ip.conf"
DIP="89.223.93.85"

while IFS= read -r SIP
do
iptables -t nat -C PREROUTING -p tcp -s "$SIP" -d "$DIP" --dport 8080 -j DNAT --to-destination 10.8.0.2:8080 2> /dev/null
if [ $? -ne 0 ];then
iptables -t nat -A PREROUTING -p tcp -s "$SIP" -d "$DIP" --dport 8080 -j DNAT --to-destination 10.8.0.2:8080
iptables -t filter -A FORWARD -p tcp -s "$SIP" -d 10.8.0.2 -j ACCEPT
fi
done < "$input"
iptables-save > /root/scripts/fireng.conf
```

Firewall Service
```
#!/bin/bash
###### Script for auto loading firewall rules at startup. It is executed by a service called fireng.service at startup.

/sbin/iptables-restore /root/scripts/fireng.conf
```

#### Resources

[https://openvpn.net/vpn-server-resources/how-to-connect-to-access-server-from-a-linux-computer/](https://openvpn.net/vpn-server-resources/how-to-connect-to-access-server-from-a-linux-computer/)

[https://openvpn.net/community-resources/how-to/](https://openvpn.net/community-resources/how-to/)

[Guide: Configure OpenVPN to autostart on systemd Linux](https://www.smarthomebeginner.com/configure-openvpn-to-autostart-linux/)

[Guide to install OpenVPN for Ubuntu](https://www.ovpn.com/en/guides/ubuntu-gui)

[OpenVPN client on Raspberry Pi](http://kernelreloaded.com/openvpn-client-on-raspberry-pi/)

[https://www.itsfullofstars.de/2018/09/openvpn-assign-static-ip-to-client/](https://www.itsfullofstars.de/2018/09/openvpn-assign-static-ip-to-client/)

[https://openvpn.net/community-resources/configuring-client-specific-rules-and-access-policies/](https://openvpn.net/community-resources/configuring-client-specific-rules-and-access-policies/)

[https://itblogsec.com/build-own-openvpn-server-by-using-raspberry-pi-12/](https://itblogsec.com/build-own-openvpn-server-by-using-raspberry-pi-12/)