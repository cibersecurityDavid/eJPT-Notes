# 📥 Importación de Resultados de Nmap a Metasploit

## 🔍 Objetivo

Una vez realizado un escaneo con Nmap, los resultados pueden importarse a Metasploit Framework (MSF) para centralizar la información de hosts y servicios, y utilizarlos posteriormente en módulos de explotación.

## ⚙️ Escaneo previo con Nmap

Antes de importar, se debe ejecutar un escaneo que genere un archivo en formato XML. En entornos donde las solicitudes de eco ICMP (ping) están bloqueadas por firewalls, se debe omitir la fase de descubrimiento de hosts con `-Pn` para asumir que los objetivos están en línea.

```bash
nmap -sV -Pn -oX myscan.xml demo.ine.local
```

| Flag   | Función |
|--------|---------|
| `-sV`  | Detección de versiones de servicios |
| `-Pn`  | Omite el descubrimiento de host (útil si el firewall bloquea pings) |
| `-oX`  | Guarda la salida en formato XML (`myscan.xml`) |

## 📂 Importación a Metasploit

Iniciar la consola de Metasploit:

```bash
msfconsole
```

Importar el archivo XML generado:

```bash
db_import myscan.xml
```

## 📊 Consulta de los datos importados

Una vez importado, se pueden consultar los hosts y servicios almacenados en la base de datos de MSF con los siguientes comandos:

```bash
hosts
services
```

**Explicación:**
- `hosts` → muestra todos los hosts detectados durante el escaneo.
- `services` → lista los puertos abiertos y los servicios asociados a cada host.
