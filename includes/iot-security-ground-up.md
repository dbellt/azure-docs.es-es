---
title: archivo de inclusión
description: archivo de inclusión
services: iot-fundamentals
author: robinsh
ms.service: iot-fundamentals
ms.topic: include
ms.date: 04/24/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 234407e6973657ba3ad0d78189e7cb1d363c15e2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95557749"
---
Internet de las cosas (IoT) plantea desafíos únicos para la seguridad, la privacidad y el cumplimiento a empresas de todo el mundo. A diferencia de la tecnología cibernética tradicional donde estos problemas giran en torno al software y cómo se implementa, a IoT le preocupa lo que sucede cuando los mundos cibernético y físico convergen. Proteger las soluciones IoT requiere garantizar el aprovisionamiento seguro de los dispositivos, proteger la conectividad entre estos dispositivos y la nube y garantizar la protección de los datos en la nube durante su procesamiento y almacenamiento. Sin embargo, trabajar con esta funcionalidad tiene sus inconvenientes: la limitación de recursos de los dispositivos, la distribución geográfica de las implementaciones y la existencia de un gran número de dispositivos dentro de una solución.

En este artículo se indica la manera en que los aceleradores de soluciones de IoT proporcionan una solución de nube de Internet de las cosas privada y segura. Los aceleradores de soluciones ofrecen una solución completa, donde la seguridad es una parte integral de cada etapa desde el principio. En Microsoft, desarrollar software seguro forma parte de la práctica de ingeniería de software, arraigada en décadas de experiencia de Microsoft en el desarrollo de software seguro. Para garantizar esto, la metodología de desarrollo fundamental es el Ciclo de vida de desarrollo de seguridad (SDL), junto con un conjunto de servicios de seguridad de nivel de infraestructura, como Garantía de la seguridad operacional (OSA), la unidad de delitos digitales de Microsoft, el Centro de respuestas de seguridad de Microsoft y el Centro de protección contra malware de Microsoft.

Los aceleradores de soluciones ofrecen características únicas para aprovisionar los dispositivos de IoT, conectarse a ellos y almacenar sus datos de manera fácil, transparente y, principalmente, segura. En este artículo se examinan las características de seguridad y las estrategias de implementación de los aceleradores de soluciones de Azure IoT, para garantizar que se abordan los desafíos relativos a la seguridad, la privacidad y el cumplimiento de objetivos.

## <a name="introduction"></a>Introducción

Internet de las cosas (IoT) es la generación del futuro al ofrecer a las empresas oportunidades inmediatas y reales para reducir los costos, aumentar los ingresos y transformar el negocio. Muchas empresas, sin embargo, tienen dudas sobre si implementar IoT en sus organizaciones, ya que les preocupa la seguridad, la privacidad y el cumplimiento. Una de las principales preocupaciones proviene del carácter único de la infraestructura IoT; al combinarse los mundos cibernético y físico, se agravan los riegos individuales inherentes a estos dos mundos. La seguridad de IoT tiene que ver con garantizar la integridad del código que se ejecuta en los dispositivos, proporcionar autenticación de dispositivos y usuarios, definir la propiedad clara de los dispositivos (así como de los datos generados por ellos) y resistir a los ataques cibernéticos y físicos.

Luego está el problema de la privacidad. Las empresas quieren transparencia en la recopilación de datos: qué se recopila y por qué, quién puede verlo, quién controla el acceso, etc. Por último, existen problemas de seguridad generales relativos al equipamiento y las personas que lo operan, y problemas de mantenimiento de los estándares de cumplimiento del sector.

A la vista de estos problemas de seguridad, privacidad, transparencia y cumplimiento, elegir el proveedor adecuado de la solución IoT sigue siendo un desafío. La unión de partes individuales del software IoT y los servicios proporcionados por diversos proveedores introduce deficiencias en estos ámbitos que pueden ser difíciles de detectar, por no hablar de solucionar. La elección del proveedor de software y servicios IoT adecuado se basa en dar con aquellos que tengan una gran experiencia en la ejecución de servicios que se distribuyen entre sectores verticales y geografías, pero que también sean capaces de escalar de forma segura y transparente. Igualmente, ayuda que el proveedor seleccionado tenga décadas de experiencia en el desarrollo de software seguro que se ejecuta en miles de millones de máquinas de todo el mundo y que sea capaz de apreciar el panorama de amenazas que plantea este nuevo mundo de Internet de las cosas.

## <a name="secure-infrastructure-from-the-ground-up"></a>Infraestructura segura desde el principio

La infraestructura de [Microsoft Cloud](https://azure.microsoft.com) tiene cabida para más de mil millones de clientes de 127 países o regiones. Gracias a las décadas de experiencia de Microsoft en la creación de software empresarial y en la ejecución de algunos de los servicios en línea más grandes del mundo, Microsoft Cloud proporciona mayores niveles de seguridad mejorada, privacidad, cumplimiento y prácticas de reducción de amenazas que los que la mayoría de los clientes podrían conseguir por su cuenta.

El [Ciclo de vida de desarrollo de seguridad (SDL)](https://www.microsoft.com/sdl/) de Microsoft proporciona un proceso de desarrollo obligatorio en toda la empresa que incorpora los requisitos de seguridad en el ciclo de vida entero del software. Para tener la seguridad de que las actividades operativas siguen prácticas de seguridad del mismo nivel, SDL usa rigurosas directrices de seguridad basadas en el proceso de Garantía de la seguridad operacional (OSA) de Microsoft. Microsoft también trabaja con empresas de auditoría de terceros para comprobar sistemáticamente que cumple con las obligaciones normativas y realiza esfuerzos por mejorar la seguridad a través de la creación de centros de excelencia, como la unidad de delitos digitales de Microsoft, el Centro de respuestas de seguridad de Microsoft y el Centro de protección contra malware de Microsoft.

## <a name="microsoft-azure---secure-iot-infrastructure-for-your-business"></a>Microsoft Azure: infraestructura de IoT segura para su negocio

Microsoft Azure ofrece una completa solución de nube, que combina una colección de servicios en la nube integrados en constante crecimiento: análisis, aprendizaje automático, almacenamiento, seguridad, redes y web, con un compromiso líder del sector a la protección y la privacidad de sus datos. La estrategia de [presunción de vulneraciones](https://azure.microsoft.com/blog/red-teaming-using-cutting-edge-threat-simulation-to-harden-the-microsoft-enterprise-cloud/) de Microsoft usa un *equipo rojo* dedicado de expertos de seguridad de software que simulan ataques con el fin de probar la capacidad de Azure para detectar las amenazas emergentes, protegerse frente a ellas y recuperarse de las vulneraciones. El equipo de [respuesta a incidentes globales](https://www.microsoft.com/en-us/TrustCenter/Security/DesignOpSecurity) de Microsoft trabaja continuamente para mitigar los efectos de ataques y actividades malintencionadas. El equipo sigue procedimientos establecidos para la administración, la comunicación y la recuperación de incidentes, y emplea interfaces detectables y predecibles con asociados internos y externos.

Los sistemas de Microsoft proporcionan continua detección y prevención de intrusiones, prevención de ataques de servicio, pruebas de penetración periódicas y herramientas forenses que ayudan a identificar y reducir las amenazas. [autenticación multifactor](../articles/active-directory/authentication/concept-mfa-howitworks.md) proporciona a los usuarios finales una capa de seguridad extra para el acceso a la red. Y para la aplicación y el proveedor de host, Microsoft ofrece control de acceso, supervisión, antimalware, examen de vulnerabilidades, revisiones y administración de configuración.

Los aceleradores de soluciones aprovechan la seguridad y la privacidad integradas en la plataforma de Azure, junto con los procesos SDL y OSA, para garantizar el desarrollo y el funcionamiento seguros de todo el software de Microsoft. Estos procedimientos proporcionan protección de infraestructura, protección de red y características de administración de identidades fundamentales para la seguridad de cualquier solución.

La instancia de [Azure IoT Hub](../articles/iot-hub/about-iot-hub.md) que se encuentra dentro de los [aceleradores de soluciones de IoT](../articles/iot-fundamentals/iot-introduction.md) ofrece un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre dispositivos de IoT y servicios de Azure, como [Azure Machine Learning](../articles/machine-learning/classic/index.yml) y [Azure Stream Analytics](../articles/stream-analytics/stream-analytics-introduction.md), mediante el uso de credenciales de seguridad y el control de acceso en cada dispositivo.

Para comunicar mejor las características de privacidad y seguridad integradas en los aceleradores de soluciones de Azure IoT, en este artículo se divide el conjunto de aplicaciones en tres áreas de seguridad principales.

![Aceleradores de soluciones de IoT de Azure](media/iot-security-ground-up/securing-iot-ground-up-fig3.png)

### <a name="secure-device-provisioning-and-authentication"></a>Seguridad en el aprovisionamiento y la autenticación de dispositivos

Azure IoT Suite protege los dispositivos mientras se usan para trabajar; para ello, proporciona una clave de identidad única a cada uno, que se puede usar en la infraestructura de IoT para comunicarse con el dispositivo mientras está en funcionamiento. El proceso es rápido y fácil de configurar. La clave generada con un identificador de dispositivo seleccionado por el usuario constituye la base de un token que se utiliza en toda la comunicación entre el dispositivo y Azure IoT Hub.

Los identificadores de dispositivo se puede asociar a un dispositivo durante la fabricación (es decir, se vacían en un módulo de confianza de hardware) o pueden usar una identidad fija existente como proxy (por ejemplo, los números de serie de CPU). Dado que no resulta fácil cambiar esta información de identificación en el dispositivo, es importante introducir identificadores de dispositivos lógicos en caso de que el hardware del dispositivo subyacente cambie pero el dispositivo lógico sea el mismo. En algunos casos, la asociación de una identidad de dispositivo puede tener lugar en el momento de la implementación del dispositivo (por ejemplo, un ingeniero de campo autenticado configura físicamente un nuevo dispositivo mientras se comunica con el back-end de la solución). El [Registro de identidades de Azure IoT Hub](../articles/iot-hub/iot-hub-devguide.md) proporciona un almacenamiento seguro de las identidades y las claves de seguridad de los dispositivos de una solución. Se pueden agregar identidades de dispositivo individuales o en grupo a una lista de elementos permitidos o bloqueados, de forma que se tiene un completo control sobre el acceso a los dispositivos.

Las directivas de control de acceso de Azure IoT Hub en la nube permiten la activación y deshabilitación de cualquier identidad de dispositivo, de tal forma que proporcionan una manera de desasociar un dispositivo de una implementación de IoT cuando es necesario. Esta asociación y desasociación de dispositivos se basa en la identidad de cada dispositivo.

Entre las características de seguridad de dispositivo adicionales se incluyen:

* Los dispositivos no aceptan conexiones de red no solicitadas. Establecen todas las conexiones y rutas solo en modo saliente. Para que un dispositivo reciba comandos del back-end, dicho dispositivo debe iniciar una conexión para comprobar si hay comandos pendientes de procesar. Cuando una conexión entre el dispositivo y Azure IoT Hub se establece de forma segura, los mensajes desde la nube hasta el dispositivo y desde el dispositivo hasta la nube se pueden enviar de forma transparente.

* Los dispositivos solo se conectan a servicios conocidos con los que están emparejados, o con los que tienen rutas establecidas, como un centro de IoT de Azure.

* En la autenticación y la autorización de nivel de sistema se emplean las identidades de cada dispositivo, lo que hace que las credenciales y los permisos de acceso se puedan revocar casi de manera instantánea.

### <a name="secure-connectivity"></a>Conectividad segura

La durabilidad de la mensajería es una característica importante de cualquier solución IoT. La necesidad de proporcionar comandos o recibir datos de dispositivos de forma duradera viene respaldada por el hecho de que los dispositivos IoT están conectados a través de Internet o de otras redes similares que pueden ser poco confiables. Azure IoT Hub ofrece durabilidad de la mensajería entre la nube y los dispositivos gracias a un sistema de confirmaciones en respuesta a los mensajes. Se consigue durabilidad adicional mediante el almacenamiento en caché de los mensajes de IoT Hub durante siete días como máximo en el caso de los datos de telemetría y de dos días para los comandos.

La eficacia es un factor importante para garantizar la conservación de recursos y el funcionamiento en un entorno con recursos limitados. HTTPS (HTTP seguro), la versión segura estándar del sector del conocido protocolo HTTP, se admite en Azure IoT Hub, lo que permite una comunicación eficaz. Advanced Message Queuing Protocol (AMQP) y Message Queuing Telemetry Transport (MQTT), admitidos en Azure IoT Hub, están diseñados no solo para la eficacia en términos del uso de recursos, sino también para la entrega confiable de mensajes. 

La escalabilidad requiere la posibilidad de interoperar con una amplia variedad de dispositivos de forma segura. El Centro de IoT de Azure permite una conexión segura tanto a dispositivos habilitados para IP como no IP. Los dispositivos habilitados para IP se pueden conectar y comunicar directamente con Azure IoT Hub a través de una conexión segura. Los dispositivos no habilitados para IP están limitados por los recursos y solo se conectan mediante protocolos de comunicación de corta distancia, como Zwave, ZigBee y Bluetooth. Se usa una puerta de enlace de campo para agregar estos dispositivos y realizar la conversión del protocolo para permitir la comunicación bidireccional segura con la nube.

Entre las características de seguridad de conexión adicionales se incluyen:

* La ruta de comunicación entre los dispositivos y Azure IoT Hub, o entre las puertas de enlace y Azure IoT Hub, está protegida mediante el protocolo Seguridad de capa de transporte (TLS) estándar del sector, estando autenticado Azure IoT Hub mediante el protocolo X.509.

* Para proteger los dispositivos de conexiones entrantes no solicitadas, Azure IoT Hub no abre ninguna conexión con el dispositivo. El dispositivo inicia todas las conexiones.

* Azure IoT Hub almacena de forma duradera los mensajes para los dispositivos y espera a que el dispositivo se conecte. Estos comandos se almacenan durante dos días, lo que permite que los dispositivos se conecten de forma esporádica, por problemas de conectividad o alimentación, para recibir estos comandos. Azure IoT Hub mantiene una cola por dispositivo para cada dispositivo.

### <a name="secure-processing-and-storage-in-the-cloud"></a>Seguridad en el procesamiento y el almacenamiento en la nube

Los aceleradores de soluciones ayudan a mantener los datos seguros mediante el cifrado de las comunicaciones y el procesamiento de los datos en la nube. En este sentido, proporciona flexibilidad para implementar cifrado adicional y administración de las claves de seguridad.

Al usar Azure Active Directory (AAD) para la autorización y la autenticación de los usuarios, los aceleradores de soluciones de Azure IoT pueden proporcionar un modelo de autorización basado en directivas para los datos de la nube, lo que permite recurrir a una administración de fácil acceso que se puede auditar y revisar. Este modelo también permite la revocación casi instantánea del acceso a los datos en la nube y de los dispositivos conectados a los aceleradores de soluciones de Azure IoT.

Cuando los datos están en la nube, se pueden procesar y almacenar en cualquier flujo de trabajo definido por el usuario. El acceso a cada parte de los datos se controla con Azure Active Directory, dependiendo del servicio de almacenamiento utilizado.

Todas las claves usadas por la infraestructura IoT se almacenan en la nube en un almacenamiento seguro, con la posibilidad de sustituirlas en caso de que sea necesario volver a aprovisionarlas. Los datos se pueden almacenar en [Azure Cosmos DB](../articles/cosmos-db/introduction.md) o en [SQL Database](../articles/azure-sql/database/sql-database-paas-overview.md), lo que permite la definición del nivel de seguridad deseado. Además, Azure proporciona una manera de supervisar y auditar todo el acceso a los datos para avisarle de cualquier intrusión o acceso no autorizado.

## <a name="conclusion"></a>Conclusión

Internet de las cosas comienza con sus cosas: las cosas que más importan a las empresas. IoT puede proporciona un valor increíble a las empresas al ayudarles a reducir los costos, aumentar los ingresos y transformar el negocio. El éxito de esta transformación depende en gran medida de la elección del proveedor de servicios y software IoT correcto. Eso significa encontrar un proveedor que no solo comprenda las necesidades y los requisitos de la empresa para hacer posible esta transformación, sino que también proporcione servicios y software en los que la seguridad, la privacidad, la transparencia y el cumplimiento integrados sean las consideraciones de diseño principales. Microsoft posee una amplia experiencia en el desarrollo y la implementación de servicios y software seguros y sigue siendo uno de los líderes en esta nueva era de Internet de las cosas.

Los aceleradores de soluciones incorporan medidas de seguridad en su diseño, lo que permite la supervisión segura de los recursos para mejorar la eficacia, impulsar el rendimiento funcional para posibilitar la innovación y emplear análisis de datos avanzados para transformar el negocio. Gracias a su enfoque dedicado a la seguridad basada en capas, sus diversas características de seguridad y sus patrones de diseño, los aceleradores de soluciones le ayudarán a implementar una infraestructura en la que se pueda confiar para transformar cualquier negocio.

## <a name="additional-information"></a>Información adicional

Cada acelerador de soluciones crea instancias de los servicios de Azure, tales como:

* [**Azure IoT Hub**](https://azure.microsoft.com/services/iot-hub/): la puerta de enlace que conecta la nube con los dispositivos. Se puede escalar a millones de conexiones por centro y procesar volúmenes masivos de datos gracias a la compatibilidad con la autenticación individual de cada dispositivo, lo que ayuda a proteger la solución.

* [**Azure Cosmos DB**](https://azure.microsoft.com/services/cosmos-db/): servicio de base de datos completamente indexado y escalable destinado a datos semiestructurados que administra los metadatos de los dispositivos aprovisionados, como los atributos, la configuración y las propiedades de seguridad. Azure Cosmos DB ofrece procesamiento de alto rendimiento, indexación de datos independiente del esquema y una completa interfaz de consultas SQL.

* [**Azure Stream Analytics**](https://azure.microsoft.com/services/stream-analytics/): procesamiento de transmisiones en tiempo real en la nube que permite desarrollar e implementar rápidamente una solución de análisis de bajo costo para desvelar datos detallados en tiempo real de dispositivos, sensores, infraestructura y aplicaciones. Los datos de este servicio completamente administrado se pueden escalar a cualquier volumen, sin dejar de obtener alto rendimiento, baja latencia y resistencia.

* [**Azure App Services**](https://azure.microsoft.com/services/app-service/): plataforma en la nube que sirve para crear sólidas aplicaciones web y móviles desde las que se puede obtener acceso a datos desde cualquier lugar, ya sea en la nube o desde la infraestructura local. Cree atractivas aplicaciones móviles para iOS, Android y Windows. Realice integraciones con aplicaciones de Software como servicio (SaaS) y empresariales gracias a la conectividad de serie a docenas de aplicaciones empresariales y servicios basados en la nube. Codifique en su lenguaje e IDE (.NET, Node.js, PHP, Python o Java) preferidos para crear aplicaciones web y API con más rapidez que nunca.

* [**Logic Apps**](https://azure.microsoft.com/services/app-service/logic/): la característica Logic Apps de Azure App Service sirve para integrar la solución de IoT con los sistemas de línea de negocio existentes y para automatizar los procesos de flujo de trabajo. Logic Apps permite a los desarrolladores diseñar flujos de trabajo que se inician desde un desencadenador y ejecutan luego una serie de pasos: reglas y acciones que usan eficaces conectores para la integración con los procesos de negocio. Logic Apps ofrece conectividad de serie a un gran ecosistema de aplicaciones SaaS, basadas en la nube y locales.

* [**Azure Blob Storage**](https://azure.microsoft.com/services/storage/): almacenamiento en la nube confiable y económico para los datos que los dispositivos envían a la nube.