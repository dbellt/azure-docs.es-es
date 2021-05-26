---
title: Comparación de colas de Azure Storage con colas de Service Bus
description: Analiza las diferencias y similitudes entre dos tipos de colas que se ofrecen en Azure.
ms.topic: article
ms.date: 04/12/2021
ms.openlocfilehash: a63ed276981a692b96b7e7369093af221b3d3a5f
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190133"
---
# <a name="storage-queues-and-service-bus-queues---compared-and-contrasted"></a>Colas de Storage y de Service Bus: comparación y diferencias
En este artículo se analizan las diferencias y similitudes entre los dos tipos de cola que ofrece Microsoft Azure: las colas de Storage y las colas de Service Bus. Con esta información, puede tomar una decisión más fundamentada sobre la solución que mejor se adapta a sus necesidades.

## <a name="introduction"></a>Introducción
Azure admite dos tipos de mecanismos de cola: **colas de Storage** y **colas de Service Bus**.

Las **colas de Storage** forman parte de la infraestructura de [Azure Storage](https://azure.microsoft.com/services/storage/). Permiten almacenar un gran número de mensajes. a los que se puede acceder desde cualquier lugar del mundo a través de llamadas autenticadas mediante HTTP o HTTPS. Un mensaje de la cola puede llegar a tener hasta 64 KB. Una cola puede contener millones de mensajes, hasta el límite de capacidad total de una cuenta de almacenamiento. Las colas se utilizan normalmente para crear un trabajo pendiente del trabajo que se va a procesar de forma asincrónica. Para obtener más información, vea [¿Qué son las colas de Azure Storage?](../storage/queues/storage-queues-introduction.md)

Las **colas de Service Bus** forman parte de una infraestructura de [mensajería de Azure](https://azure.microsoft.com/services/service-bus/) más amplia que admite la puesta en cola, la publicación o suscripción y patrones de integración más avanzados. Están diseñadas para integrar aplicaciones o componentes de aplicaciones que pueden abarcar varios protocolos de comunicación, contratos de datos, dominios de confianza o entornos de red. Para obtener más información sobre las colas, los temas o las suscripciones de Service Bus, vea [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md).


## <a name="technology-selection-considerations"></a>Consideraciones de selección de tecnología
Las colas de Storage y Service Bus tienen un conjunto de características ligeramente diferente. Puede elegir una o las dos, en función de las necesidades de la solución concreta.

A la hora de determinar qué tecnología de cola se ajusta al propósito de una solución determinada, los desarrolladores y arquitectos de soluciones deben tener en cuenta estas recomendaciones. 

### <a name="consider-using-storage-queues"></a>Consideración del uso de colas de Storage
Como arquitecto o desarrollador de soluciones, **debe considerar el uso de colas de Storage** en los siguientes casos:

* La aplicación debe almacenar más de 80 gigabytes de mensajes en una cola.
* La aplicación quiere realizar un seguimiento del progreso del procesamiento de un mensaje de la cola. Esto es útil si se bloquea el trabajo que está procesando un mensaje. Otro trabajo puede usar esa información para continuar desde donde lo ha dejado el trabajo anterior.
* Necesita registros de lado de servidor de todas las transacciones ejecutadas con las colas.

### <a name="consider-using-service-bus-queues"></a>Consideración del uso de colas de Service Bus
Como arquitecto o desarrollador de soluciones, **debe considerar el uso de colas de Service Bus** cuando:

* La solución necesita recibir mensajes sin tener que sondear la cola. Con Service Bus, esto se logra mediante una operación de recepción de sondeo largo con los protocolos basados en TCP que admite Service Bus.
* Su solución requiere que la cola ofrezca una entrega ordenada por primero en entrar es el primero en salir (FIFO).
* La solución necesita admitir la detección automática de duplicados.
* Quiera que su aplicación procese mensajes como flujos de larga ejecución en paralelo (los mensajes se asocian a un flujo mediante la propiedad **Id. de sesión** del mensaje). En este modelo, cada nodo de la aplicación de consumo compite por secuencias, en lugar de los mensajes. Cuando se proporciona una secuencia a un nodo de consumo, el nodo puede examinar el estado de la secuencia de aplicación mediante transacciones.
* Su solución requiere un comportamiento transaccional y atomicidad al enviar o recibir varios mensajes desde una cola.
* La aplicación administra mensajes que pueden superar los 64 KB pero que probablemente no van a alcanzar el límite de 256 KB.
* Aborda un requisito para ofrecer un modelo de acceso basado en roles a las colas y diferentes derechos o permisos para los remitentes y receptores. Para más información, consulte los siguientes artículos.
    - [Autenticación con identidades administradas](service-bus-managed-service-identity.md)
    - [Autenticación desde una aplicación](authenticate-application.md)
* El tamaño de la cola no va a aumentar a más de 80 GB.
* Quiere usar el protocolo de mensajería basado en estándares AMQP 1.0. Para obtener más información sobre AMQP, vea [Introducción al AMQP de Service Bus](service-bus-amqp-overview.md).
* Prevé una posible migración de la comunicación de punto a punto basada en colas a un patrón de mensajería de publicación a suscripción. Este patrón permite la integración de receptores adicionales (suscriptores). Cada receptor recibe copias independientes de algunos o todos los mensajes enviados a la cola. 
* La solución de mensajería debe admitir la garantía de entrega "Una vez como máximo" sin necesidad de compilar los componentes de infraestructura adicionales.
* La solución necesita publicar y consumir lotes de mensajes.

## <a name="compare-storage-queues-and-service-bus-queues"></a>Comparación de colas de Storage con colas de Service Bus
Las tablas de las secciones siguientes proporcionan una agrupación lógica de características de cola. Permiten comparar, de un vistazo, las capacidades disponibles tanto en las colas de Azure Storage como en las colas de Service Bus.

## <a name="foundational-capabilities"></a>Capacidades fundamentales
En esta sección se comparan algunas de las funcionalidades de puesta en cola fundamentales ofrecidas por las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de Storage | Colas de Service Bus |
| --- | --- | --- |
| Garantía de ordenación |**No** <br/><br>Para obtener más información, vea la primera nota de la sección [Información adicional](#additional-information).</br> | **Sí- Primero en caducar primero en salir (FIFO)**<br/><br>(mediante [sesiones de mensajes](message-sessions.md)) |
| Garantía de entrega |**Al menos una vez** |**Al menos una vez** (con el modo de recepción Ojear y bloquear: este es el valor predeterminado) <br/><br/>**Por lo menos un meme** (mediante el modo de recepción ReceiveAndDelete) <br/> <br/> Más información sobre los [distintos modos de recepción](service-bus-queues-topics-subscriptions.md#receive-modes).  |
| Compatibilidad con la operación atómica |**No** |**Sí**<br/><br/> |
| Comportamiento de recepción |**Sin bloqueo**<br/><br/>(se completa inmediatamente si no se encuentra ningún mensaje nuevo) |**Bloqueo con o sin un tiempo de espera**<br/><br/>(ofrece sondeo largo o ["Técnica de cometa"](https://go.microsoft.com/fwlink/?LinkId=613759))<br/><br/>**Sin bloqueo**<br/><br/>(solo con la API administrada de .NET) |
| API de estilo de inserción |**No** |**Sí**<br/><br/>Nuestros SDK de .NET, Java, JavaScript y Go proporcionan API de estilo de inserción. |
| Modo de recepción |**Ojear y alquilar** |**Ojear y bloquear**<br/><br/>**Recibir y eliminar** |
| Modo de acceso exclusivo |**Basado en concesión** |**Basado en bloqueo** |
| Duración de concesión/bloqueo |**30 segundos (valor predeterminado)**<br/><br/>**7 días (máximo)** (puede renovar o liberar una concesión de mensaje mediante la API de [UpdateMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.updatemessage)). |**30 segundos (valor predeterminado)**<br/><br/>Puede renovar el bloqueo de mensajes para la misma duración de bloqueo cada vez manualmente o usar la característica de renovación automática de bloqueos donde el cliente administra automáticamente la renovación de bloqueos. |
| Precisión de concesión/bloqueo |**Nivel de mensaje**<br/><br/>Cada mensaje puede tener un valor de tiempo de espera diferente, que luego se puede actualizar según sea necesario al procesar el mensaje, mediante la API [UpdateMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.updatemessage). |**Nivel de cola**<br/><br/>(cada cola tiene aplicada una precisión de bloqueo a todos sus mensajes, pero el bloqueo se puede renovar como se describe en la fila anterior) |
| Recepción por lotes |**Sí**<br/><br/>(especificando explícitamente el número de mensajes al recuperar mensajes, hasta un máximo de 32 mensajes) |**Sí**<br/><br/>(habilitando implícitamente una propiedad de captura previa o explícitamente mediante el uso de transacciones) |
| Envío por lotes |**No** |**Sí**<br/><br/>(mediante transacciones o procesamiento por lotes del lado cliente) |

### <a name="additional-information"></a>Información adicional
* Los mensajes de las colas de Storage son normalmente del tipo "primero en entrar, primero en salir" (FIFO), pero en ocasiones pueden estar desordenados. Por ejemplo, si expira la duración del tiempo de espera de visibilidad de un mensaje porque una aplicación cliente se ha bloqueado al procesar un mensaje. Cuando expira el tiempo de espera de visibilidad, el mensaje se vuelve visible en la cola para que otro trabajador lo quite de la cola. En ese momento, el mensaje que acaba de hacerse visible se puede colocar en la cola (para quitarlo después).
* El patrón de FIFO garantizado en las colas de Service Bus requiere el uso de sesiones de mensajería. Si la aplicación se bloquea mientras está procesando un mensaje recibido en el modo **Ojear y bloquear**, la siguiente vez que un receptor de la cola acepta una sesión de mensajería, empieza con el mensaje erróneo después de que expire el período de vida (TTL) del mensaje.
* Las colas de Storage están diseñadas para admitir escenarios estándar de puesta en cola, por ejemplo:
    - Desacoplamiento de componentes de aplicación para aumentar la escalabilidad y la tolerancia a errores
    - Redistribución de la carga
    - Compilación de flujos de trabajo de proceso.
* Se pueden evitar incoherencias con respecto al control de mensajes en el contexto de las sesiones de Service Bus al usar el estado de sesión para almacenar el estado de la aplicación relativo al progreso del control de la secuencia de mensajes de la sesión y mediante el uso de transacciones sobre la fijación de los mensajes recibidos y la actualización del estado de la sesión. Este tipo de característica de coherencia a veces se etiqueta como *procesamiento Una sola vez* en los productos de otros proveedores. Los errores de transacción obviamente hacen que se vuelvan a entregar los mensajes, por lo que el término no es del todo adecuado.
* Las colas de Storage ofrecen un modelo de programación coherente y uniforme en las colas, tablas y blobs, tanto para desarrolladores como para los equipos de operaciones.
* Las colas de Service Bus ofrecen compatibilidad con transacciones locales en el contexto de una sola cola.
* El modo **Recibir y eliminar** compatible con el Service Bus ofrece la capacidad de reducir el número de operaciones de mensajería (y el costo asociado) a cambio de una garantía de entrega menor.
* Las colas de Storage ofrecen concesiones con la posibilidad de ampliar las concesiones para mensajes. Esta característica permite que los procesos de trabajo mantengan breves concesiones sobre los mensajes. Por lo tanto, si se bloquea un trabajo, otro puede volver a procesar rápidamente el mensaje. Además, un trabajo puede ampliar la concesión sobre un mensaje si necesita procesarlo durante más tiempo que el de concesión actual.
* Las colas de Storage ofrecen un tiempo de espera de visibilidad que puede establecer al poner en cola un mensaje o quitarlo de la cola. Además, puede actualizar un mensaje con distintos valores de concesión en tiempo de ejecución y actualizar distintos valores en mensajes de la misma cola. Los tiempos de espera de bloqueo de Service Bus se definen en los metadatos de la cola. Sin embargo, puede renovar el bloqueo de mensajes para la duración de bloqueo predefinida manualmente o usar la característica de renovación automática de bloqueos donde el cliente administra automáticamente la renovación de bloqueos.
* El tiempo de espera máximo para una operación de recepción de bloqueo en las colas de Service Bus es de 24 días. Sin embargo, los tiempos de espera basados en REST tienen un valor máximo de 55 segundos.
* El procesamiento por lotes del cliente ofrecido por el Service Bus permite a un cliente de cola procesar varios mensajes por lotes en una sola operación de envío. El procesamiento por lotes solo está disponible para las operaciones de envío asincrónicas.
* Características como el límite de 200 TB de las colas de Storage (más cuando se virtualizan las cuentas) y las colas ilimitadas la convierten en la plataforma idónea para los proveedores de SaaS.
* Las colas de Storage ofrecen un mecanismo de control de acceso delegado flexible y eficiente.

## <a name="advanced-capabilities"></a>Capacidades avanzadas
En esta sección se comparan algunas de las funcionalidades avanzadas ofrecidas por las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de Storage | Colas de Service Bus |
| --- | --- | --- |
| Entrega programada |**Sí** |**Sí** |
| Mensajes fallidos automáticos |**No** |**Sí** |
| Aumentar el valor de período de vida de cola |**Sí**<br/><br/>(a través de la actualización local de tiempo de espera de visibilidad) |**Sí**<br/><br/>(proporcionado mediante una función de API dedicada) |
| Compatibilidad con mensajes dudosos |**Sí** |**Sí** |
| Actualización local |**Sí** |**Sí** |
| Registro de transacciones del servidor |**Sí** |**No** |
| Métricas de almacenamiento |**Sí**<br/><br/>**Métricas por minuto** proporciona métricas en tiempo real de disponibilidad, TPS, recuentos de llamadas API, recuentos de errores y mucho más. Todas ellas son en tiempo real, se agregan por minuto y se muestran pocos minutos después de lo que ha ocurrido en producción. Para obtener más información, vea [Acerca de las métricas de Storage Analytics](/rest/api/storageservices/fileservices/About-Storage-Analytics-Metrics). |**Sí**<br/><br/>Para información sobre las métricas admitidas por Azure Service Bus, consulte [Métricas de mensajes](monitor-service-bus-reference.md#message-metrics). |
| Administración de estados |**No** |**Sí** (Active, Disabled, SendDisabled, ReceiveDisabled. Para más información sobre estos estados, vea [Estado de la cola](entity-suspend.md#queue-status)). |
| Reenvío automático de mensajes |**No** |**Sí** |
| Purgar la función de cola |**Sí** |**No** |
| Grupos de mensajes |**No** |**Sí**<br/><br/>(mediante sesiones de mensajería) |
| Estado de la aplicación por grupo de mensajes |**No** |**Sí** |
| Detección de duplicados |**No** |**Sí**<br/><br/>(configurable en el lado del remitente) |
| Exploración de grupos de mensaje |**No** |**Sí** |
| Captura de sesiones de mensajes por id. |**No** |**Sí** |

### <a name="additional-information"></a>Información adicional
* Ambas tecnologías de cola permiten que se programe un mensaje para su entrega posteriormente.
* El reenvío automático de colas permite el reenvío automático por parte de miles de colas de sus mensajes a una única cola, desde la que la aplicación receptora consume el mensaje. Puede usar este mecanismo para lograr seguridad, flujo de control y aislar el almacenamiento aislado entre cada publicador de mensajes.
* Las colas de Storage ofrecen compatibilidad para actualizar el contenido del mensaje. Puede usar esta funcionalidad para conservar información de estado y actualizaciones incrementales de progreso en el mensaje para que se pueda procesar desde el último punto de comprobación conocido, en lugar de hacerlo desde el principio. Con las colas de Service Bus, puede habilitar el mismo escenario mediante el uso de sesiones de mensajes. Para más información, consulte [Estado de la sesión de mensajes](message-sessions.md#message-session-state).
* Las colas de Service Bus admiten [mensajes fallidos](service-bus-dead-letter-queues.md). Esto puede ser útil para aislar los mensajes que cumplen los criterios siguientes:
    - La aplicación receptora no puede procesar los mensajes correctamente. 
    - Los mensajes no pueden llegar a su destino debido a una propiedad de período de vida (TTL) expirada. El valor de TTL especifica cuánto tiempo permanece un mensaje en la cola. Con Service Bus, el mensaje se moverá a una cola especial denominada $DeadLetterQueue cuando expira el período de vida.
* Para encontrar mensajes "dudosos" en las colas de Storage, al quitar de la cola un mensaje de la aplicación se examina la propiedad [DequeueCount](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage.dequeuecount) del mensaje. Si **DequeueCount** se encuentra por encima de un umbral determinado, la aplicación mueve el mensaje a una cola de proceso como correo devuelto definida por la aplicación.
* Las colas de Storage permiten obtener un registro detallado de todas las transacciones ejecutadas en la cola y de las métricas agregadas. Ambas opciones son útiles para depurar y entender cómo usa su aplicación las colas de Storage. También son útiles para ajustar el rendimiento de la aplicación y reducir los costos del empleo de colas.
* Las [sesiones de mensajes](message-sessions.md) que admite Service Bus permiten asociar los mensajes que pertenecen a un grupo lógico a un receptor. Eso crea una afinidad de tipo sesión entre los mensajes y sus destinatarios respectivos. Puede habilitar esta funcionalidad avanzada en Service Bus estableciendo la propiedad de id. de sesión en un mensaje. Los receptores pueden escuchar entonces en un id. de sesión específico y recibir mensajes que comparten el identificador de sesión especificado.
* La característica de detección de duplicación de colas de Service Bus elimina automáticamente los mensajes duplicados enviados a una cola o un tema, según el valor de la propiedad de id. de mensaje.

## <a name="capacity-and-quotas"></a>Capacidad y cuotas
En esta sección se comparan las colas de Storage y las colas de Service Bus desde la perspectiva de la [capacidad y las cuotas](service-bus-quotas.md) que se pueden aplicar.

| Criterios de comparación | Colas de Storage | Colas de Service Bus |
| --- | --- | --- |
| Tamaño de cola máximo |**500 TB**<br/><br/>(limitado a una [capacidad de cuenta de almacenamiento única](../storage/common/storage-introduction.md#queue-storage)) |**De 1 GB a 80 GB**<br/><br/>(definido al crear una cola y [habilitar particiones](service-bus-partitioning.md): vea la sección "Información adicional") |
| Tamaño de mensaje máximo |**64 KB**<br/><br/>(48 K cuando se usa la codificación **Base64**)<br/><br/>Azure admite mensajes de gran tamaño mediante la combinación de colas y blobs, momento en el que puede poner en cola hasta 200 GB para un solo elemento. |**256 KB** o **1 MB**<br/><br/>(incluidos tanto el encabezado como el cuerpo, el tamaño máximo de encabezado es: 64 KB).<br/><br/>Depende del [nivel de servicio](service-bus-premium-messaging.md). |
| TTL de mensaje máximo |**Infinito** (api-version 2017-07-27 o posterior) |**TimeSpan.Max** |
| Número máximo de colas |**Sin límite** |**10.000**<br/><br/>(por espacio de nombres de servicio) |
| Número máximo de clientes simultáneos |**Sin límite** |**5\.000** |

### <a name="additional-information"></a>Información adicional
* Service Bus aplica límites de tamaño de cola. El tamaño máximo de cola se especifica al crear una cola. Puede ser de entre 1 GB y 80 GB. Si el tamaño de la cola alcanza este límite, se rechazan los mensajes entrantes adicionales y el autor de la llamada recibe una excepción. Para obtener más información sobre las cuotas en el Service Bus, vea [Cuotas de Service Bus](service-bus-quotas.md).
* No se admite la creación de particiones en el [nivel Premium](service-bus-premium-messaging.md). En el nivel de mensajería Estándar, puede crear colas y temas de Service Bus de 1 (valor predeterminado), 2, 3, 4 o 5 GB. Con las particiones habilitadas, Service Bus crea 16 copias (16 particiones) de la entidad, cada una del mismo tamaño especificado. Por lo tanto, si crea una cola con un tamaño de 5 GB y 16 particiones, el tamaño de cola máximo se convierte en (5 * 16) = 80 GB.  Puede ver el tamaño máximo de la cola o el tema con particiones en [Azure Portal][Azure portal].
* Con las colas de Storage, si el contenido del mensaje no es seguro para XML, debe estar codificado en **Base64**. Si codifica el mensaje con **Base64**, la carga de usuario puede ser de hasta 48 KB, en lugar de 64 KB.
* Con las colas de Service Bus, cada mensaje almacenado en una cola consta de dos partes: un encabezado y un cuerpo. El tamaño total del mensaje no puede superar el tamaño máximo admitido por el nivel de servicio.
* Cuando los clientes se comunican con colas de Service Bus por el protocolo TCP, el número máximo de conexiones simultáneas a una única cola de Service Bus se limita a 100. Este número se comparte entre remitentes y receptores. Si se alcanza esta cuota, se rechazan las solicitudes de conexiones adicionales y el código de llamada recibe una excepción. Este límite no se impone en clientes que se conectan a las colas mediante la API basada en REST.
* Si necesita más de 10.000 colas en un único espacio de nombres de Service Bus, puede ponerse en contacto con el equipo de soporte técnico de Azure y solicitar un aumento. Para escalar más allá de las 10.000 colas con Service Bus, también puede crear espacios de nombres adicionales mediante [Azure Portal][Azure portal].

## <a name="management-and-operations"></a>Administración y operaciones
En esta sección se comparan algunas de las características de administración ofrecidas por las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de Storage | Colas de Service Bus |
| --- | --- | --- |
| Protocolo de administración |**REST sobre HTTP/HTTPS** |**REST sobre HTTPS** |
| Protocolo de tiempo de ejecución |**REST sobre HTTP/HTTPS** |**REST sobre HTTPS**<br/><br/>**AMQP 1.0 estándar (TCP con TLS)** |
| API de .NET |**Sí**<br/><br/>(API de cliente de Storage para .NET) |**Sí**<br/><br/>(API de Service Bus para .NET) |
| C++ nativo |**Sí** |**Sí** |
| API de Java |**Sí** |**Sí** |
| API de PHP |**Sí** |**Sí** |
| API de Node.js. |**Sí** |**Sí** |
| Compatibilidad con metadatos arbitrarios |**Sí** |**No** |
| Reglas de nomenclatura de cola |**Hasta 63 caracteres**<br/><br/>(las letras de un nombre de cola deben estar en minúscula). |**Hasta 260 caracteres**<br/><br/>(los nombres y las rutas de acceso de las colas no distinguen mayúsculas de minúsculas). |
| Función de obtención de la longitud de la cola |**Sí**<br/><br/>(valor aproximado si los mensajes expiran más allá del TTL sin eliminarse). |**Sí**<br/><br/>(valor exacto en un momento dado). |
| Función de ojear |**Sí** |**Sí** |

### <a name="additional-information"></a>Información adicional
* Las colas de Storage ofrecen compatibilidad con atributos arbitrarios que se pueden aplicar a la descripción de la cola, en forma de pares de nombre-valor.
* Ambas tecnologías de cola ofrecen la capacidad de ojear un mensaje sin tener que bloquearlo, lo que puede resultar útil al implementar una herramienta de explorador de colas.
* Las API de mensajería asíncrona de .NET de Service Bus usan las conexiones TCP de dúplex completo para mejorar el rendimiento en comparación con REST a través de HTTP, y admiten el protocolo estándar AMQP 1.0.
* Los nombres de colas de Storage pueden tener de 3 a 63 caracteres de longitud que pueden incluir letras minúsculas, números y guiones. Para obtener más información, vea [Nomenclatura de colas y metadatos](/rest/api/storageservices/fileservices/Naming-Queues-and-Metadata).
* Los nombres de cola de Service Bus pueden tener hasta 260 caracteres y reglas de nomenclatura menos restrictivas. Los nombres de cola de Service Bus pueden contener letras, números, puntos, guiones y caracteres de subrayado.

## <a name="authentication-and-authorization"></a>Autenticación y autorización
En esta sección se describen las características de autenticación y autorización compatibles con las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de Storage | Colas de Service Bus |
| --- | --- | --- |
| Authentication |**Clave simétrica** |**Clave simétrica** |
| Modelo de seguridad |Acceso delegado a través de tokens SAS. |SAS |
| Federación de proveedor de identidad: |**Sí** |**Sí** |

### <a name="additional-information"></a>Información adicional
* Se debe autenticar cada solicitud a cualquiera de las tecnologías de cola. No se admiten colas públicas con acceso anónimo. Con [SAS](service-bus-sas.md), puede abordar este escenario publicando un SAS de solo escritura, un SAS de solo lectura o incluso un SAS de acceso completo.
* El esquema de autenticación proporcionado por las colas de Storage implica el uso de una clave simétrica. Esta clave es un código de autenticación de mensajes basado en hash (HMAC) que se calcula con el algoritmo SHA-256 y se codifica como una cadena de **Base64**. Para obtener más información sobre el protocolo respectivo, consulte [Autenticación para los servicios de Azure Storage](/rest/api/storageservices/fileservices/Authentication-for-the-Azure-Storage-Services). Las colas de Service Bus admiten un modelo similar mediante claves simétricas. Para obtener más información, vea [Autenticación con firma de acceso compartido con Service Bus](service-bus-sas.md).

## <a name="conclusion"></a>Conclusión
Una mejor comprensión de las dos tecnologías permite tomar una decisión más fundamentada sobre la tecnología de cola que se va a usar y cuándo. La decisión sobre cuándo usar las colas de Storage o las colas de Service Bus depende claramente de muchos factores. Estos factores pueden dependen en gran medida de las necesidades individuales de la aplicación y de su arquitectura. 

Quizás prefiera elegir las colas de Storage por motivos como los siguientes:

- Si la aplicación ya usa las capacidades básicas de Microsoft Azure 
- Si necesita comunicación básica y mensajería entre servicios 
- Si necesita colas que puedan tener un tamaño superior a 80 GB

Las colas de Service Bus proporcionan muchas características avanzadas, como las siguientes. Por lo tanto, pueden ser la opción preferida si va a compilar una aplicación híbrida o si la aplicación requiere estas características.

- [Sesiones](message-sessions.md)
- [Transactions](service-bus-transactions.md)
- [Detección de duplicados](duplicate-detection.md)
- [Mensajes fallidos automáticos](service-bus-dead-letter-queues.md)
- [Capacidades de publicación y suscripción duraderas](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) 



## <a name="next-steps"></a>Pasos siguientes
En los artículos siguientes se ofrece más orientación e información sobre el uso de las colas de Storage o las colas de Service Bus.

* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso del servicio Queue Storage](../storage/queues/storage-dotnet-how-to-use-queues.md)
* [Procedimientos recomendados para mejorar el rendimiento mediante la mensajería asincrónica de Service Bus](service-bus-performance-improvements.md)
* [Introducción a las colas y los temas de Azure Service Bus (publicación en un blog)](https://www.serverless360.com/blog/azure-service-bus-queues-vs-topics)
* [Guía para desarrolladores sobre el Service Bus](http://www.cloudcasts.net/devguide/Default.aspx?id=11030)
* [Uso del servicio de cola en Azure ](https://www.developerfusion.com/article/120197/using-the-queuing-service-in-windows-azure/)

[Azure portal]: https://portal.azure.com

