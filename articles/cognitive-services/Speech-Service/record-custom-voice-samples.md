---
title: 'Grabación de ejemplos de voz personalizada: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Para crear una voz personalizada con calidad de producción, prepare un guion sólido, contrate un buen actor de voz y realice la grabación de manera profesional.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/13/2020
ms.author: erhopf
ms.openlocfilehash: 0c58369392940dbb34e222693cb026f4fd84c8bd
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110458989"
---
# <a name="record-voice-samples-to-create-a-custom-voice"></a>Grabación de muestras de voz para crear una voz personalizada

Crear una voz neuronal personalizada con alta calidad de producción desde cero no es una tarea ocasional. El componente central de una voz neuronal personalizada es una gran colección de muestras de audio de voz humana. Es fundamental que estas grabaciones de audio sean de alta calidad. Elija un actor de voz que tenga experiencia en esta clase de grabaciones y un ingeniero de grabación que las grabe con un equipo profesional.

No obstante, para poder realizar estas grabaciones, necesita un guion: las palabras que dirá el actor de voz para crear las muestras de audio. Para obtener mejores resultados, el guion debe tener una buena cobertura fonética y variedad suficiente para entrenar el modelo de voz neuronal personalizada.

Muchos detalles pequeños pero importantes intervienen en la creación de una grabación de voz profesional. Esta guía es una guía básica de un proceso que le ayudará a obtener resultados buenos y uniformes.

> [!NOTE]
> Para entrenar una voz neuronal, debe especificar un perfil de actor de voz con el archivo de consentimiento de audio del actor de voz en el que acepta que se usen sus datos de voz para entrenar un modelo de voz neuronal personalizada. Cuando prepare el script de grabación, asegúrese de incluir la frase siguiente. 

> "I [state your first and last name] am aware that recordings of my voice will be used by [state the name of the company] to create and use a synthetic version of my voice." ("Yo [indique su nombre y apellido] acepto que [indique el nombre de empresa] use grabaciones de mi voz para crear y usar una versión sintética de la voz").
Esta frase se usará para comprobar si los datos de entrenamiento se corresponden con la persona que otorga el consentimiento. Para más información sobre la comprobación del talento de voz, consulte [aquí](/legal/cognitive-services/speech-service/custom-neural-voice/data-privacy-security-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext).

> Voz neuronal personalizada está disponible con acceso limitado. Asegúrese de comprender los [requisitos de IA responsable](/legal/cognitive-services/speech-service/custom-neural-voice/limited-access-custom-neural-voice?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) y [aplicar el acceso aquí](https://aka.ms/customneural). 

> [!TIP]
> Para conseguir resultados de máxima calidad, considere la posibilidad de acudir a Microsoft para ayudar a desarrollar su voz neuronal personalizada. Microsoft posee una gran experiencia en producir voces de alta calidad para sus propios productos, como Cortana y Office.

## <a name="voice-recording-roles"></a>Roles de grabación de voz

En un proyecto de grabación de voz neuronal personalizada hay cuatro roles básicos:

Role|Propósito
-|-
Actor de voz        |La voz de esta persona constituirá la base de la voz neuronal personalizada.
Ingeniero de grabación  |Supervisa los aspectos técnicos de la grabación y usa el equipo de grabación.
Director            |Prepara el guion y prepara la sesión del actor de voz.
Editor              |Finaliza los archivos de audio y los prepara para su carga en Speech Studio.

Un usuario individual puede desempeñar más de un rol. En esta guía se da por hecho que desempeñará principalmente el papel de director y que contratará un actor de voz y un ingeniero de grabación. Si quiere realizar las grabaciones usted mismo, en este artículo se incluye alguna información sobre el rol de ingeniero de grabación. El rol de editor no es necesario hasta después de la sesión, por lo que puede realizarlo el director o el ingeniero de grabación.

## <a name="choose-your-voice-talent"></a>Elección del actor de voz

Los actores con experiencia en voz en off o en poner voz a personajes serán unos buenos actores de voz neuronal personalizada. También pueden desempeñar bien este rol presentadores y locutores. Elija un actor cuya voz natural le guste. Aunque se pueden crear voces de "personaje" únicas, para los actores de voz es mucho más difícil conseguir locuciones uniformes y el esfuerzo puede dar lugar a que fuercen la voz. El único factor más importante para elegir el actor de voz es la uniformidad. Las grabaciones deben sonar como que si se realizaran el mismo día en la misma sala. Puede aproximarse a este ideal con unas buenas prácticas y una buena ingeniería de grabación.

Su actor de voz es la otra mitad de la ecuación. Debe ser capaz de hablar a una velocidad, con un nivel de volumen y un tono uniformes. Es obligatorio una dicción clara. El actor también debe ser capaz de controlar estrictamente la variación de su tono, el efecto emocional y los manierismos del habla. La grabación de muestras de voz puede causar más fatiga que otros trabajos de voz. La mayoría de los actores de voz pueden grabar durante dos o tres horas al día. Limite las sesiones a tres o cuatro a la semana, con un día de descanso entre medias si es posible.

Colabore con el actor de voz para desarrollar una "persona" que defina el sonido y el tono emocional general de la voz neuronal personalizada. En el proceso, identificará lo que suena "neutro" para esa persona. Mediante la funcionalidad de Voz neuronal personalizada, puede entrenar un modelo que hable con emociones. Defina "estilos de habla" y pida al talento de voz que lea el script de tal manera que reproduzca los estilos que desea.  

Es posible que, por ejemplo, una persona tenga una personalidad animada por naturaleza. Por tanto, "su" voz puede transmitir una nota de optimismo incluso aunque hable de forma neutra. Sin embargo, tal rasgo de personalidad debe ser sutil y uniforme. Escuche las lecturas de voces existentes para hacerse una idea de lo que busca.

> [!TIP]
> Por lo general, querrá quedarse con las grabaciones de voz que realice. Así que el actor de voz debe aceptar un contrato de trabajo a comisión para el proyecto.

## <a name="create-a-script"></a>Creación de un script

El punto de partida de cualquier sesión de grabación de voz neuronal personalizada es el guion, que contiene las expresiones que dirá el actor de voz. (El término "enunciados" abarca tanto oraciones completas como frases más cortas).

Los enunciados del guion pueden proceder de cualquier parte: ficción, no ficción, transcripciones de voces, informes de noticias y cualquier cosa disponible en formato impreso. Si quiere asegurarse de que su voz funcione bien con clases de palabras específicas (como terminología médica o jerga de programación), podría incluir oraciones de trabajos académicos o documentos técnicos. Para obtener una breve descripción de los posibles problemas legales, vea la sección ["Aspectos legales"](#legalities). También puede escribir su propio texto.

Las expresiones no tienen que proceder de la misma fuente, o de la misma clase de fuente. Incluso no es necesario que tengan nada que ver unos con otros. Sin embargo, si va a usar frases hechas (por ejemplo, "Ha iniciado sesión correctamente") en la aplicación de voz, asegúrese de incluirlas en el guion. De esta forma, la voz neuronal personalizada tendrá una mejor oportunidad de pronunciar bien esas frases. Y si decide usar una grabación en lugar de voz sintetizada, ya la tendrá en la misma voz.

Si bien la uniformidad es importante a la hora de elegir un actor de voz, la variedad es lo que distingue un buen guion. El guion debe incluir muchas palabras y frases diferentes con diversas longitudes de frase, estructuras y estados de ánimo. Cada sonido del idioma se debe representar varias veces y en varios contextos (lo que se conoce como *cobertura fonética*).

Además, el texto debe incorporar todas las formas en que puede representarse un sonido determinado al escribir, y colocar cada sonido en distintos lugares en las oraciones. Se deben incluir tanto oraciones afirmativas como preguntas y se deben leer con la entonación adecuada.

Es difícil escribir un guión que proporcione datos *suficientes* para permitir que Speech Studio genere una buena voz. En la práctica, la manera más sencilla de crear un guion que logre una sólida cobertura fonética es incluir un gran número de muestras. Las voces estándar que proporciona Microsoft se han creado a partir de decenas de miles de expresiones. Deberá estar preparado para grabar unos cuantos miles de expresiones como mínimo para generar una voz neuronal personalizada con calidad de producción.

Compruebe el guion con mucho cuidado para detectar posibles errores. Si es posible, que otra persona también lo revise. Cuando repase el guion con el actor de voz, es probable que detecte algunos errores más.

### <a name="script-format"></a>Formato del guion

Puede escribir el guion en Microsoft Word. El guion se usará durante la sesión de grabación, así que puede prepararlo de forma que sea fácil trabajar con él. Cree el archivo de texto requerido por Speech Studio por separado.

Un formato de guion básico contiene tres columnas:

* El número del enunciado, empezando por el 1. La numeración permite que todas las personas del estudio puedan consultar una expresión concreta ("probemos de nuevo el número 356"). Puede usar la característica de numeración de párrafos de Word para numerar las filas de la tabla de forma automática.
* Una columna en blanco donde escribirá el número de toma o el código de tiempo de cada expresión para ayudarle a encontrarla en la grabación finalizada.
* El propio texto del enunciado.

![Script de ejemplo](media/custom-voice/script.png)

> [!NOTE]
> La mayoría de los estudios graban en segmentos cortos conocidos como *tomas*. Cada toma contiene normalmente de 10 a 24 expresiones. Basta con anotar el número de toma para después poder encontrar una expresión. Si graba en un estudio que prefiere realizar grabaciones más largas, querrá anotar entonces el código de tiempo. El estudio tendrá una gran pantalla de tiempo.

Deje suficiente espacio después de cada fila para escribir notas. Asegúrese de que ningún enunciado se divida entre páginas. Numere las páginas e imprima el guion en una de las caras del papel.

Imprima tres copias del guion: una para el actor de voz, otra para el ingeniero y la última para el director (usted). Use un clip de papel en lugar de grapas: un actor de voz experimentado separará las páginas para evitar hacer ruido cuando se hojean.

### <a name="legalities"></a>Aspectos legales

Según la ley de derechos de autor, la lectura de texto protegido por derechos de autor por parte de un actor podría ser una representación por la que el autor de la obra debería ser compensado. Esta representación no será reconocible en el producto final, la voz neuronal personalizada. Aun así, la legalidad del uso de una obra protegida por derechos de autor con esta finalidad no está bien establecida. Microsoft no puede proporcionar asesoramiento legal sobre este problema; consulte a su asesor.

Afortunadamente, es posible evitar estos problemas por completo. Hay muchas fuentes de texto que puede usar sin licencia o permiso.

|Fuente de texto|Descripción|
|-|-|
|[Corpus CMU Arctic](http://festvox.org/cmu_arctic/)|Aproximadamente 1100 oraciones seleccionadas de obras protegidas por derechos de autor para su uso especialmente en proyectos de síntesis de voz. Un excelente punto de partida.|
|Obras que ya no<br>están protegidas por derechos de autor|Normalmente las obras publicadas antes de 1923. En inglés, el [proyecto Gutenberg](https://www.gutenberg.org/) ofrece miles de obras de este tipo. Quizás quiera centrarse en obras más modernas, dado que el lenguaje estará más próximo al inglés moderno.|
|Obras del gobierno &nbsp;|Las obras creadas por el gobierno de los Estados Unidos no están protegidas por derechos de autor en los Estados Unidos, aunque es posible que el gobierno reclame derechos de autor en otros países o regiones.|
|Dominio público|Obras en las que se ha renunciado explícitamente a los derechos de autor o que se han destinado al dominio público. Puede que en algunas jurisdicciones no sea posible renunciar totalmente a los derechos de autor.|
|Obras con licencia permisiva|Obras distribuidas con una licencia como Creative Commons o la licencia de documentación libre de GNU (GFDL). La Wikipedia usa la GFDL. Algunas licencias, sin embargo, pueden imponer restricciones sobre la representación del contenido con licencia que pueden afectar a la creación de un modelo de voz neuronal personalizada, así que lea la licencia detenidamente.|

## <a name="recording-your-script"></a>Grabación del guion

Grabe el guion en un estudio de grabación profesional especializado en trabajos de voz. Dispondrá de una cabina de grabación, el equipo adecuado y las personas adecuadas correctas utilizarlo. Vale la pena no escatimar en la grabación.

Revise el proyecto con el ingeniero de grabación del estudio y escuche sus consejos. La grabación debe tener poca o ninguna compresión de rango dinámico (máximo 4:1). Es esencial que el audio tenga un volumen uniforme y una elevada relación señal-ruido, y que esté libre de sonidos no deseados.

### <a name="do-it-yourself"></a>Hágalo usted mismo

Si quiere hacer la grabación por su cuenta y riesgo, en lugar de meterse en un estudio de grabación, este es un manual básico breve. Gracias al aumento de la grabación doméstica y la distribución de archivos multimedia, resulta más fácil que nunca encontrar buena información y recursos de grabación en Internet.

La "cabina de grabación" debe ser una habitación pequeña sin eco apreciable o "tono de la sala". Debe ser lo más silenciosa e insonorizada posible. Se pueden usar cortinas en las paredes para reducir el eco y neutralizar o "amortiguar" el sonido de la habitación.

Use un micrófono de condensador de estudio de alta calidad diseñado para la grabación de voz. Sennheiser, AKG e incluso los recientes micrófonos Zoom pueden producir buenos resultados. Puede comprar un micrófono o alquilar uno en una empresa de alquiler de equipo audiovisual. Busque uno con una interfaz USB. Este tipo de micrófono combina de forma práctica el elemento de micrófono, el preamplificador y el convertidor analógico-digital en un paquete, lo que simplifica la conexión.

También puede usar un micrófono analógico. Muchas de las empresas de alquiler ofrecen micrófonos "vintage" reconocidos por sus personajes de voz. Tenga en cuenta que el equipo analógico profesional usa conectores XLR balanceados, en lugar del conector de 1/4" que se usa en los equipos de consumo. Si se decide por el analógico, también necesitará un preamplificador y una interfaz de audio de equipo informático con estos conectores.

Coloque el micrófono en un pie o soporte tipo jirafa e instale un filtro contra chasquidos para eliminar el ruido de las consonantes oclusivas, como la "p" y la "b". Algunos micrófonos incorporan un montaje de suspensión que los aísla de las vibraciones del pie, lo que es útil.

El actor de voz debe permanecer a una distancia constante del micrófono. Use cinta en el suelo para marcar dónde se debe colocar. Si el actor prefiere sentarse, tenga especial cuidado para controlar la distancia al micrófono y evitar el ruido de la silla.

Use un atril para sostener el guion. Evite inclinar el atril para que pueda reflejar el sonido hacia el micrófono.

La persona que use el equipo de grabación, el ingeniero, debe estar en una habitación distinta de la del actor, con algún medio para comunicarse con él en la cabina de grabación (un *circuito de radio).*

La grabación debe contener el menor ruido posible; el objetivo es una relación señal-ruido de 80 db o superior.

Escuche con atención la grabación de silencio en la cabina, averigüe de dónde proceden los ruidos y elimine la causa. Las fuentes habituales de ruido son los conductos de ventilación, los balastros de tubos fluorescentes, el tráfico de carreteras cercanas y los ventiladores de los equipos (incluso los portátiles pueden tener ventiladores). Los micrófonos y cables pueden captar el ruido eléctrico de los cables cercanos de CA, normalmente un zumbido o murmullo. Un zumbido también se puede producir por un *bucle de tierra*, que se genera al conectar un equipo a más de un circuito eléctrico.

> [!TIP]
> En algunos casos, es posible usar un ecualizador o un complemento de software de reducción de ruido para ayudar a eliminar el ruido de las grabaciones, aunque siempre es mejor detenerlo en el origen.

Establezca los niveles de manera que la mayoría del intervalo dinámico de grabación digital disponible se use sin sobremodulación. Esto significa establecer un nivel de audio alto, pero sin que distorsione. En la imagen siguiente se muestra un ejemplo de la forma de onda de una grabación correcta:

![Forma de onda de una grabación correcta](media/custom-voice/good-recording.png)

Aquí, se usa la mayor parte del rango (alto), pero los picos más altos de la señal no llegan a la parte superior o inferior de la ventana. También puede ver que el silencio en la grabación se aproxima a una línea horizontal fina, que indica un ruido de fondo bajo. Esta grabación tiene un rango dinámico y una relación señal-ruido aceptables.

Grabe directamente en el equipo informático mediante una interfaz de audio de alta calidad o un puerto USB, según el micrófono que vaya a usar. Si es analógico, use lo básico: micrófono, preamplificador, interfaz de audio, equipo informático. Tanto [Avid Pro Tools](https://www.avid.com/en/pro-tools) como [Adobe Audition](https://www.adobe.com/products/audition.html) se pueden usar con licencia mensual por un módico precio. Si su presupuesto es muy reducido, pruebe la aplicación gratuita [Audacity](https://www.audacityteam.org/).

Grabe con sonido monofónico de 16 bits a 44,1 kHz (calidad de CD) o mejor. El más moderno actualmente es 24 bits a 48 kHz, si el equipo lo admite. Debe reducir la calidad del audio hasta 16 bits a 24 kHz antes de enviarlo a Speech Studio. Aun así, vale la pena tener una grabación original de alta calidad en caso de que sea necesario realizar modificaciones.

Lo ideal es tener distintas personas en los roles de director, ingeniero y actor. No intente hacerlo todo usted mismo. En caso de emergencia, la misma persona puede ser el director y el ingeniero.

### <a name="before-the-session"></a>Antes de la sesión

Para evitar perder tiempo en el estudio, repase el guion con el actor antes de la sesión de grabación. Mientras el actor se familiariza con el texto, puede resolver dudas con la pronunciación de palabras que no resulten muy conocidas.

> [!NOTE]
> La mayoría de los estudios de grabación ofrecen una pantalla electrónica de guiones en la cabina de grabación. En este caso, escriba las notas de revisión directamente en el documento del guion. También puede querer una copia en papel para tomar notas durante la sesión. La mayoría de los ingenieros también querrán una copia en papel. Y la tercera copia impresa servirá como respaldo para el actor en caso de que el equipo informático no funcione.

Es posible que el actor de voz pregunte qué palabra quiere enfatizar en una expresión (la "palabra operativa"). Indíquele que desea una lectura natural sin ningún énfasis en particular. El énfasis se puede agregar cuando se sintetice la voz; no debe formar parte de la grabación original.

Indique el actor que pronuncie las palabras claramente. Cada palabra del guion se debe pronunciar tal y como se escribe. Los sonidos no se deben omitir ni arrastrar juntos, como es habitual en el habla informal, *a menos que se hayan escrito de esa manera en el guion*.

|Texto escrito|Pronunciación informal no deseada|
|-|-|
|me has dejado helado|mehas dejao helao|
|le dijeron que estaba muy guapa|le dijeron queestaba mu guapa|
|la ciudad de Madrid|la ciuda de Madri|
|estoy cansado|estoy cansao|

El actor *no* debe agregar pausas claras entre las palabras. La frase debe fluir naturalmente, aunque suene un poco formal. Es posible que se necesite práctica para que esta distinción fina salga bien.

### <a name="the-recording-session"></a>La sesión de grabación

Al comienzo de la sesión, cree una grabación de referencia, o *archivo de ajuste,* de un enunciado típico. Pida al actor que repita esta línea cada página o página y media. Cada vez, compare la nueva grabación con la referencia. Esta práctica ayuda al actor a mantenerse en un volumen, tono y entonación constantes. Mientras tanto, el ingeniero puede usar el archivo de ajuste como referencia para los niveles y la uniformidad general del sonido.

El archivo de ajuste es especialmente importante cuando se reanuda la grabación tras un descanso, u otro día. Lo reproducirá varias veces para el actor y le hará repetirlo cada vez hasta que la coincidencia sea perfecta.

Entrene a su actor para respirar profundamente y hacer una pausa durante un momento antes de cada enunciado. Grabe un par de segundos de silencio entre enunciados. Las palabras se deben pronunciar de la misma manera cada vez que aparezcan, en función del contexto. Es importante mantener la coherencia en la pronunciación y en la entonación de las palabras.

Grabe un silencio de aproximadamente cinco segundos antes de la primera grabación para capturar el "tono de la sala". Esta práctica ayuda a Speech Studio a compensar cualquier rastro de ruido que pueda quedar en las grabaciones.

> [!TIP]
> Lo que realmente necesita es captar al actor de voz, por lo que puede hacer una grabación monofónica (un solo canal) de esas líneas concretas. Sin embargo, si graba en estéreo, puede usar el segundo canal para grabar la charla en la sala de control y capturar los comentarios de líneas o cortes determinados. Quite esta pista de la versión que se carga en Speech Studio.

Use auriculares para escuchar con atención la representación del actor de voz. Lo que busca es una dicción buena pero natural, una pronunciación correcta y ausencia de sonidos no deseados. No dude en pedir al actor que vuelva a grabar un enunciado que no satisfaga estas normas.

> [!TIP]
> Cuando se usa un número elevado de expresiones, una sola de ellas podría no tener un efecto apreciable sobre la voz neuronal personalizada resultante. Es posible que sea más adecuado anotar simplemente las expresiones con problemas, excluirlas del conjunto de datos y ver el resultado de la voz neuronal personalizada. Siempre puede volver al estudio y grabar las muestras que faltan más tarde.

Anote en el guion el número de toma o código de tiempo de cada expresión. Pídale también al ingeniero que marque todas las expresiones en los metadatos de la grabación o en la hoja de pistas.

Haga pausas cada cierto tiempo y ofrezca bebidas para ayudar a mantener la voz del actor en buen estado.

### <a name="after-the-session"></a>Después de la sesión

Los estudios de grabación modernos funcionan en equipos informáticos. Al final de la sesión, recibirá uno o varios archivos de audio, no una cinta. Estos archivos tendrán probablemente el formato WAV o AIFF en calidad de CD (16 bits a 44,1 kHz) o superior. 24 bits a 48 kHz es lo habitual y deseable. Por lo general, no se necesitan velocidades de muestreo más altas, como 96 kHz.

Speech Studio requiere que cada expresión proporcionada esté en su propio archivo. Los archivos de audio entregados por el estudio contienen varias expresiones. Así que la principal tarea de posproducción es separar las grabaciones y prepararlas para su envío. Es posible que el ingeniero de grabación haya colocado marcadores en el archivo (o proporcionado una lista de pistas independiente) para indicar dónde comienza cada expresión.

Use sus notas para encontrar exactamente los cortes que quiera y, después, use una utilidad de edición de sonido, como [Avid Pro Tools](https://www.avid.com/en/pro-tools), [Adobe Audition](https://www.adobe.com/products/audition.html), o la aplicación gratuita [Audacity](https://www.audacityteam.org/) para copiar las expresiones en un archivo nuevo.

Deje solamente unos 0,2 segundos de silencio al principio y al final de cada clip, excepto para el primero. Ese archivo debe empezar con cinco segundos completos de silencio. No use un editor de audio para "poner a cero" las partes de silencio del archivo. Incluir el "tono de la sala" ayudará a los algoritmos a compensar los ruidos de fondo que hayan podido quedar.

Escuche atentamente cada archivo. En esta fase, puede editar pequeños sonidos no deseados que pasó por alto durante la grabación, como ligeros ruidos con los labios antes de una línea, pero tenga cuidado de no quitar la voz real. Si no puede corregir un archivo, elimínelo del conjunto de datos y tome nota de que lo ha hecho.

Convierta todos los archivos a 16 bits y una velocidad de muestreo de 24 kHz antes de guardar y, si graba la charla del estudio, quite el segundo canal. Guarde cada archivo en formato WAV y nombre los archivos con el número de enunciado del guion.

Por último, cree la *transcripción* que asocia cada archivo WAV con una versión de texto del enunciado correspondiente. [Creación de una voz personalizada](./how-to-custom-voice-create-voice.md) incluye los detalles del formato necesario. Puede copiar el texto directamente del guion. Después, cree un archivo ZIP de los archivos WAV y de la transcripción de texto.

Archive las grabaciones originales en un lugar seguro en caso de que las necesite más adelante. También, conserve el guion y las notas.

## <a name="next-steps"></a>Pasos siguientes

Está listo para cargar las grabaciones y crear la voz neuronal personalizada.

> [!div class="nextstepaction"]
> [Creación y uso de un modelo de voz](./how-to-custom-voice-create-voice.md)