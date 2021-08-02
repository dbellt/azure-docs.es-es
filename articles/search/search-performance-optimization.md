---
title: Disponibilidad y continuidad
titleSuffix: Azure Cognitive Search
description: Aprenda a hacer que un servicio de búsqueda sea altamente disponible y resistente frente a interrupciones del período o incluso errores catastróficos.
author: LiamCavanagh
ms.author: liamca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/06/2021
ms.custom: references_regions
ms.openlocfilehash: 169f0b76e1009931d51339fe6b058ca24608af30
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110061055"
---
# <a name="availability-and-business-continuity-in-azure-cognitive-search"></a>Disponibilidad y continuidad empresarial en Azure Cognitive Search

En Cognitive Search, la disponibilidad se logra a través de varias réplicas, mientras que la continuidad empresarial (y la recuperación ante desastres) se logra a través de varios servicios de búsqueda. En este artículo se proporcionan instrucciones que puede usar como punto de partida para desarrollar una estrategia que cumpla los requisitos empresariales de disponibilidad y operaciones continuas.

<a name="scale-for-availability"></a>

## <a name="high-availability"></a>Alta disponibilidad

En Cognitive Search, las réplicas son copias del índice. Tener varias réplicas permite que Azure Cognitive Search realice tareas de reinicio y mantenimiento de la máquina en una réplica, mientras que la ejecución de consultas continúa en otras. Para obtener más información sobre cómo agregar réplicas, vea [Adición o reducción de réplicas y particiones](search-capacity-planning.md#adjust-capacity).

Para cada servicio de búsqueda individual, Microsoft garantiza al menos un 99,9 % de disponibilidad para las configuraciones que cumplen estos criterios: 

+ Dos réplicas para alta disponibilidad de cargas de trabajo de solo lectura (consultas)

+ Tres o más réplicas para lograr alta disponibilidad de las cargas de trabajo de lectura-escritura (consultas e indexación) 

No se proporciona ningún Acuerdo de Nivel de Servicio para el nivel Gratis. Para más información, consulte [SLA para Azure Cognitive Search](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

<a name="availability-zones"></a>

## <a name="availability-zones"></a>Zonas de disponibilidad

[Availability Zones](../availability-zones/az-overview.md) es una capacidad de la plataforma de Azure que divide los centros de datos de una región en grupos de ubicaciones físicas distintos para proporcionar alta disponibilidad, dentro de la misma región. Si usa Availability Zones para Cognitive Search, las réplicas individuales son las unidades de asignación de zona. Un servicio de búsqueda se ejecuta dentro de una región; sus réplicas se ejecutan en zonas diferentes.

Puede agregar dos o más réplicas al servicio de búsqueda para utilizar Availability Zones con Azure Cognitive Search. Cada réplica se colocará en una zona de disponibilidad distinta dentro de la región. Si tiene más réplicas que zonas de disponibilidad, las réplicas se distribuirán entre esas zonas lo más uniformemente posible. No hay ninguna acción específica que deba hacer, excepto [crear un servicio de búsqueda](search-create-service-portal.md) en una región que proporciona Availability Zones y, a continuación, configurar el servicio para que [use varias réplicas](search-capacity-planning.md#adjust-capacity).

Actualmente, Azure Cognitive Search admite zonas de disponibilidad para los servicios de búsqueda de nivel estándar o superior que se crearon en una de las siguientes regiones:

+ Este de Australia (creado a partir del 30 de enero de 2021)
+ Sur de Brasil (creado el 2 de mayo de 2021, o posteriormente)
+ Centro de Canadá (creado a partir del 30 de enero de 2021)
+ Centro de EE. UU. (creado a partir del 4 de diciembre de 2020)
+ Este de EE. UU. (creado a partir del 27 de enero de 2021)
+ Este de EE. UU. 2 (creado a partir del 30 de enero de 2021)
+ Centro de Francia (creado a partir del 23 de octubre de 2020)
+ Centro-oeste de Alemania (creado el 3 de mayo de 2021, o posteriormente)
+ Este de Japón (creado a partir del 30 de enero de 2021)
+ Norte de Europa (creado a partir del 28 de enero de 2021)
+ Centro-sur de EE. UU. (creado el 30 de abril de 2021, o posteriormente)
+ Sudeste de Asia (creado a partir del 31 de enero de 2021)
+ Sur de Reino Unido (creado a partir del 30 de enero de 2021)
+ US Gov Virginia (creado el 30 de abril de 2021, o posteriormente)
+ Oeste de Europa (creado a partir del 29 de enero de 2021)
+ Oeste de EE. UU. 2 (creado a partir del 30 de enero de 2021)

Las zonas de disponibilidad no afectan al [Acuerdo de Nivel de Servicio de Azure Cognitive Search](https://azure.microsoft.com/support/legal/sla/search/v1_0/). Todavía necesita tres o más réplicas para la alta disponibilidad de las consultas.

## <a name="multiple-services-in-separate-geographic-regions"></a>Varios servicios en regiones geográficas independientes

Aunque la mayoría de los clientes usan un solo servicio, la redundancia de servicios puede ser necesaria si los requisitos operativos son los siguientes:

+ [Continuidad empresarial y recuperación ante desastres (BCDR)](../best-practices-availability-paired-regions.md) (Cognitive Search no proporciona conmutación por error instantánea en caso de una interrupción).
+ Aplicaciones implementadas globalmente. Si las solicitudes de consulta e indexación proceden de todo el mundo, los usuarios más cercanos al centro de datos del host tendrán un rendimiento más rápido. La creación de servicios adicionales en regiones cercanas a estos usuarios puede igualar el rendimiento de todos los usuarios.
+ [Las arquitecturas multiinquilino](search-modeling-multitenant-saas-applications.md) a veces llaman a dos o más servicios.

Si necesita dos servicios de búsqueda más, crearlos en regiones diferentes puede cumplir los requisitos de la aplicación para la continuidad y la recuperación, así como tiempos de respuesta más rápidos para una base de usuarios global.

Azure Cognitive Search no proporciona actualmente un método automatizado de índices de búsqueda con replicación geográfica entre regiones, pero existen algunas técnicas que permiten simplificar la implementación y administración de este proceso. Se describen en las siguientes secciones.

El objetivo de un conjunto de servicios de búsqueda distribuido geográficamente es tener dos o más índices disponibles en dos o más regiones, en las que un usuario se enruta al servicio Azure Cognitive Search que proporciona la latencia más baja:

   ![Tablas de referencias cruzadas de servicios por región][1]

Puede implementar esta arquitectura mediante la creación de varios servicios y el diseño de una estrategia para la sincronización de datos. Opcionalmente, puede incluir un recurso como Azure Traffic Manager para las solicitudes de enrutamiento. Para más información, consulte [Creación de servicio de búsqueda](search-create-service-portal.md).

<a name="data-sync"></a>

### <a name="keep-data-synchronized-across-multiple-services"></a>Mantenimiento de los datos sincronizados entre varios servicios

Hay dos opciones para mantener sincronizados dos o más servicios de búsqueda distribuidos: usar el [indexador de Azure Cognitive Search](search-indexer-overview.md) o la Push API (también denominada [API REST de Azure Cognitive Search](/rest/api/searchservice/)). 

#### <a name="option-1-use-indexers-for-updating-content-on-multiple-services"></a>Opción 1: uso de indexadores para la actualización de contenido en varios servicios

Si ya usa un indexador en un servicio, puede configurar un segundo indexador en un segundo servicio para usar el mismo objeto de origen de datos, extrayendo datos de la misma ubicación. Cada servicio en cada región tiene su propio indexador y un índice de destino (el índice de búsqueda no se comparte, lo que significa que los datos se duplican), pero cada indexador hace referencia al mismo origen de datos.

Este es un objeto visual de alto nivel del aspecto que podría tener esa arquitectura.

   ![Origen de datos único con indexador distribuido y combinaciones de servicios][2]

#### <a name="option-2-use-rest-apis-for-pushing-content-updates-on-multiple-services"></a>Opción 2: uso de las API REST para insertar actualizaciones de contenido en varios servicios

Si usa la API REST de Azure Cognitive Search para [insertar contenido en el índice de búsqueda](tutorial-optimize-indexing-push-api.md) y quiere mantener sincronizados los distintos servicios de búsqueda, inserte los cambios en todos los servicios de búsqueda cada vez que se necesite una actualización. En el código, asegúrese de controlar los casos en los que una actualización en un servicio de búsqueda produce un error, pero se ejecuta correctamente en otros servicios de búsqueda.

### <a name="use-azure-traffic-manager-to-coordinate-requests"></a>Uso de Azure Traffic Manager para coordinar las solicitudes

[Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) permite enrutar solicitudes a diversos sitios web con ubicación geográfica atendidos por distintos servicios Azure Cognitive Search. Una ventaja de Traffic Manager es que puede sondear Azure Cognitive Search para asegurarse de que está disponible y enrutar los usuarios a servicios de búsqueda alternativos en caso de tiempo de inactividad. Además, si va a enrutar solicitudes de búsqueda a través de Azure Web Sites, Azure Traffic Manager permite equilibrar la carga de aquellos casos en los que el sitio web está activo pero Azure Cognitive Search no. Este es un ejemplo de una arquitectura que utiliza el Administrador de tráfico.

   ![Tablas de referencias cruzadas de servicios por región con Administrador de tráfico central][3]

## <a name="disaster-recovery-and-service-outages"></a>Interrupciones de servicio y recuperación ante desastres

Aunque podemos recuperar los datos, Azure Cognitive Search no proporciona conmutación por error instantánea del servicio si se produce una interrupción en centro de datos o el clúster. Si se produce un error en un clúster del centro de datos, el equipo de operaciones lo detectará y trabajará para restaurar el servicio. Experimentará tiempo de inactividad durante la restauración del servicio, pero puede solicitar créditos del servicio para compensar la falta de disponibilidad de servicio según el [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/search/v1_0/). 

Si se requiera continuidad del servicio, en caso de errores catastróficos fuera de control de Microsoft, debe [aprovisionar un servicio adicional](search-create-service-portal.md) en una región diferente e implementar una estrategia de replicación geográfica para asegurarse de que los índices sean totalmente redundantes en todos los servicios.

Los clientes que usen [indizadores](search-indexer-overview.md) para rellenar y actualizar índices pueden controlar la recuperación ante desastres mediante indizadores geográficos que aprovechan el mismo origen de datos. Dos servicios en regiones diferentes, donde cada uno ejecuta un indizador, pueden indexar desde el mismo origen de datos para lograr redundancia geográfica. Si indexa desde orígenes de datos que también tienen redundancia geográfica, tenga en cuenta que los indexadores de Azure Cognitive Search solo pueden realizar una indexación incremental (fusión de actualizaciones a partir de documentos nuevos, modificados o eliminados) desde las réplicas principales. En un evento de conmutación por error, asegúrese de que el indizador apunte de nuevo a la nueva réplica principal. 

Si no utiliza indizadores, usaría el código de aplicación para enviar objetos y datos a distintos servicios de búsqueda en paralelo. Para obtener más información, consulte [Mantenimiento de los datos sincronizados entre varios servicios](#data-sync).

## <a name="back-up-and-restore-alternatives"></a>Alternativas de copia de seguridad y restauración

Dado que Azure Cognitive Search no es una solución de almacenamiento de datos principal, Microsoft no proporciona un mecanismo formal para realizar procesos de copias de seguridad y restauración automáticos. Sin embargo, puede usar el ejemplo de código **index-backup-restore** de este [repositorio de ejemplo .NET de Azure Cognitive Search](https://github.com/Azure-Samples/azure-search-dotnet-samples) para realizar la copia de seguridad de la definición de índice y una instantánea en una serie de archivos JSON y, después, usar esos archivos para restaurar el índice, en caso de que sea necesario. Esta herramienta también se puede usar para mover índices entre niveles de servicio.

De lo contrario, el código de aplicación que se usa para crear y rellenar un índice es la opción de restauración de facto si elimina por error un índice. Para volver a generar un índice, elimínelo (si existe), vuelva a crear el índice en el servicio y cárguelo de nuevo recuperando los datos del almacén de datos principal.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de los planes de tarifa y los límites de los servicios de cada uno de ellos, consulte [Límites de servicio](search-limits-quotas-capacity.md). Consulte [Planeación de capacidad](search-capacity-planning.md) para obtener información acerca de las combinaciones de particiones y réplicas.

Para obtener una explicación sobre el rendimiento y las demostraciones de las técnicas descritas en este artículo, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON319/player]
> 

<!--Image references-->
[1]: ./media/search-performance-optimization/geo-redundancy.png
[2]: ./media/search-performance-optimization/scale-indexers.png
[3]: ./media/search-performance-optimization/geo-search-traffic-mgr.png