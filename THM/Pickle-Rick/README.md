![](adjuntos/Pasted%20image%2020260302010325.png)
# 🧪 # Pickle Rick – TryHackMe

>Compromise of a Linux host through web application exploitation, authenticated command execution, system enumeration, and privilege escalation to root by abusing an insecure sudo configuration.

---

## 1️⃣ Lab Information

**Platform:** TryHackMe  
**Environment Type:** Linux-based lab environment  
**Target Services:** Apache 2.4.41 (Ubuntu) + OpenSSH 8.2p1
**Exploited Vulnerability:**

- Information disclosure in web source
- Unsanitized command execution (RCE)
- World-writable user directory (777 permissions)
- Misconfigured sudo rule (`NOPASSWD: ALL`)

**Difficulty:** Easy

# Demonstrated Skills

- Full service enumeration (Nmap)
- Web surface analysis and manual source inspection
- Directory discovery (Gobuster)
- Authenticated web exploitation
- Remote Command Execution validation
- Linux system enumeration (`/etc/passwd`, `/home`)
- Filesystem permission analysis
- Sudo privilege assessment (`sudo -l`)
- Privilege escalation through sudo misconfiguration
- Root-level system compromise validation
# 2. Reconnaissance

## 2.1 VPN & Connectivity

Established VPN connection to access the isolated lab network.
```bash
thm on
```


Validated target reachability:
```bash
ping -c 4 <TARGET_IP>
```

✔ Host reachable  
✔ Stable network communication


## 2.2 Service Enumeration

Performed initial reconnaissance:
```bash
nmap -sC -sV -T4 <TARGET_IP>
```

## Service Enumeration

Initial Nmap scan revealed two exposed services:

- **22/tcp → SSH (OpenSSH 8.2p1 – Ubuntu)**
- **80/tcp → HTTP (Apache 2.4.41 – Ubuntu)**

### Attack Surface Prioritization

Although SSH was exposed, no brute-force attempts were performed.

Reasoning:

- No credentials identified during recon.
- No signs of weak SSH configuration.
- Web service presented visible entry points.
- CTF context suggested web-based exploitation.

Primary attack surface: **HTTP (Port 80)**.


# 🌐 3. Web Enumeration


## 3.1 Manual Inspection

- Accessed web application.
- Reviewed page structure.
- Inspected client-side source code.
- Identified exposed information.

Detected existence of an authentication portal.

## 3.2 Directory Enumeration
```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

### Discovered Endpoints

- `/login.php`
- `/portal.php`
- `/robots.txt`

This confirmed the presence of:

- Authentication mechanism
- Protected internal content
- Additional information disclosure vectors

## 3.3 Authenticated Access

Using discovered information, access was obtained to an internal **Command Execution Panel**.

Engagement shifted from unauthenticated web access to authenticated application interaction.


# 💻 4. Remote Command Execution (RCE)


## 4.1 Execution Validation
```bash
whoami
```

Output:
```bash
www-data
```


✔ Remote command execution confirmed  
✔ Running under web service account

This marks the pivot from web-layer exploitation to system-level compromise.


## 4.2 Environment Identification
```bash
pwd
```


Located within:
```bash
/var/www/html
```


Enumerated local files:
```bash
ls -la
```


Identified files of interest within web root.

## 4.3 Command Filter Bypass

The `cat` command was blocked via blacklist filtering.

Alternative utilities were used to read files:
```bash
less <file>
```


### Security Observation

Blacklist-based filtering is insufficient.  
RCE remained fully exploitable.

# 🔎 5. Post-Exploitation Enumeration


## 5.1 User Enumeration
```bash
less /etc/passwd
```


Identified human users with interactive shells.

This confirmed:

- Multi-user Linux system
- Expanded attack surface beyond web context


## 5.2 Home Directory Analysis
```bash
ls -la /home
```


Observed:

- World-writable directory (777 permissions)


### Security Impact

- Broken user isolation
- Arbitrary read/write access
- Potential privilege escalation vector

## 5.3 User Environment Enumeration
```bash
ls -la /home/<user>
```

Discovered sensitive file accessible without elevated privileges.

Second protected resource obtained at user-level.

# 🔐 6. Privilege Escalation


## 6.1 Sudo Privilege Assessment

As part of post-exploitation enumeration, sudo permissions were reviewed:
```bash
sudo -l
```


The output showed:
```bash
(ALL) NOPASSWD: ALL
```


This configuration allowed the `www-data` account to execute commands as root without authentication.

From a security standpoint, this represents a critical misconfiguration, as it effectively removes privilege boundaries.

## 6.2 Root-Level Command Execution

Due to the non-interactive nature of the web-based RCE, privilege escalation was performed by executing commands directly as root:
```bash
sudo ls -la /root
```

Access to restricted directories was confirmed.


## 6.3 Privilege Escalation Confirmation

Further verification was performed by accessing protected root-level resources:
```bash
sudo less /root/<file>
```

Root privileges were successfully obtained, confirming complete control over the system.

⚖ Disclaimer

This write-up documents my personal methodology and technical approach to solving the lab.
No flags, credentials, or proprietary lab answers are disclosed.

All activities were conducted within the authorized TryHackMe environment.