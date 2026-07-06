# LIGA CTF 2026 



Format used:

`Clue / Observation → Reasoning → Action → Confirmation → Result`

This version focuses on *how the solve direction was chosen* for each challenge, not full command dumping.

---

## Reverse Engineering

### Free Point
`Challenge gives free flag → no technical trick expected → submit provided value → accepted → warm-up solved`

### Unpackme0
`Check file type/strings → UPX indicators appear → binary is packed → unpack first before deeper reversing → strings become readable → flag found`

### Unpackme1
`Looks like UPX again → magic value looks corrupted as VQY instead of UPX → likely anti-unpack patch → repair magic bytes → decompress again → hidden strings/flag become readable`

### Unpackme2
`UPX pattern does not fit → packer indicators point to ASPack → normal UPX flow will fail → use ASPack-compatible unpacking → inspect dumped binary → flag appears after unpacking layer removed`

### Lockbox
`Normal unlock argument exists → strings show another odd hidden argument → suspicious string looks encoded → reverse it first → apply ROT13 → emergency key recovered → hidden unlock path prints flag`

### Find the C2 Server
`File is APK → treat as Android app, not normal binary → inspect Manifest for permissions/entry point → internet permission confirms network behavior → suspicious backdoor class found → decompile with JADX → C2 URL/flag clue exposed`

### Detonate2
`File name says .exe → headers show Linux ELF → Windows path in strings looks suspicious → Linux treats backslashes as normal characters → create literal path filename → program prints hash but platform rejects it → strace shows only file-existence check, no real file read → output is decoy → hash the extracted target path string manually → real flag recovered`

### Deadlocker
`Strings contain decoy → decompile algorithm instead → LCG formula discovered → only low byte is used → effective state is only 8-bit → known flag prefix gives keystream start → brute-force 256 seeds → decrypt full ciphertext → flag recovered`

### Codec-auth
`Strings suggest AES-GCM → challenge wording hints AES-GCM is distraction → inspect Python bytecode/call flow → l33tspeak functions are real validators → custom SPN/block-cipher logic found → recover required body constraints → construct valid token/flag`

### Atari_Breakout
`Strings/binwalk show embedded PNG and JPEG → automatic binwalk extraction gives messy zlib output → tool is carving wrong boundary → use exact offsets manually → carve PNG and JPEG separately → JPEG is distraction → PNG visually contains flag`

### G00fyF1NM4Ch1N3
`Program expects 16-character password → validator split into many transition functions → each function checks one character/state → reconstruct state machine instead of blind brute force → recover password sequence → run with password → flag printed`

### memdiag.ai
`Runtime prints fake flags → Ghidra shows hidden BMP dump path → dump only happens after magic value and environment checks → normal execution blocked by host validation → use debugger to skip/force success path → BMP generated → raw BMP text looks scrambled → remember BMP stores pixels BGR → swap each 3-byte group to RGB → real flag appears`

### Wraithlocker
`Base64 flag in strings is fake → visible key fragments are misleading → decompile crypto flow → anti-debug + nonce blender + LCG stream cipher discovered → reconstructed key still fails → realize memory read includes null bytes and Docker ptrace poisons key → stop trying to emulate environment perfectly → use known plaintext OWASPKL{ → recover LCG keystream/seed → decrypt ciphertext → final flag`

---

## Malware Analysis

### Malware-1
`Run static strings → Go runtime/protobuf/gRPC patterns appear → function names match C2-style capabilities → obfuscation resembles Garble → compare indicators with known frameworks → Sliver traits match best → answer is framework family`

### Malware-2
`Normal strings are noisy → use decoded-string extraction mindset → realistic host:port stands out → hostname alone does not reveal IP → run in controlled FLARE VM → capture traffic in Wireshark → outbound 443 connection reveals actual C2 IP → submit IP`

---

## Boot2Root

### B2R - Easy
`Start with full port scan → FTP/SSH/HTTP/HTTPS exposed → anonymous FTP is low-hanging fruit → creds.txt found → try creds on SSH → shell obtained → sudo permissions checked → allowed cat/ls as root → read root flag`

### Spray and Pray - 1
`Provided VM disk → likely offline credential recovery → mounting has issues → extract VMDK directly → Linux /etc/passwd + /etc/shadow found → combine hashes → hash type is yescrypt → use John instead of forcing Hashcat → weak password cracked → login as user → first flag found`

### Spray and Pray - 2
`Cracked password also works for root → password reuse confirmed → no need complex lateral movement → search /home globally for text/flags → second user desktop flag found`

### Spray and Pray - 3
`Root access already available → expected final proof in /root → enumerate root home → read final flag → machine fully compromised`

### Routine - 1
`Nmap shows Grafana on port 3000 → hint mentions MySQL plugin → Grafana plugin traversal becomes likely → test LFI using plugin path → /etc/passwd confirms file read → download Grafana SQLite DB → extract stored credentials → SSH login → user flag`

### Routine - 2
`User shell obtained → sudo/SUID checks not useful → inspect scheduled tasks → root cron runs user-writable Python script → replace script with controlled action → wait for cron → root file copied to readable path → root flag obtained`

### Chain of Attacks - 1
`OVA booted → login clue gives possible username → scan shows IMAP/web services → IMAP often stores secrets → brute/test mailbox creds → emails reveal RiteCMS path/version and rotated creds → login CMS → file manager allows PHP upload → web shell works → search web files → local flag found`

### Chain of Attacks - 2
`Web shell already exists → search for more credentials/databases → users.db found → creds work on port 9090 admin panel → need interactive access → create reverse shell from web shell → use panel/admin terminal → enumerate as privileged user/root → proof file found → flag extracted`

### The Art of Evasion and Persistence - 1
`Identify Windows target IP → scan shows FTP/IIS/XAMPP/SMB → FTP allows anonymous access → download files → dump/crack hashes → password for webadmin recovered → SMB login works → user share contains flag`

### The Art of Evasion and Persistence - 2
`Need map hash to user → test pass-the-hash over SMB → hash belongs to ligac → SMB access as ligac → pull NTUSER.DAT → registry strings reveal CMS/RiteCMS clue → login RiteCMS with known creds → special content function execution enabled → test whoami → command execution works → run flag-reading payload → flag recovered`

### REBORNE
`Discover target with netdiscover → scan shows FTP/SSH/HTTP → FTP anonymous gives PDF/hint → hint is rickroll decoy → move to web enumeration → robots/nested paths reveal external GitHub page and image → steghide image gives phrase but not SSH password → external page contains themed leetspeak words → build custom wordlist → Hydra SSH for apokalips → sudo -l shows dash as root → spawn root shell → read hidden root flag`

### FRAGNESIA
`Only HTTP open → test input for stored XSS → alert confirms stored XSS → direct admin command attempt returns Access denied → RCE requires admin session → enumerate admin_login.php → weak themed admin credential found → stored XSS logs in as admin first → same session posts cmd to admin.php → reverse shell as www-data → find flags on filesystem → collect all flags`

---

## Forensic, Log Analysis, OSINT

### Malware Investigation - 1
`Need initial malicious command/process → Sysmon Event ID 1 records process creation → filter process creation logs → phc.exe stands out and links to ligac/artifact context → submit malicious executable name`

### Malware Investigation - 2
`Known stager is phc.exe → inspect its command line/process behavior → command indicates injection/hijack target PID → PID 10424 identified → submit PID`

### Malware Investigation - 3
`Question asks process name for hijacked PID → filter logs by PID 10424 → image name resolves to M365Copilot.exe → submit process name`

### Malware Investigation - 4
`Need payload retrieval URL → DNS events show queried domain → file creation events show payload filename maindll.dll → network connection events show port 8443 → combine scheme + domain + port + DLL path → full payload URL reconstructed`

### Malware Investigation - 5
`Payload filename already known → file creation logs include path → Temp directory location identified → submit full local DLL path`

### Malware Investigation - 6
`Now focus on post-injection C2 → filter DNS from hijacked process → M365Copilot.exe queries owaspkl domain → submit C2 domain`

### Malware Investigation - 7
`Hijacked process spawns cmd.exe → follow timeline after that process creation → Event ID 1 shows commands executed → collect reconnaissance commands in order → submit command flags`

### Malware Investigation - 8
`Event logs show Cloudflare proxy IPs → proxy IPs are not origin C2 → pivot to VirusTotal relations for domain infrastructure → related non-Cloudflare origin IP discovered → submit origin IP`

### Malware Investigation - 9
`Need hash of actual payload → URL reconstructed earlier → download maindll.dll → calculate SHA-256 → submit file hash`

### OSINT — A bit to add
`Shortened link opens visual clue → image/post shows relative time → convert “8 days ago” using challenge context/date → exact date answer formed`

### OSINT — 2 Strands
`Prompt hints Alexei Pasler + Russian social platform → reverse image search/visual matching → monument identified as lab mouse knitting DNA → get plus code from Maps → convert local plus code to global format → submit location code`

### OSINT — Over the Phone
`Audio uses NATO alphabet → transcribe letters → CLASSIFIED recovered → use as password → each extracted challenge asks one OSINT fact → answer becomes next password → continue chain until final decoded material gives flag`

---

## Web

### Keluar
`Homepage source has nothing obvious → check robots.txt as basic web recon → hidden path discovered → second page source contains scattered green Base64 fragments → decode fragments → combine in order → flag reconstructed`

### Novita
`page parameter loads content → test ../ traversal → homepage/file read behavior confirms LFI → /etc/passwd confirms arbitrary read → probe web-root paths → flag displayed from included file/path`

### Nate
`Cookie value looks encoded → decode cookie → role/user state found → change value to admin → re-encode cookie → visit admin.php → access control bypassed → flag shown`

### Laufey
`Writeup only records the final flag → solving observations are missing → cannot accurately build arrow flow without inventing steps → need original notes/screenshots for this one`

### Goat2 / Imej-Anda
`Writeup has almost no solution detail → only challenge name/flag area present → thought process cannot be reconstructed safely → need missing notes such as parameter tested, vulnerability found, and confirmation step`

### The Goat
`Cloudflare note appears in writeup → likely trust/proxy logic involved → compare target behavior with Cloudflare IP/header assumptions → craft request to satisfy server-side trust check → protected content/flag reached`

### Blindless
`Only challenge name appears in the writeup → no observable clue, action, or confirmation step provided → arrow flow cannot be summarized accurately from current document`

### Courier
`Import API accepts URL → server fetches document → direct API access denied by network trust rule → clue says Cloudflare-only → use Cloudflare Worker as allowed proxy → confirm clean SVG converts to PNG → renderer is ImageMagick → SVG label:@ can read local files → render /etc/hostname first to confirm LFI → render /flag.txt → flag appears inside output PNG`

### The Amazing Digital Adventure
`Typing game looks skill-based → inspect obfuscated JavaScript → endpoints and hidden KONAMI string found → beautify and trace functions → Konami code triggers auto-type → client already knows correct lyrics and submit flow → activate cheat path → game auto-submits segments → backend returns flag`

### Quack
`/flag endpoint likely requires strange condition → challenge theme is quack → test headers instead of only URL/body → send quack across common headers → server accepts request → JSON flag returned`

---

## Labyrinth

### Scada — Plus 1
`SCADA login page exposed → try known/default SCADA credential first → citect:citect works → authenticated panel reveals flag`

### Scada — Plus 2
`Service speaks Modbus TCP → challenge likely hides state in coils/registers → write specific coil pattern → read holding registers → register bytes decode to ASCII → flag recovered`

### Windows — Tempatan
`Windows artifact challenge → check Credential Manager → cmdkey lists saved TERMSRV credential → stored credential contains flag`

### Windows — Sejarah / Always Clean Your Types
`Clue suggests history/deleted artifact → inspect Recycle Bin → suspicious reversed string found → reverse string → flag restored`

### Windows — Berlari
`Berlari/run clue suggests autorun persistence → inspect Registry Run key → suspicious SvcHelper entry found → associated artifact contains flag`

### Windows — Saluran
`Saluran/channel clue suggests named pipes → inspect suspicious binary → NamedPipeServerStream/PipeServer strings found → encoded reversed flag present → decode + reverse → flag`

### Windows — Fire
`Suspicious uncommon DLL in System32 → inspect DLL strings/metadata → embedded flag-like value found → extract flag`

### Windows — Old Feature
`Old Windows feature clue → check COM/CLSID persistence → InprocServer32 points to fakehelper.dll → default value is hex → decode hex → flag`

### Windows — Daftar
`Daftar means registry → inspect user registry hive → Console key contains FlagToken → Base64 decode value → flag`

### Android Reverse
`APK has encrypted assets + native library + fake flags → fake flags are decoys → find decryption routine → AES-CBC key derived by SHA-256(fragA+fragB+fragC) → two fragments local → Telegram bot clue gives third fragment → combine fragments → derive AES key → decrypt guestlist.enc → real flag`

---

## APT / Persistent Threat Emulation

### Asset Discovery and Attack Surfaces
`Start with known Valere domains → enumerate subdomains/services → identify staff portal, labs portal, AWS login, service pages → map exposed attack surface → choose most promising login/app targets for exploitation`

### T1213.003
`AWS login form takes username/password → test username parameter → SQLi confirmed → move from vulnerability to data objective → enumerate DB/tables/columns → internal_communications table looks valuable → UNION dump internal communications → flag and reusable credentials discovered`

### Cloudflare Origin Discovery
`Domain resolves to Cloudflare → Cloudflare IP means proxy, not real server → compare IPs against Cloudflare ranges → non-Cloudflare IP stands out → treat it as origin → use for direct service access/SSH stage`

### T1090.001
`SSH as valere obtained → enumerate local history/artifacts → shell history leaks Patricia email, company registration ID, and reused password → internal Outlook only reachable from inside network → create SSH local tunnel → access Outlook through localhost → login as Patricia → mailbox has provisioning emails → kdjebat account and temporary access key found → discovered credential submitted`

### Phantom DLL
`Challenge name hints DLL hijacking → first DLL name guess fails because output file not created → failure means DLL not loaded, not payload logic issue → build minimal DLL that writes test marker → try likely phantom DLL name wlbsctrl.dll → success confirms correct DLL name and x64 architecture → replace test payload with flag-search payload → DllMain auto-runs on load → payload searches common flag paths → writes result to C:\output\stolen.txt → flag found in C:\flag.txt`

---

## Quick Pattern Notes

`Weird string → try simple transforms first: reverse, ROT13, Base64, hex`

`Packed binary → unpack before deep reversing`

`Runtime output rejected → trace behavior and check for decoys`

`Known flag prefix → use it for XOR/stream-cipher/LCG recovery`

`Web import/fetch feature → think SSRF/file rendering issues`

`Internal-only service → use SSH tunneling after foothold`

`Windows persistence clue → check Run keys, COM hijack, DLL loads, Credential Manager`

