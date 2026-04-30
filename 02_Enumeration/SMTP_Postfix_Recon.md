# 📧 Postfix Recon: Conceptos Básicos

## 📡 Identificación del servidor SMTP

```bash
nmap -sV --script banner demo.ine.local
```

| Parámetro             | Valor |
|-----------------------|-------|
| **Servidor**          | Postfix |
| **Banner**            | `openmailbox.xyz ESMTP Postfix: Welcome to our mail server.` |

## 🔌 Conexión manual con netcat

Se utiliza `netcat` para interactuar directamente con el puerto 25 y obtener el nombre de dominio.

```bash
nc demo.ine.local 25
```

Una vez conectado, el servidor envía el banner. El nombre de host extraído es **openmailbox.xyz**.

## 👤 Verificación de usuarios (VRFY)

Dentro de la sesión netcat se pueden verificar usuarios existentes mediante el comando `VRFY`.

### ✅ Usuario `admin`

```
VRFY admin@openmailbox.xyz
252 2.0.0 admin@openmailbox.xyz
```

**Resultado:** El usuario **existe**.

### ❌ Usuario `commander`

```
VRFY commander@openmailbox.xyz
```

**Resultado:** El usuario **no existe**.

## 🧪 Comandos admitidos (HELO / EHLO)

Se utiliza `telnet` para comprobar las capacidades del servidor:

```bash
telnet demo.ine.local 25
```

Dentro de la sesión tecleamos:

```
HELO attacker.xyz
EHLO attacker.xyz
```

`EHLO` nos muestra los comandos SMTP extendidos soportados.

## 🧮 Enumeración masiva de usuarios

### smtp-user-enum

Emplea un diccionario externo para contar cuántos nombres existen en el servidor.

```bash
smtp-user-enum -U /usr/share/commix/src/txt/usernames.txt -t demo.ine.local
```

**Resultado:** **8** usuarios válidos del diccionario proporcionado.

### Módulo Metasploit `smtp_enum`

Utiliza la lista `unix_users.txt` y devuelve el número exacto de coincidencias.

```bash
msfconsole -q
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS demo.ine.local
exploit
```

**Resultado:** **22** usuarios encontrados.

| Herramienta         | Diccionario usado                                       | Usuarios encontrados |
|---------------------|---------------------------------------------------------|----------------------|
| `smtp-user-enum`    | `/usr/share/commix/src/txt/usernames.txt`              | 8                    |
| `auxiliary/scanner/smtp/smtp_enum` | `/usr/share/metasploit-framework/data/wordlists/unix_users.txt` | 22 |

## ✉️ Envío de correos falsos

### Mediante telnet

Enviar un correo manualmente al usuario `root`:

```bash
telnet demo.ine.local 25
```

```
HELO attacker.xyz
mail from: admin@attacker.xyz
rcpt to: root@openmailbox.xyz
data
Subject: Hi Root
Hello,
This is a fake mail sent using telnet command.
From,
Admin
.
```

El punto final (`.`) indica la terminación del mensaje.

### Mediante `sendemail`

```bash
sendemail -f admin@attacker.xyz -t root@openmailbox.xyz -s demo.ine.local -u Fakemail -m "Hi root, a fake from admin" -o tls=no
```

| Opción       | Valor                       |
|--------------|-----------------------------|
| `-f`         | Remitente                   |
| `-t`         | Destinatario                |
| `-s`         | Servidor SMTP               |
| `-u`         | Asunto                      |
| `-m`         | Mensaje                     |
| `-o tls=no`  | Desactiva TLS para la demo  |

## 🧠 Conclusión

El reconocimiento del servidor Postfix incluye la obtención del banner, la verificación manual de usuarios con `VRFY`, la enumeración masiva con `smtp-user-enum` y `Metasploit`, y la capacidad de enviar correos falsos con `telnet` o `sendemail`. Estas técnicas permiten evaluar la exposición del servidor de correo.
