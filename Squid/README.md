lab Squid

target 192.168.56.189

```bash
sudo nmap -sC -sV squid -sT -vvv -p- -Pn -T5 --min-rate=5000
```

```bash
Nmap scan report for squid (192.168.56.189)
Host is up, received user-set (0.00040s latency).
Scanned at 2026-03-13 09:30:45 UTC for 120s
Not shown: 65529 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON  VERSION
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack
3128/tcp  open  http-proxy    syn-ack Squid http proxy 4.14
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/4.14
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
```


SMB

Right away I notice that 445 - SMB seems to be open which is on the top of my priorities for Windows machines. I go ahead and investigate it using various tools.
```
bash
445/tcp   open  microsoft-ds? syn-ack
```
```bash
smbmap -H 192.168.56.189
```
```bash
┌──(kali㉿kali)-[~]
└─$ smbmap -H 192.168.56.189

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
 -----------------------------------------------------------------------------
     SMBMap - Samba Share Enumerator | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB
[*] Established 0 SMB session(s)                                                                            [*] Detected 1 hosts serving SMB
```

```bash
enum4linux -U -S 192.168.56.189
```
```bash
┌──(kali㉿kali)-[~]
└─$ enum4linux -U -S 192.168.56.189
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Wed Nov  6 04:18:31 2024

 =========================================( Target Information )=========================================

Target ........... 192.168.217.189
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==========================( Enumerating Workgroup/Domain on 192.168.217.189 )==========================

[E] Can't find workgroup/domain

 ==================================( Session Check on 192.168.217.189 )==================================

[E] Server doesn't allow session using username '', password ''.  Aborting remainder of tests.
```

```bash
smbclient -L 192.168.56.189
```
```bash
┌──(kali㉿kali)-[~]
└─$ smbclient -L 192.168.56.189
Password for [WORKGROUP\kali]:
session setup failed: NT_STATUS_ACCESS_DENIED
```

Since I was not able to get an anonymous session I tried one of the other ports first to gain more information on possible creds.
3128/TCP - HTTP

I went to the website and found the following page:
