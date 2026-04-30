# 🔍 Recopilación Pasiva de Información

## Introducción a la Fase de Recolección

La primera fase de cualquier prueba de penetración consiste en recolectar información sobre una persona, empresa, sitio web o sistema. Cuanto más se conozca al objetivo, mayores serán las probabilidades de éxito en las etapas finales.

La información obtenida permite, por ejemplo:

- Acceder mediante **pivoting** a una red interna.
- Realizar **ataques de phishing** a empleados.
- Enviar **archivos maliciosos** (Word/Excel con macros) para obtener credenciales o acceso al sistema.

## Tipos de Recopilación

### ✅ Recopilación Pasiva
Consiste en obtener información **sin interactuar directamente** con el objetivo. Se evalúa desde la perspectiva de un usuario final, analizando tecnologías, la IP del servidor real, etc.

**Datos recolectables:**
- Direcciones IP y DNS.
- Nombres de dominio y subdominios.
- Correos electrónicos y redes sociales.
- Tecnologías empleadas por el sitio.
- Identificación de subdominios.

### ⚡ Recopilación Activa
Implica **participar activamente** con el sistema objetivo. Por ejemplo, tras identificar la IP en la fase pasiva, se puede realizar un escaneo de puertos con Nmap para detectar puertos abiertos y los servicios que se ejecutan en ellos. Esta información resulta clave para encontrar vulnerabilidades explotables.

> ⚠️ **Se requiere permiso escrito de la empresa** para efectuar pruebas de penetración. Escanear sin autorización es ilegal.

## Herramientas y Técnicas de Recopilación Pasiva

### Comandos básicos en Kali

```bash
whatis host
host hackersploit.org          # Muestra la IP del dominio
```

### Archivos públicos del sitio

- **robots.txt**  
  Indica qué rutas no deben ser indexadas por los motores de búsqueda.
  ```
  Disallow: /wp-admin/   (WordPress)
  ```

- **sitemap.xml**  
  Mapa del sitio con rutas útiles.

### Identificación de tecnologías

| Herramienta | Tipo | Uso |
|------------|------|------|
| **BuiltWith** | Extensión Firefox/Chrome | Perfilador de tecnología que muestra CMS, frameworks, etc. |
| **Wappalyzer** | Extensión navegador | Similar a BuiltWith |
| **WhatWeb** | Línea de comandos (Kali) | `whatweb hackersploit.org` |

### Descarga completa del sitio con HTTrack

```bash
# Instalación en Kali
sudo apt-get install webhttrack
```

HTTrack copia el sitio web en local para analizar el código fuente sin conexión.

### WHOIS y NetCraft

```bash
whois hackersploit.org    # Información del dominio: registrador, fechas, etc.
```

NetCraft (vía navegador) también ofrece datos históricos y de hosting.

### Reconocimiento de DNS (pasivo)

- `dnsrecon` en modo básico:
  ```bash
  dnsrecon -d hackersploit.org
  ```
- **dnsdumpster.com** organiza la información de forma gráfica y práctica.

### Detección de WAF con wafw00f

```bash
wafw00f -l                     # Lista los firewalls que detecta
wafw00f hackersploit.org       # Indica si el sitio usa Cloudflare u otro WAF
```

### Enumeración pasiva de subdominios con Sublist3r

```bash
sublist3r -d hackersploit.org -e google,yahoo
```

> 🔒 Se recomienda usar VPN para que Google no bloquee las solicitudes.

### Google Dorks (filtros de búsqueda)

Permiten localizar información sensible expuesta públicamente. A continuación se muestran los filtros más utilizados:

| Operador/Filtro       | Ejemplo                                      | Descripción |
|-----------------------|----------------------------------------------|-------------|
| `site:`               | `site:ine.com inurl:admin`                  | Páginas con "admin" en la URL |
| `site:` + `inurl:`    | `site:ine.com inurl:forum`                  | "forum" en la URL |
| `site:*`              | `site:*.ine.com`                            | Todos los subdominios |
| `site:*` + `inurl:`   | `site:*ine.com inurl:admin`                 | Subdominios con admin en la URL |
| `site:*` + `intitle:` | `site:*ine.com intitle:admin`              | Subdominios con admin en el título |
| `filetype:`           | `site:ine.com filetype:pdf`                | Archivos PDF |
| `filetype:`           | `site:ine.com filetype:pdf marketing`      | PDF con la palabra "marketing" |
| `filetype:`           | `site:ine.com filetype:xlsx`               | Hojas de cálculo |
| `filetype:`           | `site:ine.com filetype:docx`               | Documentos Word |
| `filetype:`           | `site:ine.com filetype:zip`                | Archivos comprimidos |
| `filetype:`           | `site:ine.com filetype:docs`               | Documentos genéricos |
| `site:` + texto       | `site:ine.com employees`                    | Búsqueda de "employees" |
| `site:` + texto       | `site:ine.com instructors`                  | Búsqueda de "instructors" |
| `intitle:`            | `intitle:index of`                           | Directorios con listado público |
| `inurl:`              | `inurl:auth_user_file.txt`                  | Posibles archivos de contraseñas |
| `inurl:`              | `inurl:passwd.txt`                           | Archivos de contraseñas en texto plano |

Más combinaciones se encuentran en la **Google Hacking Database (GHDB)**. También se puede emplear **Wayback Machine** para ver versiones anteriores de los sitios.

### Enumeración de correos electrónicos

#### theHarvester

```bash
sudo apt search theharvester
theHarvester --help
```

Ejemplos de uso:
```bash
theHarvester -d INE -b duckduckgo
theHarvester -d INE -b duckduckgo,bing,yahoo
theHarvester -d ine.com -b duckduckgo,bing,yahoo,urlscam,pentesttool
```

### Filtraciones públicas de credenciales

**Have I Been Pwned?** permite comprobar si una dirección de correo o un número de teléfono ha sido comprometido en filtraciones de datos.
