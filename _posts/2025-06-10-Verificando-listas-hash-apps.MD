---
layout: post
title:  "Verificando apps de Android en VirusTotal con sus hashes"
categories: [android, forense]
tags: [android, forense, virustotal, vt, hash, colecciones, collections, malware]
author: ZoqueLabs
description: Mini-tutorial para investigar listas de hashes de apps en Virus Total
lang: es
---

<div align=center>
<h1>--[ Verificando <em>muchas</em> apps de Android en VirusTotal con sus hashes ]--</h1>
Por: ZoqueLabs
<br>
<br>
Este escrito se distribuye con una licencia Creative Commons CC BY-SA (Reconocimiento - Compartir Igual)
<br>
<a href="/android/forensics/2025/06/10/Verifying-lists-of-hash-apps.html">English version</a>
</div>


## Intro

Este mini tutorial está pensado para ayudarte a revisar si un listado de aplicaciones instaladas en un dispositivo Android ha sido reportado como malicioso, usando su hash (una especie de huella digital única del archivo).

Herramientas como **Colander** o **[MVT](https://docs.mvt.re/en/latest/)** (Mobile Verification Toolkit) permiten hacer esta verificación, pero lo hacen de una en una. Si solo tienes unos pocos hashes, puede funcionar. Pero si estás trabajando con muchos —como cuando se hace una extracción forense completa con `androidqf`— este proceso se vuelve muy lento y rápidamente consume el límite de la API de VirusTotal.

En este tutorial mostramos cómo hacer esta verificación en lote (bulk), es decir, revisar varios hashes al mismo tiempo, ahorrando tiempo y energía.

Partimos de una extracción forense del dispositivo Android generada con `androidqf`, que incluye un archivo llamado `packages.json` donde se encuentran los hashes SHA256 de las aplicaciones instaladas. También se puede usar `hashes.csv` si está disponible, pero aquí nos enfocamos principalmente en `packages.json` porque es lo que suele estar incluso en extracciones más básicas.

La idea es ayudarte a identificar comportamientos sospechosos o directamente maliciosos en las apps del dispositivo, sin necesidad de revisar una por una.

---

## Preparar el entorno con androidqf

Para comenzar, necesitaremos realizar una extracción forense del dispositivo Android usando la herramienta [androidqf](https://github.com/mvt-project/androidqf). Obtendrás una carpeta con varios archivos, entre ellos el importante `packages.json`.

```bash
$ ls
acquisition.json  dumpsys.txt  logcat.txt          selinux.txt          tmp
apks              env.txt      logs                services.txt
backup.ab         files.json   packages.json       settings_global.txt
bugreport.zip     getprop.txt  processes.txt       settings_secure.txt
command.log       hashes.csv   root_binaries.json  settings_system.txt
```
&nbsp;

## Extrayendo hashes de las aplicaciones de `packages.json`

El  `packages.json` contiene información detallada sobre todas las aplicaciones instaladas en el dispositivo, incluyendo los hashes criptográficos de cada apk y sus subcomponentes.

```bash
 {
        "name": "com.android.pacprocessor",
        "files": [
            {
                "path": "/system/app/PacProcessor/PacProcessor.apk",
                "local_name": "",
                "md5": "95ed855e694de1ad40e4d3500a24952f",
                "sha1": "e40cd26f03becf29a4f887978ea57f21c7e30cc4",
                "sha256": "946cb5797f9a26a93709a7f01175b890bc010ba80182f8847ede24a39c9b9660",
                "sha512": "77c2fb531da7b2751e3abb9ef25c3d715e75d3978301ee00b66edb3c171205fa97d7daa3fb21c2725aece9d7392b7a7ad90da58f1ea64520398ccebf0c3a6d67",
```
&nbsp;
Puedes extraer la lista completa de hashes SHA256 de las aplicaciones desde `packages.json`, usando la terminal de Linux como te mostramos más adelante (el proceso puede ser similar en MacOS o Windows).

Sin embargo, en este caso solo nos interesan las aplicaciones que podrían ser sospechosas —es decir, las que no son nativas del sistema o que fueron instaladas desde fuentes desconocidas—, para ello usaremos filtros con `jq` para excluir apps preinstaladas o que parecen "confiables", y concentrarnos en las que realmente valen la pena revisar.

Ahora si, vamos...

Primero, navega hasta el directorio donde está ubicada la extracción:
```bash
cd path/to/your/backup/
```
&nbsp;
Si quieres extraer todos los hashes puedes ejecutar este comando:

```
jq -r '.[].files.[].sha256' packages.json
```
&nbsp;
Pero como nuestro objetivo es concentrarnos en aplicaciones "sospechosas" vamos a usar filtros más especificos con este comando: 
```bash
jq -r '.[] | select (((.name | startswith("com.google.android.") or startswith("com.android.") or startswith("com.samsung") or startswith("com.sec.")) and (.system == true)) | not)' packages.json | jq -r '.files.[].sha256'
```
&nbsp;
**¿Qué hace este comando y cuál es nuestro plan?**

Este comando usa `jq` (una herramienta de línea de comandos para procesar archivos JSON) para procesar el archivo `packages.json`. y extraer únicamente los hashes SHA256 de las aplicaciones que no parecen ser del sistema o de fabricantes conocidos como Google o la empresa fabricante -nuestro caso Samsung, pero este lo puedes reemplazar-.

Entonces, vamos por partes:

En primer lugar, `select(...)` aplica un filtro para excluir:

1. Apps cuyo nombre empieza por `com.google.android.`, `com.android.`, `com.samsung` o `com.sec`.
2. Y además que están marcadas como del sistema `(.system == true)`
3. `| not` invierte la condición para quedarnos solo con las que no cumplen ese patrón, es decir, apps instaladas desde fuentes desconocidas.

Luego, `.files[].sha256` extrae el hash de cada archivo asociado a esas apps filtradas.

Así, al ejecutar este comando obtendrás una lista filtrada de aplicaciones más interesantes para analizar con VirusTotal.

```bash
$ jq -r '.[] | select (((.name | startswith("com.google.android.") or startswith("com.android.") or startswith("com.samsung") or startswith("com.sec.")) and (.system == true)) | not)' packages.json | jq -r '.files.[].sha256

5f4144359f8fdf52e6d4471a11438aa2c209e4af2d8bc90a4111975d1227c0aa
9948eaa76138f8a45943cdd81838e4a053a2734ef8a326e108dc3b0b5c4f409d
5014e20fb03bca12d456f278faf2b1f2f43326930c062e0705fb2cebedfe23b2
... mas hashes
... mas hashes
```
&nbsp;
Si quieres guardar la lista de hashes en un archivo para luego subirlo a VirusTotal, puedes hacerlo así:
```bash
$ jq -r '.[] | select (((.name | startswith("com.google.android.") or startswith("com.android.") or startswith("com.samsung") or startswith("com.sec.")) and (.system == true)) | not)' packages.json | jq -r '.files.[].sha256' > hashesobtenidos.txt
```
&nbsp;
Con este archivo tendrás todos los hashes en bulk.

---

## Crear una colección en VirusTotal con los hashes extraídos

Una vez que tenemos la lista de hashes de las aplicaciones sospechosas, podemos usar la funcionalidad de _colecciones_ de VirusTotal para analizarlos en conjunto.

En este tutorial usamos la versión gratuita de VirusTotal, porque la idea es facilitar recursos accesibles sin depender de licencias pagas.

Eso sí: en la versión gratuita, cuando subes muchos hashes a una colección, el filtrado de resultados no es tan amigable. No puedes, por ejemplo, ordenar fácilmente por número de detecciones o aplicar filtros avanzados. Por eso es importante que antes de subir, ya tengas muy bien filtradas las apps que podrían ser realmente sospechosas, como lo hicimos en el paso anterior. 

Si tienes acceso a una cuenta premium de VirusTotal, ese análisis en _bulk_ será mucho más fluido, pero en este caso trabajamos con lo que tenemos disponible - la versión gratis-. 


### Qué son las colecciones de VirusTotal y por qué usarlas?

La sección [_colecciones_](https://blog.virustotal.com/2021/11/introducing-virustotal-collections.html) en VirusTotal es muy útil porque nos permite reunir en un solo lugar varios indicadores relacionados con un mismo análisis o caso, como hashes, URLs, dominios o direcciones IP. Podemos ir actualizando estas colecciones con nueva información, compartirlas fácilmente con otras personas a través de un enlace, y revisar en conjunto los metadatos que ofrece VirusTotal, como los motores antivirus que detectan amenazas o las etiquetas que describen su comportamiento. Esto hace que el análisis sea más claro, ordenado y colaborativo, especialmente cuando trabajamos en equipo o necesitamos documentar hallazgos para informes.

**Crear una colección paso a paso**

1. Ingresa a Visrus total, crea una cuenta o logueate si ya tienes una.
2. Puedes dar clic en el icono de busqueda en VT o dar clic [acá](https://www.virustotal.com/gui/collections).

&nbsp;
![Image VT](/assets/vt1.png)
&nbsp;

3. Da clic en **Create new collection** (crear nueva colección).

&nbsp;
![Image VT](/assets/vt2.png)
&nbsp;

4. Asigna un nombre a tu colección pj: `hashapps`.
5. Pega los hashes que extrajiste desde `packages.json`, uno por línea.
6. Guarda la colección.

## ¿Qué datos arroja el análisis de VirusTotal?

VirusTotal se encarga de analizar los hashes automáticamente y te muestra varios datos importantes para revisar las aplicaciones.

&nbsp;
![Image VT](/assets/vt3.png)
&nbsp;

**Detecciones por motores antivirus**: te dice si algún motor ha marcado el archivo como malicioso, y qué nombre le da (por ejemplo, `Android/TrojanSpy.Agent`).

**Etiquetas descriptivas**: estas etiquetas no solo indican si es un virus, sino que también señalan otros comportamientos o características, como:
  - `has-trackers`: si la app incluye rastreadores conocidos (como Google, Facebook, etc.).
  - `uses-permission`: si utiliza permisos sensibles como acceso a SMS, cámara, ubicación o almacenamiento.
  - `obfuscator`: si el código está ofuscado, lo que puede indicar que intenta esconder algo.
  - Otras etiquetas como `spyware`, `dropper`, `apk-downloader`, entre otras que ayudan a entender mejor el comportamiento de la app.


**Otra información del archivo**:
  - Tamaño y tipo de archivo.
  - Fecha del análisis.
  - Nombre del archivo (si se conserva).
  - Información del certificado o firma digital, si está disponible.
  - Detalles técnicos del `AndroidManifest.xml`, como los permisos y servicios que declara.

**Relaciones con otros elementos**: VirusTotal también puede mostrar si ese hash está vinculado a otros archivos, URLs, dominios o campañas conocidas.

Todo esto permite revisar de forma más completa si una aplicación representa un riesgo, incluso cuando no está clasificada directamente como malware.


### Hashes listos

Y listo. Con estos pasos ya tienes una forma rápida y sencilla de verificar aplicaciones Android. Esperamos que le saques provecho al mini-tutorial. 


---




