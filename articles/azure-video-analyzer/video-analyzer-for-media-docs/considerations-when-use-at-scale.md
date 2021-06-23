---
title: 'Aspectos que se deben tener en cuenta al usar Azure Video Analyzer for Media (anteriormente, Video Indexer) a gran escala: Azure'
titleSuffix: Azure Media Services
description: En este tema se explican los aspectos que se deben tener en cuenta al usar Azure Video Analyzer for Media (anteriormente, Video Indexer) a gran escala.
services: media-services
author: Juliako
manager: femila
ms.topic: how-to
ms.date: 11/13/2020
ms.author: juliako
ms.openlocfilehash: bfad40e55deae4ebad930907221517ae3ef4e5f4
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388612"
---
# <a name="things-to-consider-when-using-video-analyzer-for-media-at-scale"></a>Aspectos que se deben tener en cuenta al usar Video Analyzer for Media a gran escala

Si se usa Azure Video Analyzer for Media (anteriormente, Video Indexer) para indexar vídeos y el archivo de vídeos está creciendo, considere la posibilidad de usar la opción de escalado. 

En este artículo se responden las siguientes preguntas:

* ¿Hay alguna restricción tecnológica que deba tener en cuenta?
* ¿Se puede hacer de forma inteligente y eficaz?
* ¿Puedo evitar gastar mucho dinero en el proceso?

En el artículo se incluyen seis procedimientos recomendados para usar Video Analyzer for Media a gran escala.

## <a name="when-uploading-videos-consider-using-a-url-over-byte-array"></a>Al cargar vídeos, considere la posibilidad de usar una dirección URL a través de una matriz de bytes.

Video Analyzer for Media ofrece la opción de cargar vídeos desde una dirección URL, o directamente mediante el envío del archivo como matriz de bytes, aunque esta última opción tiene algunas restricciones. Para más información, consulte el apartado [Consideraciones y limitaciones para la carga](upload-index-videos.md#uploading-considerations-and-limitations).

En primer lugar, tiene limitaciones con respecto al tamaño de los archivos. El archivo de la matriz de bytes no puede superar los 2 GB, en comparación con el límite de tamaño de carga de 30 GB cuando se usa la dirección URL.

En segundo lugar, tenga en cuenta algunos de los problemas que pueden afectar al rendimiento y, por ende, a su capacidad de escalado:

* El envío de archivos en varias partes conlleva una gran dependencia de la red, 
* confiabilidad del servicio, 
* conectividad, 
* velocidad de carga y 
* pérdida de paquetes en alguna parte de World Wide Web.

:::image type="content" source="./media/considerations-when-use-at-scale/first-consideration.png" alt-text="Primera consideración para usar Video Analyzer for Media a gran escala":::

Si se cargan vídeos mediante la dirección URL, solo hay que especificar una ruta de acceso a la ubicación de un archivo multimedia ya que Video Indexer se encarga del resto (consulte el campo `videoUrl` en la API de [carga de vídeo](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Upload-Video)).

> [!TIP]
> Use el parámetro opcional `videoUrl` de la API de carga de vídeo.

Para ver un ejemplo de cómo cargar vídeos mediante una dirección URL, consulte [este ejemplo](upload-index-videos.md#code-sample). También puede usar [AzCopy](../../storage/common/storage-use-azcopy-v10.md), ya que es una forma rápida y confiable de enviar su contenido a una cuenta de almacenamiento, desde la que puede enviarlo a Video Analyzer for Media mediante la [dirección URL de SAS](../../storage/common/storage-sas-overview.md).

## <a name="increase-media-reserved-units-if-needed"></a>Aumento de las unidades reservadas de multimedia en caso de necesidad

Normalmente, en la fase de prueba de concepto, cuando se acaba de empezar a usar Video Analyzer for Media, no se necesita una gran capacidad de cálculo. Si el archivo de vídeos empieza a ser grande y tiene que indexarlo, y quiere que este proceso se realice a una velocidad que se ajuste a sus necesidades concretas, deberá escalar verticalmente el uso de Video Analyzer for Media. Por consiguiente, convendría que se planteara la posibilidad de aumentar el número de recursos de proceso que utiliza en caso de que la potencia de computación actual no sea suficiente.

En Azure Media Services, si desea aumentar la potencia de computación y la paralelización, es preciso prestar atención a las [unidades reservadas](../../media-services/latest/concept-media-reserved-units.md)(RU) de multimedia. Estás son las unidades de proceso que determinan los parámetros de las tareas de procesamiento de elementos multimedia. El número de unidades reservadas afecta al número de tareas multimedia que se pueden procesar simultáneamente en cada cuenta y su tipo determina la velocidad de procesamiento y, si la indexación de un vídeo es compleja, puede requerir más de una unidad reservada. Si las unidades reservadas están ocupadas, las tareas nuevas se pondrán en una cola hasta que haya otro recurso disponible.

Para funcionar con eficacia y no tener recursos que estén inactivos parte del tiempo, Video Indexer ofrece un sistema de escalabilidad automática que reduce el uso de unidades reservadas cuando se necesita menos procesamiento y lo aumenta en las horas punta (hasta usar absolutamente todas ellas). Para habilitar esta funcionalidad, [active la escalabilidad automática](manage-account-connected-to-azure.md#autoscale-reserved-units) en la configuración de la cuenta o mediante [Update-Paid-Account-Azure-Media-Services API](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Update-Paid-Account-Azure-Media-Services).

:::image type="content" source="./media/considerations-when-use-at-scale/second-consideration.jpg" alt-text="Segunda consideración para usar Video Analyzer for Media a gran escala":::

:::image type="content" source="./media/considerations-when-use-at-scale/reserved-units.jpg" alt-text="Unidades reservadas de Azure Media Services":::

Para minimizar la duración de la indexación y un rendimiento bajo, es aconsejable que empiece con 10 unidades reservadas del tipo S3. Más adelante, si realiza un escalado vertical para admitir más contenido o mayor simultaneidad, y necesita más recursos para hacerlo, puede [ponerse en contacto con nosotros mediante el sistema de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) (solo en cuentas de pago) para solicitar la asignación de más unidades reservadas.

## <a name="respect-throttling"></a>Respeto de los límites

Video Analyzer for Media se creó para realizar indexación a gran escala y, para sacarle el máximo partido, es preciso conocer las funcionalidades del sistema y diseñar la integración en consecuencia. No desea enviar una solicitud de carga de un lote de vídeos solo para detectar que algunas de las películas no se han cargado y que recibe un código de respuesta HTTP 429 (demasiadas solicitudes). Esto puede deberse a que ha enviado un número de solicitudes que supera el [límite de películas por minuto que se admiten](upload-index-videos.md#uploading-considerations-and-limitations). Video Analyzer for Media agrega un encabezado `retry-after` en la respuesta HTTP. Este encabezado especifica cuándo se debe realizar el siguiente reintento. Asegúrese de que lo respeta antes de intentar la siguiente solicitud.

:::image type="content" source="./media/considerations-when-use-at-scale/respect-throttling.jpg" alt-text="Diseñe bien la integración, respete los límites":::

## <a name="use-callback-url"></a>Uso de una URL de devolución de llamada

Es aconsejable que, en lugar de sondear el estado de una solicitud constantemente desde el segundo posterior al envío de la solicitud de carga, agregue una [URL de devolución de llamada ](upload-index-videos.md#callbackurl) y espere a que Video Analyzer for Media la actualice. En cuanto se produzca cualquier cambio de estado en la solicitud de carga, recibirá una notificación POST a la dirección URL que haya especificado.

Puede agregar una URL de devolución de llamada como uno de los parámetros de la [API de carga de vídeo](https://api-portal.videoindexer.ai/api-details#api=Operations&operation=Upload-Video). Vea los ejemplos de código en el [repositorio de GitHub](https://github.com/Azure-Samples/media-services-video-indexer/tree/master/). 

Para la URL de devolución de llamada también se puede usar Azure Functions, una plataforma basada en eventos sin servidor que se puede desencadenar mediante HTTP e implementar un flujo de seguimiento.

### <a name="callback-url-definition"></a>Definición de URL de devolución de llamada

[!INCLUDE [callback url](./includes/callback-url.md)]

## <a name="use-the-right-indexing-parameters-for-you"></a>Uso de los parámetros de indexación correctos

Cuando tome decisiones relacionadas con el uso de Video Analyzer for Media a gran escala, examine cómo sacarle el máximo partido a los parámetros que más se ajusten a sus necesidades. Piense en su caso concreto, ya que mediante la definición de diferentes parámetros puede ahorrar dinero y acelerar el proceso de indexación de los vídeos.

Antes de cargar e indexar cualquier vídeo, lea esta breve [documentación](upload-index-videos.md) y consulte [indexingPreset](upload-index-videos.md#indexingpreset) y [streamingPreset](upload-index-videos.md#streamingpreset) para obtener una idea más clara de las opciones con las que cuenta.

Por ejemplo, no establezca el valor preestablecido en streaming si no tiene pensado ver el vídeo, y no indexe la información de vídeo si solo necesita información de audio.

## <a name="index-in-optimal-resolution-not-highest-resolution"></a>Indexación en una resolución óptima, no en la resolución máxima

Puede que desee saber qué calidad de vídeo necesita para indexar los vídeos. 

En muchos casos, el rendimiento de la indexación es casi idéntico entre los vídeos de alta definición (720P) y los vídeos 4K. Al final, obtendrá prácticamente la misma información con la misma confianza. Cuanto mayor sea la calidad de la película que se carga, mayor será el tamaño del archivo, lo que requerirá una mayor potencia de computación y se necesitará más tiempo para cargar el vídeo.

Por ejemplo, para la característica de detección de caras, una mayor resolución puede servir de ayuda en un escenario en el que hay muchas caras pequeñas, pero importantes para el contexto. Sin embargo, esto provoca un aumento cuadrático del tiempo de ejecución y un mayor riesgo de falsos positivos.

Por tanto, se recomienda comprobar que obtiene los resultados correctos para cada caso y realizar antes una prueba en un entorno local. Cargue el mismo vídeo en alta definición y en 4K, y compare la información que obtiene.

## <a name="next-steps"></a>Pasos siguientes

[Examen de la salida de Azure Video Analyzer for Media producida por la API](video-indexer-output-json-v2.md)