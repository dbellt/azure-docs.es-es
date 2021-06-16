---
title: ¿Qué es un grupo de Instancia administrada de Azure SQL?
titleSuffix: Azure SQL Managed Instance
description: Información acerca de los grupos de Instancia administrada de Azure SQL (versión preliminar), una característica que proporciona una forma cómoda y rentable de migrar bases de datos de SQL Server más pequeñas a la nube a escala y administrar varias instancias administradas.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: service-overview
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: urosmil
ms.author: urmilano
ms.reviewer: mathoma
ms.date: 09/05/2019
ms.openlocfilehash: b84ff58517d413be884b81b6498f5b626ba02016
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697250"
---
# <a name="what-is-an-azure-sql-managed-instance-pool-preview"></a>¿Qué es un grupo de Instancia administrada de Azure SQL (versión preliminar)?
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Los grupos de instancias de Instancia administrada de Azure SQL proporcionan una forma cómoda y rentable de migrar instancias de SQL Server más pequeñas a la nube a escala.

Los grupos de instancias permiten aprovisionar previamente recursos de proceso según los requisitos totales de la migración. Después, puede implementar varias instancias administradas individuales hasta el nivel de proceso aprovisionado previamente. Por ejemplo, si aprovisiona previamente 8 núcleos virtuales, puede implementar dos instancias de 2 núcleos virtuales y una instancia de 4 núcleos virtuales y, a continuación, migrar las bases de datos a estas instancias. Antes de que los grupos de instancias estuvieran disponibles, las cargas de trabajo de procesos intensivos más pequeñas y menos frecuentes a menudo hubiesen tenido que consolidarse en una instancia administrada más grande al migrar a la nube. La necesidad de migrar grupos de bases de datos a una instancia grande normalmente requería un cuidadoso planeamiento de la capacidad y gobernanza de los recursos, consideraciones de seguridad adicionales y algunos trabajos de consolidación de datos adicionales en el nivel de instancia.

Además, los grupos de instancias admiten la integración de redes virtuales nativas, por lo que puede implementar varios grupos de instancias y varias instancias únicas en la misma subred.

## <a name="key-capabilities"></a>Principales capacidades

Los grupos de instancias ofrece las ventajas siguientes:

1. Capacidad de hospedar instancias de 2 núcleos virtuales. *\*Solo para las instancias en grupos de instancias*.
2. Tiempo de implementación de instancia predecible y rápida (hasta 5 minutos).
3. Asignación mínima de direcciones IP.

En el diagrama siguiente se muestra un grupo de instancias con varias instancias administradas implementadas dentro de una subred de red virtual.

![Grupo de instancias con varias instancias](./media/instance-pools-overview/instance-pools1.png)

Los grupos de instancias permiten la implementación de varias instancias en la misma máquina virtual en la que el tamaño de proceso de la máquina virtual se basa en el número total de núcleos virtuales asignados para el grupo. Esta arquitectura permite la *creación de particiones* de la máquina virtual en varias instancias, que pueden ser de cualquier tamaño admitido, incluidos 2 núcleos virtuales (las instancias de 2 núcleos virtuales solo están disponibles para instancias en grupos).

Después de la implementación inicial, las operaciones de administración en las instancias de un grupo son mucho más rápidas. Esto se debe a que la implementación o la extensión de un [clúster virtual](connectivity-architecture-overview.md#high-level-connectivity-architecture) (un conjunto dedicado de máquinas virtuales) no forma parte del aprovisionamiento de la instancia administrada.

Dado que todas las instancias de un grupo comparten la misma máquina virtual, la asignación de direcciones IP total no depende del número de instancias implementadas, lo que resulta práctico para la implementación en subredes con un intervalo IP restringido.

Cada grupo tiene una asignación de direcciones IP fija de solo nueve direcciones IP (sin incluir las cinco direcciones IP de la subred que están reservadas para sus propias necesidades). Para información detallada, consulte los [requisitos de tamaño de la subred para instancias únicas](vnet-subnet-determine-size.md).

## <a name="application-scenarios"></a>Escenarios de aplicación

En la lista siguiente se proporcionan los casos de uso principales en los que se deben tener en cuenta los grupos de instancias:

- Migración de *un grupo de instancias de SQL Server* al mismo tiempo, donde la mayoría es de un tamaño más pequeño (por ejemplo, 2 o 4 núcleos virtuales).
- Escenarios en los que es importante *la creación y el escalado de instancias cortas y predecibles*. Por ejemplo, la implementación de un inquilino nuevo en un entorno de aplicación SaaS multiinquilino que requiere funcionalidades de nivel de instancia.
- Escenarios en los que es importante tener un *costo fijo* o un *límite de gasto*. Por ejemplo, la ejecución de entornos compartidos de desarrollo y pruebas o de demostración de un tamaño fijo (o que cambia con poca frecuencia), donde se implementan periódicamente instancias administradas cuando es necesario.
- Escenarios en los que es importante la *asignación mínima de direcciones IP* en una subred de red virtual. Todas las instancias de un grupo comparten una máquina virtual, por lo que el número de direcciones IP asignadas es menor que en el caso de las instancias únicas.

## <a name="architecture"></a>Architecture

Los grupos de instancias tienen una arquitectura similar a las instancias administradas normales (*únicas*). Para admitir las [implementaciones dentro de redes virtuales de Azure](../../virtual-network/virtual-network-for-azure-services.md) y proporcionar aislamiento y seguridad para los clientes, los grupos de instancias también dependen de [clústeres virtuales](connectivity-architecture-overview.md#high-level-connectivity-architecture). Los clústeres virtuales representan un conjunto dedicado de máquinas virtuales aisladas implementadas dentro de la subred de la red virtual del cliente.

La principal diferencia entre los dos modelos de implementación es que los grupos de instancias permiten varias implementaciones de procesos de SQL Server en el mismo nodo de máquina virtual, que se rigen por los recursos mediante [objetos de trabajo de Windows](/windows/desktop/ProcThread/job-objects), mientras que las instancias únicas siempre están solas en un nodo de máquina virtual.

En el diagrama siguiente se muestra un grupo de instancias y dos instancias individuales implementadas en la misma subred y se muestran los detalles principales de la arquitectura de ambos modelos de implementación:

![Grupo de instancias y dos instancias individuales](./media/instance-pools-overview/instance-pools2.png)

Cada grupo de instancias crea un clúster virtual independiente debajo. Las instancias de un grupo y las instancias únicas implementadas en la misma subred no comparten recursos de proceso asignados a los componentes de la puerta de enlace y los procesos de SQL Server, lo que garantiza la predictibilidad del rendimiento.

## <a name="resource-limitations"></a>Limitaciones de recursos

Existen varias limitaciones de recursos con respecto a los grupos de instancias y a las instancias dentro de los grupos:

- Los grupos de instancias solo están disponibles en hardware Gen5.
- Las instancias administradas de un grupo tienen CPU y RAM dedicadas, por lo que el número agregado de núcleos virtuales entre todas las instancias tiene que ser menor o igual que el número de núcleos virtuales asignados al grupo.
- Todos los [límites de nivel de instancia](resource-limits.md#service-tier-characteristics) se aplican a las instancias creadas dentro de un grupo.
- Además de los límites de nivel de instancia, también hay dos límites impuestos *en el nivel de grupo de instancias*:
  - Tamaño de almacenamiento total por grupo (8 TB).
  - Número total de bases de datos de usuario por grupo. Este límite depende del valor de núcleos virtuales del grupo:
    - Un grupo de 8 núcleos virtuales admite hasta 200 bases de datos.
    - Un grupo de 16 núcleos virtuales admite hasta 400 bases de datos.
    - Un grupo de 24 núcleos virtuales y superior admite hasta 500 bases de datos.
- No se puede establecer el administrador de AAD para las instancias implementadas dentro del grupo de instancias, por lo tanto, no se puede usar la autenticación de AAD.

La asignación de almacenamiento total y el número de bases de datos en todas las instancias tiene que ser inferior o igual que los límites expuestos por los grupos de instancias.

- Los grupos de instancias admiten 8, 16, 24, 32, 40, 64 y 80 núcleos virtuales.
- Las instancias administradas dentro de los grupos admiten 2, 4, 8, 16, 24, 32, 40, 64 y 80 núcleos virtuales.
- Las instancias administradas dentro de los grupos admiten tamaños de almacenamiento entre 32 GB y 8 TB, excepto:
  - Las instancias con 2 núcleos virtuales admiten tamaños de entre 32 y 640 GB.
  - Las instancias con 4 núcleos virtuales admiten tamaños de entre 32 GB y 2 TB.
- Las instancias administradas dentro de los grupos tienen un límite de hasta 100 bases de datos de usuario por instancia, excepto las instancias de 2 núcleos virtuales que admiten hasta 50 bases de datos de usuario por instancia.

La [propiedad de nivel de servicio](resource-limits.md#service-tier-characteristics) está asociada al recurso de grupo de instancias, por lo que todas las instancias de un grupo tienen que ser del mismo nivel de servicio que el nivel de servicio del grupo. En este momento, solo está disponible el nivel de servicio De uso general (consulte la sección siguiente sobre las limitaciones de la versión preliminar actual).

### <a name="public-preview-limitations"></a>Limitaciones de la vista preliminar pública

La versión preliminar pública tiene las limitaciones siguientes:

- Actualmente, solo está disponible el nivel de servicio De uso general.
- Los grupos de instancias no se pueden escalar durante la versión preliminar pública, por lo que es importante planear la capacidad antes de la implementación.
- Todavía no está disponible la compatibilidad de Azure Portal con la creación y configuración de los grupos de instancias. Todas las operaciones en grupos de instancias se admiten solo a través de PowerShell. La implementación de instancia inicial en un grupo creado previamente también se admite solo a través de PowerShell. Una vez implementadas en un grupo, las instancias administradas se pueden actualizar con Azure Portal.
- Las instancias administradas creadas fuera del grupo no se pueden mover a un grupo existente, y las instancias creadas dentro de un grupo no se pueden migrar fuera como instancia única o a otro grupo.
- Los precios para instancia de [capacidad reservada](../database/reserved-capacity-overview.md) no están disponibles.

## <a name="sql-features-supported"></a>Características de SQL admitidas

Las instancias administradas creadas en grupos admiten los mismos [niveles de compatibilidad y características compatibles en instancias administradas únicas](sql-managed-instance-paas-overview.md#sql-features-supported).

Cada instancia administrada implementada en un grupo tiene una instancia independiente del Agente SQL.

Las características opcionales o las que requieren que elija valores específicos (como la intercalación de nivel de instancia, la zona horaria, el punto de conexión público para el tráfico de datos, los grupos de conmutación por error) se configuran en el nivel de instancia y pueden ser diferentes para cada instancia de un grupo.

## <a name="performance-considerations"></a>Consideraciones de rendimiento

Aunque las instancias administradas dentro de los grupos tienen RAM y núcleos virtuales dedicados, comparten el disco local (para el uso de tempdb) y los recursos de red. No es probable, pero es posible experimentar el efecto del *entorno ruidoso* si varias instancias del grupo tienen un consumo elevado de recursos al mismo tiempo. Si observa este comportamiento, considere la posibilidad de implementar estas instancias en un grupo más grande o como instancias únicas.

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Dado que las instancias implementadas en un grupo comparten la misma máquina virtual, es posible que quiera considerar la posibilidad de deshabilitar las características que presentan mayores riesgos de seguridad o de controlar estrictamente los permisos de acceso a estas características. Por ejemplo, la integración con CLR, la copia de seguridad y restauración nativa, el correo electrónico de base de datos, etc.

## <a name="instance-pool-support-requests"></a>Solicitudes de soporte técnico de un grupo de instancias

Cree y administre solicitudes de soporte técnico para grupos de instancias en [Azure Portal](https://portal.azure.com).

Si tiene problemas relacionados con la implementación de un grupo de instancias (creación o eliminación), asegúrese de especificar **Grupos de instancias** en el campo **Subtipo de problema**.

![Solicitud de soporte técnico de grupos de instancias](./media/instance-pools-overview/support-request.png)

Si tiene problemas relacionados con bases de datos o instancias administrada únicas dentro de un grupo, debe crear una incidencia de soporte técnico normal para Instancias administrada de Azure SQL.

Para crear implementaciones de instancias administradas de SQL más grandes (con o sin grupos de instancias), es posible que tenga que obtener una cuota regional más grande. Para más información, consulte [Solicitud de aumentos de cuota para Azure SQL Database](../database/quota-increase-request.md). La lógica de implementación para grupos de instancias compara el consumo total de núcleos virtuales *en el nivel de grupo* con respecto a la cuota para determinar si se permite crear recursos nuevos sin aumentar aún más la cuota.

## <a name="instance-pool-billing"></a>Facturación de un grupo de instancias

Los grupos de instancias permiten escalar el proceso y el almacenamiento de manera independiente. Los clientes pagan por el proceso asociado al recurso de grupo medido en núcleos virtuales y el almacenamiento asociado a cada instancia medido en gigabytes (los primeros 32 GB son gratuitos para cada instancia).

El precio de núcleo virtual para un grupo se cobra independientemente del número de instancias que se implementen en ese grupo.

Para el precio de proceso (medido en núcleos virtuales), hay disponibles dos opciones de precios:

  1. *Con licencia incluida*: se incluye el precio de las licencias de SQL Server. Esto es para los clientes que eligen no aplicar las licencias de SQL Server existentes con Software Assurance.
  2. *Ventaja híbrida de Azure*: precio reducido que incluye la Ventaja híbrida de Azure para SQL Server. Los clientes pueden optar a este precio si utilizan sus licencias de SQL Server con Software Assurance. Para información sobre la idoneidad y otros detalles, consulte [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/).

No es posible establecer diferentes opciones de precios para las instancias individuales de un grupo. Todas las instancias del grupo primario deben tener un precio con licencia incluida o precio con Ventaja híbrida de Azure. El modelo de licencia para el grupo se puede modificar una vez creado el grupo.

> [!IMPORTANT]
> Si especifica un modelo de licencia para la instancia que sea diferente que en el grupo, se usa el precio del grupo y se omite el valor de nivel de instancia.

Si crea grupos de instancias en [suscripciones válidas para la ventaja de desarrollo y pruebas](https://azure.microsoft.com/pricing/dev-test/), recibirá automáticamente tarifas con descuento de hasta el 55 % en Instancia administrada de Azure SQL.

Para información detallada sobre los precios de un grupo de instancias, consulte la sección sobre los *grupos de instancias* en la [página de precios de Instancia administrada de SQL](https://azure.microsoft.com/pricing/details/sql-database/managed/).

## <a name="next-steps"></a>Pasos siguientes

- Para una introducción al trabajo con grupos de instancias, consulte [Guía paso a paso sobre los grupos de Instancias administrada de SQL](instance-pools-configure.md).
- Para más información sobre cómo crear su primera instancia administrada, vea la [Guía de inicio rápido](instance-create-quickstart.md).
- Para obtener una lista de características y una comparación, consulte [Características comunes de Azure SQL](../database/features-comparison.md).
- Para más información sobre la configuración de redes virtuales, consulte [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
- Para obtener información sobre la supervisión avanzada del rendimiento de las bases de datos de Instancia administrada de SQL con inteligencia integrada para la solución de problemas, consulte [Supervisión de instancias de Azure SQL Database con Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Para obtener información sobre precios, vea la página de [precios de Instancia administrada de SQL](https://azure.microsoft.com/pricing/details/sql-database/managed/).
