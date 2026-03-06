lab ClamAV - easy

target 192.168.55.42

recon

```bash
sudo nmap -sC -sV clamav -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV clamav -sT -vvv -p- -Pn -T5 --min-rate=5000

PORT    STATE SERVICE     REASON  VERSION
22/tcp  open  ssh         syn-ack OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
25/tcp  open  smtp        syn-ack Sendmail 8.13.4/8.13.4/Debian-3sarge3
80/tcp  open  http        syn-ack Apache httpd 1.3.33 ((Debian GNU/Linux))
| http-methods:
|   Supported Methods: GET HEAD OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Ph33r
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
139/tcp open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp open  smux        syn-ack Linux SNMP multiplexer
445/tcp open  netbios-ssn syn-ack Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel
```

Port 80
There’s a webserver (Apache 1.3.33) hosted on the box, and the default page shows the following binary code:

<img width="1919" height="129" alt="image" src="https://github.com/user-attachments/assets/97117bda-24c0-4598-b0dc-dde5971f59ab" />

I went ahead and ran a searchsploit search for all the versions, and did find one that seemed interesting for the Apache server:

```bash
searchsploit "Apache 1.3.33"
```
<img width="1044" height="144" alt="image" src="https://github.com/user-attachments/assets/f17d9ee6-a114-4418-a86a-42a84b39821a" />


i’ll note it for now but enumerate further.. i see an unusual high number port:

<img width="924" height="29" alt="image" src="https://github.com/user-attachments/assets/1a22a0a1-859b-443d-8de2-bb1735f2f80f" />

So for now we have a SMB server and an Apache webserver. I’ll check port 80 first.


Initial Foothold
80/TCP - HTTP

Going to the site I found nothing of interest:

<img width="811" height="26" alt="image" src="https://github.com/user-attachments/assets/1ebf69bd-829a-4326-9215-952a467b9bed" />

I converted the binary to text: : ifyoudontpwnmeuran00b


199/TCP - SNMP

<img width="637" height="26" alt="image" src="https://github.com/user-attachments/assets/010db856-2984-49e8-bc43-3237ffcf77b6" />

I went ahead and enumerated this port using snmp-check:

```bash
snmp-check 192.168.55.42
```
<img width="845" height="134" alt="image" src="https://github.com/user-attachments/assets/0248c515-e39a-4726-b609-ccd37ca3c8c2" />

Scrolling a bit down I found this:

<img width="1058" height="37" alt="image" src="https://github.com/user-attachments/assets/72fda1c6-ceaf-4bb4-bf08-7f60bb5bd7a0" />

I looked it up to see if there’s a PoC for it.

<img width="1373" height="859" alt="image" src="https://github.com/user-attachments/assets/6bf8e032-6ae9-4b81-b86d-a9b22a00cb60" />

There appears to be a RCE PoC available. Let’s try it out.

first start listner in another terminal
```bash
nc clamav 31337
```
and then 
```bash
perl black-hole.pl clamav
```

<img width="1716" height="518" alt="image" src="https://github.com/user-attachments/assets/2d049101-b97e-49f3-9550-fab8235c6165" />

boom we're done

