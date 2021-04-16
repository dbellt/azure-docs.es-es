---
title: 'Parámetros del servidor (Azure Database for MySQL): servidor flexible'
description: 'En este tema se proporcionan instrucciones para configurar parámetros del servidor en Azure Database for MySQL: servidor flexible.'
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 11/10/2020
ms.openlocfilehash: d64dc4f3c034279aee7401503bbb60883c9ed4e7
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106492246"
---
# <a name="server-parameters-in-azure-database-for-mysql---flexible-server"></a>Parámetros del servidor en Azure Database for MySQL: servidor flexible

> [!IMPORTANT]
> Actualmente, la opción de implementación Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

En este artículo se incluyen consideraciones e instrucciones para configurar parámetros del servidor en el servidor flexible de Azure Database for MySQL.

## <a name="what-are-server-variables"></a>¿Qué son las variables de servidor? 

El motor MySQL proporciona muchas [variables o parámetros de servidor](https://dev.mysql.com/doc/refman/5.7/en/server-option-variable-reference.html) diferentes que se pueden usar para configurar y ajustar el comportamiento del motor. Algunos parámetros se pueden establecer de forma dinámica en tiempo de ejecución, mientras que otros son "estáticos", requiriendo un reinicio del servidor para su aplicación.

El servidor flexible de Azure Database for MySQL expone la capacidad de cambio del valor de diversos parámetros del servidor MySQL mediante [Azure Portal](./how-to-configure-server-parameters-portal.md) y la [CLI de Azure](./how-to-configure-server-parameters-cli.md) para satisfacer las necesidades de la carga de trabajo.

## <a name="configurable-server-parameters"></a>Parámetros configurables del servidor

Puede administrar la configuración de Azure Database for MySQL: servidor flexible con los parámetros de servidor. Los parámetros de servidor se configuran con los valores predeterminados y recomendados al crear el servidor. La hoja de parámetros de servidor de Azure Portal muestra el parámetro de servidor modificable y no modificable. Los parámetros de servidor no modificables están atenuados.

La lista de parámetros del servidor admitidos crece constantemente. Use la pestaña Parámetros del servidor de Azure Portal para ver la lista completa y configurar valores de los parámetros del servidor.

Consulte las siguientes secciones para obtener más información sobre los límites de los diferentes parámetros del servidor actualizados con más frecuencia. Los límites vienen determinados por el nivel de proceso y el tamaño (núcleos virtuales) del servidor.

> [!NOTE]
> Si quiere modificar un parámetro de servidor que no es modificable pero desea verlo como modificable para su entorno, abra un elemento de [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql) o, si el comentario ya existe, vótelo para ayudarnos a priorizar.

### <a name="log_bin_trust_function_creators"></a>log_bin_trust_function_creators

En el servidor flexible de Azure Database for MySQL, siempre están habilitados los registros binarios (p. ej., `log_bin` está establecido en ACTIVADO). log_bin_trust_function_creators se establece en ON de manera predeterminada en servidores flexibles. 

El formato de registro binario siempre es **ROW** y todas las conexiones al servidor **ALWAYS** usan un registro binario basado en filas. Con el registro binario basado en filas, no existen problemas de seguridad y el registro binario no se puede interrumpir, de modo que puede habilitar [`log_bin_trust_function_creators`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators) para que se establezca en **ON** de manera segura.

Si [`log_bin_trust_function_creators`] está establecido en OFF, cuando intente crear desencadenadores podría obtener errores similares a *No tiene el privilegio SUPER y el registro binario está habilitado (se recomienda usar la variable menos segura`log_bin_trust_function_creators`)* . 

### <a name="innodb_buffer_pool_size"></a>innodb_buffer_pool_size

Consulte la [documentación de MySQL](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_buffer_pool_size) para más información acerca de este parámetro.

|**Plan de tarifa**|**Núcleos virtuales**|**Tamaño de memoria (GiB)**|**Valor predeterminado (bytes)**|**Valor mínimo (bytes)**|**Valor máximo (bytes)**|
|---|---|---|---|---|---|
|Ampliable (B1s)|1|1|134217728|33554432|134217728|
|Ampliable (B1ms)|1|2|536870912|134217728|536870912|
|Flexible|2|4|2147483648|134217728|2147483648|
|De uso general|2|8|6442450944|134217728|6442450944|
|De uso general|4|16|12884901888|134217728|12884901888|
|De uso general|8|32|25769803776|134217728|25769803776|
|De uso general|16|64|51539607552|134217728|51539607552|
|De uso general|32|128|103079215104|134217728|103079215104|
|De uso general|48|192|154618822656|134217728|154618822656|
|De uso general|64|256|206158430208|134217728|206158430208|
|Memoria optimizada|2|16|12884901888|134217728|12884901888|
|Memoria optimizada|4|32|25769803776|134217728|25769803776|
|Memoria optimizada|8|64|51539607552|134217728|51539607552|
|Memoria optimizada|16|128|103079215104|134217728|103079215104|
|Memoria optimizada|32|256|206158430208|134217728|206158430208|
|Memoria optimizada|48|384|309237645312|134217728|309237645312|
|Memoria optimizada|64|504|405874409472|134217728|405874409472|

### <a name="innodb_file_per_table"></a>innodb_file_per_table

MySQL almacena la tabla InnoDB en distintos espacios de tabla en función de la configuración proporcionada durante la creación de la tabla. El [espacio de tablas del sistema](https://dev.mysql.com/doc/refman/5.7/en/innodb-system-tablespace.html) es el área de almacenamiento del diccionario de datos de InnoDB. Un [espacio de tabla de archivo por tabla](https://dev.mysql.com/doc/refman/5.7/en/innodb-file-per-table-tablespaces.html) contiene datos e índices para una sola tabla de InnoDB y se almacena en el sistema de archivos, en su propio archivo de datos. Este comportamiento se controla mediante el parámetro de servidor `innodb_file_per_table`. Si `innodb_file_per_table` se establece en `OFF`, InnoDB crea tablas en el espacio de tablas del sistema. De lo contrario, InnoDB crea tablas en espacios de tabla de archivo por tabla.

El servidor flexible de Azure Database for MySQL admite **4 TB** como máximo en un solo archivo de datos. Si el tamaño de la base de datos es superior a 4 TB, debería crear la tabla en el espacio de tabla [innodb_file_per_table](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table). Si tiene una sola tabla de tamaño superior a 4 TB, debería usar la tabla de particiones.

### <a name="max_connections"></a>max_connections

El valor de max_connection viene determinado por el tamaño de la memoria del servidor. 

|**Plan de tarifa**|**Núcleos virtuales**|**Tamaño de memoria (GiB)**|**Valor predeterminado**|**Valor mínimo**|**Valor máximo**|
|---|---|---|---|---|---|
|Ampliable (B1s)|1|1|85|10|171|
|Ampliable (B1ms)|1|2|171|10|341|
|Flexible|2|4|341|10|683|
|De uso general|2|8|683|10|1365|
|De uso general|4|16|1365|10|2731|
|De uso general|8|32|2731|10|5461|
|De uso general|16|64|5461|10|10923|
|De uso general|32|128|10923|10|21845|
|De uso general|48|192|16384|10|32 768|
|De uso general|64|256|21845|10|43691|
|Memoria optimizada|2|16|1365|10|2731|
|Memoria optimizada|4|32|2731|10|5461|
|Memoria optimizada|8|64|5461|10|10923|
|Memoria optimizada|16|128|10923|10|21845|
|Memoria optimizada|32|256|21845|10|43691|
|Memoria optimizada|48|384|32 768|10|65536|
|Memoria optimizada|64|504|43008|10|86016|

Si las conexiones superan el límite, puede que reciba el error siguiente:
> ERROR 1040 (08004): Demasiadas conexiones

> [!IMPORTANT]
> Para obtener la mejor experiencia posible, se recomienda usar un agrupador de conexiones, como ProxySQL, para administrar las conexiones de forma eficaz.

La creación de conexiones de cliente a MySQL lleva tiempo y, una vez establecidas, estas conexiones ocupan recursos de bases de datos, incluso cuando están inactivas. La mayoría de las aplicaciones solicitan muchas conexiones de corta duración, y esto es lo que conforma esta situación. El resultado es que hay menos recursos disponibles para la carga de trabajo real, lo que baja el rendimiento. Esto se puede evitar con un agrupador de conexiones, ya que reduce las conexiones inactivas y reutiliza las conexiones existentes. Para más información sobre cómo configurar ProxySQL, consulte nuestra [entrada de blog](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/load-balance-read-replicas-using-proxysql-in-azure-database-for/ba-p/880042).

>[!Note]
>ProxySQL es una herramienta de la comunidad de código abierto. Es compatible con Microsoft dentro de lo posible. Para obtener soporte a la producción con instrucciones autoritativas, puede evaluar y ponerse en contacto con el [servicio de soporte técnico de ProxySQL](https://proxysql.com/services/support/).

### <a name="innodb_strict_mode"></a>innodb_strict_mode

Si recibe un error similar a "Tamaño de la fila demasiado grande (> 8126)", puede desactivar el parámetro **innodb_strict_mode**. El parámetro de servidor **innodb_strict_mode** no se puede modificar globalmente en el nivel de servidor porque, si el tamaño de los datos de fila es superior a 8 KB, se truncarán los datos sin que se produzca ningún error, lo que puede causar pérdida de datos. Se recomienda modificar el esquema para ajustarlo al límite de tamaño de página. 

Este parámetro se puede establecer en un nivel de sesión mediante `init_connect`. Para establecer **innodb_strict_mode** en el nivel de sesión, consulte cómo [ajustar un parámetro que no aparece en la lista](./how-to-configure-server-parameters-portal.md#setting-non-modifiable-server-parameters).

> [!NOTE]
> Si tiene un servidor de réplica de lectura, al establecer **innodb_strict_mode** como desactivado en el nivel de sesión de un servidor de origen, se interrumpirá la replicación. Se recomienda mantener el parámetro establecido así si tiene réplicas de lectura.

### <a name="time_zone"></a>time_zone

Tras la implementación inicial, un servidor flexible de Azure for MySQL incluye tablas de sistemas para la información de zona horaria, pero estas tablas no se rellenan. Las tablas de la zona horaria se pueden rellenar mediante una llamada al procedimiento almacenado `mysql.az_load_timezone` desde una herramienta como la línea de comandos de MySQL o MySQL Workbench. Vea los artículos de [Azure Portal](./how-to-configure-server-parameters-portal.md#working-with-the-time-zone-parameter) o de la [CLI de Azure](./how-to-configure-server-parameters-cli.md#working-with-the-time-zone-parameter) sobre cómo llamar al procedimiento almacenado y establecer las zonas horarias globales o de nivel de sesión.

## <a name="non-modifiable-server-parameters"></a>Parámetros de servidor no modificables

La hoja de parámetros de servidor de Azure Portal muestra los parámetros de servidor modificables y no modificables. Los parámetros de servidor no modificables están atenuados. Si quiere configurar un parámetro de servidor no modificable en el nivel de sesión, consulte el artículo de [Azure Portal](./how-to-configure-server-parameters-portal.md#setting-non-modifiable-server-parameters) o la [CLI de Azure](./how-to-configure-server-parameters-cli.md#setting-non-modifiable-server-parameters) para establecer el parámetro en el nivel de conexión mediante `init_connect`.

## <a name="next-steps"></a>Pasos siguientes

- Cómo configurar [parámetros del servidor en Azure Portal](./how-to-configure-server-parameters-portal.md)
- Cómo configurar [parámetros de servidor en la CLI de Azure](./how-to-configure-server-parameters-cli.md)
