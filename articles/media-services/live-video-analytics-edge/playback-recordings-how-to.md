---
title: 'Reproducción de grabaciones con Live Video Analytics: Azure'
description: Puede usar Azure Live Video Analytics on IoT Edge para la grabación continua de vídeo, lo que permite grabar vídeo en la nube durante semanas o meses. También puede limitar la grabación a clips de interés mediante la grabación basada en eventos. En este artículo se habla sobre cómo reproducir las grabaciones.
ms.topic: how-to
ms.date: 04/27/2020
ms.openlocfilehash: bc8698fc9317421513677eb64403ac2cdc3905c3
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110372086"
---
# <a name="playback-of-recordings-with-live-video-analytics"></a>Reproducción de grabaciones con Live Video Analytics

[!INCLUDE [redirect to Azure Video Analyzer](./includes/redirect-video-analyzer.md)]

## <a name="pre-read"></a>Lectura previa  

* [Reproducción de vídeo](video-playback-concept.md)
* [Grabación de vídeo continua](continuous-video-recording-concept.md)
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)

## <a name="background"></a>Información previa  

Puede usar Live Video Analytics on IoT Edge para la [grabación continua de vídeo](continuous-video-recording-concept.md), lo que permite grabar vídeo en la nube durante semanas o meses. También puede limitar la grabación a clips de interés mediante la [grabación de vídeo basada en eventos](event-based-video-recording-concept.md) (EVR). 

La cuenta de Media Services está vinculada a una cuenta de Azure Storage y, al grabar vídeo en la nube, el contenido se escribe en un [recurso de Media Services](terminology.md#asset). Media Services le permite [transmitir en secuencias el contenido](terminology.md#streaming) al finalizar la grabación o mientras está en curso. Media Services le ofrece las funciones necesarias para proporcionar secuencias mediante los protocolos HLS o MPEG-DASH a dispositivos de reproducción (clientes). Consulte el artículo [reproducción de vídeo](video-playback-concept.md) para obtener más detalles. Usaría las API de Media Services para generar las direcciones URL de streaming necesarias que usan los clientes para reproducir el vídeo y el audio. Para construir la dirección URL de streaming para un recurso, consulte [Creación de un localizador de streaming y compilación de direcciones URL](../latest/create-streaming-locator-build-url.md). Una vez creada la dirección URL de streaming para un recurso, puede y debe almacenar la asociación de la dirección URL con el origen de vídeo (es decir, la cámara) en el sistema de administración de contenido.

Por ejemplo, al realizar streaming mediante HLS, la dirección URL de streaming tendría este aspecto: `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl).m3u8`. En MPEG-DASH, sería similar a esta: `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=mpd-time-cmaf).mpd`.

Esto devuelve un archivo de manifiesto, que, entre otras cosas, describe la duración total de la secuencia que se está entregando y si representa el contenido grabado previamente o la fuente "en directo" actual.

### <a name="live-vs-vod"></a>Directo y VoD  

Los protocolos de streaming, como HLS y MPEG-DASH, se crearon para controlar escenarios como el streaming de vídeos en directo o el de contenido a petición/grabado previamente, como programas de TV y películas. En el caso de los vídeos en directo, los clientes HLS y MPEG-DASH están diseñados para iniciar la reproducción desde el momento "más reciente" en adelante. Sin embargo, con las películas, los espectadores esperan empezar desde el principio y saltar a otro momento si quieren. Los manifiestos de HLS y MPEG-DASH tienen marcas que indican a los clientes si el vídeo representa streaming en vivo o si se trata de contenido grabado previamente.
Este concepto también se aplica a las secuencias HLS y MPEG-DASH de los recursos que contienen vídeo grabado mediante Live Video Analytics on IoT Edge.

## <a name="browsing-and-selective-playback-of-recordings"></a>Exploración y reproducción selectiva de grabaciones  

Tenga en cuenta el escenario en el que ha usado Live Video Analytics on IoT Edge para grabar vídeo solo de 8:00 a 10:00 en los días lectivos de todo el año académico. O quizás solo grabe vídeo de 7:00 a 19:00 en días festivos nacionales. En cualquiera de estos dos escenarios, al intentar examinar y ver la grabación de vídeo, necesitaría:

* Una manera de determinar qué fechas y horas de vídeo tiene en una grabación.
* Una manera de seleccionar una parte (por ejemplo, de 9:00 a 11:00 del primer día del año) de esa grabación para la reproducción.

Media Services proporciona una API de consulta (availableMedia) para resolver el primer problema y filtros de intervalo de tiempo (startTime, endTime) para el segundo.

## <a name="query-api"></a>API de consulta 

Cuando se usa CVR, los dispositivos de reproducción (clientes) no pueden solicitar un manifiesto que abarque la reproducción de toda la grabación.  Una grabación de varios años produciría un archivo de manifiesto demasiado grande para la reproducción y sería difícil de analizar en partes útiles en el lado del cliente.  El cliente debe saber qué intervalos de fecha y hora tienen datos en la grabación para poder realizar solicitudes válidas que no generen demasiado trabajo de estimación. Aquí es donde entra en juego la nueva API de consulta: ahora, los clientes pueden usar esta API del lado servidor para detectar contenido.

El valor de precisión puede ser un año, mes, día o intervalo completo (como se muestra a continuación). 

|Precision|year|month|day|full|
|---|---|---|---|---|
|Consultar|`/availableMedia?precision=year&startTime=2018&endTime=2019`|`/availableMedia?precision=month& startTime=2018-01& endTime=2019-02`|`/availableMedia?precision=day& startTime=2018-01-15& endTime=2019-02-02`|`/availableMedia?precision=full& startTime=2018-01-15T10:08:11.123& endTime=2019-01-015T12:00:01.123`|
|Response|`{  "timeRanges":[{ "start":"2018", "end":"2019" }]}`|`{  "timeRanges":[{ "start":"2018-03", "end":"2019-01" }]}`|`{  "timeRanges":[    { "start":"2018-03-01", "end":"2018-03-07" },    { "start":"2018-03-09", "end":"2018-03-31" }  ]}`|Respuesta de plena fidelidad. En caso de que no haya ninguna separación, el inicio sería startTime y el final, endTime.|
|Restricciones|&#x2022;startTime <= endTime<br/>&#x2022;Ambos valores deben tener el formato AAAA. De lo contrario, devuelve un error.<br/>&#x2022;Los valores pueden estar separados por cualquier cantidad de años.<br/>&#x2022;Los valores son inclusivos.|&#x2022;startTime <= endTime<br/>&#x2022;Ambos valores deben tener el formato AAAA-MM. De lo contrario, devuelve un error.<br/>&#x2022;Los valores pueden tener una separación máxima de 12 meses.<br/>&#x2022;Los valores son inclusivos.|&#x2022;startTime <= endTime<br/>&#x2022;Ambos deben tener el formato AAAA-MM-DD. De lo contrario, devuelve un error.<br/>&#x2022;Los valores pueden tener una separación máxima de 31 días.<br/>Los valores son inclusivos.|&#x2022;startTime < endTime<br/>&#x2022;Los valores pueden tener una separación máxima de 25 horas.<br/>&#x2022;Los valores son inclusivos.|

#### <a name="additional-request-format-considerations"></a>Consideraciones adicionales sobre el formato de la solicitud

* Todas las horas se muestran en UTC.
* El parámetro de la cadena de consulta de precisión es obligatorio.  
* Los parámetros de cadena de consulta startTime y endTime son necesarios para los valores de precisión de mes, día e intervalo completo.  
* Los parámetros de cadena de consulta startTime y endTime son opcionales para el valor de precisión de año (se puede especificar uno de los dos, los dos o ninguno).  

    * Si se omite startTime, se asume que es el primer año de la grabación.
    * Si se omite endTime, se asume que es el último año de la grabación.
    * Por ejemplo, si la grabación comenzó en 2011 y ha continuado hasta 2020, entonces:

        * /availableMedia?precision=year equivale a /availableMedia?precision=year&startTime=2011&endTime=2020
        * /availableMedia?precision=year&startTime=2015 equivale a /availableMedia?precision=year&startTime=2015&endTime=2020
        * /availableMedia?precision=year&endTime=2018 equivale a /availableMedia?precision=year&startTime=2011&endTime=2018

Si no hay datos multimedia disponibles para los intervalos de tiempo, se devuelve una lista vacía.

Si el recurso no contiene ninguna grabación de un grafo multimedia, se devolverá una respuesta HTTP 400 con un mensaje de error que explicará que esta característica solo está disponible para el contenido grabado mediante un grafo multimedia.

Si el tiempo especificado por los parámetros es mayor que el permitido según el intervalo de tiempo máximo de un tipo de consulta concreto, se devolverá un error HTTP 400 con un mensaje de error que indicará el intervalo de tiempo máximo permitido para el tipo de consulta solicitado.

Suponiendo que el intervalo de tiempo sea válido para los parámetros especificados, no se devolverá ningún error para las consultas que estén fuera del período de tiempo de los datos de la grabación.  Es decir, si la grabación comenzó hace 7 horas, pero el cliente solicita el contenido multimedia de {UtcNow – 24 hours} a UtcNow, solo devolveríamos {UtcNow – 7} horas de datos.

### <a name="response-format"></a>Formato de respuesta  

La llamada a availableMedia devuelve un conjunto de valores de tiempo.

La respuesta será un cuerpo JSON, con valores timeRanges de matriz de fechas UTC ISO 8601 para el año (en formato AAAA), mes (en formato AAAA-MM) o día (AAAA-MM-DD), en función de la precisión solicitada.  El valor completo de timeRanges contendrá un valor de inicio y de finalización con formato de fecha y hora UTC ISO 8601 (AAA-MM-DDThh:mm:ss.sssZ).

Para la consulta availableMedia, la API usará la escala de tiempo del vídeo. Cualquier interrupción en la escala de tiempo se mostrará como una separación en la respuesta.

#### <a name="response-example-for-availablemedia"></a>Ejemplo de respuesta para availableMedia

##### <a name="example-1-live-recording-with-no-gaps"></a>Ejemplo 1: Grabación en directo sin separación

A continuación, se incluye una respuesta en la que se muestran todos los contenidos multimedia disponibles para el 21 de diciembre de 2019, cuando la grabación se había completado al 100 %. La respuesta solo tiene un par de inicio y fin.

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
   {
         "start":"2019-12-21T00:00:00.000Z", 
         "end":"2019-12-22T00:00:00.000Z"
    }
   ]
}
```

##### <a name="example-2-live-recording-with-a-2-second-gap"></a>Ejemplo 2: Grabación en directo con una separación de 2 segundos

Imagine que, por algún motivo, la cámara no pudiera enviar un vídeo válido durante un intervalo de dos segundos en un día concreto. Esta es una respuesta que muestra todo el contenido multimedia para el 21 de diciembre de 2019:

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:01:08Z"
    },
    {
         "start":"2019-12-21T04:01:10Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

##### <a name="example-3-live-recording-with-an-8-hour-gap"></a>Ejemplo 3: Grabación en directo con una separación de 8 segundos

Supongamos que la cámara o la instalación local perdieran el suministro eléctrico durante un período de 8 horas durante el día, de las 4:00 a las 12:00 y que no hubiera ninguna fuente de alimentación auxiliar. Esta es una respuesta que muestra todo el contenido multimedia disponible para ese día.

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:00:00Z"
    },
    {
         "start":"2019-12-21T12:00:00Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

#### <a name="example-4-live-recording-where-no-video-is-recorded-over-summer-holidays"></a>Ejemplo 4: Grabación en directo en la que no se registra ningún vídeo durante las vacaciones de verano

Imagine que solo grabó vídeo durante el período lectivo y que la grabación se detuvo del 17 de junio al 6 de septiembre. Una consulta para los meses disponibles tendría el siguiente aspecto:

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=month&startTime=2019-01&endTime=2019-12
{
   "timeRanges":[
    {
         "start":"2019-01", 
         "end":"2019-06"
    },
    {
         "start":"2019-09", 
         "end":"2019-12"
    }
   ]
}
```

Si, después, preguntara por los días disponibles en junio, vería lo siguiente:

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=day&startTime=2019-06-01&endTime=2019-06-30
{
   "timeRanges":[
    {
         "start":"2019-06-01", 
         "end":"2019-06-17"
    }
   ]
}
```

##### <a name="example-5-live-recording-where-no-video-is-recorded-over-weekends-or-holidays"></a>Ejemplo 5: Grabación en vivo en la que no se registró ningún vídeo durante los fines de semana o los días festivos.

Supongamos que solo grabó vídeo durante el horario comercial. Una consulta de los días disponibles tendría este aspecto:

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=day&startTime=2020-02-01&endTime=2020-02-29
{
   "timeRanges":[
    {
         "start":"2020-02-03", 
         "end":"2020-02-07"
    },
    {
         "start":"2020-02-10", 
         "end":"2020-02-14"
    },
    {
         "start":"2020-02-18", // Monday Feb 17th was a holiday
         "end":"2020-02-21"
    },
    {
         "start":"2020-02-24", 
         "end":"2020-02-28"
    }
   ]
}
```

## <a name="time-range-filters"></a>Filtros de intervalo de tiempo

Como se mencionó anteriormente, estos filtros le ayudan a seleccionar partes de la grabación (por ejemplo, de 9:00 a 11:00 del primer día del año) para la reproducción. Si realiza streaming mediante HLS, la dirección URL de streaming tendría este aspecto: `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl).m3u8`. Para seleccionar una parte de la grabación, debe agregar los parámetros startTime y endTime de este modo: `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T08:00:00Z,endTime=2019-12-21T10:00:00Z).m3u8`. Por lo tanto, los filtros de intervalo de tiempo son modificadores de dirección URL que se usan para describir la parte de la escala de tiempo de la grabación incluida en el manifiesto de streaming:

* `starttime` es una marca de fecha y hora ISO 8601 que describe la hora de inicio deseada de la escala de tiempo del vídeo en el manifiesto devuelto.
* `endtime` es una marca de fecha y hora ISO 8601 que describe la hora de finalización deseada de la escala de tiempo del vídeo que se devuelve en el manifiesto.

La longitud máxima (de tiempo) de este tipo de manifiesto no puede superar las 24 horas.

Estas son las restricciones de los filtros.

|startTime| endTime |Resultado|
|---|---|---|
|Absent |Absent |Devuelve la parte más reciente del vídeo en el recurso, hasta una longitud máxima de 4 horas.<br/><br/>Si el recurso no se ha escrito (no se han incorporado nuevos datos de vídeo) recientemente, se devuelve un manifiesto a petición (VoD). En caso contrario, se devuelve un manifiesto en directo.|
|Presente|Absent|    Devuelve un manifiesto con el vídeo que esté disponible que sea más reciente que startTime, si el manifiesto abarca menos de 24 horas.<br/>Si la longitud del manifiesto supera las 24 horas, devuelve un error.<br/>Si el recurso no se ha escrito (no se han incorporado nuevos datos de vídeo) recientemente, se devuelve un manifiesto a petición (VoD). En caso contrario, se devuelve un manifiesto en directo.
|Absent|Presente |Error: Si hay un valor startTime, se necesita un valor endTime.|
|Presente|Presente|Devuelve un manifiesto de VoD con cualquier vídeo disponible entre startTime y endTime.<br/>El intervalo (startTime, endTime) no puede superar las 24 horas.|

### <a name="response-examples"></a>Ejemplos de respuesta  

#### <a name="example-1-live-recording-with-no-gaps"></a>Ejemplo 1: Grabación en directo sin separación

A continuación, se incluye una respuesta en la que se muestran todos los contenidos multimedia disponibles para el 21 de diciembre de 2019, cuando la grabación se había completado al 100 %. La respuesta solo tiene un par de inicio y fin.

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
   {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

Cuando la grabación es continua, puede solicitar manifiestos HLS o DASH para cualquier parte del tiempo dentro del par de inicio/fin, siempre que el intervalo sea de 24 horas o menos. Un ejemplo de una solicitud de manifiesto HLS de 4 horas para lo anterior sería:

`GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T14:00:00.000Z,endTime=2019-12-21T18:00:00.000Z).m3u8`

#### <a name="example-2-live-recording-with-a-2-second-gap"></a>Ejemplo 2: Grabación en directo con una separación de 2 segundos

Imagine que, por algún motivo, la cámara no pudiera enviar un vídeo válido durante un intervalo de dos segundos en un día concreto. Esta es una respuesta que muestra el contenido multimedia para el 21 de diciembre de 2019:

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:01:08Z"
    },
    {
         "start":"2019-12-21T04:01:10Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

Con una grabación como la anterior, podría solicitar manifiestos HLS y DASH con cualquier par startTime/endTime, siempre y cuando el intervalo fuera inferior a 24 horas. Si estos valores abarcan la separación en 04:01:08 UTC, el manifiesto devuelto señalaría la discontinuidad en la escala de tiempo multimedia según las especificaciones pertinentes para esos protocolos.

#### <a name="example-3-live-recording-with-an-8-hour-gap"></a>Ejemplo 3: Grabación en directo con una separación de 8 horas

Supongamos que la cámara o la instalación local perdiera el suministro eléctrico durante un período de 8 horas durante el día, de las 4:00 a las 12:00 y que no hubiera ninguna fuente de alimentación auxiliar. Esta es una respuesta que muestra todo el contenido multimedia para ese día.

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:00:00Z"
    },
    {
         "start":"2019-12-21T12:00:00Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

Con este tipo de grabación:

* Si solicita un manifiesto en el que los filtros del intervalo startTime/endTime estén dentro de las partes "disponibles" de la escala de tiempo (p. ej., de medianoche a 4:00 o de mediodía a medianoche), el servicio devolvería un manifiesto que cubre el tiempo de startTime a endTime, como el siguiente:

    `GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T14:01:00.000Z,endTime=2019-12-21T03:00:00.000Z).m3u8`
* Si solicita un manifiesto en que startTime y endTime están dentro de la "separación" del medio (p. ej., de 8:00 a 10:00 UTC), el servicio se comportará de la misma manera que si un filtro de recursos tuviera un resultado vacío.

    [Esta es una solicitud que obtiene una respuesta vacía] `GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T08:00:00.000Z,endTime=2019-12-21T10:00:00.000Z).m3u8`
* Si solicita un manifiesto donde solo uno de los valores de startTime o endTime está dentro de la "separación", el manifiesto devuelto solo incluirá una parte de ese intervalo de tiempo. Ajustaría el valor startTime o endTime al límite válido más cercano. Por ejemplo, si solicita una secuencia de 3 horas de 10:00 a 13:00, la respuesta contendrá una hora de contenido multimedia, de las 12:00 a las 13:00.

    `GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T10:00:00.000Z,endTime=2019-12-21T13:00:00.000Z).m3u8`
    
    El manifiesto devuelto empezaría a las 2019-12-21T12:00:00.000Z, aunque en la solicitud se pedía el inicio a las 10:00. Al compilar un sistema de administración de vídeo con un complemento para el reproductor, es necesario indicárselo al usuario, ya que un usuario que lo ignore podría no entender por qué empieza la escala de tiempo de reproducción a mediodía y no a las 10:00, como se solicitó.

## <a name="recording-and-playback-latencies"></a>Latencias de grabación y reproducción

Al usar Live Video Analytics on IoT Edge para grabar un recurso, se especifica una propiedad segmentLength que indica al módulo que agregue una duración mínima de vídeo (en segundos) antes de que se grabe en la nube. Por ejemplo, si segmentLength se establece en 300, el módulo acumulará 5 minutos de vídeo antes de cargar un "fragmento" de 5 minutos y, a continuación, entrará en el modo de acumulación durante los 5 minutos siguientes y volverá a cargarse. Aumentar el valor de segmentLength tiene la ventaja de reducir los costos de las transacciones de Azure Storage, ya que el número de lecturas y escrituras no tendrá una frecuencia superior a una vez cada segmentLength segundos.

Por lo tanto, el streaming del vídeo de Media Services se retrasará al menos durante ese tiempo. 

Otro factor que determina la latencia de reproducción (el retraso entre el momento en que se produce un evento delante de la cámara hasta el momento en que se puede ver en un dispositivo de reproducción) es la duración del grupo de imágenes ([GOP](https://en.wikipedia.org/wiki/Group_of_pictures)). Tal como se describe en [Reducción del retraso del streaming en vivo mediante tres sencillas técnicas](https://medium.com/vrt-digital-studio/reducing-the-delay-of-live-streams-by-using-3-simple-techniques-e8e028b0a641), cuanto mayor sea la duración del GOP, mayor será la latencia. Es habitual que las cámaras IP que se usan en escenarios de vigilancia y seguridad se configuren con un valor de GOP de más de 30 segundos. Esto tiene un gran impacto sobre la latencia general.

## <a name="next-steps"></a>Pasos siguientes

[Tutorial de grabación continua de vídeo](continuous-video-recording-tutorial.md)
