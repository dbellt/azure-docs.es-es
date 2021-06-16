---
title: 'Selección del tipo de implementación adecuado: Azure Database for MariaDB'
description: En este artículo se describen los factores que deben tenerse en cuenta antes de implementar Azure Database for MariaDB como infraestructura como servicio (IaaS) o plataforma como servicio (PaaS).
author: mksuni
ms.author: sumuth
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: e99d94fbd4ab91c2ba51ca7922873b8338f7da85
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112033309"
---
# <a name="choose-the-right-mariadb-server-option-in-azure"></a>Selección de la opción adecuada del servidor MariaDB en Azure

Con Azure, las cargas de trabajo del servidor MariaDB se pueden ejecutar en una infraestructura como servicio (IaaS) de máquina virtual hospedada o como una plataforma como servicio (PaaS) hospedada. PaaS dispone de varias opciones de implementación y, dentro de cada una, hay niveles de servicio. Al decidir entre PaaS o IaaS, debe decidir si quiere administrar la base de datos, aplicar revisiones y realizar copias de seguridad, o si quiere delegar estas operaciones en Azure.

Cuando tome la decisión, tenga en cuenta las dos siguientes opciones:

- **Azure Database for MariaDB:** Esta opción es un motor de base de datos MariaDB totalmente administrado basado en la versión estable de la edición Community de MariaDB. Esta base de datos relacional como servicio (DBaaS), hospedada en plataforma en la nube de Azure, se engloba en la categoría del sector denominada PaaS.

  Con una instancia administrada de MariaDB en Azure, puede usar características integradas que de lo contrario requieren una configuración extensiva cuando MariaDB Server está instalado de forma local o en una máquina virtual de Azure.

  Al usar MariaDB como servicio, paga por el uso de opciones de escalado vertical u horizontal a fin de aumentar el control de forma ininterrumpida. Y, a diferencia del servidor MariaDB independiente, Azure Database for MariaDB tiene características adicionales como alta disponibilidad, inteligencia y administración integradas.

- **MariaDB en máquinas virtuales de Azure:** Esta opción pertenece a la categoría del sector de IaaS. Con este servicio, puede ejecutar MariaDB Server en una máquina virtual totalmente administrada en la plataforma en la nube de Azure. Todas las versiones y ediciones recientes de MariaDB se pueden instalar en una máquina virtual IaaS.

  La diferencia más importante con Azure Database for MariaDB es que MariaDB en máquinas virtuales de Azure permite el control sobre el motor de base de datos. Sin embargo, este control conlleva el costo de la responsabilidad de administrar las máquinas virtuales y muchas tareas de administración de bases de datos (DBA). Estas tareas incluyen el mantenimiento y la revisión de los servidores de las bases de datos, la recuperación de las bases de datos y el diseño de la alta disponibilidad.

Las principales diferencias entre estas opciones se mencionan en la siguiente tabla:

| Atributo          | Azure Database for MariaDB | MariaDB en máquinas virtuales de Azure    |
|:-------------------|:-----------------------------|:--------------------|
| Acuerdo de Nivel de Servicio (SLA)                | Ofrece un Acuerdo de Nivel de Servicio del 99,99 % de disponibilidad.| Hasta el 99,95 % de disponibilidad con dos o más instancias del mismo conjunto de disponibilidad.<br/><br/>El 99,9 % de disponibilidad con una máquina virtual de instancia única con Premium Storage.<br/><br/>El 99,99 % con Availability Zones con varias instancias en varios conjuntos de disponibilidad.<br/><br/>Consulte [Acuerdo de Nivel de Servicio para Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/). |
| Aplicación de revisiones del sistema operativo        | Automático  | Administrado por los clientes |
| Aplicación de revisiones de MariaDB     | Automático  | Administrado por los clientes |
| Alta disponibilidad | El modelo de alta disponibilidad se basa en los mecanismos de conmutación por error integrados cuando se produce una interrupción en el nivel de nodo. En tales casos, el servicio crea automáticamente una nueva instancia y asocia el almacenamiento a esta instancia. | Los clientes pueden diseñar, implementar, probar y mantener una alta disponibilidad. Entre las funcionalidades se incluyen los clústeres de conmutación por error Always On, la replicación de grupos Always On, el trasvase de registros o la replicación transaccional.|
| Redundancia de zona | Actualmente no se admite | Las máquinas virtuales de Azure se pueden configurar para que se ejecuten en distintas zonas de disponibilidad. En el caso de una solución local, los clientes deben crear, administrar y mantener su propio centro de datos secundario.|
| Escenarios híbridos | Con la [replicación de datos de entrada](concepts-data-in-replication.md), puede sincronizar los datos de un servidor MariaDB externo con el servicio Azure Database for MariaDB. El servidor externo puede ser local, de máquinas virtuales o un servicio de base de datos hospedado por otros proveedores de nube.<br/><br/> Con la característica de [réplica de lectura](concepts-read-replicas.md) puede replicar los datos de un servidor de origen de Azure Database for MariaDB en hasta cinco servidores de réplica de solo lectura. Las réplicas están dentro de la misma región de Azure o en varias regiones. Las réplicas de solo lectura se actualizan de forma asincrónica mediante la tecnología de replicación binlog.<br/><br/>La replicación de lectura entre regiones se encuentra actualmente en versión preliminar pública.| Administrado por los clientes
| Copia de seguridad y restauración | Crea automáticamente [copias de seguridad del servidor](concepts-backup.md#backups) y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. El servicio realiza copias de seguridad completas, diferenciales y del registro de transacciones. | Administrado por los clientes |
| Supervisión de las operaciones de base de datos | Ofrece a los clientes la capacidad de [definir alertas](concepts-monitoring.md) en la operación de base de datos y realizar acciones al alcanzar los umbrales. | Administrado por los clientes |
| Protección contra amenazas avanzada | Proporciona Advanced Threat Protection. Esta protección detecta actividades anómalas que indiquen intentos inusuales y potencialmente perjudiciales de acceder a sus bases de datos o aprovechar sus vulnerabilidades.<br/><br/>Advanced Threat Protection está actualmente en versión preliminar pública.| Los clientes deben compilar esta protección por sí mismos.
| Recuperación ante desastres | Almacena copias de seguridad automatizadas en el [almacenamiento con redundancia geográfica o con redundancia local](howto-restore-server-portal.md) configurado por el usuario. Las copias de seguridad pueden restaurar también un servidor a un momento dado. El período de retención es de 7 a 35 días. La restauración se realiza mediante Azure Portal. | Totalmente administradas por los clientes. Las responsabilidades incluyen, entre otras tareas, la programación, la prueba, el archivado, el almacenamiento y la retención. Una opción adicional es usar el almacén de Recovery Services para realizar la copia de seguridad de máquinas virtuales de Azure y bases de datos en máquinas virtuales. Esta opción se encuentra en versión preliminar. |
| Recomendaciones de rendimiento | Proporciona a los clientes [recomendaciones de rendimiento](https://techcommunity.microsoft.com/t5/Azure-Database-for-MariaDB/Azure-brings-intelligence-and-high-performance-to-Azure-Database/ba-p/769110) basadas en archivos de registro de uso generados por el sistema. Las recomendaciones ayudan a optimizar las cargas de trabajo.<br/><br/>Las recomendaciones de rendimiento se encuentran actualmente en versión preliminar pública. | Administrado por los clientes |

## <a name="business-motivations-for-choosing-paas-or-iaas"></a>Motivaciones empresariales para elegir PaaS o IaaS

Hay varios factores que pueden influir en la decisión para elegir PaaS o IaaS para hospedar las bases de datos MariaDB.

### <a name="cost"></a>Coste

La capacidad limitada de financiación suele ser la consideración principal que determina la mejor solución para hospedar las bases de datos. Esto suele darse en el caso de las startups con poca disponibilidad de efectivo o el equipo de una empresa bien establecida que opera con presupuestos ajustados. En esta sección se describen los conceptos básicos de facturación y licencias en Azure con respecto a Azure Database for MariaDB y MariaDB en máquinas virtuales de Azure.

#### <a name="billing"></a>Facturación

Azure Database for MariaDB está disponible como servicio en varios niveles con recursos de distintos precios. Todos los recursos se facturan por hora a una tarifa fija. Para acceder a la información más reciente sobre los niveles de servicio, los tamaños de proceso y las cantidades de almacenamiento admitidos actualmente, consulte el [modelo de compra basado en núcleo virtual](concepts-pricing-tiers.md). Los niveles de servicio y los tamaños de proceso se pueden ajustar de forma dinámica para satisfacer las necesidades variables de rendimiento de la aplicación. Se le factura por el tráfico saliente de Internet según las [tarifas de transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) habituales.

Con Azure Database for MariaDB, Microsoft configura, aplica revisiones y actualiza el software de base de datos de forma automática. Estas acciones automatizadas reducen los costos de administración. Además, Azure Database for MariaDB tiene capacidades de [copia de seguridad integradas](concepts-backup.md). Estas funcionalidades le ayudan a obtener un ahorro significativo, sobre todo, cuando se tiene gran cantidad de base de datos. En contraste, con MariaDB en máquinas virtuales de Azure, puede elegir y ejecutar cualquiera de las versiones de MariaDB. Con independencia de la versión de MariaDB que use, pagará por la máquina virtual aprovisionada y los costos del tipo de licencia específico de MariaDB que utilice.

Azure Database for MariaDB proporciona alta disponibilidad integrada para cualquier tipo de interrupción de nivel de nodo y mantiene al mismo tiempo la garantía del 99,99 % establecida por el SLA para el servicio. Sin embargo, para conseguir una alta disponibilidad de la base de datos en las máquinas virtuales, los clientes deben usar las opciones de alta disponibilidad que están disponibles en una base de datos de MariaDB, como [la replicación de MariaDB](https://mariadb.com/kb/en/library/setting-up-replication/). El uso de una opción de alta disponibilidad compatible no proporciona un SLA adicional, pero permite lograr una disponibilidad de base de datos del 99,99 % a cambio de un costo adicional y sobrecarga administrativa.

Para más información sobre los precios, consulte los siguientes artículos:
* [Precios de Azure Database for MariaDB](https://azure.microsoft.com/pricing/details/MariaDB/)
* [Precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/)
* [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/)

### <a name="administration"></a>Administración

En muchas empresas, la decisión de pasar a un servicio en la nube está tan relacionada con la posibilidad de reducir la carga de complejidad de administración como con el costo. Con IaaS y PaaS, Microsoft:

- Administra la infraestructura subyacente.
- Replica automáticamente todos los datos para proporcionar recuperación ante desastres.
- Configura y actualiza el software de la base de datos.
- Se ocupa del equilibrio de carga.
- Realiza una conmutación por error transparente si se produce un error en el servidor.

En la lista siguiente se describen las consideraciones administrativas para cada opción:

* Con Azure Database for MariaDB, puede seguir administrando la base de datos. Sin embargo, ya no necesita administrar el motor de base de datos, el sistema operativo o el hardware. Entre los ejemplos de elementos que puede seguir administrando se incluyen:

  - Bases de datos
  - Inicio de sesión
  - Ajuste del índice
  - Ajuste de las consultas
  - Auditoría
  - Seguridad

  Además, la configuración de alta disponibilidad en otro centro de datos no requiere configuración y administración o, si acaso, ajustes mínimos.

* Con MariaDB en máquinas virtuales de Azure, tiene un control completo sobre la configuración del sistema operativo y la instancia de servidor MariaDB. Con una máquina virtual, en sus manos está la decisión de cuándo actualizar o renovar el sistema operativo y el software de la base de datos. También puede decidir cuándo instalar cualquier software adicional, como una aplicación antivirus. Se proporcionan algunas características automatizadas para simplificar considerablemente la aplicación de revisiones, las copias de seguridad y la alta disponibilidad. Puede controlar el tamaño de la máquina virtual, el número de discos y sus configuraciones de almacenamiento. Para más información, consulte [Tamaños de las máquinas virtuales y los servicios en la nube de Azure](../virtual-machines/sizes.md).

### <a name="time-to-move-to-azure"></a>Es hora de cambiar a Azure

* Azure Database for MariaDB es la solución adecuada para las aplicaciones diseñadas para la nube cuando la productividad del desarrollador y un plazo de comercialización rápido de las nuevas soluciones son factores críticos. Con una funcionalidad mediante programación de tipo DBA, el servicio es idóneo para arquitectos y desarrolladores de la nube, puesto que reduce la necesidad de administrar el sistema operativo y la base de datos subyacentes.

* Si quiere evitar el tiempo y los gastos de adquirir nuevo hardware local, MariaDB en máquinas virtuales de Azure es la solución idónea para aplicaciones que requieren una base de datos MariaDB o el acceso a sus características en Windows o Linux. Esta solución también es idónea para migrar aplicaciones y bases de datos locales existentes a Azure tal cual, en casos en los que Azure Database for MariaDB no es una buena solución.

  Dado que no es necesario cambiar las capas de presentación, aplicación y datos, se ahorra tiempo y presupuesto a la hora de renovar la arquitectura de la solución existente. Así, puede centrarse en migrar todas las soluciones a Azure y en abordar algunas optimizaciones de rendimiento que puede requerir la plataforma Azure.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Precios de Azure Database for MariaDB](https://azure.microsoft.com/pricing/details/MariaDB/).
* Comenzar por [crear el primer servidor](quickstart-create-mariadb-server-database-using-azure-portal.md).
