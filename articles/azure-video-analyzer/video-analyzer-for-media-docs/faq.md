---
title: 'Preguntas más frecuentes sobre Azure Video Analyzer for Media (anteriormente Video Indexer): Azure'
titleSuffix: Azure Media Services
description: En este artículo se responde a las preguntas más frecuentes sobre Azure Video Analyzer for Media (anteriormente Video Indexer).
services: media-services
author: Juliako
manager: femila
ms.topic: article
ms.date: 05/06/2021
ms.author: juliako
ms.openlocfilehash: 620ebb81462f807b46c12aeee596e8873c99a1e4
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386779"
---
# <a name="video-analyzer-for-media-frequently-asked-questions"></a>Preguntas más frecuentes sobre Video Analyzer for Media

En este artículo se responde a las preguntas más frecuentes sobre Azure Video Analyzer for Media (anteriormente Video Indexer).

## <a name="general-questions"></a>Preguntas generales

### <a name="what-is-video-analyzer-for-media"></a>¿Qué es Video Analyzer for Media?

Video Analyzer for Media es un servicio de inteligencia artificial que forma parte de Microsoft Azure Media Services. Video Analyzer for Media proporciona una orquestación de varios modelos de aprendizaje automático que permiten extraer fácilmente información detallada de un vídeo. Para proporcionar información precisa y avanzada, Video Analyzer for Media utiliza varios canales del vídeo: audio, voz y objetos visuales. La información de Video Analyzer for Media se puede usar de muchas maneras, como mejorar la detectabilidad de contenido y la accesibilidad, crear oportunidades de monetización o crear experiencias que usen esta información. Video Analyzer for Media proporciona una interfaz web para pruebas, configuración y personalización de los modelos en la cuenta. Los desarrolladores pueden usar una API de REST para integrar Video Analyzer for Media en el sistema de producción. 

### <a name="what-can-i-do-with-video-analyzer-for-media"></a>¿Qué puedo hacer con Video Analyzer for Media?

Algunas de las operaciones que Video Analyzer for Media puede realizar en los archivos multimedia son:

* Identificación y extracción de diálogos e identificación de hablantes.
* Identificación y extracción del texto en pantalla de un vídeo.
* Detección de objetos en un archivo de vídeo.
* Identificación de marcas (por ejemplo: Microsoft) en las pistas de audio y en el texto en pantalla de un vídeo.
* Detección y reconocimiento de caras en una base de datos de caras de personas famosas y en una base de datos de caras definida por el usuario.
* Extracción de los temas tratados, aunque no necesariamente mencionados, en el contenido de audio y vídeo.
* Creación de subtítulos a partir de la pista de audio.

Para obtener más información y características de Video Analyzer for Media, consulte [Información general](video-indexer-overview.md).

### <a name="how-do-i-get-started-with-video-analyzer-for-media"></a>¿Cómo empiezo a trabajar con Video Analyzer for Media?

Video Analyzer for Media incluye una oferta de evaluación gratuita que proporciona 600 minutos en la interfaz web y 2400 minutos a través de la API. Puede [iniciar sesión en la interfaz web de Video Analyzer for Media](https://www.videoindexer.ai/) y probarlo usted mismo con cualquier identidad web y sin tener que configurar una suscripción de Azure. Siga [este sencillo laboratorio de introducción](https://github.com/Azure-Samples/media-services-video-indexer/blob/master/IntroToVideoIndexer.md) para obtener una idea más clara de cómo usar Video Analyzer for Media.

Para indexar archivos de vídeo y audio a escala, puede conectar Video Analyzer for Media a una suscripción de Microsoft Azure de pago. Puede obtener más información sobre los precios en la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/video-indexer/).

Puede obtener más información sobre cómo comenzar en [Comenzar](video-indexer-get-started.md).

### <a name="do-i-need-coding-skills-to-use-video-analyzer-for-media"></a>¿Necesito habilidades de codificación para usar Video Analyzer for Media?

Puede usar la interfaz web de Video Analyzer for Media para evaluar, configurar y administrar la cuenta **sin necesidad de código**.  Cuando esté listo para desarrollar aplicaciones más complejas, puede usar [Video Analyzer for Media API](https://api-portal.videoindexer.ai/) para integrarlo en sus propias aplicaciones, sitios web o [flujos de trabajo personalizados que usen tecnologías sin servidor, como Azure Logic Apps](https://azure.microsoft.com/blog/logic-apps-flow-connectors-will-make-automating-video-indexer-simpler-than-ever/) o Azure Functions.

### <a name="do-i-need-machine-learning-skills-to-use-video-analyzer-for-media"></a>¿Necesito habilidades de aprendizaje automático para usar Video Analyzer for Media?

No, Video Analyzer for Media proporciona la integración de varios modelos de aprendizaje automático en una canalización. Con la indexación de un archivo de audio o vídeo a través de Video Analyzer for Media se recupera un conjunto completo de información extraído en una escala de tiempo compartida sin habilidades de aprendizaje automático ni conocimientos sobre los algoritmos necesarios del lado del cliente.

### <a name="what-media-formats-does-video-analyzer-for-media-support"></a>¿Qué formatos multimedia admite Video Analyzer for Media?

Video Analyzer for Media admite los formatos multimedia más comunes. Vea la lista de los [formatos estándar de Azure Media Encoder](../../media-services/latest/encode-media-encoder-standard-formats-reference.md) para más detalles.

### <a name="how-do-i-upload-a-media-file-into-video-analyzer-for-media-and-what-are-the-limitations"></a>¿Cómo cargo un archivo multimedia en Video Analyzer for Media y cuáles son las limitaciones?

En el portal web de Video Analyzer for Media puede cargar un archivo multimedia mediante el cuadro de diálogo de carga de archivos o indicando una dirección URL que directamente hospede el archivo de origen (consulte el [ejemplo](https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/Ignite-short.mp4)). Cualquier dirección URL que hospede el contenido multimedia mediante iFrame o código para insertar no funcionará (consulte el [ejemplo](https://www.videoindexer.ai/accounts/7e1282e8-083c-46ab-8c20-84cae3dc289d/videos/5cfa29e152/?t=4.11)). 

Para obtener más información, vea esta [guía paso a paso](./upload-index-videos.md).

#### <a name="limitations"></a>Limitaciones

* El nombre del vídeo no puede superar los 80 caracteres.
* Si carga un vídeo mediante una matriz de bytes, el tamaño del vídeo se limita a 2 GB (30 GB al usar la dirección URL). 

Para obtener la lista completa, consulte [Consideraciones y limitaciones para la carga](upload-index-videos.md#uploading-considerations-and-limitations).

### <a name="how-long-does-it-take-video-analyzer-for-media-to-extract-insights-from-media"></a>¿Cuánto tiempo tarda Video Analyzer for Media en extraer información de los elementos multimedia?

El tiempo necesario para indexar un archivo de audio o vídeo, tanto mediante Video Analyzer for Media API como mediante la interfaz web, depende de varios parámetros, como la longitud del archivo y su calidad, la cantidad de información que se encuentra en él, el número de [unidades reservadas](../../media-services/previous/media-services-scale-media-processing-overview.md) disponibles y si el [punto de conexión de streaming](../../media-services/previous/media-services-streaming-endpoints-overview.md) está habilitado o no. Recomendamos que ejecute algunos archivos de prueba con su propio contenido y tome un promedio para obtener una idea más clara.

### <a name="can-i-create-customized-workflows-to-automate-processes-with-video-analyzer-for-media"></a>¿Puedo crear flujos de trabajo personalizados para automatizar los procesos con Video Analyzer for Media?

Sí, puede integrar Video Analyzer for Media en tecnologías sin servidor, como Logic Apps, Flow y [Azure Functions](https://azure.microsoft.com/services/functions/). Más detalles sobre los conectores de [Logic Apps](https://azure.microsoft.com/services/logic-apps/) y [Flow](https://flow.microsoft.com/en-us/) para Video Analyzer for Media [aquí](https://azure.microsoft.com/blog/logic-apps-flow-connectors-will-make-automating-video-indexer-simpler-than-ever/). Puede ver algunos proyectos de automatización realizados por asociados en el repositorio de [ejemplos de Video Analyzer for Media](https://github.com/Azure-Samples/media-services-video-indexer).

### <a name="in-which-azure-regions-is-video-analyzer-for-media-available"></a>¿En qué regiones de Azure está disponible Video Analyzer for Media?

Puede ver en qué regiones de Azure está disponible Video Analyzer for Media en la página [regiones](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services&regions=all).

### <a name="can-i-customize-video-analyzer-for-media-models-for-my-specific-use-case"></a>¿Puedo personalizar los modelos de Video Analyzer for Media para mi caso de uso concreto? 

Sí. En Video Analyzer for Media puede personalizar algunos de los modelos disponibles para que se adapten mejor a sus necesidades. 

Por ejemplo, el modelo Person admite de serie 1 millón de caras de reconocimiento de famosos, pero también puede entrenarlo para que reconozca otras caras que no están en esa base de datos. 

Para obtener detalles, vea los artículos sobre la personalización de los modelos [Person](customize-person-model-overview.md), [Brands](customize-brands-model-overview.md) y [Language](customize-language-model-overview.md). 

###  <a name="can-i-edit-the-videos-in-my-library"></a>¿Puedo editar los vídeos de mi biblioteca?

Sí. Presione el botón **Editar el vídeo** de la pantalla de la biblioteca o el botón **Abrir en el editor** de la pantalla del reproductor para ir a la pestaña **Proyectos**. Puede crear un proyecto nuevo y agregar más vídeos de la biblioteca para editarlos a la vez. Una vez que haya terminado, puede representar el vídeo y descargarlo. 

Si quiere obtener información sobre el nuevo vídeo, puede indexarlo con Video Analyzer for Media y aparecerá en la biblioteca con su información.

### <a name="can-i-index-multiple-audio-streams-or-channels"></a>¿Puedo indexar varios canales o secuencias de audio?

Si hay varias secuencias de audio, Video Analyzer for Media toma la primera que encuentra y procesa solo esa. En cualquier secuencia de audio que procesa Video Analyzer for Media, toma los distintos canales (si los hay) y los procesa como mono. En el caso de la manipulación de secuencias/canales, puede usar comandos ffmpeg en el archivo antes de indexarlo.

### <a name="what-is-the-sla-for-video-analyzer-for-media"></a>¿Cuál es el Acuerdo de Nivel de Servicio de Video Analyzer for Media?

El Acuerdo de Nivel de Servicio de Azure Media Services abarca Video Analyzer for Media y se encuentra en la página del [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/media-services/v1_2/). El Acuerdo de Nivel de Servicio solo se aplica a las cuentas de pago de Video Analyzer for Media y no a la evaluación gratuita.

## <a name="privacy-questions"></a>Cuestiones de privacidad

### <a name="are-video-and-audio-files-indexed-by-video-analyzer-for-media-stored"></a>¿Se almacenan los archivos de audio y vídeo indexados por Video Analyzer for Media?

Sí, a menos que elimine el archivo de Video Analyzer for Media mediante el sitio web o Video Analyzer for Media API, los archivos de audio y vídeo se almacenan. Para la evaluación gratuita, los archivos de audio y vídeo que indexe se almacenan en la región de Azure Este de EE. UU. En caso contrario, los archivos de audio y vídeo se almacenan en la cuenta de almacenamiento de su suscripción de Azure.

### <a name="can-i-delete-my-files-that-are-stored-in-video-analyzer-for-media-portal"></a>¿Puedo eliminar mis archivos almacenados en el portal de Video Analyzer for Media?

Sí, siempre puede eliminar los archivos de audio y vídeo, así como los metadatos y la información extraída de ellos por Video Analyzer for Media. Una vez que se elimina un archivo de Video Analyzer for Media, el archivo y sus metadatos y conclusiones se quitan permanentemente de Video Analyzer for Media. Sin embargo, si ha implementado su propia solución de copia de seguridad en Azure Storage, el archivo permanece en el almacenamiento de Azure.

### <a name="can-i-control-user-access-to-my-video-analyzer-for-media-account"></a>¿Puedo controlar el acceso de los usuarios a mi cuenta de Video Analyzer for Media?

Sí, solo los administradores de cuenta pueden invitar y dejar de invitar a personas a sus cuentas, así como asignar quién tiene privilegios de edición y quién tiene acceso de solo lectura.

### <a name="who-has-access-to-my-video-and-audio-files-that-have-been-indexed-andor-stored-by-video-analyzer-for-media-and-the-metadata-and-insights-that-were-extracted"></a>¿Quién tiene acceso a los archivos de audio y vídeo que Video Analyzer for Media ha indexado o almacenado y a los metadatos y la información que se extrajeron?

Cualquiera que tenga el vínculo al contenido de vídeo o audio y su información puede acceder al contenido cuya privacidad haya configurado como pública. Al contenido de vídeo o audio que tiene Privado como configuración de privacidad solo pueden acceder los usuarios que han sido invitados a la cuenta de ese contenido. La configuración de privacidad del contenido también se aplica a los metadatos y la información que Video Analyzer for Media extrae. La configuración de privacidad se asigna al carga el archivo de vídeo o audio. También puede cambiar la configuración de privacidad después de indexación.

### <a name="what-access-does-microsoft-have-to-my-video-or-audio-files-that-have-been-indexed-andor-stored-by-video-analyzer-for-media-and-the-metadata-and-insights-that-were-extracted"></a>¿Qué tipo de acceso tiene Microsoft a mis archivos de audio y vídeo que Video Analyzer for Media ha indexado o almacenado y a los metadatos y la información que se extrajeron?

De conformidad con los [Términos de los servicios en línea de Azure](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31) (OST), el contenido es totalmente suyo y Microsoft solo puede acceder a él y a los metadatos e información que Video Analyzer for Media extrae de él de conformidad con los Términos de los servicios en línea y la declaración de privacidad de Microsoft.

### <a name="are-the-custom-models-that-i-build-in-my-video-analyzer-for-media-account-available-to-other-accounts"></a>¿Los modelos personalizados que creo en mi cuenta de Video Analyzer for Media están disponibles para otras cuentas?

 No, los modelos personalizados que cree en su cuenta no están disponibles para ninguna otra cuenta. Video Analyzer for Media actualmente le permite crear modelos personalizados de [marcas](customize-brands-model-overview.md), [idioma](customize-language-model-overview.md) y [persona](customize-person-model-overview.md) en su cuenta. Estos modelos solo están disponibles en la cuenta en la que los ha creado.
    
### <a name="is-the-content-indexed-by-video-indexer-kept-within-the-azure-region-where-i-am-using-video-indexer"></a>¿Se mantiene el contenido indexado por Video Indexer dentro de la región de Azure donde estoy usando Video Indexer?

Sí, el contenido y su información se mantienen dentro de la región de Azure (excepto las regiones de Singapur y Sur de Brasil) a menos que tenga una configuración manual en la suscripción de Azure que use varias regiones de Azure.

Los datos del cliente de una región se replican por motivos de BCDR en la [región emparejada](https://docs.microsoft.com/azure/best-practices-availability-paired-regions#azure-regional-pairs).

### <a name="what-is-the-privacy-policy-for-video-analyzer-for-media"></a>¿Cuál es la directiva de privacidad de Video Analyzer for Media?

Video Analyzer for Media está cubierto por la [declaración de privacidad de Microsoft](https://privacy.microsoft.com/privacystatement). En la declaración de privacidad se explican los datos personales que Microsoft procesa, cómo los procesa y los propósitos de este procesamiento. Para más información sobre la privacidad, visite [Microsoft Trust Center](https://www.microsoft.com/trustcenter).

### <a name="what-certifications-does-video-analyzer-for-media-have"></a>¿Qué certificaciones tiene Video Analyzer for Media?

Video Analyzer for Media tiene actualmente la certificación SOC. Para revisar la certificación de Video Analyzer for Media, consulte [Microsoft Trust Center](https://www.microsoft.com/trustcenter/compliance/complianceofferings?product=Azure).

### <a name="what-is-the-difference-between-private-and-public-videos"></a>¿Cuál es la diferencia entre los vídeos privados y públicos? 

Cuando se cargan vídeos en Video Analyzer for Media, puede elegir entre dos valores de privacidad: privado y público. Los vídeos públicos son accesibles para cualquier usuario, incluidos los usuarios anónimos y no identificados. Los privados están restringidos exclusivamente a los miembros de la cuenta. 

### <a name="i-tried-to-upload-a-video-as-public-and-it-was-flagged-for-inappropriate-or-offensive-content-what-does-that-mean"></a>He intentado cargar un vídeo como público y se ha marcado como contenido inadecuado u ofensivo, ¿qué significa eso? 

Al cargar un vídeo en Video Analyzer for Media, los algoritmos y los modelos realizan un análisis de contenido automático con el fin de asegurarse de que no se presente públicamente ningún contenido inadecuado. Si se detecta que un vídeo es sospechoso de tener contenido explícito, no es posible establecerlo como público. Pero los miembros de la cuenta pueden acceder a él como un vídeo privado (verlo, descargar la información y los artefactos extraídos, y realizar otras operaciones disponibles para los miembros de la cuenta).   

Para establecer el vídeo para acceso público, puede: 

* Crear su propia capa de interfaz (como aplicación o sitio web) y usarla para interactuar con el servicio Video Analyzer for Media. De este modo, el vídeo sigue siendo privado en el portal y los usuarios pueden interactuar con él a través de su interfaz. Por ejemplo, puede seguir obteniendo la información o permitir la visualización del vídeo en su propia interfaz. 
* Solicitar una revisión humana del contenido, lo que daría lugar a la eliminación de la restricción siempre que el contenido no sea explícito. 

    Esta opción se puede contemplar si el sitio web de Video Analyzer for Media es usado directamente por los usuarios como la capa de interfaz y para visualización pública (sin autenticar). 

## <a name="api-questions"></a>Preguntas sobre la API

### <a name="what-apis-does-video-analyzer-for-media-offer"></a>¿Qué API ofrece Video Analyzer for Media?

Las API de Video Analyzer for Media permiten indexar, extraer metadatos, administrar recursos, traducir, insertar, personalizar modelos, y mucho más. Para información más detallada sobre el uso de Video Analyzer for Media API, consulte [Video Analyzer for Media Developer Portal](https://api-portal.videoindexer.ai/) (Portal para desarrolladores de Video Analyzer for Media).

### <a name="what-client-sdks-does-video-analyzer-for-media-offer"></a>¿Qué SDK de cliente ofrece Video Analyzer for Media?

Actualmente no se ofrecen SDK de cliente. El equipo de Video Analyzer for Media está trabajando en los SDK y los planes para entregarlos pronto.

### <a name="how-do-i-get-started-with-video-analyzer-for-medias-api"></a>¿Cómo empiezo a trabajar con la API de Video Analyzer for Media?

Siga [Tutorial: get started with the Video Analyzer for Media API](video-indexer-use-apis.md) (Tutorial: introducción a la API de Video Analyzer for Media).

### <a name="what-is-the-difference-between-the-video-analyzer-for-media-api-and-the-azure-media-service-v3-api"></a>¿Cuál es la diferencia entre Video Analyzer for Media API y Azure Media Services v3 API?

Actualmente algunas de las características de Video Analyzer for Media API y Azure Media Services v3 API se superponen. Más información sobre la comparación de ambos servicios [aquí](compare-video-indexer-with-media-services-presets.md).

### <a name="what-is-an-api-access-token-and-why-do-i-need-it"></a>¿Qué es un token de acceso de API y por qué lo necesito?

Video Analyzer for Media API contiene una API de autorización y una API de operaciones. Authorizations API contiene las llamadas que ofrecen el token de acceso. Cada llamada a Operations API debe asociarse con un token de acceso, correspondiente al ámbito de autorización de la llamada.

Los tokens de acceso son necesarios para usar Video Analyzer for Media API con fines de seguridad. Esto garantiza que todas las llamadas proceden de usted o los usuarios que tengan permisos de acceso a su cuenta. 

### <a name="what-is-the-difference-between-account-access-token-user-access-token-and-video-access-token"></a>¿Cuál es la diferencia entre el token de acceso de cuenta, el token de acceso de usuario y el token de acceso de vídeo?

* Nivel de cuenta: los tokens de acceso de nivel de cuenta permiten realizar operaciones en el nivel de cuenta o de vídeo. Por ejemplo, cargar un vídeo, enumerar todos los vídeos u obtener información detallada de un vídeo.
* Nivel de usuario: los tokens de acceso de nivel de usuario permiten realizar operaciones en el nivel de usuario. Por ejemplo, obtener cuentas asociadas.
* Nivel de vídeo: los tokens de acceso de nivel de vídeo permiten realizar operaciones en el nivel de vídeo. Por ejemplo, obtener información detallada del vídeo, descargar títulos, obtener widgets, etc.

### <a name="how-often-do-i-need-to-get-a-new-access-token-when-do-access-tokens-expire"></a>¿Con qué frecuencia es necesario obtener un nuevo token de acceso? ¿Cuándo expiran los tokens de acceso?

Los tokens de acceso expiran cada hora, por lo que tendrá que generar un nuevo token de acceso cada hora. 

### <a name="what-are-the-login-options-to-video-analyzer-for-media-developer-portal"></a>¿Cuáles son las opciones de inicio de sesión para el portal para desarrolladores de Video Analyzer for Media?

Vea una nota de la versión relacionada con la [información de inicio de sesión](release-notes.md#october-2020).

Una vez que haya registrado su cuenta de correo electrónico mediante un proveedor de identidades, no podrá usarla con otro proveedor de identidades.

## <a name="billing-questions"></a>Preguntas sobre facturación

### <a name="how-much-does-video-analyzer-for-media-cost"></a>¿Cuánto cuesta Video Analyzer for Media?

Video Analyzer for Media usa un sencillo modelo de precios de pago por uso según la duración de entrada del contenido que se indexa. Es posible que se apliquen cargos adicionales para la codificación, el streaming, el almacenamiento, el uso de la red y las unidades reservadas multimedia. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cognitive-services/video-indexer/)para más información.

### <a name="when-am-i-billed-for-using-video-analyzer-for-media"></a>¿Cuándo se me factura el uso de Video Analyzer for Media?

Al enviar un vídeo para a indexarlo, el usuario define la indexación de modo que sea análisis de vídeo, de audio o ambos. Esto determina la SKU que se va a cobrar. Si se produce un error de nivel crítico durante el procesamiento, se devuelve un código de error como respuesta. En este caso, no se realiza facturación.  Un error crítico puede provocarlo un error en nuestro código o un fallo crítico en una dependencia interna del servicio. Errores como una incorrecta identificación o la extracción de conclusiones no se consideran críticos y se devuelve una respuesta. En los casos en los que se devuelva una respuesta válida (código sin errores), se realiza facturación.
 
### <a name="does-video-analyzer-for-media-offer-a-free-trial"></a>¿Ofrece Video Analyzer for Media una evaluación gratuita?

Sí, Video Analyzer for Media ofrece una evaluación gratuita que ofrece un servicio completo y la funcionalidad de API. Hay una cuota de 600 minutos de vídeos para los usuarios de la interfaz web y de 2400 minutos para los usuarios de la API. 

## <a name="next-steps"></a>Pasos siguientes

* [Información general](video-indexer-overview.md)
* [Stack Overflow](https://stackoverflow.com/search?q=video-indexer)
