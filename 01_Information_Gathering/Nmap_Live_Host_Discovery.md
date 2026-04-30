# 🔎 01_Information_Gathering – Nmap_Live_Host_Discovery.md

## 📡 Identificación de Hosts Activos (Ping Sweep)

Antes de cualquier escaneo de puertos, es necesario saber qué dispositivos están encendidos en la red. Nmap permite hacer esta comprobación sin realizar un escaneo de puertos real.

### 🟢 Nmap `-sn` (Ping Sweep)

```bash
sudo nmap -sn <ip>/<subred>
```

**Ejemplo práctico:**
```bash
sudo nmap -sn 192.168.1.0/24
```

Este comando **no escanea puertos** (`-sn`), únicamente descubre hosts activos mediante peticiones ICMP y otras sondas.

### 🔵 NetDiscover (Pasivo con ARP)

Alternativa para redes locales. Escucha peticiones ARP y muestra los dispositivos activos.

```bash
# Instalación
sudo apt-get install netdiscover

# Ejecución en interfaz eth0 para la subred indicada
netdiscover -i eth0 -r 192.168.1.0/24
```

Así se pueden enumerar los dispositivos de una red local sin enviar tráfico activo de escaneo de puertos.

---

## 🚪 Escaneo de Puertos con Nmap

Una vez identificado un host activo (`<ip>`), se procede a escanear sus puertos.

### 🔸 Escaneo básico (1000 puertos más comunes)

```bash
nmap <ip>
```

Nmap por defecto escanea los **1000 puertos TCP más utilizados**. No recorre todo el rango (1-65535).

### 🔸 Bandera `-Pn` (Forzar escaneo)

En ocasiones los firewalls bloquean las sondas de descubrimiento. La opción `-Pn` **omite el descubrimiento de host** y fuerza el escaneo de puertos como si el objetivo estuviera activo.

```bash
nmap -Pn <ip>
```

Esto muestra una lista de puertos (por ejemplo, 993 puertos cerrados o filtrados) y también el servicio en los puertos abiertos.

### 🔸 Control del rango de puertos

| Comando / Flag | Descripción |
|----------------|-------------|
| `nmap -Pn -p- <ip>` | Todos los puertos (1-65535) |
| `-p 80` | Solo puerto 80 |
| `-p 8080` | Solo puerto 8080 |
| `-p 1-1000` | Del puerto 1 al 1000 |
| `-F` | Escaneo rápido de los 100 puertos principales |
| `-sU` | Escaneo de puertos **UDP** |

### 🔸 Detección de versiones, sistema operativo y scripts

```bash
nmap -Pn -F -sV                # Versiones de servicios en puertos rápidos
nmap -Pn -F -sV -O -sC <ip> -v # Con detección de SO, scripts por defecto y verboso
```

**Detalle de flags:**

| Flag | Descripción |
|------|-------------|
| `-Pn` | Omite el ping previo (host discovery) |
| `-F` | Escaneo rápido (100 puertos) |
| `-sV` | Detecta versión de servicios |
| `-O` | Intenta identificar el sistema operativo |
| `-sC` | Ejecuta los scripts por defecto de Nmap |
| `-v` | Modo verbose (detalles durante el escaneo) |

---

## 🛡️ Entendiendo los Puertos Filtrados

Nmap no puede determinar si un puerto está abierto porque el filtrado de paquetes impide que sus sondas lleguen al puerto. El filtrado puede originarse en un firewall dedicado, reglas de router o software de firewall en el host.

Estos puertos **frustran a los atacantes** porque proporcionan muy poca información. A veces responden con mensajes ICMP de error (tipo 3, código 13: destino inalcanzable, comunicación prohibida), pero lo más común es que los filtros simplemente descarten las sondas sin responder.

Para manejar esta incertidumbre, Nmap reintenta varias veces por si la sonda se perdió por congestión de red, lo que ralentiza el escaneo.

---

**Nota:** Los comandos aquí mostrados se ejecutan sobre una IP o rango de IP de ejemplo; en un entorno real se sustituyen por los datos del objetivo.
