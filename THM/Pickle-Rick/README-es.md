![](adjuntos/Pasted%20image%2020260302010325.png)
# 🧪 Pickle Rick – TryHackMe

> Compromiso de un host Linux mediante explotación de aplicación web, ejecución autenticada de comandos, enumeración del sistema y escalada de privilegios a root abusando de una configuración insegura de sudo.

---

## 1️⃣ Información del Laboratorio

**Plataforma:** TryHackMe  
**Tipo de Entorno:** Entorno Linux  
**Servicios Objetivo:** Apache 2.4.41 (Ubuntu) + OpenSSH 8.2p1  
**Vulnerabilidades Explotadas:**

- Divulgación de información en el código fuente web
- Ejecución de comandos sin sanitización (RCE)
- Directorio de usuario con permisos inseguros (777)
- Regla sudo mal configurada (`NOPASSWD: ALL`)

**Dificultad:** Easy


#  Habilidades Demostradas

- Enumeración completa de servicios (Nmap)
- Análisis manual de superficie web
- Descubrimiento de recursos ocultos (Gobuster)
- Explotación web autenticada
- Validación de Remote Command Execution
- Enumeración de sistema Linux (`/etc/passwd`, `/home`)
- Análisis de permisos en sistema de archivos
- Evaluación de privilegios sudo (`sudo -l`)
- Escalada de privilegios mediante mala configuración
- Validación de compromiso total del sistema (root)

# 2️⃣ Reconocimiento

## 2.1 VPN y Conectividad

Se estableció conexión VPN para acceder al entorno aislado del laboratorio:
```bash
thm on
```

Validación de conectividad:
```bash
ping -c 4 <TARGET_IP>
```


✔ Host accesible  
✔ Comunicación estable

## 2.2 Enumeración de Servicios

Reconocimiento inicial:
```bash
nmap -sC -sV -T4 <TARGET_IP>
```

### Servicios Detectados

- **22/tcp → SSH (OpenSSH 8.2p1 – Ubuntu)**
- **80/tcp → HTTP (Apache 2.4.41 – Ubuntu)**

### Priorización de Superficie de Ataque

Aunque SSH estaba expuesto, no se realizaron ataques de fuerza bruta.

Motivos:

- No se identificaron credenciales durante el reconocimiento.
- No se observaron configuraciones débiles en SSH.
- El servicio web mostraba funcionalidad visible.
- El contexto indicaba una explotación orientada a aplicación web.

Superficie principal seleccionada: **HTTP (Puerto 80)**.



# 🌐 3️⃣ Enumeración Web



## 3.1 Inspección Manual

- Acceso a la aplicación web
- Revisión del código fuente
- Identificación de información expuesta
- Confirmación de existencia de portal de autenticación


## 3.2 Descubrimiento de Directorios

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

### Endpoints Descubiertos

- `/login.php`
- `/portal.php`
- `/robots.txt`

Se confirmó la existencia de contenido protegido y posibles vectores adicionales de información.


## 3.3 Acceso Autenticado

Con la información descubierta se obtuvo acceso al **Command Execution Panel**.

En este punto, el enfoque pasó de reconocimiento web a interacción autenticada con la aplicación.



# 💻 4️⃣ Ejecución Remota de Comandos (RCE)



## 4.1 Validación de Ejecución

```bash
whoami
```

Salida:

```bash
www-data
```
✔ RCE confirmada  
✔ Ejecución bajo el usuario del servicio web

Este punto marca el cambio de explotación a nivel aplicación hacia compromiso a nivel sistema.



## 4.2 Identificación del Entorno

```bash
pwd
```
Ubicación:
```bash
/var/www/html
```


Enumeración local:
```bash
ls -la
```

Se identificaron archivos accesibles desde el entorno web.



## 4.3 Bypass del Filtro de Comandos

El comando `cat` estaba bloqueado mediante filtrado por blacklist.

Se utilizaron herramientas alternativas:
```bash
less <file>
```

### Observación

El filtrado por blacklist no mitiga realmente la ejecución remota de comandos.  
La RCE seguía siendo completamente explotable.



# 🔎 5️⃣ Enumeración Post-Explotación



## 5.1 Enumeración de Usuarios
```bash
less /etc/passwd
```

Se identificaron usuarios humanos con shell interactiva.

Confirmación de:

- Entorno Linux multiusuario

- Ampliación de superficie de ataque



## 5.2 Análisis de `/home`

```bash
ls -la /home
```

Se detectó un directorio con permisos 777.

### Impacto de Seguridad

- Ruptura del aislamiento entre usuarios
- Posibilidad de lectura/escritura arbitraria
- Vector potencial de escalada


## 5.3 Enumeración del Entorno de Usuario

```bash
ls -la /home/<user>
```

Se localizó un archivo sensible accesible sin privilegios elevados.

Se obtuvo un segundo recurso protegido a nivel usuario.



# 🔐 6️⃣ Escalada de Privilegios



## 6.1 Evaluación de Privilegios Sudo

```bash
sudo -l
```
Salida:

```bash
(ALL) NOPASSWD: ALL
```


Esto permitió al usuario `www-data` ejecutar cualquier comando como root sin autenticación.

Desde el punto de vista de seguridad, representa una configuración crítica que elimina completamente la separación de privilegios.



## 6.2 Ejecución como Root

Debido a la naturaleza no interactiva del panel web, se ejecutaron comandos directamente como root:
```bash
sudo ls -la /root
```

Se confirmó acceso al directorio restringido.



## 6.3 Confirmación de Escalada

```bash
sudo less /root/<file>
```
Privilegios root obtenidos con éxito.

✔ Escalada completada  
✔ Control total del sistema confirmado

## ⚖ Disclaimer

Este write-up documenta mi metodología y enfoque técnico personal para resolver el laboratorio.  
No se incluyen flags, credenciales ni respuestas propietarias del reto.

Todas las actividades fueron realizadas dentro de un entorno autorizado de aprendizaje (TryHackMe).