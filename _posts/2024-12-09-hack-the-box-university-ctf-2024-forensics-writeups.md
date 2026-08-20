---
layout: post
title: "Hack The Box University CTF 2024 — Forensics Writeups"
date: 2024-12-09 12:00:00 +0100
categories: [ctf, forensics]
---

Full forensics writeups from my Hack The Box University CTF 2024 run — three challenges, three infection chains, zero quiet moments. All the artifacts and scripts I used are in the repo: [github.com/Zyyz2/Hack-The-Box-University-CTF-2024](https://github.com/Zyyz2/Hack-The-Box-University-CTF-2024).

## Frontier Exposed

### Description

The chaos within the Frontier Cluster is relentless, with malicious actors exploiting vulnerabilities to establish footholds across the expanse. During routine surveillance, an open directory vulnerability was identified on a web server, suggesting suspicious activities tied to the Frontier Board. Your mission is to thoroughly investigate the server and determine a strategy to dismantle their infrastructure. Any credentials uncovered during the investigation would prove invaluable in achieving this objective. Spawn the Docker instance and start the investigation.

### Environment Setup

To access the Docker instance, copy the provided IP address and port into your web browser's URL bar.

### Steps to Solve

**1. Access the server** — browse to the Docker instance.

**2. Directory listing** — the server leaks everything:

```
.bash_history
.bash_logout
.bashrc
.profile
dirs.txt
exploit.sh
nmap_scan_results.txt
vulmap-linux.py
```

**3. Analyze `.bash_history`** — the operator(s) behind the Frontier Board left their whole reconnaissance trail behind:

```
nmap -sC -sV nmap_scan_results.txt jackcolt.dev
cat nmap_scan_results.txt
gobuster dir -u http://jackcolt.dev -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -o dirs.txt
nc -zv jackcolt.dev 1-65535
curl -v http://jackcolt.dev
nikto -h http://jackcolt.dev
sqlmap -u "http://jackcolt.dev/login.php" --batch --dump-all
searchsploit apache 2.4.49
```

…and buried in there, a Base64 string:

```
SFRCe0MyX2NyM2QzbnQxNGxzXzN4cDBzM2R9
```

**4. Decode the flag:**

```bash
echo "SFRCe0MyX2NyM2QzbnQxNGxzXzN4cDBzM2R9" | base64 --decode
```

> **HTB{C2_cr3d3nt14ls_3xp0s3d}**

Morale of the story: don't keep your C2 credentials in your bash history — or, you know, on a publicly listed directory.

## Wanter Alive

### Description

A routine patrol through the Frontier Cluster's shadowy corners uncovered a sinister file embedded in a bounty report — one targeting Jack Colt himself. The file's obfuscated layers suggest it's more than a simple message; it's a weaponized code from the Frontier Board, aiming to tighten their grip on the stars. As a trusted ally, it's your task to peel back the layers of deception, trace its origin, and turn their tools against them. Every domain found in the challenge should resolve to your Docker instance — don't forget the port when visiting the URLs.

### Steps to Solve

In this forensics challenge, we are tasked with analyzing a potentially malicious `.hta` (HTML Application) file, decoding its obfuscated data, and tracing the malicious actions it attempts to perform.

**Step 1 — Inspect the `.HTA` file.** Opening the file reveals a long URL-encoded string:

```bash
cat wanted.hta
```

**Step 2 — Decode the URL-encoded data.** Using CyberChef, we decode the string 5 times. After decoding, we find a `<script>` tag containing JavaScript with embedded malicious code.

**Step 3 — Analyze the JavaScript.** Inside the decoded script tag, we identify nested JavaScript and VBScript. The VBScript is the key component of the attack, designed to execute a PowerShell command silently:

- **Obfuscated VBScript** — constructs a PowerShell command using obfuscated variable names, then uses `WScript.Shell` to run it while bypassing execution policies, disabling profiles, and running in hidden mode.
- **PowerShell command** — decodes a Base64-encoded payload and executes it with `Invoke-Expression (iex)`. This payload is likely used to download and execute additional malicious instructions.

**Step 4 — Decode the Base64 payload.** The decoded payload is a PowerShell script that imports a function from a DLL (`uRLmON.dll`) and uses it to download a malicious `.vbs` file from a remote server:

- **URL:** `http://wanted.alive.htb/35/wanted.tIF`
- **Download location:** saved in the user's `APPDATA` directory
- **Execution:** runs after a 3-second pause to ensure the download completes

**Step 5 — Execute the downloaded VBS script.** To retrieve it, map the malicious domain to the Docker IP in `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Then fetch the file:

```bash
curl http://wanted.alive.htb:port/35/wanted.tIF
```

**Step 6 — Deconstruct the VBS script.** Inside the downloaded `.vbs` we find multiple Base64-encoded strings that, when decoded, reveal executable commands and system-level actions:

- **Base64 decoding** — the script hides its commands; decoded strings get concatenated into a command line executed using `incentiva.Run`.
- **Obfuscation** — layers of encoding to evade analysis and AV.
- **Background execution** — `incentiva.Run` executes the built command in the background, requiring no user interaction and avoiding detection.

**Step 7 — Decode additional payloads.** A suspicious Base64 string (`latifoliado`) doesn't decode directly — but we notice a repetitive pattern `d2FudGVkCg`. After cleaning up the repeated sections, decoding reveals another PowerShell script that:

- **Disables security** — bypasses SSL validation and sets the execution policy to `Bypass`
- **Downloads another payload** — fetches a Base64-encoded string from `http://wanted.alive.htb/cdba/_rp`, decodes it into a PowerShell script, and runs it with `iex`

The nested encoding ensures each payload stays invisible to standard security tooling until fully unraveled.

**Step 8 — Retrieve the flag.** Fetch the final script:

```bash
curl http://wanted.alive.htb:port/cdba/_rp
```

And there it is:

> **HTB{c4tch3d_th3_m4lw4r3_w1th_th3_l4ss0_9829c8ef2650507eed3f7a63361074ae}**

Five layers of URL-encoding, nesting VBS→PowerShell→PowerShell, all to deliver one evil lasso. Classic.

## Binary Badresources

### Challenge Overview

In this forensics challenge, we analyze a file named `wanted.msc`, which — when executed — doesn't produce any visible output. Inspecting the file contents reveals several layers of obfuscation: Base64-encoded JavaScript and VBScript. The goal: reverse the obfuscation and uncover the hidden flag.

### Steps to Solve

**Step 1 — Initial analysis of `wanted.msc`.** Executing it yields nothing. Opening it in a text editor reveals large amounts of obfuscated JavaScript; focus shifts to the JavaScript section.

**Step 2 — Deobfuscating the JavaScript.** Heavily obfuscated, but after deobfuscation we find a string that turns out to be VBScript. Analyzing it, we identify a function that processes a specific string, then convert the VBScript function to Python.

**Step 3 — Transforming VBScript to Python.** Running the converted script reveals the deobfuscated VBScript:

```python
TpHCM = ""

source_string = "Stxmsr$I|tpmgmx..."

for i in range(1, 3223):
    char = source_string[i - 1]
    ascii_value = ord(char) - 5 + 1
    TpHCM += chr(ascii_value)

with open("output.txt", "w") as file:
    file.write(TpHCM)

print(TpHCM)
```

A simple Caesar shift (`-5 + 1`) over the whole payload — the classic ROT dance.

**Step 4 — Downloading files.** Using `curl`, we download the payload files:

```
csrss.exe
csrss.exe.config
wanted.pdf
csrss.dll
```

**Step 5 — XOR operation.** Each of the files is XORed using `csrss.dll` as the key:

```python
import os

def xor_file(file_path, key_path):
    try:
        with open(key_path, 'rb') as key_file:
            key = key_file.read()

        with open(file_path, 'rb') as file:
            file_content = bytearray(file.read())

        key_length = len(key)

        for i in range(len(file_content)):
            file_content[i] ^= key[i % key_length]

        with open(file_path, 'wb') as file:
            file.write(file_content)

        print(f"File '{file_path}' has been successfully XORed.")
    except Exception as e:
        print(f"An error occurred while processing the file: {e}")

file_paths = [
    "csrss.exe",
    "csrss.exe.config",
    "wanted.pdf"
]

key_path = "csrss.dll"

for file_path in file_paths:
    xor_file(file_path, key_path)
```

After the XOR, `wanted.pdf` is no longer corrupted and displays an image of a cowboy. No hidden data or flag inside the PDF, though.

**Step 6 — Executing the XOR'd executable.** Analyzing `csrss.exe` with `strings`, we find keywords such as "AES" and "crypto" — time to reverse further.

**Step 7 — Reverse engineering the executable.** Decompiling `csrss.exe` with dnSpy reveals the real script hidden inside — a new domain:

```
http://windowsupdate.htb/ec285935b46229d40b95438707a7efb2282f2f02.xml
```

**Step 8 — Retrieving the flag:**

```bash
curl http://windowsupdate.htb:port/ec285935b46229d40b95438707a7efb2282f2f02.xml
```

> **HTB{mSc_1s_b31n9_s3r10u5ly_4buSed}**

### Conclusion

By deobfuscating the JavaScript, reversing the XOR operation, and analyzing the executable with dnSpy, we uncover the hidden flag. This challenge required a multi-layered approach — cryptographic analysis, reverse engineering, and web requests.

## Final Thoughts

Three challenges, three distinct evasion chains: open-directory credential leaks, a five-layer HTA dropper, and an MSC misused as an XOR-encrypted malware delivery vehicle. HTB University CTF 2024's forensics category was genuinely great practice for real-world malware triage. All artifacts — the obfuscated scripts, decoded payloads, and tooling — are attached in the repo: [github.com/Zyyz2/Hack-The-Box-University-CTF-2024](https://github.com/Zyyz2/Hack-The-Box-University-CTF-2024).