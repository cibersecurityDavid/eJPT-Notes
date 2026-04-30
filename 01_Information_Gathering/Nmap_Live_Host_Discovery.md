# 📡 Descubrimiento de Hosts Activos (Ping Sweep)

Antes de cualquier escaneo de puertos, es necesario identificar qué dispositivos están encendidos en la red. Nmap permite esta comprobación sin realizar un escaneo de puertos real.

## 🟢 Nmap `-sn` (Ping Sweep)

```bash
sudo nmap -sn <ip>/<subred>
```

**Ejemplo práctico:**
```bash
sudo nmap -sn 192.168.1.0/24
```

El flag `-sn` **inhabilita el escaneo de puertos** y se limita a descubrir hosts activos mediante peticiones ICMP y otras sondas.

## 🔵 NetDiscover (Escucha ARP pasiva)

Alternativa para redes locales. Escucha las peticiones ARP para mostrar los dispositivos activos sin enviar tráfico de escaneo de puertos.

```bash
# Instalación
sudo apt-get install netdiscover

# Ejecución en interfaz eth0 para la subred indicada
netdiscover -i eth0 -r 192.168.1.0/24
```

## 🚪 Escaneo de Puertos con Nmap

Una vez identificado un host activo (`<ip>`), se procede a escanear sus puertos.

### 🔸 Escaneo básico (1000 puertos más comunes)

```bash
nmap <ip>
```

Por defecto, Nmap escanea los **1000 puertos TCP más utilizados**. No recorre el rango completo 1-65535.

### 🔸 Bandera `-Pn` (Forzar escaneo)

Cuando los firewalls bloquean las sondas de descubrimiento, la opción `-Pn` **omite el descubrimiento de host** y fuerza el escaneo de puertos como si el objetivo estuviera activo.

```bash
nmap -Pn <ip>
```

Esto muestra una lista de puertos (por ejemplo, 993 cerrados o filtrados) junto con el servicio detectado en los puertos abiertos.

### 🔸 Control del rango de puertos

| Comando / Flag | Descripción |
|----------------|-------------|
| `nmap -Pn -p- <ip>` | Todos los puertos (1-65535) |
| `-p 80` | Solo el puerto 80 |
| `-p 8080` | Solo el puerto 8080 |
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

## 🛡️ Entendiendo los Puertos Filtrados

Nmap no puede determinar si un puerto está abierto cuando el filtrado de paquetes impide que sus sondas lleguen a él. El filtrado puede provenir de un firewall dedicado, reglas de router o software de firewall en el host.

Estos puertos **frustran a los atacantes** porque apenas aportan información. A veces responden con mensajes ICMP de error (tipo 3, código 13: destino inalcanzable, comunicación prohibida), pero lo más habitual es que los filtros descarten las sondas sin responder.

Para mitigar falsos negativos, Nmap reintenta varias veces, lo que retrasa el escaneo.
