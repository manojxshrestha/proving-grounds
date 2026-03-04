access

sudo nano /etc/hosts

192.168.56.187 access

Enumeration and Information Gathering

```bash
sudo nmap -sC -sV access -sT -vvvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV access -sT -vvvv -p- -Pn -T5 --min-rate=5000

PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Simple DNS Plus
80/tcp    open  http          syn-ack Apache httpd 2.4.48 ((Win64) OpenSSL/1.1.1k PHP/8.0.7)
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
|_http-title: Access The Event
|_http-server-header: Apache/2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
| http-methods:
|   Supported Methods: POST OPTIONS HEAD GET TRACE
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2025-05-19 08:13:45Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: access.offsec0., Site: Default-First-Site-Name)
443/tcp   open  ssl/http      syn-ack Apache httpd 2.4.48 (OpenSSL/1.1.1k PHP/8.0.7)
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: access.offsec0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
47001/tcp open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
49670/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49671/tcp open  msrpc         syn-ack Microsoft Windows RPC
49674/tcp open  msrpc         syn-ack Microsoft Windows RPC
49679/tcp open  msrpc         syn-ack Microsoft Windows RPC
49701/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Hosts: SERVER, www.example.com; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Open port and services:

    53/tcp — domain: Simple DNS Plus
    80/tcp — http: Apache httpd 2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
    88/tcp — kerberos-sec: Microsoft Windows Kerberos
    135/tcp — msrpc: Microsoft Windows RPC
    139/tcp — netbios-ssn: Microsoft Windows netbios-ssn
    389/tcp — ldap: Microsoft Windows Active Directory LDAP (Domain: access.offsec0)
    443/tcp — ssl/http: Apache httpd 2.4.48 (Win64) OpenSSL/1.1.1k PHP/8.0.7
    445/tcp — microsoft-ds: Unknown service (likely SMB)
    464/tcp — kpasswd5: Unknown service
    593/tcp — ncacn_http: Microsoft Windows RPC over HTTP 1.0
    636/tcp — tcpwrapped: Secure LDAP (SSL/TLS)
    3268/tcp — ldap: Microsoft Windows Active Directory LDAP (Global Catalog)
    3269/tcp — tcpwrapped: Secure LDAP (SSL/TLS)
    5985/tcp — http: Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

Port 80

The first step in our exploration was to investigate the web service running on port 80 . This port hosted an Apache HTTP server (Apache httpd 2.4.48) with PHP support (PHP/8.0.7).

When accessing the website, we discovered that it is a platform designed to sell tickets for a marketing conference 

lets run ffuf
```bash
ffuf -u http://192.168.56.187/FUZZ -w directory-list-2.3-medium.txt -fc 404,403
```

<img width="801" height="45" alt="image" src="https://github.com/user-attachments/assets/5e82c274-56ca-4daa-af9b-296a93237863" />

<img width="814" height="17" alt="image" src="https://github.com/user-attachments/assets/9974b0ee-df0c-4400-910f-425a7aa1019d" />

<img width="1136" height="286" alt="image" src="https://github.com/user-attachments/assets/12ec7503-58b9-4d3d-9ac4-1485c69ffc6b" />

<img width="1445" height="765" alt="image" src="https://github.com/user-attachments/assets/34c8d2af-34b0-4e43-b913-cd00ee6ff20d" />

<img width="1443" height="761" alt="image" src="https://github.com/user-attachments/assets/63b1288d-2cbd-43fe-8d3c-7f1ce2dc213b" />

Upload Feature

We can access an upload feature on the website when we click “Buy ticket” → “Buy Now”

This could be a File Upload attack.

<img width="501" height="172" alt="image" src="https://github.com/user-attachments/assets/da317932-f93c-452f-95e7-ca3892694448" />

After the file is uploaded, it will be available in the ‘uploads’ directory.

<img width="1146" height="324" alt="image" src="https://github.com/user-attachments/assets/6615439e-1099-4cfe-9fbb-a51431f36851" />

We are able to access it directly via the browser.
<img width="1160" height="321" alt="image" src="https://github.com/user-attachments/assets/b98e1fe7-94a1-4ab6-acb7-3f3e8fb8e52f" />


We can bypass file upload restriction by leveraging the .htaccess file. Uploading a malicious .htaccess file, we were able to override the server's file extension restrictions, allowing the execution of unauthorized file types, such as PHP files. This misconfiguration could enable an attacker to upload and execute arbitrary code, potentially leading to remote code execution (RCE) and full compromise of the web server.

Let’s try a the PHP Ivan Sincek shell from https://www.revshells.com/

<img width="1424" height="965" alt="image" src="https://github.com/user-attachments/assets/a61cf630-8537-4799-9f1c-477a3553370d" />


Enter the appropriate tun0 IP address and I’ll use port 443. Copy the code and save it (shell.php) and try to upload it.

<img width="1443" height="811" alt="image" src="https://github.com/user-attachments/assets/7dbdb95a-7120-47d1-8504-54f27b685094" />

We are redirected and discover that they are wise to our ways. PHP is not allowed.

<img width="480" height="147" alt="image" src="https://github.com/user-attachments/assets/845ce645-c02a-425c-bd01-322c23180ec8" />

We are then shuffled back to the home page where we are free to try another file type. By exhaustion, we learn the none of the PHP type extensions will be accepted. Here’s where it gets fun!

We create our own .htaccess file that outlines a NEW PHP file type and upload it.

```bash
echo "AddType application/x-httpd-php .pwn" > .htaccess
```
Be aware that to be able to select this file for upload, you will have to change your view setting with a right-click after browsing to its location.

<img width="559" height="391" alt="image" src="https://github.com/user-attachments/assets/0b4b9868-40d3-4bd8-a64b-fab6153dfd62" />

<img width="475" height="150" alt="image" src="https://github.com/user-attachments/assets/7e14afe0-9f82-4111-8b77-b96435d7c359" />

Change the name of your shell.php file to end in the new pwn extension type.

<img width="490" height="125" alt="image" src="https://github.com/user-attachments/assets/e48f0760-f353-4ce6-a3aa-1707cd824325" />


Now we will be able to upload it. However, we should set up our listener first.

<img width="559" height="393" alt="image" src="https://github.com/user-attachments/assets/fd5333bc-9729-4230-991a-4d41ab8abf8f" />


```bash
sudo rlwrap nc -lnvp 443
```

<img width="307" height="79" alt="image" src="https://github.com/user-attachments/assets/1de1352d-4a8b-4b61-b200-4bd737120c9f" />

I use rlwrap so that I retain use of my arrow key functionality upon connection.

Now finally we upload the shell.pwn file.

<img width="560" height="392" alt="image" src="https://github.com/user-attachments/assets/30ce6605-f053-4eb9-a68a-1a309f497a16" />

<img width="479" height="151" alt="image" src="https://github.com/user-attachments/assets/6ff49a00-517f-4739-9967-5c22956d0f66" />

Now go back to the /uploads directory and see file uploaded sucessfully.

<img width="973" height="256" alt="image" src="https://github.com/user-attachments/assets/f1a2b14f-c413-4c1f-b776-60403f35e907" />

Click on our uploaded shell and notice that the browser hangs and will not resolve. This is promising. Check the listener







lab doesnt work will do it later

