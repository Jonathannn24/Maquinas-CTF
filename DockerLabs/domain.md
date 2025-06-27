# Maquina Trust - DockerLabs

* Nivel : Medio
* Máquina : Linux
* Útil para el Ejptv2
* Ip Maquina : 172.17.0.3

## Escaneo de puertos

```bash
sudo nmap -A -sV -O -T4 -p- -Pn 172.17.0.3
```
```bash

```

## Escaneo web

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt -t 30 -x php,txt,htm
```
