---
title: Introducción a Azure Cosmos DB
description: Información acerca de Azure Cosmos DB. Esta base de datos de varios modelos y distribución global se ha creado con latencia baja, escalabilidad elástica y alta disponibilidad, y ofrece compatibilidad nativa con datos de NoSQL.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: overview
ms.date: 05/07/2021
ms.openlocfilehash: f435ed599e39b2dd0c204ef8c2cdcbb23146bc08
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367874"
---
# <a name="welcome-to-azure-cosmos-db"></a>Bienvenido a Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Las aplicaciones actuales deben estar siempre en línea y tener una alta capacidad de respuesta. Para lograr baja latencia y alta disponibilidad, las instancias de estas aplicaciones deben implementarse en centros de datos que están cerca de sus usuarios. Es necesario que las aplicaciones respondan en tiempo real a grandes cambios, en cuanto a uso, en las horas punta, que almacenen volúmenes de datos que cada vez son mayores y que dichos datos estén a disposición de los usuarios en milisegundos.

Azure Cosmos DB es una base de datos NoSQL totalmente administrada para el desarrollo de aplicaciones modernas. Tiempos de respuesta de milisegundos de un solo dígito y la escalabilidad automática e instantánea garantizan la velocidad a cualquier escala. La continuidad empresarial está garantizada por un [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/cosmos-db) y seguridad de clase empresarial. El desarrollo de aplicaciones es más rápido y productivo gracias a la distribución de datos de varias regiones integrada en cualquier parte del mundo y las API y los SDK de código abierto para los lenguajes más populares. Como servicio totalmente administrado, Azure Cosmos DB le libera de la administración de bases de datos con administración, actualizaciones y revisiones automáticas. También controla la administración de la capacidad con opciones de escalado automático y sin servidor rentables que responden a las necesidades de la aplicación para hacer coincidir la capacidad con la demanda.

> [!NOTE]
> ¿Desea ayudar a mejorar los documentos de Azure Cosmos DB participando en un estudio de usuario? Dedique unos minutos a rellenar esta [encuesta de filtrado](https://aka.ms/cosmosdb-documentation-screener-survey) de 5 minutos. Si cumple con los requisitos, se le redirigirá a un programador en el que puede reservar un espacio para unirse a una sesión de investigación interactiva. Durante este proceso no se recopilan datos personales de conformidad con nuestra [declaración de privacidad](https://go.microsoft.com/fwlink/?LinkId=521839).

Puede [probar Azure Cosmos DB de forma gratuita](https://azure.microsoft.com/try/cosmosdb/) sin necesidad de una suscripción de Azure y sin compromisos o bien usar una [cuenta de nivel Gratis de Azure Cosmos DB](free-tier.md) para obtener una cuenta con los primeros 1000 RU/s y 25 GB de almacenamiento totalmente gratis.

> [!div class="nextstepaction"]
> [Probar gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)

:::image type="content" source="./media/introduction/azure-cosmos-db.png" alt-text="Azure Cosmos DB es una base de datos NoSQL totalmente administrada para el desarrollo de aplicaciones modernas." border="false":::

## <a name="key-benefits"></a>Ventajas principales

### <a name="guaranteed-speed-at-any-scale"></a>Velocidad garantizada a cualquier escala

Obtenga una velocidad y un rendimiento sin precedentes [con respaldo de SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db), acceso global rápido y elasticidad instantánea.

- Acceso en tiempo real con latencias globales de lectura y escritura rápidas, rendimiento y coherencia, todo ello respaldado por distintos [Acuerdos de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/cosmos-db).
- Escrituras en varias regiones y distribución de datos en cualquier región de Azure con un clic en un botón.
- Escale el almacenamiento y el rendimiento de forma independiente y elástica en cualquier región de Azure, incluso durante ráfagas de tráfico imprevisibles, para una escala ilimitada en todo el mundo.

### <a name="simplified-application-development"></a>Desarrollo de aplicaciones simplificado

Desarrolle rápidamente con las API de código abierto, varios SDK, datos sin esquema y análisis sin ETL sobre datos operativos.

- Está profundamente integrado con los principales servicios de Azure que se usan en el desarrollo moderno de aplicaciones (nativo de la nube), como Azure Functions, IoT Hub, AKS (Azure Kubernetes Service), App Service y mucho más.
- Elija entre varias API de base de datos, incluidas Core (SQL) API nativa, MongoDB API, Cassandra API, Gremlin API y Table API.
- Cree aplicaciones en Core (SQL) API con los lenguajes que prefiera con los SDK para .NET, Java, Node.js y Python. O con los controladores de su elección para cualquiera de las otras API de base de datos.
- Ejecute análisis sin ETL en los datos operativos casi en tiempo real almacenados en Azure Cosmos DB con Azure Synapse Analytics.
- La fuente de cambios facilita el seguimiento y la administración de los cambios en los contenedores de base de datos y la creación de eventos desencadenados con Azure Functions.
- El servicio sin esquema de Azure Cosmos DB indexa automáticamente todos los datos, independientemente del modelo de datos, para entregar consultas asombrosamente rápidas.

### <a name="mission-critical-ready"></a>Preparado para situaciones críticas

Garantice la continuidad empresarial, con una disponibilidad del 99,999 % y seguridad de nivel empresarial para todas las aplicaciones.

- Azure Cosmos DB ofrece un conjunto completo de [Acuerdos de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/cosmos-db), incluida una disponibilidad líder del sector en todo el mundo.
- Distribuya los datos fácilmente a cualquier región de Azure con la replicación automática de datos. Disfrute de cero tiempo de inactividad con escrituras de varias regiones o RPO 0 al usar la coherencia fuerte.
- Disfrute de un cifrado en reposo de clase empresarial con claves autoadministradas.
- El control de acceso basado en roles de Azure mantiene los datos seguros y ofrece un control ajustado.

### <a name="fully-managed-and-cost-effective"></a>Totalmente administrado y rentable

Administración de bases de datos de un extremo a otro, con escalado automático sin servidor en línea con las necesidades de la aplicación y de TCO.

- Servicio de base de datos totalmente administrado. Mantenimiento, revisiones y actualizaciones automáticos, lo que permite ahorrar tiempo y dinero a los desarrolladores.
- Opciones rentables para cargas de trabajo impredecibles o esporádicas de cualquier tamaño o escala, lo que permite a los desarrolladores empezar a trabajar fácilmente sin tener que planear o administrar la capacidad.
- El modelo sin servidor ofrece un servicio con capacidad de respuesta para cargas de trabajo con picos para administrar las ráfagas de tráfico a petición.
- El escalado automático de rendimiento aprovisionado escala automáticamente y al instante la capacidad para cargas de trabajo impredecibles, al tiempo que mantiene los [Acuerdos de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/cosmos-db).

## <a name="solutions-that-benefit-from-azure-cosmos-db"></a>Soluciones que se benefician de Azure Cosmos DB

Cualquier [aplicación web, para dispositivos móviles, de juegos y de IoT](use-cases.md) que necesite controlar ingentes cantidades de datos y operaciones de lectura y escritura a [escala global](distribute-data-globally.md), con tiempos de respuesta casi en tiempo real para una variedad de datos se beneficiará de la alta disponibilidad, el elevado rendimiento, la latencia baja y la coherencia ajustable [garantizados](https://azure.microsoft.com/support/legal/sla/cosmos-db/) de Cosmos DB. Aprenda a usar Azure Cosmos DB para crear aplicaciones de [IoT y telemática](use-cases.md#iot-and-telematics), [comercio minorista y marketing](use-cases.md#retail-and-marketing), y [juegos](use-cases.md#gaming), así como [aplicaciones web y para dispositivos móviles](use-cases.md#web-and-mobile-applications).

## <a name="next-steps"></a>Pasos siguientes

Comience a usar Azure Cosmos DB con una de nuestras guías rápidas:

- [Introducción a SQL API de Azure Cosmos DB](create-sql-api-dotnet.md)
- [Introducción a la API de Azure Cosmos DB para MongoDB](create-mongodb-nodejs.md)
- [Introducción a la API Cassandra de Azure Cosmos DB](create-cassandra-dotnet.md)
- [Introducción a Gremlin API de Azure Cosmos DB](create-graph-dotnet.md)
- [Introducción a Table API de Azure Cosmos DB](create-table-dotnet.md)
- [Documento técnico sobre el desarrollo de aplicaciones de próxima generación con Azure Cosmos DB](https://azure.microsoft.com/resources/microsoft-azure-cosmos-db-flexible-reliable-cloud-nosql-at-any-scale/)

> [!div class="nextstepaction"]
> [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)
