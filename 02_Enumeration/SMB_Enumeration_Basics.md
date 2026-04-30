# 🪟 Samba Recon: Conceptos Básicos

## 📡 Puertos predeterminados de Samba

Samba utiliza dos servicios principales:

- **smbd** (intercambio de archivos e impresión): puertos **TCP 139 y 445**.
- **nmbd** (resolución de nombres NetBIOS): puertos **UDP 137 y 138**.

### 🔍 Escaneo inicial

```bash
nmap demo.ine.local
```

Para detectar los puertos UDP de nmbd:

```bash
nmap -sU --top-ports 25 demo.ine.local
```

Salida esperada: **137, 138** (UDP abiertos).

## 🏢 Nombre del grupo de trabajo

El grupo de trabajo se puede obtener con una detección de versiones en el puerto 445.

```bash
nmap -sV -p 445 demo.ine.local
```

En el laboratorio de ejemplo, el grupo de trabajo identificado fue **RECONLABS**.

## 📋 Versión exacta del servidor Samba

### Mediante script Nmap

```bash
nmap --script smb-os-discovery.nse -p 445 demo.ine.local
```

Salida: **Samba 4.3.11-Ubuntu**

### Mediante módulo Metasploit (`smb_version`)

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_version
set RHOSTS demo.ine.local
exploit
```

Resultado: **Samba 4.3.11-Ubuntu**

## 🧾 Nombre NetBIOS del servidor

### Mediante el mismo script Nmap

```bash
nmap --script smb-os-discovery.nse -p 445 demo.ine.local
```

El nombre NetBIOS aparece en la salida del script. En este caso: **SAMBA-RECON**

### Mediante `nmblookup`

```bash
nmblookup -A demo.ine.local
```

Salida: **SAMBA-RECON**

## 👤 Conexión anónima (sesión nula)

### Verificación con `smbclient`

```bash
smbclient -L demo.ine.local -N
```

**Interpretación:** Si se muestran los recursos compartidos sin solicitar contraseña, la sesión nula está **permitida**.

### Verificación con `rpcclient`

```bash
rpcclient -U "" -N demo.ine.local
```

Si se obtiene una sesión interactiva, la conexión anónima está **permitida**.

---

## 🧠 Conclusión

La combinación de Nmap, scripts `smb-os-discovery`, `nmblookup`, `smbclient` y `rpcclient` permite enumerar de forma rápida la configuración básica de un servidor Samba:
- Puertos activos.
- Grupo de trabajo.
- Versión exacta.
- Nombre NetBIOS.
- Posibilidad de acceso anónimo (sesión nula).
