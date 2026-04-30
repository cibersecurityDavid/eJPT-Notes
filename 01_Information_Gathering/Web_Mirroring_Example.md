# 🪞 Clonado de Sitios Web con HTTrack

## 🔍 Objetivo

Descargar una copia completa de un sitio web para analizar archivos que a veces no son visibles mediante un simple escaneo de directorios. La pista _"ciertos archivos pueden revelar algo interesante cuando se reflejan"_ hace referencia precisamente al uso de HTTrack.

## ⚙️ Descarga del sitio

```bash
httrack http://target.ine.local -O /home/kali/Desktop/target_mirror
```

**Salida de ejemplo:**

```
WARNING! You are running this program as root!
It might be a good idea to run as a different user
Mirror launched on Sun, 04 Jan 2026 03:54:04 by HTTrack Website Copier/3.49-5 [XR&CO'2014]
mirroring http://target.ine.local with the wizard help..
* target.ine.local/wp-admin/load-scripts.php?c=0&load%5Bchunk_0%5D=jquery-core,jquery-migrate,zxcvbn-async,wp-polyfill-inert,regenerator-runtime,wp-polyfill,wp-hooks40/52: target.ine.local/wp-admin/load-scripts.php?c=0&load%5Bchunk_0%5D=jquery-core,jquery-migrate,zxcvbn-async,wp-polyfill-inert,regenerator-runtime,wp-polyfill,wp-Done.: target.ine.local/wp-login.php (4105 bytes) - OK
Thanks for using HTTrack!
```

## 📂 Búsqueda del archivo oculto

Una vez finalizada la descarga, se navega a la carpeta clonada y se busca un archivo con un nombre inusual que suele aparecer **solo tras el proceso de clonación**.

**Archivo objetivo:** `xmlrpc0db0.php`

```bash
cat /home/kali/Desktop/target_mirror/target.ine.local/xmlrpc0db0.php
```

## 🏁 Contenido del archivo (bandera incluida)

```xml
<?xml version="1.0" encoding="UTF-8"?><rsd version="1.0" xmlns="http://archipelago.phrasewise.com/rsd">
        <service>
                <engineName>WordPress</engineName>
                <engineLink>https://wordpress.org/</engineLink>
                <homePageLink>http://target.ine.local</homePageLink>
                  <apis>
                        <api name="WordPress" blogID="1" preferred="true" apiLink="http://target.ine.local/xmlrpc.php" />
                        <api name="Movable Type" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
                        <api name="MetaWeblog" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
                        <api name="Blogger" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
                        <api name="FLAG5{c9c52f584e364e719faaa4bb83d1aa33}" blogID="1" preferred="false" apiLink="http://target.ine.local/xmlrpc.php" />
                        <api name="WP-API" blogID="1" preferred="false" apiLink="http://target.ine.local/index.php/wp-json/" />
                  </apis>
        </service>
</rsd>
```

La bandera aparece dentro de un nodo `<api>`:  
`FLAG5{c9c52f584e364e719faaa4bb83d1aa33}`

## 🧠 Explicación

HTTrack clona el sitio web completo. En este caso, el proceso reveló un archivo `xmlrpc0db0.php` que no era visible mediante un escaneo de directorios tradicional. Al leerlo, se obtuvo la bandera embebida en la estructura del archivo.

Esta técnica es útil para descubrir archivos de configuración, backups y otros recursos que el servidor oculta a peticiones normales pero que quedan expuestos al generar una copia exacta del sitio.
