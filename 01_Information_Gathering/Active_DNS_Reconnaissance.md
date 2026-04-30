# 🌐 Reconocimiento DNS Activo

## Tipos de Registros DNS

Antes de ejecutar cualquier comando, es fundamental conocer los distintos tipos de registro que pueden devolver los servidores DNS.

| Registro | Nombre Completo         | Descripción |
|----------|-------------------------|-------------|
| **A**    | Address                 | Asocia un dominio a una dirección **IPv4** |
| **AAAA** | IPv6 Address            | Asocia un dominio a una dirección **IPv6** |
| **NS**   | Name Server             | Indica los servidores de nombres autoritativos del dominio |
| **MX**   | Mail Exchange           | Resuelve el servidor de correo y su prioridad |
| **CNAME**| Canonical Name          | Define un alias para un dominio (apunta a otro dominio) |
| **TXT**  | Text                    | Contiene texto descriptivo; se usa a menudo para SPF, DKIM, etc. |
| **HINFO**| Host Information        | Describe el hardware y sistema operativo del host (poco utilizado) |
| **SOA**  | Start of Authority      | Contiene el servidor DNS primario, el correo del administrador y parámetros de refresco |
| **SRV**  | Service Record          | Especifica la ubicación de servicios concretos (LDAP, XMPP, etc.) |
| **PTR**  | Pointer                 | Resuelve una dirección IP en un nombre de host (registro **inverso**) |

## Transferencia de Zona DNS (AXFR)

Cuando un servidor DNS está mal configurado, permite a un servidor secundario (o a un atacante) solicitar una **copia completa** de todos los registros de la zona. Esta operación se denomina **transferencia de zona completa** (AXFR).

> ⚠️ Una transferencia de zona no autorizada expone la estructura interna del dominio y se considera un fallo de seguridad.

## Herramientas

### dnsrecon

Enumera registros e intenta la transferencia de zona.

```bash
dnsrecon -d zonetransfer.me          # Enumera todos los registros (si se permite AXFR)
dnsrecon -d hackersploit.org         # Intenta la enumeración sobre el dominio indicado
```

### El archivo /etc/hosts

Kali puede resolver nombres localmente mediante el archivo `/etc/hosts`, que contiene una lista de nombres de host y sus IPs asociadas.

```bash
sudo vim /etc/hosts
```

### dnsenum

Cumple una función similar a `dnsrecon`, con capacidades de transferencia de zona.

```bash
dnsenum zonetransfer.me
dnsenum hackersploit.org
```

### dig (Domain Information Groper)

Consulta DNS clásica. Permite solicitar una transferencia de zona completa.

```bash
whatis dig
```

Ejemplo de transferencia de zona con `dig`:
```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```
Explicación:  
`axfr` → solicita la transferencia de zona.  
`@nsztm1.digi.ninja` → especifica el servidor de nombres autoritativo.  
`zonetransfer.me` → dominio de prácticas que permite AXFR.

### fierce

Herramienta de fuerza bruta de subdominios y enumeración DNS; especialmente útil cuando la transferencia de zona no está permitida.

```bash
man fierce                  # Página de manual
fierce -h                   # Ayuda
fierce -dns zonetransfer.me # Descubre subdominios y registros
```

`zonetransfer.me` es un entorno de pruebas diseñado para practicar transferencias de zona sin implicaciones legales.
