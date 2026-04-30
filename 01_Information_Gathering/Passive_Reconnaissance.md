# 🔍 Etapa 1: 01_Information_Gathering – Passive_Reconnaissance.md

> **Nota importante:** Este archivo conserva **todas** las notas originales, comandos, herramientas y técnicas descritas en el laboratorio. Se ha ordenado y formateado sin omitir ningún paso.

---

## 📌 Introducción a la Fase de Recolección

La **primera fase de cualquier prueba de penetración** consiste en recolectar información sobre una persona, empresa, sitio web o sistema.  
*“El que más sabe sobre su objetivo es el que más éxito tendrá en las etapas finales de la prueba”*.

Esta información puede ser útil en la fase de explotación, por ejemplo para:

* Acceder mediante **pivoting** a una red interna.
* Realizar **ataques de phishing** a empleados.
* Enviar **archivos maliciosos** (Word/Excel con macros) para obtener credenciales o acceso al sistema.

---

## 🛡️ Tipos de Recopilación

### ✅ Recopilación Pasiva
Se obtiene información **sin interactuar directamente** con el objetivo.  
**Ejemplo:** evaluar un sitio web desde la perspectiva de un usuario final, tecnologías, IP del servidor, etc.

📋 **Datos recolectables:**
* IP y DNS
* Nombres de dominio y subdominios
* Correos electrónicos y redes sociales
* Tecnologías usadas por el sitio
* Identificación de subdominios

### ⚡ Recopilación Activa
Se recopila información **participando activamente** con el sistema objetivo.  
**Ejemplo:** después de la recolección pasiva, ejecutar un escaneo de puertos con **Nmap** sobre la IP real.

📋 **Datos recolectables:**
* Puertos abiertos en el sistema objetivo
* Infraestructura interna de la red objetivo
* Enumerar información del sistema objetivo

> ⚠️ **Se necesita permiso escrito de la empresa** para realizar pruebas de penetración. Escanear sin autorización es ilegal.

---

## 🧰 Herramientas y Técnicas de Recopilación Pasiva

### 🔧 Comandos básicos en Kali

```bash
whatis host
host hackersploit.org          # Muestra la IP del dominio
```

---

### 🕸️ Archivos públicos del sitio

* **robots.txt**  
  Indica qué rutas no deben ser indexadas por motores de búsqueda.
  ```text
  Disallow: /wp-admin/   (WordPress)
  ```

* **sitemap.xml**  
  Mapa del sitio con rutas útiles.

---

### 🔎 Identificación de tecnologías

| Herramienta | Tipo | Uso |
|------------|------|------|
| **BuiltWith** | Extensión Firefox/Chrome | Perfilador de tecnología que muestra CMS, frameworks, etc. |
| **Wappalyzer** | Extensión navegador | Similar a BuiltWith |
| **WhatWeb** | Línea de comandos (Kali) | `whatweb hackersploit.org` |

---

### 📥 Descarga de sitio completo con HTTrack

```bash
# Instalación en Kali
sudo apt-get install webhttrack
```

HTTrack permite descargar el sitio web completo para analizar el código fuente en local.

---

### 🌍 WHOIS y NetCraft

```bash
whois hackersploit.org    # Información del dominio, registrador, fechas, etc.
```

**NetCraft** (vía navegador) también da información histórica y de hosting.

---

### 🌐 Reconocimiento de DNS (pasivo)

* **dnsrecon (uso básico)**
  ```bash
  dnsrecon -d hackersploit.org
  ```
* **dnsdumpster.com**  
  Recomendado por su forma práctica de organizar la información.

---

### 🚧 Detección de WAF con wafw00f

```bash
wafw00f -l                     # Lista de firewalls que detecta
wafw00f hackersploit.org       # Indica si está detrás de Cloudflare, etc.
```

---

### 🧩 Enumeración pasiva de subdominios con Sublist3r

```bash
sublist3r -d hackersploit.org -e google,yahoo
```
> 🔒 Usar VPN para evitar que Google bloquee las solicitudes.

---

### 🎯 Google Dorks (filtros de búsqueda)

Permiten encontrar información sensible pública. A continuación se muestran los ejemplos de las notas:

| Operador/Filtro | Ejemplo | Descripción |
|-----------------|---------|-------------|
| `site:` | `site:ine.com inurl:admin` | Buscar páginas con "admin" en la URL |
| `site:` + `inurl:` | `site:ine.com inurl:forum` | Buscar "forum" en la URL de ese sitio |
| `site:*` | `site:*.ine.com` | Todos los subdominios |
| `site:*` + `inurl:` | `site:*ine.com inurl:admin` | Subdominios con admin en la URL |
| `site:*` + `intitle:` | `site:*ine.com intitle:admin` | Subdominios con admin en el título |
| `filetype:` | `site:ine.com filetype:pdf` | Archivos PDF en el sitio |
| `filetype:` | `site:ine.com filetype:pdf marketing` | PDF que contengan "marketing" |
| `filetype:` | `site:ine.com filetype:xlsx` | Hojas de cálculo |
| `filetype:` | `site:ine.com filetype:docx` | Documentos Word |
| `filetype:` | `site:ine.com filetype:zip` | Archivos comprimidos |
| `filetype:` | `site:ine.com filetype:docs` | Documentos genéricos |
| `site:` + texto | `site:ine.com employees` | Buscar la palabra "employees" |
| `site:` + texto | `site:ine.com instructors` | Buscar "instructors" |
| `intitle:` | `intitle:index of` | Directorios de listado público |
| `inurl:` | `inurl:auth_user_file.txt` | Posibles archivos de contraseñas |
| `inurl:` | `inurl:passwd.txt` | Archivos de contraseñas planas |

Más información en **Google Hacking Database (GHDB)**.

También se puede utilizar **Wayback Machine** para ver versiones anteriores del sitio.

---

### 📧 Enumeración de correos electrónicos

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

---

### 🔓 Verificar filtraciones públicas

**have i been pwned?**  
Sitio web donde se comprueba si un correo o teléfono ha sido vulnerado en filtraciones.
