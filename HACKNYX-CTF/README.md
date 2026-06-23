# HACKNYX CTF Writeup Summary

Simple version of the writeup. This focuses on the thinking process, tools used, and how each flag was reached. No payloads or shell commands are included.

> Note: Some flags in the original writeup are written as `HYNX{...}` instead of `HNYX{...}`. Keep the exact flag format shown by the challenge when submitting.

---

## Reverse Engineering

### Lang of Roblox

**Tools to use:** Lua bytecode decompiler, strings viewer, hex editor.

**Thought process:**
The challenge name points to Roblox, which commonly uses Lua. The goal is to inspect or decompile the Lua bytecode and check the recovered constants or readable strings. Once the script logic is readable, the flag can be found directly.

**Flag:** `HNYX{lua5_byt3c0d3_g0t_d3c0mpil3d}`

---

### Sanity Check

**Tools to use:** Challenge platform, Discord/rules page, browser.

**Thought process:**
This is a basic warm-up challenge. The flag is usually placed in the challenge description, rules page, announcement, or CTF platform page.

**Flag:** `HNYX{771cd10bc38e5c6141fef689daf5a0b7}`

---

### The Confused Donut

**Tools to use:** dnSpy, Detect It Easy, FLOSS, capa, shellcode analysis tools, .NET decompiler.

**Thought process:**
The executable is a .NET Windows Forms program protected with ConfuserEx. Initial analysis shows suspicious API usage, suggesting the program loads hidden shellcode. Inside dnSpy, the program performs a Windows username check before continuing. Instead of guessing the username, the branch logic is patched to bypass the check.

After bypassing it, the program reveals that the next stage is hidden in an embedded resource named `d0nut_glaz3`. That resource is reversed, Base64-encoded data. After decoding it, the result is raw Donut shellcode. The shellcode contains an embedded .NET DLL. Inspecting the extracted DLL reveals the flag as a UTF-16 string.

**Flag:** `HNYX{c0nfu$1nG_tH3_d0nUt_4_c0ff33}`

---

## Forensics

### General Forensic Flag

**Tools to use:** Not enough information provided in the writeup.

**Thought process:**
The PDF lists a forensic flag but does not include the challenge name or solving steps for this specific item.

**Flag:** `HNYX{74ccdaf686bb385d2b54e56b3426bda5}`

---

### Ghost in the RAM

**Tools to use:** 7-Zip, strings tool, memory string search, BitLocker image tools, Sleuth Kit, hex editor, CyberChef.

**Thought process:**
The challenge gives two artifacts: an encrypted disk image and a Windows memory dump. Because the clue mentions a text editor, the memory dump should be searched for text editor activity, saved notes, passwords, and BitLocker-related strings.

The memory contains a note showing the BitLocker password. That password is used to decrypt the disk image. After decryption, the NTFS partition is inspected and a suspicious image file is extracted.

The extracted image has a corrupted header, so the file signature needs to be repaired. After repairing it, the image opens correctly, but the real flag is hidden after the JPEG end marker. The hidden data is Base64 encoded, so decoding it gives the final flag.

**Flag:** `HNYX{d4S_cR4zy!}`

---

### Fr1dg3OS

**Tools to use:** Sleuth Kit, Linux disk mounting tools, LVM tools, strings tool, LUKS tools, SQLite browser, CyberChef.

**Thought process:**
The challenge provides a disk image and a memory dump from a smart fridge. The disk image contains an LVM-based Linux filesystem. After mounting the filesystem, a suspicious systemd service named `fridge-vault.service` is found.

The service points to a LUKS-encrypted vault and shows that the vault password was stored in a runtime environment variable. The environment file was deleted from disk, so the password must be recovered from memory.

Searching the memory dump reveals the real runtime token. That token unlocks the LUKS vault. Inside the vault is a SQLite telemetry database. The database had a deleted Base64 evidence blob, but because secure deletion was not enabled, the deleted value is still recoverable from the raw database content. Decoding the Base64 value gives the flag.

**Flag:** `HNYX{fr1dg3_m3m0ry_k3pt_1t_c0ld}`

---

## Web

### Warm Up 1

**Tools to use:** Browser, Burp Suite, SQL error analysis.

**Thought process:**
The product search page leaks SQL errors. By intentionally triggering an SQL error, the application exposes useful information and eventually displays the flag through the error/output behavior.

**Flag:** `HNYX{MySqL_3rR0r_4nYwh3R3???!!}`

---

### Warm Up 2

**Tools to use:** Browser, Burp Suite, SQL injection testing.

**Thought process:**
The product search is vulnerable to SQL injection. The goal is to use UNION-based SQL injection to control what appears in the product results. The flag is stored in another database/table and can be displayed by selecting it through the injection.

**Flag:** `HNYX{D0_y0u_3xPeCt_th3_fl4g_1s_ln_4n0th3r_d4t4B@s3??}`

---

### Warm Up 3

**Tools to use:** Browser, Burp Suite, SQL injection testing.

**Thought process:**
The login form is vulnerable to SQL injection. Instead of knowing the admin password, the query can be manipulated so that it matches an admin-like username. Once logged in as admin, the flag is visible in the dashboard or response.

**Flag:** `HNYX{3nD_0f_w4rM_uP_cH4LleNg3_G00dLucK_h4ck3rS!}`

---

### No Slang

**Tools to use:** Burp Suite, browser, MySQL knowledge, Jinja/SSTI knowledge.

**Thought process:**
The login form uses a blacklist, but the filtering is incomplete. The operator and keycode fields can be abused together to break the SQL query and inject controlled output. The injected value later gets rendered by the template engine, creating a second vulnerability: server-side template injection.

The solve chain is SQL injection first, then template injection. After reaching template execution, the flag-reading functionality is triggered through the vulnerable rendering path.

**Flag:** `HYNX{b4cKsL4sH_br34Ks_7h3_Ch41n}`

---

### Middle Management

**Tools to use:** Browser, Burp Suite, Next.js middleware/auth bypass knowledge.

**Thought process:**
The challenge is about trusting middleware too much. Middleware checks can sometimes be bypassed if the real protected route does not enforce authorization again. By reaching the internal/admin route directly through the right request path or headers, the admin console becomes visible and shows the production secret.

**Flag:** `HYNX{m1ddl3w4r3_1snt_4n_4uth_b0undary}`

---

### No Slang 2

**Tools to use:** Burp Suite, browser, MySQL knowledge, Jinja/SSTI knowledge, error analysis.

**Thought process:**
This challenge continues the SQL injection and SSTI idea but adds more filtering. The first useful result is an error message that leaks the internal path. That path helps confirm where the application is running and how the flag file can be reached.

The solve requires bypassing the blacklist, using SQL injection to place a template payload into the rendered page, and then using the template engine to read the flag.

**Flag:** `HYNX{1nF0_sCh3m4_l34Ks_3v3RyTh1nG}`

---

### Wisadel

**Tools to use:** Burp Suite, SOAP request editor, PostgreSQL SQL injection knowledge.

**Thought process:**
The application exposes a SOAP endpoint. The SOAP body is passed into a backend PostgreSQL query without proper sanitization. By modifying the XML request, SQL injection becomes possible. The injection is used to read server-side data and retrieve the flag.

**Flag:** `HNYX{s04P_P0stgr3s_sUcC3sS_rC3_Pwn3d}`

---

### Import

**Tools to use:** Burp Suite, XML parser knowledge, external DTD concept, public callback server/tunnel, CyberChef.

**Thought process:**
The import feature parses XML unsafely, making it vulnerable to blind XXE. Since the flag is not shown directly in the page response, an external DTD is used to make the server send the flag data out to a callback server.

The leaked value arrives Base64-encoded. Decoding the callback data reveals the flag.

**Flag:** `HNYX{Bl1nD_XmL_t0_Lf1_w1tH_0Ob_t3cHn1Qu3}`

---

### No Slang 3

**Tools to use:** Burp Suite, browser, MySQL knowledge, SHA256/hash verification understanding, Jinja/SSTI knowledge.

**Thought process:**
This version adds a stronger server-side check: the returned database row must match the submitted operator and the SHA256 hash of the submitted keycode. A normal SQL injection is not enough because the post-query verification will fail.

The key idea is to make the SQL injection return exactly the same operator value and a hash that matches the exact submitted keycode. After bypassing login, the operator value is rendered by Jinja, allowing server-side template injection. That final template execution path is used to read the flag.

**Flag:** `HYNX{h4sH_qU1n3_m4sT3rY_unL0cK3d}`

---

### PrintNightMare

**Tools to use:** Not enough information provided in the writeup.

**Thought process:**
The PDF only lists the challenge name and category area, but does not include the solving steps or final flag for this item.

**Flag:** Not provided in the writeup.

---

## OSINT

### Highway 1

**Tools to use:** Google Maps, Google Street View, Zoomyd highway map.

**Thought process:**
The image shows a Malaysian highway scene. Match the road view with highway map references and Street View. Once the correct location is found, submit the latitude and longitude rounded to three decimal places.

**Flag:** `HNYX{2.284,102.367}`

---

### Highway 2

**Tools to use:** Google Maps, Google Street View, Zoomyd highway map.

**Thought process:**
Use the highway map and kilometer marker clue to identify the correct road section. The writeup points to the East Coast route around kilometer marker 143. After matching the map position and Street View, the coordinate is submitted in the required format.

**Flag:** `HNYX{3.555,102.547}`

---

### Highway 3 / Find Me

**Tools to use:** Username search, Chess.com, TikTok, Google Maps, What3Words.

**Thought process:**
Start from the given identity or username and pivot across public profiles. The writeup shows the target username leading to social media activity. A repost or location clue points to a place. After identifying the place, convert the location into What3Words format and submit the three-word location.

**Flag:** `HNYX{tripped.boarding.playing}`

---

### Deleted Tweet

**Tools to use:** X/Twitter, deleted tweet lookup, Wayback Machine, CyberChef/Base64 decoder.

**Thought process:**
Start with the username `dk_cloudnomad`. Search for deleted or archived tweets. The archived tweet leads to exposed configuration-style text. One value in the config is encoded. Decoding that value reveals the flag.

**Flag:** `HNYX{@rch1v3$_n3ver_f0rg3t}`

---

## Boot2Root / Pwn

### Transfer

**Tools to use:** Nmap, FTP client, SSH client, Linux enumeration checklist.

**Thought process:**
Enumeration shows an FTP service that allows access to useful files. The files include a user flag and SSH material for logging in as the low-privileged user.

After logging in, system enumeration reveals a writable cleanup script in a privileged location. The important observation is that the script is owned or writable by the current user but executed later by root. By abusing that scheduled execution path, privilege escalation is achieved and the root flag is recovered.

**User flag:** `HNYX{AN0N_M1SC0NF1G_FTP}`

**Root flag:** Present in the screenshots, but not clearly written as text in the parsed writeup.

---

### Nocturnal

**Tools to use:** Nmap, browser, Burp Suite, file-read testing, Linux process awareness.

**Thought process:**
Enumeration shows web services and indicates that PhantomJS is involved. The key issue is an exposed file-read behavior running with high privileges. By testing likely file paths, web files and user files can be read. Since the process runs as root, the root flag can also be reached through the same file-read weakness.

**User flag:** Present in the screenshots, but not clearly written as text in the parsed writeup.

**Root flag:** Present in the screenshots, but not clearly written as text in the parsed writeup.

---

### Glaux

**Tools to use:** Nmap, browser, Burp Suite, PHP version research, Docker awareness, SSH client, Linux privilege escalation checklist.

**Thought process:**
Enumeration finds SSH, nginx, and a PHP development server. The PHP server is running the vulnerable PHP 8.1.0-dev build, which allows command execution through a backdoored HTTP header.

Initial access lands inside a Docker container as root. Since container root is not always host root, the filesystem and history are checked for credentials. Leaked SSH credentials allow login to the host as user `strix`.

On the host, privilege escalation comes from a sudo misconfiguration that allows `strix` to run Wine as root without a password. That misconfiguration is abused to obtain root-level access and read the final flag.

**User flag:** `HNYX{30FD4C6999FED6EB19179D323D0C502DED96C2D7DC949D70}`

**Root flag:** `HNYX{026EBC25E942AE296D96E25B6C0832365603923F250C41E8}`

---

## Overall Pattern

Most of the challenges follow a clear CTF workflow:

1. Identify the file type, service, or clue.
2. Use the correct tool for the category.
3. Look for weak validation, hidden data, leaked secrets, or bad configuration.
4. Decode or extract the final hidden value.
5. Submit the flag exactly as shown.

Recommended general tools:

- **Reverse:** dnSpy, Detect It Easy, FLOSS, capa, strings, hex editor.
- **Forensics:** Sleuth Kit, Volatility-style memory analysis, strings, CyberChef, SQLite Browser, hex editor.
- **Web:** Burp Suite, browser DevTools, SQL injection methodology, SSTI/XXE knowledge.
- **OSINT:** Google Maps, Street View, Wayback Machine, username search, What3Words.
- **Boot2Root:** Nmap, FTP/SSH clients, Linux enumeration checklist, privilege escalation checklist.
