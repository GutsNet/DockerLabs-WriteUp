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
Este comando revela que el puerto `80` está ejecutando Apache y el puerto `22` está ejecutando OpenSSH.
De igual forma, podemos observar que, la cookie `PHPSESSID` no tiene la flag `httponly`, lo cual representa un fallo de seguridad.

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

## Inyección SQL

#### Página principal (http://172.170.2/)
![image](https://github.com/user-attachments/assets/1a710629-010e-4b9b-8a4a-d04398fa4bbb)

La página principal es un Login (Inicio de sesión), lo primero que se puede intentar al ver un inicio de sesión y sabiendo que se trata de una web vulnerable, es realizar una inyección SQL.

La inyección SQL que se utilzará será la siguiente `admin' OR 1=1 -- -`, se aplicará dentro del campo de User y en la contraseña se ingresará cualquier cosa como `password`. 
#### Explicación

  - `admin` es un valor que el atacante inserta en un campo de entrada, en este caso en el del nombre de usuario.
  - La comilla simple `'` justo después de admin cierra el valor del nombre de usuario en la consulta SQL original.
  - `OR 1=1` es una condición lógica añadida por el atacante. La condición `1=1` siempre es verdadera, lo que significa que esta parte   de la consulta siempre se cumplirá sin importar lo que venga antes.
  - `--` es un operador de comentario en SQL. Todo lo que viene después de `--` en la misma línea será ignorado por la base de datos.
  - El espacio y guión adicional ` -`, es porque en algunos casos se puede requerir un espacio después de `--` para ser considerado un comentario

 Ahora bien, suponiendo que la consulta que busca un usuario en la base de datos es la siguiente:
 ```SQL
SELECT * FROM usuarios WHERE username = 'admin' AND password = 'password';
 ```
 Si insertamos la inyección SQL (`admin' OR 1=1 -- -`) en donde está el texto "admin", la consulta quedará de esta manera:
 ```SQL
SELECT * FROM usuarios WHERE username = 'admin' OR 1=1 -- - AND password = 'password';
 ```
 La consulta original que se esperaba filtrar usuarios por nombre de usuario y contraseña válidos se convierte en una consulta que siempre devuelve al menos un resultado (porque `1=1` es siempre verdadero), permitiendo al atacante "anular" la autenticación.

### Aplicación de la inyección
Los campos del login quedarán de la siguiente forma:

![image](https://github.com/user-attachments/assets/76a6c541-743a-4852-82f6-8c688ff88847)

Al hacer clic en "Login" y si ya inyección fue exitosa, nos redireccionará a la ruta http://172.17.0.2/acceso_valido_dylan.php, el contenido de dicha ruta, es lo que se muestra en la imagen de abajo.

![image](https://github.com/user-attachments/assets/2c22cc65-55db-49ef-93f0-fbe9f9fe10cd)

En la pantalla se puede apreciar el nombre `Dylan` y la contraseña `KJSDFG789FGSDF78` y, como se pudo ver en el escaneo de puertos, el servicio SSH está habilitado, por ende, podrían ayudarnos a acceder a la máquina.
