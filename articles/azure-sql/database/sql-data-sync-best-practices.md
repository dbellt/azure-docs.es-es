---
title: Procedimientos recomendados de Azure SQL Data Sync
description: Conozca los procedimientos recomendados para configurar y ejecutar Azure SQL Data Sync.
services: sql-database
ms.service: sql-database
ms.subservice: sql-data-sync
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MaraSteiu
ms.author: masteiu
ms.reviewer: mathoma
ms.date: 12/20/2018
ms.openlocfilehash: e4a08dca9fe30b632b2653270ff6a41c6c081b41
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689757"
---
# <a name="best-practices-for-azure-sql-data-sync"></a>Procedimientos recomendados de Azure SQL Data Sync 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo, se describen los procedimientos recomendados para Azure SQL Data Sync.

Para obtener información general acerca de SQL Data Sync, consulte [Sincronización de datos entre varias bases de datos locales y de la nube con Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Azure SQL Data Sync **no** es compatible de momento con Instancia administrada de Azure SQL.

## <a name="security-and-reliability"></a><a name="security-and-reliability"></a>Seguridad y fiabilidad

### <a name="client-agent"></a>Agente cliente

-   Instale el agente cliente con la cuenta de usuario con menos privilegios que tenga acceso al servicio de red.  
-   Instale el agente cliente en un equipo que no sea el de SQL Server.  
-   No registre ninguna base de datos local con más de un agente.    
    -   Evite esto aunque vaya a sincronizar tablas diferentes para distintos grupos de sincronización.  
    -   Registrar una base de datos local con varios agentes cliente puede causar problemas al eliminar uno de los grupos de sincronización.

### <a name="database-accounts-with-least-required-privileges"></a>Cuentas de base de datos con los mínimos privilegios requeridos

-   **Para la configuración de la sincronización**. Crear/modificar tabla, modificar base de datos, crear procedimiento, seleccionar/modificar esquema, crear el tipo definido por el usuario.

-   **Para la sincronización en curso**. Seleccionar/insertar/actualizar/eliminar en tablas seleccionadas para la sincronización y en tablas de seguimiento y metadatos de la sincronización, ejecutar un permiso en procedimientos almacenados creados por el servicio, ejecutar un permiso en tipos de tablas definidas por el usuario.

-   **Para el desaprovisionamiento**. Modificar en tablas que formen parte de la sincronización, seleccionar/eliminar en tablas de metadatos de la sincronización, controlar en tablas de seguimiento de la sincronización, procedimientos almacenados y tipos definidos por el usuario.

Azure SQL Database solo admite un único conjunto de credenciales. Para realizar estas tareas dentro de esta restricción, tenga en cuenta las siguientes opciones:

-   Cambie las credenciales para las distintas fases (por ejemplo, utilice *credencial1* para la configuración y *credencial2* para la sincronización en curso).  
-   Cambie el permiso de las credenciales (es decir, cambie el permiso una vez configurada la sincronización).

### <a name="auditing"></a>Auditoría

Se recomienda habilitar la auditoría en el nivel de las bases de datos en los grupos de sincronización. 

## <a name="setup"></a>Configurar

### <a name="database-considerations-and-constraints"></a><a name="database-considerations-and-constraints"></a> Restricciones y consideraciones de la base de datos

#### <a name="database-size"></a>Tamaño de base de datos

Al crear una nueva base de datos, establezca el tamaño máximo de forma que siempre sea mayor que la base de datos que implemente. Si no lo hace así, se producirá un error de sincronización. Aunque SQL Data Sync no permite el crecimiento automático, puede ejecutar el comando `ALTER DATABASE` para aumentar el tamaño de la base de datos una vez creada. Asegúrese de que no se sale de los límites de tamaño de la base de datos.

> [!IMPORTANT]
> SQL Data Sync almacena metadatos adicionales con cada base de datos. No olvide tener en cuenta estos metadatos al calcular el espacio necesario. La cantidad de sobrecarga adicional guarda relación con la cantidad de tráfico y con el ancho de las tablas (por ejemplo, las tablas estrechas necesitan más sobrecarga).

### <a name="table-considerations-and-constraints"></a><a name="table-considerations-and-constraints"></a> Restricciones y consideraciones de la tabla

#### <a name="selecting-tables"></a>Selección de tablas

No deben incluir en un grupo de sincronización todas las tablas que se encuentran en una base de datos. Las tablas que se incluyen en un grupo de sincronización afectan a la eficiencia y los costos. Incluya tablas y también las tablas que de ellas dependan en un grupo de sincronización solo si así lo requieren las necesidades empresariales.

#### <a name="primary-keys"></a>Claves principales

Cada tabla de un grupo de sincronización debe tener una clave principal. SQL Data Sync no puede sincronizar una tabla que no tiene una clave principal.

Antes de usar SQL Data Sync en producción, compruebe el rendimiento de la sincronización inicial y en curso.

#### <a name="empty-tables-provide-the-best-performance"></a>Las tablas vacías proporcionan el mejor rendimiento.

Las tablas vacías ofrecen el mejor rendimiento en el momento de la inicialización. Si la tabla de destino está vacía, las sincronización de datos utiliza la inserción masiva para cargar los datos. En caso contrario, la sincronización de datos realiza una comparación fila a fila y la inserción para comprobar si hay conflictos. Aunque, si el rendimiento no es un problema, puede configurar la sincronización entre tablas que ya contengan datos.

### <a name="provisioning-destination-databases"></a><a name="provisioning-destination-databases"></a> Aprovisionamiento de bases de datos de destino

SQL Data Sync permite aprovisionar automáticamente la base de datos básica.

En esta sección, se describen los límites del aprovisionamiento de SQL Data Sync.

#### <a name="autoprovisioning-limitations"></a>Limitaciones del aprovisionamiento automático

SQL Data Sync tiene las siguientes limitaciones de aprovisionamiento automático:

-   Seleccione solo las columnas creadas en la tabla de destino. Todas las columnas que no forman parte del grupo de sincronización no se aprovisionan en las tablas de destino.
-   Los índices se crean únicamente para las columnas seleccionadas. Si el índice de la tabla de origen tiene columnas que no forman parte del grupo de sincronización, estos índices no se aprovisionan en las tablas de destino.  
-   Los índices de las columnas de tipo XML no se aprovisionan.  
-   Las restricciones CHECK no se aprovisionan.  
-   Los desencadenadores existentes en las tablas de origen no se aprovisionan.  
-   No se crean vistas ni procedimientos almacenados en la base de datos de destino.
-   Las acciones ON UPDATE CASCADE Y ON DELETE CASCADE en las restricciones clave externas no se vuelven a crear en las tablas de destino.
-   Si tiene columnas decimales o numéricas con una precisión mayor a 28, SQL Data Sync puede encontrar un problema de desbordamiento de conversión durante la sincronización. Se recomienda que limite la precisión de las columnas decimales o numéricas a 28 o menos.

#### <a name="recommendations"></a>Recomendaciones

-   Use la funcionalidad de aprovisionamiento automático de SQL Data Sync solo cuando vaya a probar el servicio.  
-   Para entornos de producción, aprovisione el esquema de base de datos.

### <a name="where-to-locate-the-hub-database"></a><a name="locate-hub"></a> Dónde encontrar la base de datos central

#### <a name="enterprise-to-cloud-scenario"></a>Escenario de empresa a la nube

Para minimizar la latencia, mantenga la base de datos central cerca de la mayor concentración del tráfico de la base de datos del grupo de sincronización.

#### <a name="cloud-to-cloud-scenario"></a>Escenario de nube a nube

-   Cuando todas las bases de datos de un grupo de sincronización están en un centro de datos, el concentrador debe estar en el mismo centro de datos. Esta configuración reduce la latencia y el costo de la transferencia de datos entre centros de datos.
-   Cuando las bases de datos de un grupo de sincronización están en varios centros de datos, el concentrador debe estar en el mismo centro de datos que la mayoría de las bases de datos y del tráfico de la base de datos.

#### <a name="mixed-scenarios"></a>Escenarios combinados

Aplique las instrucciones anteriores a configuraciones de grupos de sincronización complejas, como aquellas en las que existe una combinación de escenarios de la empresa a la nube y de la nube a la nube.

## <a name="sync"></a>Sync

### <a name="avoid-slow-and-costly-initial-sync"></a><a name="avoid-a-slow-and-costly-initial-synchronization"></a> Evitar una sincronización inicial lenta y costosa

En esta sección se describe la sincronización inicial de un grupo de sincronización. Obtenga información sobre cómo ayudar a evitar que una sincronización inicial tarde más tiempo y sea más costosa de lo necesario.

#### <a name="how-initial-sync-works"></a>Funcionamiento de la sincronización inicial

Al crear un grupo de sincronización, comience con los datos en una sola base de datos. Si tiene datos en varias bases de datos, SQL Data Sync trata cada fila como un conflicto que hay que resolver. La resolución de conflictos hace que la sincronización inicial sea lenta. Si tiene datos en varias bases de datos, la sincronización inicial puede tardar entre varios días y varios meses, según el tamaño de la base de datos.

Si las bases de datos están en distintos centros de datos, cada fila debe desplazarse entre ellos. Esto aumenta el costo de una sincronización inicial.

#### <a name="recommendation"></a>Recomendación

Si es posible, comience con datos en una sola base de datos del grupo de sincronización.

### <a name="design-to-avoid-sync-loops"></a><a name="design-to-avoid-synchronization-loops"></a> Diseño para evitar bucles de sincronización

Un bucle de sincronización se produce cuando hay referencias circulares dentro de un grupo de sincronización. En ese caso, cada cambio realizado en una base de datos resulta interminable y se replica de forma circular a través de las bases de datos del grupo de sincronización.   

Asegúrese de evitar los bucles de sincronización, ya que causan una degradación del rendimiento y podrían aumentar significativamente los costos.

### <a name="changes-that-fail-to-propagate"></a><a name="handling-changes-that-fail-to-propagate"></a> Cambios que no se propagan

#### <a name="reasons-that-changes-fail-to-propagate"></a>Razones por las que los cambios no se propagan

Puede que los cambios no se propaguen por alguno de los siguientes motivos:

-   Incompatibilidad de esquemas o tipos de datos.
-   Insertar valores NULL en columnas que no aceptan valores NULL.
-   Infringir restricciones de claves principales.

#### <a name="what-happens-when-changes-fail-to-propagate"></a>¿Qué ocurre cuando los cambios no se propagan?

-   Se muestra un estado de **advertencia** en el grupo de sincronización.
-   Los detalles se encuentran en el visor de registro de la interfaz de usuario del portal.
-   Si el problema no se resuelve en 45 días, la base de datos se convierte en obsoleta.

> [!NOTE]
> Estos cambios no se propagan nunca. La única manera de recuperarse en este caso es volver a crear el grupo de sincronización.

#### <a name="recommendation"></a>Recomendación

Supervise el estado del grupo de sincronización y la base de datos con regularidad desde la interfaz del portal y de registro.


## <a name="maintenance"></a>Mantenimiento

### <a name="avoid-out-of-date-databases-and-sync-groups"></a><a name="avoid-out-of-date-databases-and-sync-groups"></a> Evitar bases de datos y grupos de sincronización obsoletos

Un grupo de sincronización o una base de datos de un grupo de sincronización pueden llegar a estar obsoletos. Cuando el estado de un grupo de sincronización es **obsoleto**, deja de funcionar. Cuando el estado de la base de datos es **obsoleto**, se pueden perder datos. Es mejor evitar esta situación en lugar de intentar recuperarse de este error.

#### <a name="avoid-out-of-date-databases"></a>Evitar bases de datos obsoletas

El estado de la base de datos se establece en **obsoleto** cuando ha estado sin conexión durante cuarenta y cinco días o más. Para evitar el estado **obsoleto** en una base de datos, asegúrese de que ninguna de las bases de datos esté sin conexión durante cuarenta y cinco días o más.

#### <a name="avoid-out-of-date-sync-groups"></a>Evitar grupos de sincronización obsoletos

El estado de un grupo de sincronización se establece en **obsoleto** cuando cualquier cambio en el grupo de sincronización no puede propagarse al resto del grupo de sincronización durante cuarenta y cinco días o más. Para evitar el estado **obsoleto** de un grupo de sincronización, compruebe periódicamente el registro de historial del grupo de sincronización. Asegúrese de que todos los conflictos se resuelvan y los cambios se propaguen correctamente en las bases de datos del grupo de sincronización.

Un grupo de sincronización puede producir un error al aplicar un cambio por alguno de estos motivos:

-   Incompatibilidad de esquemas entre tablas.
-   Incompatibilidad de datos entre tablas.
-   Insertar una fila con un valor NULL en una columna que no admite valores NULL.
-   Actualizar una fila con un valor que infringe una restricción de clave externa.

Para impedir grupos de sincronización obsoletos, haga lo siguiente:

-   Actualice el esquema para permitir los valores incluidos en las filas con errores.
-   Actualice los valores de la clave externa para incluir los valores incluidos en las filas con errores.
-   Actualice los valores de datos en la fila con errores para que sean compatibles con el esquema o con las claves externas de la base de datos de destino.

### <a name="avoid-deprovisioning-issues"></a><a name="avoid-deprovisioning-issues"></a> Evitar problemas de cancelación del aprovisionamiento

En determinadas circunstancias, anular el registro de una base de datos con un agente cliente puede provocar errores de sincronización.

#### <a name="scenario"></a>Escenario

1. El grupo de sincronización A se ha creado con una instancia de SQL Database y una base de datos de SQL Server asociada al agente local 1.
2. La misma base de datos local está registrada con el agente local 2 (este agente no está asociado a ningún grupo de sincronización).
3. Al anular el registro de la base de datos local del agente local 2, se eliminan las tablas de seguimiento y de metadatos del grupo de sincronización A en la base de datos local.
4. En las operaciones del grupo de sincronización A, se producirá el error: "La operación actual no se pudo completar porque la base de datos no está aprovisionada para la sincronización o no tiene permisos para las tablas de configuración de sincronización".

#### <a name="solution"></a>Solución

Para evitar esta situación, no registre una base de datos con más de un agente.

Para recuperarse de esta situación:

1. Quite la base de datos de todos los grupos de sincronización a los que pertenezca.  
2. Vuelva a agregar la base de datos a cada uno de los grupos de sincronización de los que la ha quitado.  
3. Implemente cada grupo de sincronización afectado (esta acción aprovisiona la base de datos).  

### <a name="modifying-a-sync-group"></a><a name="modifying-your-sync-group"></a> Modificación de un grupo de sincronización

No intente quitar una base de datos de un grupo de sincronización para después editar el grupo de sincronización sin implementar primero uno de los cambios.

En su lugar, primero quite una base de datos de un grupo de sincronización. A continuación, implemente el cambio y espere a que el desaprovisionamiento finalice. Una vez finalizado el desaprovisionamiento, puede editar el grupo de sincronización e implementar los cambios.

Si intenta quitar una base de datos para después modificar un grupo de sincronización sin implementar primero uno de los cambios, se produce un error en una de las dos operaciones. La interfaz del portal puede quedar en un estado incoherente. En este caso, actualice la página para restaurar el estado correcto.

### <a name="avoid-schema-refresh-timeout"></a>Evitar el tiempo de espera de actualización del esquema

Si tiene un esquema complejo para la sincronización, es posible que experimente un "tiempo de espera de la operación" durante una actualización del esquema si la base de datos de metadatos de sincronización tiene una SKU inferior (ejemplo: básica). 

#### <a name="solution"></a>Solución

Para mitigar este problema, escale verticalmente la base de datos de metadatos de sincronización para que tenga una SKU superior, como S3. 

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de SQL Data Sync, consulte:

-   Introducción: [Sincronización de datos entre varias bases de datos locales y en la nube con Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
-   Configuración de SQL Data Sync
    - En el portal, [Tutorial: Configuración de SQL Data Sync para sincronizar datos entre Azure SQL Database y SQL Server](sql-data-sync-sql-server-configure.md)
    - Con PowerShell
        -  [Uso de PowerShell para sincronizar varias bases de datos de Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Uso de PowerShell para sincronizar una base de datos de SQL Database y una base de datos de una instancia de SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Agente de sincronización de datos: [Agente de sincronización de datos para Azure SQL Data Sync](sql-data-sync-agent-overview.md)
-   Supervisión: [Monitor SQL Data Sync with Azure Monitor logs](./monitor-tune-overview.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)
-   Solución de problemas: [Solución de problemas de Azure SQL Data Sync](sql-data-sync-troubleshoot.md)
-   Actualización del esquema de sincronización
    -   Con Transact-SQL: [Automatización de la replicación de los cambios de esquema en Azure SQL Data Sync](sql-data-sync-update-sync-schema.md)
    -   Con PowerShell: [Usar PowerShell para actualizar el esquema de sincronización en un grupo de sincronización existente](scripts/update-sync-schema-in-sync-group.md)

Para obtener más información sobre SQL Database, vea:

-   [Información general de SQL Database](sql-database-paas-overview.md)
-   [Administración del ciclo de vida de las aplicaciones](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))
