# Fase 1: Recopilación de Información (Pasiva y Activa)

Esta es la primera fase de cualquier prueba de penetración. Consiste en recolectar información sobre una persona, empresa, sitio web o sistema objetivo.
Los datos obtenidos (nombres, emails, tecnologías) pueden ser cruciales en fases posteriores, por ejemplo para ataques de phishing o para acceder a la red interna.

> “El que más sabe sobre su objetivo es el que más éxito tendrá en las etapas finales de la prueba.” – Por eso es la fase más importante.

---

## 1.1 Recopilación Pasiva
Implica recopilar información **sin interactuar directamente** con el objetivo. Ejemplo: analizar un sitio web desde la perspectiva del usuario final, descubrir tecnologías, IP real, subdominios, etc.

### Áreas de interés
- IP y DNS
- Nombres de dominio y otros dominios relacionados
- Emails y redes sociales
- Tecnologías que usa el sitio
- Identificar subdominios

### Herramientas y comandos

#### whatis & host
whatis host: muestra una breve descripción del comando host (utilidad de consulta DNS).
host hackersploit.org
host <dominio>: realiza una consulta DNS para obtener la dirección IP del dominio.

### robots.txt
Se accede añadiendo /robots.txt a la URL del sitio. Es un archivo que indica qué rutas no deben ser indexadas por los motores de búsqueda.
Ejemplo de contenido:
Disallow: /wp-admin/

Esto sugiere que el sitio usa WordPress y que la ruta /wp-admin/ está deshabilitada para los buscadores.

### sitemap.xml
Se accede con /sitemap.xml. Es un mapa del sitio que lista las URLs disponibles, útil para descubrir contenido.

### BuiltWith (extensión navegador) y Wappalyzer
Extensiones para Firefox/Chrome que perfilan las tecnologías usadas por un sitio web.

### whatweb
```whatweb hackersploit.org```
whatweb: herramienta de línea de comandos que identifica tecnologías en un sitio web (CMS, frameworks, servidores, etc.).

### HTTrack (clonado de sitio web)
```sudo apt-get install webhttrack```
```httrack http://target.ine.local -O /home/kali/Desktop/target_mirror```
httrack: descarga una copia completa del sitio web para análisis offline de su código fuente y archivos.

### WHOIS
```whois hackersploit.org```
whois: consulta la base de datos WHOIS y devuelve información de registro del dominio (titular, fechas, servidores DNS, etc.).

### NetCraft
Sitio web que proporciona informes detallados sobre tecnologías, hosting e historial de un sitio.

### Reconocimiento DNS con dnsrecon y dnsdumpster
```dnsrecon -h```
```dnsrecon -d hackersploit.org```

```dnsrecon -d <dominio>: herramienta de enumeración DNS que realiza varios tipos de consultas.```

Recomendación: usar dnsdumpster.com para obtener la información DNS organizada visualmente.

### WAF Detección con wafw00f
```wafw00f -l                   # Lista los firewalls que puede detectar```

```wafw00f hackersploit.org     # Detecta si hay un WAF (ej. Cloudflare)```

wafw00f: identifica la presencia y el tipo de Web Application Firewall.

### Enumeración de subdominios con Sublist3r
```sublist3r -d hackersploit.org -e google,yahoo```

sublist3r -d <dominio> -e <motores>: búsqueda pasiva de subdominios usando motores de búsqueda. Usar VPN para evitar bloqueos de Google.

### Google Dorks
Filtros avanzados de Google para encontrar información pública específica.
```site:ine.com inurl:admin```

```site:ine.com inurl:forum```

```site:*.ine.com```

```site:*ine.com inurl:admin```

```site:*ine.com intitle:admin```

```site:ine.com filetype:pdf```

```site:ine.com filetype:pdf marketing```

```site:ine.com filetype:xlsx```

```site:ine.com filetype:docs```


```site:ine.com filetype:zip```

```site:ine.com filetype:docx```

```site:ine.com employees```

```site:ine.com instructors```

```intitle:index of```

```inurl:auth_user_file.txt   # Posible archivo de contraseñas```

```inurl:passwd.txt```

site:, inurl:, intitle:, filetype:: operadores para filtrar por dominio, ruta, título o tipo de archivo.

Wayback Machine (web.archive.org): permite ver versiones históricas de un sitio.

### Enumeración de emails con theHarvester
```sudo apt search theharvester```

```theHarvester --help```

```theHarvester -d INE -b duckduckgo```

```theHarvester -d INE -b duckduckgo,bing,yahoo```

```theHarvester -d ine.com -b duckduckgo,bing,yahoo,urlscam,pentesttool```

theHarvester: recopila correos electrónicos, nombres y subdominios de fuentes públicas.

### Base de datos de contraseñas filtradas
Comprobar si un email o teléfono está en filtraciones: haveibeenpwned.com.

## 1.2 Recopilación Activa
Implica interactuar directamente con el sistema objetivo: escaneo de puertos, descubrimiento de servicios, enumeración de infraestructura interna.
Precaución: Realizar escaneos sin permiso por escrito es ilegal.

### Tipos de registros DNS

| Registro | Descripción |
|----------|-------------|
| A        | Dirección IPv4 |
| AAAA     | Dirección IPv6 |
| NS       | Servidores de nombres del dominio real |
| MX       | Servidor de correo |
| CNAME    | Alias del dominio |
| TXT      | Texto arbitrario (verificación, spf, etc.) |
| HINFO    | Información del host |
| SOA      | Autoridad del dominio (Start of Authority) |
| SRV      | Servicios disponibles |
| PTR      | Resolución inversa (IP → nombre de host) |

### Zona de transferencia DNS (AXFR)
```dnsrecon -d zonetransfer.me```

```sudo vim /etc/hosts          # Contiene mapeos locales IP ↔ nombre```

```dnsenum zonetransfer.me```

dnsenum: enumeración DNS con múltiples técnicas, incluida la transferencia de zona.

### whatis dig
```dig axfr @nsztm1.digi.ninja zonetransfer.me```

dig axfr @<servidor_dns> <dominio>: intenta una transferencia de zona completa si el servidor lo permite.
```dnsrecon -d hackersploit.org```

```dnsenum hackersploit.org```

### fierce
```man fierce```

```fierce -h```

```fierce -dns zonetransfer.me```

fierce: herramienta de enumeración DNS diseñada para localizar espacios de nombres internos y externos.

# 2. Escaneo y Enumeración con Nmap

### Descubrimiento de hosts activos
```sudo nmap -sn 192.168.1.0/24```
-sn: Ping Scan, no realiza escaneo de puertos; solo descubre equipos activos.

### NetDiscover
```sudo apt-get install netdiscover```

```netdiscover -i eth0 -r 192.168.1.0/24```

netdiscover: escaneo ARP pasivo/activo para identificar hosts en una red local.

### Opciones principales de Nmap
```nmap <IP>                       # Escanea los 1000 puertos TCP más comunes```

```nmap -Pn <IP>                   # Omite el descubrimiento (trata todos los hosts como online)```

```nmap -Pn -p- <IP>               # Escanea los 65535 puertos TCP```

```nmap -p 80 <IP>                 # Sólo el puerto 80```

```nmap -p 8080 <IP>               # Sólo el puerto 8080```

```nmap -p1-1000 <IP>              # Rango de puertos del 1 al 1000```

```nmap -F <IP>                    # Escaneo rápido (top 100 puertos)```

```nmap -sU <IP>                   # Escaneo UDP```

```nmap -Pn -F -sV <IP>            # Omite host discovery, rápido, detección de versiones```

```nmap -Pn -F -sV -O -sC <IP> -v  # + Detección de SO, scripts por defecto, verbose```

-Pn: omite la fase de descubrimiento; útil cuando el firewall bloquea pings.

-sV: sondeo de versiones de servicios.

-O: detección del sistema operativo.

-sC: ejecución de scripts NSE por defecto.

-v: modo verbose (más información durante el escaneo).

### Sobre puertos filtrados en Nmap
Nmap no puede determinar si están abiertos porque un firewall o filtro descarta los paquetes. Esto ralentiza el escaneo al reintentar las sondas. A veces responden con ICMP tipo 3 código 13 (comunicación prohibida).

# 3. Enumeración de Servicios Específicos

### 3.1 Enumeración Web y Directorios

### gobuster
```gobuster dir -u http://target.ine.local -w /usr/share/wordlists/dirb/common.txt -x bak,old,zip,sql,tar.gz,php.bak```

```gobuster dir: fuerza bruta de directorios/archivos en un servidor web.```
-x: extensiones a probar.

### HTTrack para descubrir archivos ocultos

Pista: "ciertos archivos pueden revelar algo interesante cuando se reflejan" (cuando se clona el sitio).
httrack http://target.ine.local -O /home/kali/Desktop/target_mirror
Al clonar el sitio, aparecen archivos no visibles en un escaneo normal, por ejemplo xmlrpc0db0.php.

### Revisar el archivo descargado:
```cat /home/kali/Desktop/target_mirror/target.ine.local/xmlrpc0db0.php```

Contenido de ejemplo con bandera incrustada:
```<api name="FLAG5{c9c52f584e364e719faaa4bb83d1aa33}" ... />```

### curl para inspeccionar cabeceras y contenido
```curl -I http://target.ine.local/secret-info/```
Respuesta:
HTTP/1.1 200 OK
Server: FLAG1_befcdbc2a7224bf1ab808c3872c03443

```curl http://target.ine.local/secret-info/```
["flag.txt"]

```curl http://target.ine.local/secret-info/flag.txt```
FLAG2_138a312a8afd48369c72e91bd1cdd05b

### curl -I: muestra solo los encabezados HTTP.
A veces la bandera está en un encabezado del servidor y no en el cuerpo.

### 3.2 Enumeración FTP

### Verificar si el puerto FTP (21) está abierto:
```nmap -p 21 target.ine.local```

```nmap -sV target.ine.local```

### Conexión FTP anónima:
```ftp target.ine.local```
Name: anonymous
Password: anonymous (o vacío)

### Comandos FTP dentro de la sesión:
```ftp> ls```
```ftp> get flag.txt```
```ftp> get creds.txt```
```ftp> bye```

### Ver archivos descargados:
```ls```
```cat flag.txt       # FLAG3_b09e93fc63d24140846e16ec1115bca5```

```cat creds.txt      # db_admin:password@123```

### 3.3 Enumeración MySQL

### Verificar puerto 3306:
```nmap -p 3306 target.ine.local```

### Conectarse con las credenciales obtenidas:
```mysql -h target.ine.local -u db_admin -p```
Password: password@123

### Dentro de MySQL:
SHOW DATABASES;
-- Aparece: FLAG4_7591346de419484d98af421a412d9833

# 4. Uso de Metasploit para Enumeración

### 4.1 Importar resultados de Nmap a MSF

### 1. Realizar escaneo con salida XML:
```nmap -sV -Pn -oX myscan.xml demo.ine.local```

### 2. Cargar en Metasploit:
```msfconsole -q```

```db_import myscan.xml```

### 3. Ver resultados:
```msf6 > hosts```

```msf6 > services```

### 4.2 Enumeración FTP con módulos auxiliares

### Versión del FTP:
```msf6 > use auxiliary/scanner/ftp/ftp_version```

```msf6 > set RHOSTS demo.ine.local```

```msf6 > run```

### Fuerza bruta FTP:
```msf6 > use auxiliary/scanner/ftp/ftp_login```

```msf6 > set RHOSTS demo.ine.local```

```msf6 > set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt```

```msf6 > set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt```

```msf6 > run```

### Sesión anónima:
```msf6 > use auxiliary/scanner/ftp/anonymous```
```msf6 > set RHOSTS demo.ine.local```
```msf6 > run```

Si es exitosa, conectarse manualmente:
```ftp demo.ine.local```

### 4.3 Enumeración SMB (Samba)

### 1. Encontrar puertos TCP predeterminados de smbd:
```nmap demo.ine.local```

### 2. Puertos UDP usados por nmbd:
```nmap -sU --top-ports 25 demo.ine.local   # 137, 138```

### 3. Nombre del grupo de trabajo y versión:
```nmap -sV -p 445 demo.ine.local```

### 4. Versión exacta con script NSE:
```nmap --script smb-os-discovery.nse -p 445 demo.ine.local```

### 5. Versión exacta con Metasploit:
```msf6 > use auxiliary/scanner/smb/smb_version```

```msf6 > set RHOSTS demo.ine.local```

```msf6 > exploit```

### 6. Nombre NetBIOS:
```nmap --script smb-os-discovery.nse -p 445 demo.ine.local   # Nombre: SAMBA-RECON```

```nmblookup -A demo.ine.local                                # Nombre: SAMBA-RECON```

### 7. Conexión anónima con smbclient:
```smbclient -L demo.ine.local -N```
-N: sin contraseña; si se listan recursos, la sesión nula está permitida.

### 8. Conexión anónima con rpcclient:
```rpcclient -U "" -N demo.ine.local```

### 4.4 Enumeración de Apache (HTTP)

### 1. Verificar conectividad:
```ping -c 5 victim-1```

### 2. Abrir msfconsole:
```msfconsole -q```

### 3. Módulos de enumeración web (ejecutar uno por uno):
http_version – versión del servidor:
```use auxiliary/scanner/http/http_version```

```set RHOSTS victim-1```

```run```

### robots_txt – analiza robots.txt:
```use auxiliary/scanner/http/robots_txt```

```set RHOSTS victim-1```

```run```

### http_header – cabeceras HTTP (útiles para información de servidor y posibles flags):
```use auxiliary/scanner/http/http_header```

```set RHOSTS victim-1```

```run```

```set TARGETURI /secure```

```run```

### brute_dirs – fuerza bruta de directorios con wordlist interna:
```use auxiliary/scanner/http/brute_dirs```

```set RHOSTS victim-1```

```run```

### dir_scanner – fuerza bruta de directorios con diccionario personalizado:
```use auxiliary/scanner/http/dir_scanner```

```set RHOSTS victim-1```

```set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt```

```run```

### dir_listing – comprueba si un directorio tiene listado habilitado:
```use auxiliary/scanner/http/dir_listing```

```set RHOSTS victim-1```

```set PATH /data```

```run```

### files_dir – busca archivos comunes en una ruta:
```use auxiliary/scanner/http/files_dir```

```set RHOSTS victim-1```

```set VERBOSE false```

```run```

### http_put – prueba el método HTTP PUT para subir archivos:
```use auxiliary/scanner/http/http_put```

```set RHOSTS victim-1```

```set PATH /data```

```set FILENAME test.txt```

```set FILEDATA "Welcome To AttackDefense"```

```run```

### Verificar archivo subido:
```wget http://victim-1:80/data/test.txt```

```cat test.txt```
Contenido: Welcome To AttackDefense

### Eliminar el archivo con DELETE:
```use auxiliary/scanner/http/http_put```

```set RHOSTS victim-1```

```set PATH /data```

```set FILENAME test.txt```

```set ACTION DELETE```

```run```

### Confirmar que ya no existe (error 404):
```wget http://victim-1:80/data/test.txt```

### http_login – fuerza bruta de inicio de sesión HTTP:
```use auxiliary/scanner/http/http_login```

```set RHOSTS victim-1```

```set AUTH_URI /secure/```

```set VERBOSE false```

```run```

### apache_userdir_enum – enumeración de usuarios de Apache (mod_userdir):
```use auxiliary/scanner/http/apache_userdir_enum```

```set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt```

```set RHOSTS victim-1```

```set VERBOSE false```

```run```




