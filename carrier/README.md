## Nmap
![](https://i.imgur.com/XaMYmVO.png)
![](https://i.imgur.com/tmKsFVD.png)
I found http, snmp service is running, and next tried to analyze it.
## SNMP
By snmpwalk, I got a stranger string.
```
$ snmpwalk 10.10.10.105 -c public -v 2c > result.txt
```
![](https://i.imgur.com/RJi5UbD.png)
## gobuster
```
$ gobuster -u 10.10.10.105 -w /usr/share/seclists/Discovery/Web_Content/common.txt -s '200,204,301,302,307,403,500' -e 
```
![](https://i.imgur.com/uzbGi18.png)

## doc
That's tried to enter the doc dir, there are two files inside, which are the errorcodes.pdf and a picture.
![](https://i.imgur.com/VeJZAEZ.png)

In the errorcodes.pdf file, that has a tip about account.

![](https://i.imgur.com/AsiwqIf.png)
Through this document, I found default user **"admin"**.
Success log in by admin/NET_45JDX23 !
## diag.php
Enter this page, it has a command injection.
First encode reverse code to base64:
![](https://i.imgur.com/7VQHNCW.png)

And using burpsuite to got a reverse shell.

![](https://i.imgur.com/099AEVl.png)

![](https://i.imgur.com/V6Ti0g8.png)

The user flag is:
![](https://i.imgur.com/xJ27w2f.png)

But through ifconfig, I found that the IP address of this host is actually **10.99.64.2**.
## Ticket.php
![](https://i.imgur.com/goTJsmb.png)
According to the content, one of the three networks should be problematic with FTP.
```
$ for i in $(seq 254); do nc -w 5 -vz 10.120.15.$i 21; done;
```
![](https://i.imgur.com/VhzSunS.png)

Through this command, I found out that the only host open the FTP service should be 10.120.15.10.
## BGP hijacking
Try an intranet scan:
```
$ netstat -pantu
```
![](https://i.imgur.com/2EYhJmY.png)

I find the bgp service running on port 2601.

My shell is in AS100, according to the picture in ticket, the FTP service in CastCom(AS200), the other machine connected to FTP maybe in AS300.
According to the NIC condition, eth1 should be connected to AS300, and eth2 should be connected to AS200.
```
$ ifconfig eth2 10.120.15.10 netmask 255.255.255.0
```
Next, I want to attack the bgp protocol and do subnet hijacking.
I tried to transfer the access in AS200 to this controlled host, and capture packet to analysis is there have login credentials.

To do this, I need to modify the bgp routing table, and restart the network and quagga service.

- modify bgpd.conf
![](https://i.imgur.com/luqW21r.png)

- restart service
```
$ service networking restart
$ service quagga restart
```

Finally, set up a complete FTP server on the target, using python3 to execute.
![](https://i.imgur.com/5CEvoUN.png)
use ssh to log in.
```
$ ssh root@10.10.10.105
```
root flag:
![](https://i.imgur.com/8q3Y2eT.png)


