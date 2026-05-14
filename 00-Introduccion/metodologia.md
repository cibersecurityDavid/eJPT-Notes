# 🎓 Introducción y Metodología de Estudio

## 📌 ¿Qué es el eJPT?

El **eLearnSecurity Junior Penetration Tester (eJPT)** es una certificación de **INE** que valida los conocimientos fundamentales en **pruebas de penetración** (pentesting).  
A diferencia de otros exámenes teóricos, el eJPT es **100% práctico** y se basa en un entorno de laboratorio real donde deberás:

- Enumerar servicios y sistemas.
- Identificar vulnerabilidades.
- Explotarlas para obtener acceso.
- Escalar privilegios.
- Capturar banderas que demuestren el compromiso de los objetivos.

---

## 🧠 Metodología de estudio recomendada

Para aprovechar al máximo los laboratorios de INE y estas notas, se sugiere seguir un flujo de trabajo ordenado:

| Fase | Acción | Objetivo |
|------|--------|----------|
| **1. Comprender el concepto** | Leer la teoría y ver las demostraciones en video | Entender el "por qué" antes del "cómo" |
| **2. Reproducir el laboratorio** | Ejecutar cada comando manualmente, sin ayudas | Desarrollar memoria muscular y resolución de errores |
| **3. Tomar notas propias** | Escribir lo aprendido con tus palabras | Facilitar el repaso posterior |
| **4. Comparar con estas notas** | Revisar el archivo `.md` correspondiente | Completar vacíos o corregir errores |
| **5. Repetir en variaciones** | Modificar escenarios (diferentes IPs, puertos, herramientas) | Ganar flexibilidad ante el examen |

---

## 🗺️ Flujo de un Pentest (según el eJPT)

El examen sigue la estructura clásica de una prueba de penetración. Las carpetas de este repositorio están organizadas en ese mismo orden:

```
1. Information Gathering   → Recopilar información pasiva y activa del objetivo
2. Enumeration              → Enumerar servicios, usuarios y recursos
3. Vulnerability Assessment → Identificar vulnerabilidades (Nessus, WMAP, manual)
4. Exploitation             → Explotar las vulnerabilidades encontradas
5. Post-Exploitation        → Mantener acceso, extraer credenciales, keylogging
6. Privilege Escalation     → Elevar privilegios a administrador/root
7. CTF Complete Writeups    → Ejercicios integradores que combinan todas las fases
```

---

## 📂 Cómo usar este repositorio

- Cada carpeta representa una **fase del pentest**.
- Dentro de cada carpeta hay un archivo `.md` por **laboratorio o técnica**.
- Los archivos contienen:
  - 🔍 Procedimientos paso a paso.
  - 💻 Comandos exactos con sus parámetros.
  - 📊 Salidas de terminal esperadas.
  - 🏁 Banderas capturadas (en formato hash MD5).
  - 🧠 Explicaciones de los conceptos clave.

### ⚠️ Recomendaciones importantes

- **No** uses estas notas como un atajo para saltarte los laboratorios. El valor del eJPT está en la práctica real.
- Las IPs, nombres de dominio y banderas corresponden exclusivamente a los **laboratorios de INE**. No intentes replicarlas fuera de ese entorno.
- Ante cualquier duda, vuelve a la plataforma de INE o consulta la documentación oficial de cada herramienta.

---

## 🛠️ Herramientas principales utilizadas

| Herramienta       | Categoría              | Uso en los laboratorios |
|-------------------|------------------------|-------------------------|
| **Nmap**          | Escaneo de red         | Descubrimiento de hosts, puertos y versiones |
| **Metasploit**    | Explotación            | Módulos auxiliares, exploits y post-explotación |
| **Hydra**         | Fuerza bruta           | Ataques de diccionario a SSH, FTP, SMB, RDP, HTTP |
| **Smbclient**     | Enumeración SMB        | Conexión anónima y autenticada a recursos compartidos |
| **Enum4linux**    | Enumeración SMB        | Extracción masiva de información de Samba |
| **Cadaver**       | WebDAV                 | Cliente WebDAV interactivo |
| **Davtest**       | WebDAV                 | Probar capacidades de subida y ejecución |
| **Xfreerdp**      | Escritorio Remoto      | Conexión RDP desde Linux |
| **WPScan**        | WordPress              | Enumeración de plugins y vulnerabilidades |
| **Searchsploit**  | Exploits               | Búsqueda de exploits públicos en Exploit-DB |
| **Netcat**        | Red                    | Conexiones TCP/UDP y listeners para shells inversas |

---

## 🌟 Consejos finales para el examen

- **Enumeración, enumeración y enumeración**. La mayoría de las banderas se obtienen escaneando correctamente.
- **Guarda todas las salidas** de Nmap y otros escaneos; ahorrarás tiempo si necesitas volver a consultarlas.
- **Prueba herramientas manuales** antes de usar Metasploit. El examen valora la comprensión de bajo nivel.
- **Controla el tiempo**. Si te atascas en una bandera, pasa a la siguiente y vuelve después con otra perspectiva.
- **Confía en tu metodología**. Si has completado estos laboratorios con éxito, tienes todo lo necesario para aprobar.

---

*¡Mucho éxito en tu preparación y en el examen!* 🛡️
