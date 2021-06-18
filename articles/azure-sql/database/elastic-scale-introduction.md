---
title: Escalado horizontal
description: Los desarrolladores de Software como servicio (SaaS) pueden crear bases de datos elásticas y escalables con facilidad en la nube mediante estas herramientas
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.topic: conceptual
ms.custom: sqldbrb=1
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: b0e577a8f0d8b77a9a8fcd1a5e9fdf4ca909c14e
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705559"
---
# <a name="scaling-out-with-azure-sql-database"></a>Escalado horizontal con Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Es muy fácil escalar horizontalmente bases de datos de Azure SQL Database mediante las herramientas de **Base de datos elástica**. Estas herramientas y características permiten usar los recursos de bases de datos de **Azure SQL Database** con el fin de crear soluciones para cargas de trabajo transaccionales y, especialmente, aplicaciones de software como servicio (SaaS). Las características de Elastic Database se componen de lo siguiente:

* [Biblioteca de cliente de Elastic Database](elastic-database-client-library.md): la biblioteca de cliente es una característica que permite crear y mantener bases de datos particionadas.  Consulte [Introducción a las herramientas de Elastic Database](elastic-scale-get-started.md).
* [Herramienta de división y combinación de Elastic Database](elastic-scale-overview-split-and-merge.md): mueve datos entre bases de datos particionadas. Esta herramienta es útil para mover datos de una base de datos de varios inquilinos a una de un solo inquilino (o viceversa). Consulte el [tutorial de la herramienta de división y combinación de Base de datos elástica](elastic-scale-configure-deploy-split-and-merge.md).
* [Trabajos de Elastic Database](elastic-jobs-overview.md) (versión preliminar): use trabajos para administrar un gran número de bases de datos en Azure SQL Database. Realice fácilmente operaciones administrativas, como cambios de esquema, administración de credenciales, actualizaciones de datos de referencia, recopilación de datos de rendimiento o de trabajos de recolección de telemetría de inquilinos (cliente).
* [Consulta de Elastic Database](elastic-query-overview.md) (versión preliminar): le permite ejecutar una consulta Transact-SQL que abarca varias bases de datos. Esto permite la conexión con herramientas de informes, como Excel, Power BI, Tableau, etc.
* [Transacciones elásticas](elastic-transactions-overview.md): esta característica permite ejecutar transacciones que abarcan varias bases de datos. Las transacciones de bases de datos elásticas están disponibles para aplicaciones .NET mediante ADO .NET y se integran con la conocida experiencia de programación en la que se emplean las clases [System.Transaction](/dotnet/api/system.transactions).

El siguiente gráfico muestra una arquitectura que incluye las **características de Elastic Database** en relación con una colección de bases de datos.

En este gráfico, los colores de la base de datos representan esquemas. Las bases de datos con el mismo color comparten el mismo esquema.

1. Un conjunto de **bases de datos SQL** se hospeda en Azure con la arquitectura de particionamiento.
2. La **biblioteca de cliente de Elastic Database** se usa para administrar un conjunto de particiones.
3. Un subconjunto de las bases de datos se coloca en un **grupo elástico**. (Consulte [¿Qué es un grupo?](elastic-pool-overview.md)).
4. Un **trabajo de Elastic Database** ejecuta scripts de T-SQL programados o ad-hoc en todas las bases de datos.
5. La **herramienta de división y combinación** se usa para mover datos de una partición a otra.
6. La **consulta de Elastic Database** le permite escribir una consulta que abarque todas las bases de datos del conjunto de particiones.
7. Las **transacciones elásticas** le permiten ejecutar transacciones que abarcan varias bases de datos. 

![Herramientas de Elastic Database][1]

## <a name="why-use-the-tools"></a>¿Por qué hay que usar las herramientas?

Dotar de elasticidad y escalabilidad a las aplicaciones en la nube ha sido sencillo para el almacenamiento de blobs y las máquinas virtuales; basta con sumar o restar unidades, o bien con aumentar la potencia. Pero sigue siendo un desafío para el procesamiento de datos con estado en bases de datos relacionales. Estos son los desafíos que han surgido en estos escenarios:

* Aumento y disminución de la capacidad de la parte de la base de datos relacional de la carga de trabajo.
* La administración de las zonas activas puede afectar a un subconjunto específico de datos, como un usuario final (inquilino) ocupado.

Tradicionalmente, escenarios similares se han abordado a través de la inversión en servidores a gran escala para admitir la aplicación. Sin embargo, esta opción está limitada en la nube donde todo el procesamiento ocurre en hardware estándar predefinido. En su lugar, la distribución de datos y el procesamiento a través de muchas bases de datos con una estructura idéntica (un patrón de escalado horizontal conocido como "particionamiento") proporciona una alternativa a los enfoques de escalado vertical tradicionales, tanto en términos de costo como de elasticidad.

## <a name="horizontal-and-vertical-scaling"></a>Escalado horizontal y vertical

En la siguiente ilustración se muestran las dimensiones horizontales y verticales de escalado, que son las formas básicas en que se pueden escalar las bases de datos elásticas.

![Escalado horizontal frente a vertical][2]

Escalar horizontalmente es agregar o quitar bases de datos con el fin de ajustar la capacidad o el rendimiento general; también se conoce como "escalado horizontal". El particionamiento, en el que los datos se particionan en toda una recopilación de bases de datos con estructura idéntica, es un enfoque común para implementar el escalado horizontal.  

Escalar verticalmente es aumentar o disminuir el tamaño de proceso de una base de datos individual; también se conoce como "escalado vertical".

La mayoría de las aplicaciones de bases de datos a escala de la nube usan una combinación de estas dos estrategias. Por ejemplo, una aplicación de software como servicio podría utilizar el escalado horizontal para aprovisionar usuarios finales nuevos y el escalado vertical para permitir que la base de datos de cada cliente final aumente o disminuya los recursos, según lo requiera la carga de trabajo.

* El escalado horizontal se administra mediante la [biblioteca de cliente de Elastic Database](elastic-database-client-library.md).
* El escalado vertical se logra utilizando cmdlets de Azure PowerShell para cambiar el nivel de servicio o colocando bases de datos en un grupo elástico.

## <a name="sharding"></a>Particionamiento

*Particionamiento* es una técnica para distribuir grandes cantidades de datos estructurados de manera idéntica entre un número de bases de datos independientes. Su uso se ha extendido entre los desarrolladores de nube que crean ofertas de software como servicio (SAAS) para empresas o clientes finales. Estos clientes finales a menudo se conocen como "inquilinos". El particionamiento puede ser necesario por diversos motivos:  

* La cantidad total de datos es demasiado grande para caber dentro de las restricciones de una base de datos individual
* El rendimiento de transacciones de la carga de trabajo total supera la capacidad de una base de datos individual
* Los inquilinos pueden requerir el aislamiento físico entre sí, por lo que se necesitan bases de datos independientes para cada inquilino
* Es posible que distintas secciones de una base de datos tengan que residir en diferentes geografías por motivos de cumplimiento, rendimiento o geopolíticos.

En otros escenarios, como la recopilación de datos desde dispositivos distribuidos, el particionamiento se puede usar para llenar un conjunto de bases de datos organizadas de manera temporal. Por ejemplo, puede dedicarse una base de datos independiente a cada día o semana. En ese caso, la clave de particionamiento puede ser un entero que representa la fecha (presente en todas las filas de las tablas particionadas) y la aplicación debe enrutar las consultas que recuperan información de un intervalo de fechas al subconjunto de bases de datos que abarca el intervalo en cuestión.

El particionamiento funciona mejor cuando todas las transacciones de una aplicación pueden restringirse a un único valor de una clave de particionamiento. De este modo se garantiza que todas las transacciones sean locales con respecto a una base de datos.

## <a name="multi-tenant-and-single-tenant"></a>De un solo inquilino y multiinquilino

Algunas aplicaciones usan el enfoque más simple de crear una base de datos independiente para cada inquilino. Este enfoque es el **patrón de particionamiento de un solo inquilino** que proporciona aislamiento, capacidad de copia de seguridad y restauración, y ajuste de escala de recursos en la granularidad del inquilino. Con el particionamiento de un solo inquilino, cada base de datos se asocia a un determinado valor de identificador de inquilino (o valor de clave de cliente), pero no es necesario que esa clave esté siempre presente en los propios datos. Es responsabilidad de la aplicación enrutar cada solicitud a la base de datos adecuada y la biblioteca de cliente puede simplificar este proceso.

![Un solo inquilino frente a multiinquilinos][4]

Otros escenarios empaquetan varios inquilinos juntos en bases de datos, en lugar de aislarlos en bases de datos independientes. Este es un **patrón de particionamiento multiinquilino** típico y puede estar impulsado por el hecho de que una aplicación administra grandes cantidades de pequeños inquilinos. En el particionamiento de varios inquilinos, las filas de las tablas de bases de datos están diseñadas para contener una clave que identifique la clave de particionamiento o el identificador del inquilino. De nuevo, la capa de aplicación es la responsable de enrutar la solicitud de un inquilino a la base de datos adecuada, y esto puede admitirlo la biblioteca de cliente de bases de datos elásticas. Además, es posible usar seguridad en el nivel de fila para filtrar las filas a las que puede tener acceso cada inquilino; si desea obtener detalles, consulte [Aplicaciones multiinquilinos con herramientas de bases de datos elásticas y seguridad de nivel de fila](saas-tenancy-elastic-tools-multi-tenant-row-level-security.md). La redistribución de datos entre las bases de datos puede ser necesaria con el patrón de particionamiento multiinquilino, lo que se ve facilitado por la herramienta de división y combinación de bases de datos elásticas. Para más información acerca de los modelos de diseño de las aplicaciones SaaS que usan grupos elásticos, consulte [Modelos de diseño para aplicaciones SaaS multiinquilino con Azure SQL Database](saas-tenancy-app-design-patterns.md).

### <a name="move-data-from-multiple-to-single-tenancy-databases"></a>Mover datos de bases de datos de multinIiquilino a inquilino único
Al crear una aplicación SaaS, es típico ofrecer a los clientes potenciales una versión de prueba del software. En este caso, resulta más rentable usar una base de datos multiinquilino para los datos. Sin embargo, cuando un cliente potencial se convierte en un cliente, una base de datos de inquilino único es mejor, puesto que ofrece un mejor rendimiento. Si el cliente había creado datos durante el período de prueba, use la [herramienta de división y combinación](elastic-scale-overview-split-and-merge.md) para mover los datos de la base de datos multiinquilino a la nueva base de datos de inquilino único.

## <a name="next-steps"></a>Pasos siguientes
Para ver una aplicación de ejemplo que demuestra la biblioteca de cliente, consulte [Introducción a las herramientas de Elastic Database](elastic-scale-get-started.md).

Para convertir las bases de datos existentes con el fin de emplear las herramientas, consulte [Migración de bases de datos existentes de escalado horizontal](elastic-convert-to-use-elastic-tools.md).

Para ver los detalles del grupo elástico, consulte [Consideraciones de precio y rendimiento para un grupo elástico](elastic-pool-overview.md) o cree un nuevo grupo con [grupos elásticos](elastic-pool-manage.md).  

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->
[1]:./media/elastic-scale-introduction/tools.png
[2]:./media/elastic-scale-introduction/h_versus_vert.png
[3]:./media/elastic-scale-introduction/overview.png
[4]:./media/elastic-scale-introduction/single_v_multi_tenant.png
