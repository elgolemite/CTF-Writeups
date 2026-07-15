# UiTM Cyberheroes Internal CTF - Challenge Summary

> Team: **Hati2Hati**  
> Purpose: Quick-reference summary of each challenge's main theme, solving idea, and tools.  
> Flags are intentionally omitted so this document stays focused on techniques.

## Competition Overview

| Category | Challenges | Main skills covered |
|---|---:|---|
| Web | 8 | Web enumeration, IDOR, XSS, SQLi, upload bypasses, token forgery, CVE exploitation |
| OSINT | 6 | Reverse image search, geolocation, archive research, historical imagery |
| Misc | 6 | Esolangs, steganography, platform observation, prompt injection, game logic |
| Forensic | 8 | Office macros, memory strings, PCAP analysis, disk recovery, malware triage |
| Reverse Engineering | 7 | ELF/PE analysis, custom VMs, PRNGs, bytecode, ransomware |
| Pwn | 6 | Ret2win, format strings, shellcode, ret2libc, heap exploitation, ROP |
| Crypto | 3 | Modular algebra, RSA common factors, layered classical decoding |
| **Total** | **44** | **44 challenge summaries** |

## Core Tool Stack

| Area | Common tools |
|---|---|
| Web | Browser DevTools, Burp Suite, curl, CyberChef, Python |
| OSINT | Google Search, Google Lens, Maps, Street View, Earth, GitHub |
| Forensic | oletools, Wireshark, Sleuth Kit, ewf-tools, strings, grep |
| Reverse | file, nm, strings, objdump, readelf, xxd, Ghidra, Dogbolt |
| Pwn | checksec, GDB/pwndbg, ROPgadget, pwntools |
| Crypto | Python, modular arithmetic, RSA utilities, CyberChef |

## Web

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **Baby Web** | Information disclosure / hidden endpoint | Checked `robots.txt`, opened the hidden path, inspected the HTML source, and decoded a Base64 value. | Browser, DevTools, View Source, CyberChef |
| **OTP Can Be Broken** | OTP brute force / missing rate limiting | Found a valid username, requested a password reset, captured the session cookie, and brute-forced the 4-digit OTP because the endpoint had no attempt limit. | Browser DevTools, Burp Suite or curl, Python, cookies |
| **Guest Book** | Stored XSS / admin-bot interaction | Stored a JavaScript payload in the guestbook and triggered the admin bot so its browser submitted protected data back into the application. | JavaScript, curl, Browser DevTools, admin bot |
| **Unhive** | IDOR and insecure password recovery | Changed the exposed profile ID to access the admin profile, obtained the security-answer data, reset the admin password, and logged in. | Browser, URL manipulation, DevTools |
| **Unhive2** | Unrestricted file upload / null-byte filename bypass | Modified a normal image upload in Burp so a PHP payload was stored with a crafted `.jpg%00.php` name, then used double encoding to request and execute it. | Burp Suite Repeater, curl, PHP web shell |
| **Array Escape** | Secret leakage, token forgery, parameter pollution and path traversal | Recovered the HMAC signing secret from client-side data, forged an admin token, then supplied the `file` parameter twice so Express parsed it as an array and bypassed path validation. | Browser source, CyberChef, curl, Python HMAC |
| **SQLI MAYBE?** | SQL injection filter bypass | Tested individual characters, discovered that MySQL accepted `\|\|` as logical OR, and used a balanced payload because comments and spaces were blocked. | Browser, Burp Suite, curl, manual SQLi testing |
| **Agent** | Known-CVE exploitation / Joomla JCE RCE | Enumerated the Joomla/JCE version, matched it to a vulnerable release, used a public proof of concept for unauthenticated code execution, and read environment variables. | curl, version enumeration, Git, Python PoC, Linux commands |

## OSINT

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **The Best Teh Tarik** | Historical business-name investigation | Identified the shop and used historical Google Street View imagery to find its previous name. | Google Maps, Street View history, image inspection |
| **Tax Me If You Can** | Geolocation from visual landmarks | Zoomed into the supplied photo, identified the municipal building, matched the entrance in Street View, and extracted the coordinates. | Google Search, Google Maps, Street View |
| **My Childhood Game** | Reverse image search / game identification | Uploaded the game screenshot to an image-search service and identified the original game. | Google Lens or reverse image search |
| **The Last Message** | Transit-station geolocation | Recognized the KLIA Transit system, used visible station clues, compared the route map, and identified the correct station. | Image zoom, Google Search, transit route maps |
| **Solliitt!!!!** | Cross-platform identity pivoting | Pivoted from a GitHub username to commit history, found a linked Facebook account, located the full video, extracted a visual clue, and reverse-searched it. | GitHub, commit history, Facebook, screenshots, Google Lens |
| **Epstein** | Archive research and terrain geolocation | Decoded the clue, traced the document serial to public archives, identified the ranch, and compared historical Google Earth terrain to pinpoint the photo location. | CyberChef, web archives, reverse image search, Google Earth historical imagery |

## Misc

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **The Hardest** | Sanity / feedback challenge | Completed the linked feedback form to obtain the answer. | Web browser, Google Forms |
| **Brainduck** | Custom esoteric language / reversible byte transforms | Traced the `.bd` control flow, mapped duck-themed opcodes to byte operations, and brute-forced each byte against the validation targets. | grep, sed, text editor, Python |
| **Shades of Duck** | Color-based steganography | Extracted the hex color of every grid cell, selected the meaningful byte from each color code, and decoded the resulting hexadecimal ASCII. | Canva eyedropper or color picker, CyberChef |
| **Ready, Set, Go** | Announcement / platform observation | Found the flag directly in the competition announcement rather than in a downloadable artifact. | Discord, careful observation |
| **SOFEA** | LLM prompt injection and sensitive-data disclosure | Used the bot's private report form, enumerated its diagnostic template, discovered the protected variable name, and abused placeholder resolution to reveal it. | Discord bot, prompt injection, structured diagnostic prompts |
| **FOR SOFEA** | Game logic and event-path exploration | Played the RPG, collected unlimited currency, bought the required item, completed the dialogue path, and selected the flag option. | Game executable, RPG Maker data files, text editor |

## Forensic

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **The One That Got Away** | Malicious Office macro analysis | Extracted the VBA macro without opening the document, found fragmented strings and their reconstruction order, and rebuilt the final message. | oletools, olevba, FlareVM or Kali |
| **Memory Soup** | Memory-string carving and mixed encodings | Extracted strings with offsets, filtered for recipe markers, rejected decoys, decoded Base64/hex/reversed fragments, and assembled them in the numbered order. | strings, grep, CyberChef, Python |
| **Aduhai** | Phishing-email and macro triage | Extracted the DOCM attachment from the email, identified suspicious auto-running VBA, traced the macro logic, and recovered the first-stage information. | munpack, oleid, olevba, grep, Python |
| **Aduhai 2** | PowerShell downloader analysis | Downloaded the next stage safely, decoded it without execution, and inspected the PowerShell script for the embedded result. | curl, Base64 tools, cat, text editor |
| **Aduhai 3** | PyInstaller malware analysis | Identified the executable as PyInstaller, extracted its Python bytecode, disassembled it, and searched the recovered logic for collection and C2 behavior. | strings, pyinstxtractor, xdis/pydisasm, grep |
| **Missing Iroha** | Deleted-file recovery from an E01 image | Verified and mounted the forensic image, listed deleted FAT entries, recovered the target image by inode, and opened it. | ewfverify, ewfmount, fls, icat, Sleuth Kit |
| **Irohaverse** | Network forensics / WebSocket credential leak | Filtered the PCAP, found the WebSocket archive endpoint, followed the stream, recovered staff credentials, and used them on the website. | Wireshark, display filters, Follow TCP Stream |
| **Sus Exe** | Packed Python malware and configuration decoding | Detected UPX/PyInstaller indicators, extracted and decompiled the Python payload, then reversed Base64 plus repeating-key XOR to decode the hidden configuration. | file, strings, pyinstxtractor, decompyle3, Python |

## Reverse Engineering

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **Cleveland JR** | State-machine reversing and byte decryption | Used preserved symbols to map token handlers, reconstructed the state transformations, located the ciphertext, and either entered the required token sequence or brute-forced the final state. | file, nm, strings, objdump, Python |
| **ClockWork** | PRNG/LCG keystream reversal | Reversed the gear generator and byte check, converted the in-memory data address to a file offset, extracted 67 ciphertext bytes, and regenerated the keystream. | file, nm, objdump, readelf, xxd, Python |
| **DuckVM** | Custom virtual-machine bytecode | Determined that the real checker ran inside a custom VM, identified its bytecode and interpreter behavior, and wrote a solver for the VM logic. | strings, Dogbolt, Ghidra, Python |
| **MAMU OH MAMU** | Runtime string decoding | Located the encoded recipe-answer table, reconstructed the decoder, recovered the eight exact inputs, and submitted them in order. | strings, objdump, Ghidra, Python, netcat |
| **SUPERBABYRE** | Symbol-table and pointer analysis | Used the unstripped `real_flag` symbol, followed its little-endian pointer into `.rodata`, and decoded the bytes as ASCII. | file, nm, objdump/readelf, xxd |
| **KIRK** | Ransomware reverse engineering | Extracted a PyInstaller 3.13 payload, disassembled its bytecode, recovered obfuscated constants, identified scrypt plus AES-256-CBC, and wrote a decryptor for `.kirked` files. | 7z, sha256sum, strings, pyinstxtractor-ng, Python `marshal`/`dis`, PyCryptodome |
| **GUESS THE LOGO** | Roblox source analysis and repeating-key XOR | Opened the game in Roblox Studio, inspected client/server Lua scripts, derived the key from the known `OPUCC{` prefix, and decoded the real flag data. | Roblox Studio, Lua source review, Python |

## Pwn

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **Warm Up** | Ret2win stack overflow | Confirmed a 64-byte stack buffer was filled with up to 256 bytes, calculated the return offset, and redirected execution to the hidden `win()` function. | checksec, nm, objdump, GDB, pwntools |
| **Echo** | Format-string arbitrary read | Found `printf(user_input)`, determined the controlled argument index with a marker, and used `%s` with the global flag-buffer address. | nm, objdump, format-string probes, pwntools |
| **Shell Coder** | Stack shellcode execution | Used the leaked stack address and executable stack, overflowed the saved return address, and jumped to injected shellcode. | checksec, readelf, objdump, pwntools, shellcraft |
| **Borrowed Code** | Two-stage ret2libc | Leaked a libc address through `puts@GOT`, calculated the libc base, then built a second ROP chain for `system('/bin/sh')`. | checksec, objdump, ROPgadget, pwntools, supplied libc |
| **The HEAP** | Use-after-free and tcache poisoning | Used dangling pointers for read/write-after-free and double-free, leaked libc and safe-linking data, poisoned tcache toward `free@GOT`, and replaced `free` with `system`. | checksec, GDB/pwndbg, heap analysis, pwntools |
| **Ghost** | Format-string memory dump plus ROP | Used an arbitrary-read format string to reconstruct the remote PIE binary, leaked the canary/PIE/libc addresses, identified the exact libc, then overflowed the `auth` buffer with a ROP chain. | pwntools, file, readelf, objdump, libc.rip, Python |

## Crypto

| Challenge | Theme / vulnerability | Summary | Main tools |
|---|---|---|---|
| **One Line Duck** | Modular polynomial recovery | Interpreted the checker targets as evaluations of a polynomial whose coefficients were the flag bytes, then solved the modular system to recover all bytes. | Python, modular arithmetic, polynomial interpolation or linear algebra |
| **Sharing is Caring** | RSA common-prime attack | Computed `gcd(n1, n2)` to recover the reused prime, reconstructed the private key, and decrypted the ciphertext. | Python, `math.gcd`, RSA arithmetic |
| **Table for Two** | Layered puzzle: A1Z26, number sequences and hex | Decoded the initial clue, selected Perrin-number binder sheets, removed the Erdős-Woods overlap, read the corner stamps, decoded the combined hex string, and followed the final reference. | PDF reader, A1Z26, sequence lookup, CyberChef or Python, Pastebin |

## Fast Study Order

For a beginner-friendly progression:

1. **Start with Web and OSINT** - the feedback is immediate and the tools are easy to observe.
2. **Continue with Forensic and basic Reverse Engineering** - practise `strings`, `grep`, Office analysis, and Ghidra.
3. **Move to Crypto** - focus on recognizing standard mathematical weaknesses before writing code.
4. **Finish with Pwn** - begin with ret2win and format strings, then ret2libc, heap exploitation, and multi-stage ROP.

## Reusable CTF Workflow

1. Identify the file, service, protocol, and challenge category.
2. Perform low-cost enumeration first: source code, `robots.txt`, headers, `strings`, metadata, symbols, and obvious encodings.
3. Reproduce the vulnerable logic locally or with a minimal request.
4. Automate only after the manual behavior is understood.
5. Record the vulnerability, evidence, payload or algorithm, result, and mitigation.

---

Source: *Writeup Internal CTF UiTM Cyberheroes - Team Hati2Hati*.
