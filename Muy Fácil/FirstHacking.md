
# 7. Máquina: FirstHacking  

<a href="https://github.com/GutsNet"><img title="Author" src="https://img.shields.io/badge/Author-GutsNet-purple.svg?style=for-the-badge&logo=github"></a>

**Dificultad:** 🔵 *Muy Fácil*

**Autor:** *[El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)*

**Fecha de creación:** *14/06/2024*

**Proceso:**
- [1. Escaneo de Puertos](#escaneo-de-puertos-con-nmap)
- [2. Búsqueda de Exploits](#exploit-vsftpd-234)
- [3. Ejecución de Exploit](#ejecución-del-exploit)
- [4. Acceso a la Máquina](#acceso-a-la-máquina)

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
