# 3. Máquina: Injection  

<a href="https://github.com/GutsNet"><img title="Author" src="https://img.shields.io/badge/Author-GutsNet-red.svg?style=for-the-badge&logo=github"></a>

**Dificultad:** 🔵 *Muy Fácil*

**Autor:** *[El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)*

**Fecha de creación:** *09/04/2024*

**Proceso:**
- [1. Despliegue](#desplegando-la-máquina-vulnerable)
- [2. Escaneo de puertos](#escaneo-de-puertos-con-nmap)

---

## Desplegando la Máquina Vulnerable

```python
~/Injection ᐅ sudo bash auto_deploy.sh injection.tar
```
Este comando ejecuta un script de bash (`auto_deploy.sh`) que despliega la máquina vulnerable a partir del archivo `injection.tar`.

**Salida:**

```python
Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

#### Verificación de Conectividad

```python
~/Injection ᐅ ping -c 2 172.17.0.2
```
El comando `ping -c 2` envía 2 paquetes ICMP a la dirección IP `172.17.0.2` para verificar la conectividad y la respuesta del host.

**Salida:**

```python
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=6.17 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.089 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.089/3.130/6.171/3.041 ms
```

---

## Escaneo de Puertos con Nmap

```python
~/Injection ᐅ nmap -p- -sCV 172.17.0.2
```
Este comando revela que ..

**Salida:**

```python
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-title: Iniciar Sesi\xC3\xB3n
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Explicación de parámetros:**

- `-p-`: Escanea todos los puertos (del 1 al 65535).
- `-sCV`: Realiza un escaneo con detección de versión (`-sV`) y utiliza scripts básicos (`-sC`) para obtener más información del servicio.

---
