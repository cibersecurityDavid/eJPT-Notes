# 🌐 01_Information_Gathering – Active_DNS_Reconnaissance.md

> **Nota:** En esta fase se interactúa directamente con los servidores DNS del objetivo para obtener registros, intentar transferencias de zona y recopilar información detallada.

---

## 📚 Tipos de Registros DNS

Antes de lanzar comandos, es fundamental conocer los tipos de registro que se pueden encontrar durante la enumeración activa.

| Registro | Nombre Completo         | Descripción |
|----------|-------------------------|-------------|
| **A**    | Address                 | Mapea un nombre de dominio a una dirección **IPv4**. |
| **AAAA** | IPv6 Address            | Mapea un nombre de dominio a una dirección **IPv6**. |
| **NS**   | Name Server             | Indica cuáles son los servidores de nombres autoritativos del dominio. |
| **MX**   | Mail Exchange           | Resuelve el dominio del servidor de correo y su prioridad. |
| **CNAME**| Canonical Name          | Define un alias para un nombre de dominio (apunta a otro dominio). |
| **TXT**  | Text                    | Contiene información textual; a menudo se usa para verificación SPF, DKIM, etc. |
| **HINFO**| Host Information        | Describe el hardware y sistema operativo del host (poco usado). |
| **SOA**  | Start of Authority      | Indica el servidor DNS primario, el correo del administrador, tiempos de refresco, etc. |
| **SRV**  | Service Record          | Especifica la ubicación de servicios específicos (ej. LDAP, XMPP). |
| **PTR**  | Pointer                 | Resuelve una dirección IP en un nombre de host (registro **inverso**). |

---

## 🔄 DNS Zone Transfer (Transferencia de Zona)

Cuando un servidor DNS está mal configurado, permite que un servidor secundario (o un atacante) realice una **copia completa de todos los registros** de una zona. A esto se le llama **AXFR** (full zone transfer).

> ⚠️ Las transferencias de zona **no autorizadas** son un fallo de seguridad grave, ya que exponen la estructura interna del dominio.

### 🔍 Herramientas para intentar la transferencia de zona

---

## 🧰 Comandos en Kali

### 1. dnsrecon

Se usa para enumerar registros y probar transferencias de zona.

```bash
dnsrecon -d zonetransfer.me          # Enumera todos los registros de la zona (si se permite)
dnsrecon -d hackersploit.org         # Intenta enumerar el dominio hackersploit.org
```

### 2. Consulta del archivo /etc/hosts

Antes de realizar consultas DNS, Kali puede resolver nombres localmente mediante el archivo `/etc/hosts`.

```bash
sudo vim /etc/hosts                  # Contiene una lista de nombres de host y sus IPs; se puede modificar para pruebas
```

### 3. dnsenum

Herramienta similar a dnsrecon, especializada en enumeración y transferencias de zona.

```bash
dnsenum zonetransfer.me              # Intenta transferencia y enumera registros
dnsenum hackersploit.org             # Sobre el dominio de ejemplo
```

### 4. dig (Domain Information Groper)

Comando clásico para consultas DNS. Ver su definición:

```bash
whatis dig
```

Ejemplo de **transferencia de zona completa** usando `dig`:

```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```
> **Explicación:**  
> `axfr` solicita la transferencia de zona.  
> `@nsztm1.digi.ninja` especifica el servidor de nombres autoritativo.  
> `zonetransfer.me` es el dominio de práctica (permite AXFR).

### 5. fierce

Herramienta de fuerza bruta de subdominios y enumeración DNS. Se usa cuando la transferencia de zona no está permitida.

```bash
man fierce                          # Página de manual
fierce -h                           # Ayuda
fierce -dns zonetransfer.me        # Escanea zonetransfer.me en busca de subdominios y registros
```

---

📌 **Todos estos comandos se ejecutan directamente contra el dominio objetivo.**  
El dominio **zonetransfer.me** es un entorno de pruebas diseñado para que los estudiantes practiquen transferencias de zona sin temor a consecuencias legales.
