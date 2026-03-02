---
layout: post
title: "Experimento 0x03: Los Diarios de Blind Eagle (parte 1): Analizando SVGs Maliciosos"
date: 2026-02-28 00:00:45 -0500 
categories: experimento
tags: inteligencia de amenazas, blind eagle, svg, forence, apt 
author: ZoqueLabs
description: Primera entrega de un serie de articulos sobre Blind Eagle, un APT colombiano conocido por aprovechar infraestructura debil de entidades gubernamentales para engañar personas y hacerlas instalar malware. En esta ocación analizamos un nuevo vector de ataque de este grupo usando imagenes .SVG.
lang: es
---

<div align=center>
<h1>--[ Experimento 0x03: Los Diarios de Blind Eagle (parte 1): Analizando SVGs Maliciosos ]--</h1>
<h3>Febrero 2026</h3>
Por: ZoqueLabs y amigxs
<br>
<br>
Este escrito se distribuye con una licencia Creative Commons CC BY-SA (Reconocimiento - Compartir Igual).
<br> 
<br>
<a href="https://zoquelabs.xyz/experiment/2026/02/28/blind-eagle-diaries-1.html">English version</a>
</div>

## -[ 0x00 Intro ]-

Esta es la primera parte de una serie sobre Blind Eagle (en adelante, BE), un “APT” colombiano activo al menos desde 2016 y con campañas registradas en varios países de Latinoamérica.

La narrativa mediática —e incluso algunos análisis técnicos— suele presentar a BE como un grupo dedicado a atacar directamente entidades gubernamentales o financieras colombianas. La realidad es un poco más mundana (y más efectiva): BE suele suplantar entidades del Estado en campañas de correo malicioso dirigidas a personas del común. Demandas, multas, procesos judiciales, supuestos problemas tributarios: el anzuelo perfecto para lograr que alguien haga clic y termine ejecutando un RAT en su máquina Windows.

Una táctica frecuente consiste en aprovechar credenciales filtradas de entidades públicas para comprometer cuentas de correo reales y enviar desde allí los mensajes maliciosos. Muchas veces, los destinatarios son los propios contactos del correo vulnerado. No hay nada más convincente que un correo legítimo que ya estaba en tu libreta.

El “éxito” de BE no está necesariamente en malware sofisticado, sino en ingeniería social suficientemente bien hecha. Correos plausibles, pasos graduales, presión psicológica y, al final del camino, la descarga y ejecución del payload de turno. Con frecuencia utilizan herramientas de código abierto como AsyncRAT, DCRat o NjRAT. De eso hablaremos más adelante en esta serie.

En septiembre de 2025, VirusTotal publicó un [reporte](https://blog.virustotal.com/2025/09/uncovering-colombian-malware-campaign.html) sobre una táctica interesante: el uso de archivos .SVG (gráficos vectoriales) como vehículo de engaño en campañas dirigidas a usuarios en Colombia. Rápidamente, esta táctica fue atribuida a BE. El mecanismo es sencillo y elegante: el correo incluye un archivo .SVG que, al abrirse, simula ser una aplicación legítima de la entidad suplantada. La experiencia es más fluida que en campañas anteriores, reduce fricción y aumenta la probabilidad de ejecución. Ingeniería social refinada.

El write-up que sigue fue escrito por alguien cercano a ZoqueLabs (quien prefirió mantenerse en el anonimato). Picado por la curiosidad, tomó uno de estos archivos, lo desarmó pieza por pieza y documentó el análisis de forma clara y replicable. Hay sorpresas interesantes en el camino.

¡Vamos pues!

## -[ 0x01 Este escrito ]-

Este escrito es un análisis simple de las características técnicas de un ataque observado en casilleros de correo electrónico de Colombia, su enfoque es técnico y requiere un conocimiento básico de conceptos de programación web y de redes informáticas. Su objetivo es desmitificar el funcionamiento de estos ataques y acercarlos en la medida de lo posible a la mayor cantidad de profesionales y aficionados a la seguridad informática.

## -[ 0x02 El SVG ]-

El ataque inicial consiste en la entrega por correo de un archivo SVG, el cual “es un formato de gráficos vectoriales bidimensionales, tanto estáticos como animados, en formato de lenguaje de marcado extensible XML (Extensible Markup Language), es decir que se compone por código” (https://es.wikipedia.org/wiki/Gr%C3%A1ficos_vectoriales_escalables).


Un archivo básico de SVG se vería así si se abre con un editor de texto:

```html
<svg height="100" width="100" xmlns="http://www.w3.org/2000/svg">
  <circle r="45" cx="50" cy="50" fill="red" />
  Sorry, your browser does not support inline SVG.  
</svg> 
```
<br> 

Como se observa, un SVG es un formato en texto basado en XML, lo cual en la práctica lo hace compatible con cualquier visualizador de HTML como los navegadores comerciales y otras herramientas similares.

Para analizar el archivo malicioso vamos a usar la librería BeautifulSoup de Python la cual “permite analizar y extraer datos de documentos HTML y XML de forma sencilla”

El primer paso es cargar el archivo que contiene el SVG malicioso, el cual se ha renombrado como sample.svg, y cargarlo desde una shell de comandos de Python así:

```python
with open("sample.svg") as fp:
...     soup = BeautifulSoup(fp)
```
<br> 

Este código crea una estructura de datos que contiene el árbol de nodos que componen cualquier formato en XML.

Cada elemento de la estructura que crea soup tiene un array llamado contents que contiene los hijos del nodo, en este caso son 3:

```python
len(soup.contents)
3
```
<br> 

Al imprimir el nombre del primer nodo hijo observamos que es una tag svg:

```python
print(soup.contents[0].name)
“svg”
```
<br> 

Luego de asignar este elemento a su propia variable node_svg, listamos los nombres de sus hijos:

```python
svg = soup.contents[0]
for x in svg:
...     print(x.name)
```
<br> 

El anterior comando despliega un listado de todos los elementos que componen el primer nivel del árbol de nodos:

```bash
clippath
g
None
Script
None
text
None
line
None
…
```
<br> 

El listado completo incluye 71 elementos, de los cuales la mitad no tiene nombre (None), o sea que son texto o comentario por lo tanto no son importantes para nuestro análisis,  y la otra mitad tiene nombres de los elementos normales que contiene un svg como clippath, g, text, polygon entre otros, los cuales describen los elementos gráficos que componen el gráfico.

Pero lo más llamativo de todo es uno con nombre script. Esto es muy interesante ya que no es normal que un svg contenga código javascript, o al menos, es algo muy sospechoso en un archivo que ha sido categorizado como malware,  o que tiene un origen sospechoso.

Para continuar con el análisis vamos dividir el archivo en dos partes, la primera va a contener todos los elementos originales del SVG pero vamos a remover el script, lo cual nos permite manipular el archivo con más facilidad y por ejemplo usar una aplicación web para ver su presentación gráfica.

```python
from bs4 import BeautifulSoup

with open("sample.svg") as fp:
	soup = BeautifulSoup(fp, features="html.parser")

svg = soup.contents[0]

with open('cleaned.svg', 'w') as file:
    ##Como estamos agregando el tag svg manualmente debemos mapear los atributos que contiene en el archivo original
	file.write('<svg ')
	for (key, value) in svg.attrs.items():
		file.write(key + '=' + '"' + value + '"' )
	file.write('>')
    
	for tag_content in svg.contents:
        ## NO agregamos el tag con el script
		if tag_content.name =='script':
			continue
		else:
			file.write(str(tag_content))
	file.write('</svg>')

Adicionalmente listamos los atributos del tag principal svg:

for x in svg.attrs:
...     print(x)
```
<br> 

Y en el listado aparece un elemento muy interesante: onclick, este atributo permite a cualquier elemento activar un fragmento de código javascript al ser pulsado

La siguiente instrucción imprime el contenido del atributo:

```python
print(svg.attrs['onclick’])
_
openDocument()
```
<br> 

Esto  nos indica que el SVG es un artefacto gráfico que debe motivar a la víctima del ataque a dar clic y activar el código javascript adjunto.

Esto lo verificamos abriendo el svg depurado en un visor SVG online como https://www.svgviewer.dev/ y observando que efectivamente el SVG dibuja una interfaz que simula ser una sección de un sistema de la Fiscalía General de la Nación de Colombia, donde se indica que se puede descargar documentación relacionada con un proceso con un botón grande que dice DESCARGAR BOLETA:

![descargar_boleta](/assets/images/exp0x03/descargar_boleta.png)

Esto finaliza la primera parte del análisis concluyendo que se usa el archivo SVG como vector de infección no solamente por su capacidad de insertar código javascript en un navegador sino también como un mecanismo de ingeniería social para incentivar al usuario a interactuar con el artefacto de ataque y activar el código malicioso.

## -[ 0x03 El Script ]-

Para analizar el script lo primero que hacemos es insertarlo en un archivo de texto:

```python
with open('script.txt', 'w') as script_output:
	file2.write(svg.script)
```
<br> 

 Al abrirlo en un editor de texto lo primero que observamos es una serie de comentarios MUY particulares

```javascript
        // POLIMORFISMO_MASIVO_SEGURO: 2025-09-15T15:07:48.273546
        // HASH_PRINCIPAL: 6a8aa3db5195
        // UUID_PRINCIPAL: d9b9b8687a6a4e3cae8da30599bf8970
        // TIMESTAMP_MICRO: 1757966868273547
        // ENTROPY: 0.6700673669742895
        // SIGNATURE: 
```
<br> 

Lo primero llamativo es que los nombres de los campos estan en español, y son muy descriptivos, lo cual no es para nada común en un archivo de malware ya que por la naturaleza del mismo se busca opacar y ocultar al máximo la naturaleza de cada elemento, aquí al contrario se usan nombres explícitos y descriptivos, algo paradójico. Peor aún la primera cadena de texto “POLIMORFISMO_MASIVO_SEGURO” es única y particular, en más de dos décadas de interactuar con código fuente nunca la había visto, y su contenido es la paradoja mayor, el polimorfismo es una técnica en la que el código fuente se transforma en cada instancia del software, ¡lo cual está en absoluta oposición a usar un texto que no cambia de una variación a otra!

Este mecanismo al parecer indica que el programador por un lado tiene ciertos conocimientos de técnicas de programación de malware, pero por otro lado no las entiende en absoluto. O puede ser que esa sea la intención,  ¿ocultar el verdadero perfil del atacante y confundir acerca de sus capacidades reales? 🤔

Buscamos a continuación la función que se ejecuta al dar clic en el svg (openDocument)y se revela un fragmento de código muy interesante:

```javascript
        function openDocument() {
            // MÉTODO EXACTO del SVG que funciona - Genera URLs blob únicas
            try {
                var htmlCompleto = atob(ztymrglbczwsotpc);
                var blob = new Blob([htmlCompleto], { type: 'text/html' });
                var url = URL.createObjectURL(blob);
                window.open(url, '_blank');
                setTimeout(() => URL.revokeObjectURL(url), 5000);
            } catch(e) {
                console.error('Error:', e);
            }
        }
```
<br> 

Continuamos con las “excentricidades”, un código nos explica EXACTAMENTE que hace la función pero igual vamos analizar, lo que hace es tomar una cadena de texto y la pasa por la función atob que decodifica una cadena de base 64 en ASCII y la convierte a binario. 

```javascript
var htmlCompleto = atob(ztymrglbczwsotpc);
```
<br> 

Esta estructura binaria se pasa a la función Blob, la cual devuelve un objeto Blob que representa una colección de datos binarios que se pueden manipular como un archivo.

```javascript
var blob = new Blob([htmlCompleto], { type: 'text/html' });
```
<br> 

Luego se llama una función que codifica el objeto binario en una URL, la cual al accederse desde el navegador genera una descarga del archivo.

Finalmente se usa una función que abre el archivo URL y luego lo elimina de memoria.

```javascript
window.open(url, '_blank');
setTimeout(() => URL.revokeObjectURL(url), 5000);
```
<br> 

Antes de continuar analizando el archivo generado analizamos el resto del código compuesto por dos bloques extensos, uno compuesto por más de 70 variables con la siguiente estructura :

```javascript
// Variables dummy MASIVAS (40-80)

 var var_36db1df8742aa5e1592dea0e = '6208ea7dcde02003b407ae514e5381cbff35e529567319fad470ea6a615bb007cd6b49bc98f371bb9974f3b43297811b2fcfd690190256dd0be6ae687eaba669';
…..
```
<br> 

Un truco muy simple nos permite entender el uso de estas variables, copiamos el nombre y lo buscamos encontrando que no están referenciadas en ninguna parte del código, por lo que son simple relleno y no tienen ninguna funcionalidad dentro del flujo de ejecución.

El otro bloque está compuesto por 38 funciones con este formato:

```javascript
    function func_0d50fb55bba8e7af6301() {
            var x_0 = 'e13f916002c2944bb96667b1caabab676ca9d17d..';
            var y_0 = Math.random() * 10000;
            var z_0 = '6a8aa3db5195_d9b9b8687a6a4e3cae8da30599bf8970_0';
            var w_0 = Date.now() + Math.random();
            var hash_0 = 'c742569249011b432ea4807188dd5e1c3ab0e4e8dcb3175d3d35f8561923640f';
            var uuid_0 = 'a81908ab-4d30-411c-9c2e-dc5d4904c7d5';
            var entropy_0 = Math.random() * Math.PI;
            var crypto_0 = '9d74acfa3c9b0c6133d1aeecbe1ab4b….';
            var nonce_0 = '87cfe205e6880d577c24c9dbcf810e1d4784c356….';
            var timestamp_0 = new Date().getTime();
            return x_0 + y_0 + z_0 + w_0 + hash_0 + uuid_0 + entropy_0 + crypto_0 + nonce_0 + timestamp_0;
      }
```
<br> 

Aunque a simple vista parece una función compleja antes de analizarla hacemos el mismo procedimiento de buscar el nombre de la función dentro del archivo  y encontrar que esta función no se llama desde ninguna parte por lo que es también código de relleno sin ninguna otra funcionalidad que engañar análisis heurísticos rudimentarios..

Procedemos a analizar el contenido del archivo generado por la función `openDocument()`, lo podemos hacer usando cualquier consola JS, por ejemplo la de node, copiando la variable maliciosa, para luego pasarla por la función `atob()` y escribir el resultado en un nuevo archivo.

```javascript
const fsPromises = require('fs').promises
var ztymrglbczwsotpc = 'PCFET0NU.....';

fn = async () => { await fsPromises.writeFile('./blob_file.txt', atob(ztymrglbczwsotpc)); }
fn();
```
<br> 

## -[ 0x04 El otro Script ]-

Al abrir el nuevo archivo `blob_file.txt` en un editor de texto vemos  que es un archivo HTML con la estructura básica de un sitio web sencillo: header, estilo, cuerpo y scripts.

Usando un código python similar con el que se analizó el SVG creamos una copia nueva del archivo eliminando el script y luego previsualizarlo en un editor html online como https://html.onlineviewer.net/, observaremos que esta es otra interfaz visual similar a la anterior que emula un sistema de la Fiscalía General de la Nación que descarga un archivo cifrado y provee un password para desencriptar.

![preparando_descarga](/assets/images/exp0x03/preparando_descarga.png)

Ahora analizamos el script que se encarga de continuar con el ataque,y a simple vista observamos la función window.onload, la cual es un elemento clásico en el ecosistema javascript ya se dispara cada vez que el archivo se carga en un navegador:

```javascript
window.onload = function() {
animarProgreso();
	setTimeout(descargarArchivoBinario, 2500);
}
```
<br> 

El código ejecuta otras dos funciones, la primera es una función que manipula elementos gráficos para simular una barra de descarga (tal como su nombre nos lo explica)  y la segunda crea un cronómetro que pasados 2.5 segundos ejecuta otra función: `descargarArchivoBinario()`, muy similar a la del anterior script:

```javascript
function descargarArchivoBinario() {
            const base64Data = 'UEsDBBQAA....';
            const binaryString = atob(base64Data);
            const bytes = new Uint8Array(binaryString.length);
            for (let i = 0; i < binaryString.length; i++) {
                bytes[i] = binaryString.charCodeAt(i);
            }
            const blob = new Blob([bytes], { type: 'application/octet-stream' });
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.charCodeAthref = url;
            a.download = 'NOTIFICACION_OFICIAL_DEMANDA_POR_DAÑOS_Y_PERJUICIOS_GRAVES_JUZGADO.zip';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            window.URL.revokeObjectURL(url);
 }
```
<br> 

Esta función toma una cadena de texto en base64, la decodifica y la convierte en una estructura de Javascript Uint8Array que usa para crear un blob que finalmente se convierte en un archivo comprimido en formato zip.

En este punto del ataque el usuario ya ha iniciado el flujo tal como el atacante lo incentivó seguramente engañado por la apariencia oficial y sofisticada del mensaje, por lo que la siguiente acción que consiste en la aparición de un cuadro de diálogo que le pregunta al usuario si desee abrir el archivo comprimido será probablemente también realizada. 

Este archivo zip está encriptado con la contraseña que se provee al usuario, lo cual impide el análisis de los motores de antivirus en cualquier momento previo a la desencriptación y contiene un archivo ejecutable de windows con varios DLLs y un par de archivos con extensiones no comunes. 

## -[ 0x05 Conclusión ]-

En este momento del proceso es claro que el objetivo del archivo analizado es desplegar un ataque de ingeniería social usando para su funcionalidad algunos trucos no comunes como incluir el sitio de phishing en un archivo SVG,  otros que se vienen usando desde hace más de 20 años como enviar un ejecutable dentro de un comprimido con contraseña. Es también notable el uso de ciertos mecanismos estándar en el malware como polimorfismo implementados de una manera ingenua, y casi infantil, por ejemplo describiendo el código con comentarios como:

```javascript
// Funciones dummy MASIVAS
// Variables dummy MASIVAS (40-80)
```
<br> 

Y otras cadenas que anulan cualquier logro en el polimorfismo ya que son constantes entre cada sample analizado, por lo que se podría detectar el ataque con cualquier análisis estático que busque estos poco comunes textos. Increiblemente o aunque no tanto, la mayoria de engines de antivirus son engañados por el payload inicial, o al menos asi lo indican los analisis en virustotal de uno de los samples (https://www.virustotal.com/gui/file/7ecc8a25f51d4de1097f05eb68619d4b21e7de8dad077422ede4015941645be3)

En la siguiente parte del análisis usaremos las técnicas explicadas anteriormente, para analizar varios archivos automáticamente, generar vectores que describen las features de cada uno y usan un algoritmo de clusterización para identificar si todos los samples se relacionan entre sí o pertenecen a otra campaña/familia de malware.

---

FIN de la primera parte.
