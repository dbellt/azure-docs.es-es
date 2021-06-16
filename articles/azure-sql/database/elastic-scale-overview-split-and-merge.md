---
title: Moving data between scaled-out cloud databases (Mover datos entre bases de datos en la nube escaladas horizontalmente)
description: Explica cómo manipular las particiones y mover los datos a través de un servicio autohospedado mediante las API de bases de datos elásticas.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: conceptual
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 03/12/2019
ms.openlocfilehash: cfd33ad39e48201208c41cf7351910dafa614e84
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708698"
---
# <a name="moving-data-between-scaled-out-cloud-databases"></a>Moving data between scaled-out cloud databases (Mover datos entre bases de datos en la nube escaladas horizontalmente)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Si es desarrollador de Software como servicio y su aplicación experimenta de pronto una demanda enorme, es necesario adaptarse al crecimiento. De este modo, agrega más bases de datos (particiones). ¿Cómo redistribuye los datos en las nuevas bases de datos sin interrumpir la integridad de los datos? Use la **herramienta de división y combinación** para mover datos desde bases de datos limitadas a las nuevas bases de datos.  

La herramienta de división y combinación se ejecuta como servicio web de Azure. Un administrador o desarrollador usa la herramienta para mover shardlets (datos de una partición) entre distintas bases de datos (particiones). La herramienta usa la administración de mapas de particiones para mantener la base de datos de metadatos de servicio y garantizar asignaciones coherentes.

![Información general][1]

## <a name="download"></a>Descargar

[Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge/)

## <a name="documentation"></a>Documentación

1. [Tutorial de la herramienta de división y combinación de bases de datos elásticas](elastic-scale-configure-deploy-split-and-merge.md)
2. [Configuración de seguridad de división y combinación](elastic-scale-split-merge-security-configuration.md)
3. [Split-merge security considerations](elastic-scale-split-merge-security-configuration.md)
4. [Administración de mapas de particiones.](elastic-scale-shard-map-management.md)
5. [Migrate existing databases to scale-out](elastic-convert-to-use-elastic-tools.md)
6. [Elastic database tools](elastic-scale-introduction.md)
7. [Glosario de las herramientas de bases de datos elásticas](elastic-scale-glossary.md)

## <a name="why-use-the-split-merge-tool"></a>¿Por qué usar la herramienta de división y combinación?

- **Flexibilidad**

  Las aplicaciones necesitan extenderse de manera flexible más allá de los límites de una base de datos única en Azure SQL Database. Use la herramienta para mover datos según sea necesario a nuevas bases de datos conservando la integridad.

- **Dividir para aumentar**

  Para aumentar la capacidad general de cara a controlar el crecimiento explosivo, cree capacidad adicional; para ello, particione los datos y distribúyalos entre más bases de datos de manera incremental hasta satisfacer las necesidades de capacidad. Este es un buen ejemplo de la característica de **división**.

- **Combinar para reducir**

  La capacidad debe reducirse debido a la naturaleza estacional de una empresa. La herramienta permite reducir verticalmente a menos unidades de escalado cuando la empresa va lenta. La característica "combinar" del servicio de División y combinación de escalado elástico abarca este requisito.

- **Administrar zonas activas moviendo shardlets**

  Con varios inquilinos por base de datos, la asignación de shardlets a particiones puede generar cuellos de botella en la capacidad de algunas particiones. Esto requiere volver a asignar shardlets o mover shardlets ocupados a particiones nuevas o menos utilizadas.

## <a name="concepts--key-features"></a>Conceptos y características clave

- **Servicios hospedados por el cliente**

  El servicio División y combinación es un servicio hospedado por el cliente. Debe implementar y hospedar el servicio en su suscripción a Microsoft Azure. El paquete que descarga de NuGet contiene una plantilla de configuración que se debe completar con la información correspondiente a la implementación específica. Consulte el [tutorial de División y combinación](elastic-scale-configure-deploy-split-and-merge.md) para obtener más detalles. Dado que el servicio se ejecuta en su suscripción de Azure, es posible controlar y configurar la mayoría de los aspectos de seguridad del servicio. La plantilla predeterminada incluye las opciones para configurar TLS, autenticación de cliente basada en certificados, cifrado para credenciales almacenadas, protección ante denegación de servicio y restricciones de IP. Puede encontrar más información sobre los aspectos de seguridad en el siguiente documento [Configuración de seguridad de división y combinación](elastic-scale-split-merge-security-configuration.md).

  El servicio implementado de manera predeterminada se ejecuta con un rol de trabajo y un rol web. Cada uno de ellos usa el tamaño A1 de máquina virtual en Azure Cloud Services. A pesar de que no puede modificar esta configuración cuando implementa el paquete, sí puede cambiarla después de una implementación exitosa en el servicio en la nube que está en ejecución (a través del portal de Azure). Tenga en cuenta que el rol de trabajo, por razones técnicas, no se debe configurar para más de una instancia.

- **Integración de mapas de particiones**

  El servicio División y combinación interactúa con el mapa de particiones de la aplicación. Cuando use el servicio División y combinación para dividir o combinar intervalos o para mover shardlets entre particiones, el servicio mantiene actualizado automáticamente el mapa de particiones. Para ello, el servicio se conecta a la base de datos del administrador de mapa de particiones de la aplicación y mantiene los intervalos y asignaciones a medida que progresan las solicitudes de división/combinación/desplazamiento. Esto garantiza que el mapa de particiones siempre presente una vista actualizada cuando las operaciones de División y combinación se estén realizando. Las operaciones de división, combinación y desplazamiento de shardlets se implementan moviendo un lote de shardlets desde la partición de origen a la partición de destino. Durante la operación de desplazamiento de shardlets, los shardlets sujetos al lote actual se marcan como sin conexión en el mapa de particiones y no están disponibles para las conexiones de enrutamiento dependiente de los datos mediante la API **OpenConnectionForKey** .

- **Conexiones coherentes de shardlets**

  Cuando se inicia el movimiento de datos para un lote de shardlets nuevo, se cierra toda conexión de enrutamiento dependiente de los datos proporcionada por el mapa de particiones a la partición que almacena el shardlet, además, las conexiones subsiguientes desde las API de mapa de particiones a los shardlets se bloquean mientras el desplazamiento de los datos está en curso a fin de evitar incoherencias. También se cierran las conexiones a otros shardlets de la misma partición, pero se realizarán correctamente de nuevo inmediatamente al reintentarlo. Una vez que se mueve el lote, los shardlets se marcan nuevamente como en línea para la partición de destino y los datos de origen se quitan de la partición de origen. El servicio pasa por estos pasos para cada lote, hasta que se han movido todos los shardlets. Esto dará lugar a operaciones de cierre de varias conexiones durante el curso de la operación de división/combinación/desplazamiento completa.  

- **Administración de la disponibilidad de shardlets**

  limitar el cierre de las conexiones al lote actual de shardlets, tal como se mencionó anteriormente, restringe el ámbito de la no disponibilidad a un lote de shardlets a la vez. Esto se prefiere sobre un enfoque en el que la partición completa deba permanecer sin conexión para todos sus shardlets durante el curso de una operación de división o combinación. El tamaño de un lote, definido como el número de los distintos shardlets que se deben mover a la vez, es un parámetro de configuración. Se puede definir para cada operación de división y combinación, dependiendo de la disponibilidad de la aplicación y de las necesidades de rendimiento. Tenga en cuenta que el intervalo que se está bloqueando en el mapa de particiones puede ser más grande que el tamaño del bloque especificado. Esto se debe a que el servicio escoge el tamaño del intercalo de manera tal que el número real de valores clave de particionamiento en los datos coincida, de manera aproximada, con el tamaño del lote. Es importante recordar esto, principalmente para las claves de particionamiento rellenas de manera dispersa.

- **Almacenamiento de metadatos**

  El servicio División y combinación usa una base de datos para mantener su estado y conservar registros durante el procesamiento de las solicitudes. El usuario crea esta base de datos en su suscripción y proporciona la cadena de conexión para esta en el archivo de configuración para la implementación del servicio. Los administradores de la organización del usuario también pueden conectarse a esta base de datos para revisar el progreso de la solicitud y para investigar información detallada relacionada con posibles errores.

- **Reconocimiento de particionamiento**

  El servicio División y combinación hace la diferencia entre (1) tablas particionadas, (2) tablas de referencia y (3) tablas normales. La semántica de una operación de división/combinación/desplazamiento depende del tipo de tabla usada y se define de la siguiente manera:

  - **Tablas particionadas**

    Las operaciones de división, combinación y desplazamiento mueven los shardlets desde la partición de origen hacia la partición de destino. Tras finalizar correctamente la solicitud general, esos shardlets dejan de estar presentes en la partición de origen. Observe que las tablas de destino deben existir en la partición de destino y no pueden contener datos en el intervalo de destino antes del procesamiento de la operación.

  - **Tablas de referencia**

    En el caso de las tablas de referencia, las operaciones de división, combinación y desplazamiento copian los datos de la partición de origen a la partición de destino. Observe que, sin embargo, no se producen cambios en la partición de destino de una tabla determinada si ya hay presente alguna fila en esta tabla en la partición de destino. La tabla debe estar vacía para que se procese la operación de copia de cualquier tabla de referencia.

  - **Otras tablas**

    Es posible que existan otras tablas en el origen o el destino de una operación de división y combinación. El servicio División y combinación omite estas tablas en cualquier operación de copia o de desplazamiento de datos. Sin embargo, observe que pueden interferir con estas operaciones en caso de restricciones.

    Las API `SchemaInfo` proporcionan información comparativa sobre las tablas de referencia y compartidas en el mapa de particiones. En el siguiente ejemplo se ilustra el uso de estas API en un objeto dado de Shard Map Manager.

    ```csharp
    // Create the schema annotations
    SchemaInfo schemaInfo = new SchemaInfo();

    // reference tables
    schemaInfo.Add(new ReferenceTableInfo("dbo", "region"));
    schemaInfo.Add(new ReferenceTableInfo("dbo", "nation"));

    // sharded tables
    schemaInfo.Add(new ShardedTableInfo("dbo", "customer", "C_CUSTKEY"));
    schemaInfo.Add(new ShardedTableInfo("dbo", "orders", "O_CUSTKEY"));

    // publish
    smm.GetSchemaInfoCollection().Add(Configuration.ShardMapName, schemaInfo);
    ```

    Las tablas "region" y "nation" se definen como tablas de referencia y se copiarán con las operaciones de división, combinación o desplazamiento. A su vez, las tablas "customer" y "orders" se definen como tablas particionadas. `C_CUSTKEY` y `O_CUSTKEY` sirven como clave de particionamiento.

- **Integridad referencial**

  El servicio División y combinación analiza las dependencias entre las tablas y usa relaciones de clave externa-clave principal para realizar las operaciones de mover las tablas de referencia y los shardlets. En general, las tablas de referencia primero se copian en orden de dependencia, luego los shardlets se copia según el orden de sus dependencias dentro de cada lote. Esto es necesario para que las restricciones de clave foránea-clave principal de la partición de destino se procesen tan pronto como lleguen los datos nuevos.

- **Coherencia de mapa de particiones y eventual finalización**

  Si hay errores, el servicio División y combinación reanuda las operaciones después de cualquier interrupción y apunta a completar cualquier solicitud en curso. Sin embargo, puede haber situaciones irrecuperables, por ejemplo, cuando la partición de destino se pierde o se ve comprometida más allá de la reparación. En esas circunstancias, algunos shardlets que iban a moverse podrían seguir residiendo en la partición de origen. El servicio asegura que las asignaciones de shardlets solo se actualicen después de que se copian los datos necesarios a la partición de destino. Los shardlets se eliminan del origen solo una vez que todos los datos se han copiado al destino y que se han actualizado correctamente las asignaciones correspondientes. La operación de eliminación ocurre en segundo plano mientras el intervalo ya está en línea en la partición de destino. El servicio División y combinación siempre garantiza la exactitud de las asignaciones almacenadas en el mapa de particiones.

## <a name="the-split-merge-user-interface"></a>La interfaz de usuario de División y combinación

El paquete del servicio División y combinación incluye un rol de trabajo y un rol web. El rol web se usa para enviar solicitudes de división y combinación de manera interactiva. Los componentes principales de la interfaz de usuario son los siguientes:

- **Tipo de operación**

  El tipo de operación es un botón de radio que controla la clase de operación que realiza el servicio para esta solicitud. Puede elegir entre los escenarios de división, combinación y movimiento. También puede cancelar una operación anteriormente enviada. Puede usar las solicitudes de división, combinación y movimiento para los mapas de particiones de intervalo. Los mapas de particiones de lista solo admiten operaciones de movimiento.

- **Mapa de particiones**

  La sección siguiente de parámetros de solicitud incluye información sobre el mapa de particiones y la base de datos que lo hospeda. En concreto, debe proporcionar el nombre del servidor y la base de datos que hospeda el mapa de particiones, las credenciales para conectarse a la base de datos del mapa de particiones y, finalmente, el nombre del mapa de particiones. Actualmente, la operación solo acepta un conjunto de credenciales. Estas credenciales deben tener permisos suficientes para realizar cambios en el mapa de particiones, así como también en los datos de usuario en las particiones.

- **Intervalo de origen (división y combinación)**

  Una operación de división y combinación procesa un intervalo con su clave superior e inferior. Para especificar una operación con un valor de clave superior desvinculado, active la casilla “La clave superior es máxima" y deje en blanco el campo de clave superior. Los valores de clave de intervalo que especifique no tienen que coincidir de manera precisa con una asignación y su límites en el mapa de particiones. Si no especifica ningún límite de intervalo, todo el servicio deducirá el intervalo más próximo automáticamente. Puede usar el script de PowerShell GetMappings.ps1 para recuperar las asignaciones actuales de un mapa de partición determinado.

- **Comportamiento del origen de división (división)**

  Para las operaciones de división, defina el punto para dividir el intervalo de origen. Para ello, proporcione la clave de particionamiento donde desee que se produzca la partición. Utilice el botón de radio para especificar si desea mover la parte inferior del intervalo (excluyendo la clave de división) o si desea mover la parte superior (incluida la clave de división).

- **Shardlet de origen (desplazamiento)**

  Las operaciones de desplazamiento son diferentes de las operaciones de división o combinación, puesto que no requieren un intervalo para describir el origen. Un origen para el desplazamiento simplemente se identifica con el valor de la clave de particionamiento que planea mover.

- **Partición de destino (división)**

  Una vez que haya proporcionado la información sobre el origen de la operación de división, debe definir dónde desea copiar los datos, para lo cual debe proporcionar el nombre del servidor y de la base de datos de destino.

- **Intervalo de destino (combinación)**

  Las operaciones de combinación mueven shardlets a una partición existente. Para identificar la partición existente, proporcione lo límites de intervalo del intervalo existente con el que desea combinar.

- **Tamaño de lote**

  El tamaño del lote controla el número de shardlets que quedarán sin conexión a la vez durante el movimiento de datos. Este es un valor entero en el que puede usar valores más pequeños cuando sea susceptible ante largos períodos de tiempo de inactividad para los shardlets. Los valores mayores aumentarán el tiempo durante el cual un shardlet determinado está sin conexión, pero podrían mejorar el rendimiento.

- **Identificador de operación (cancelación)**

  Si tiene en curso una operación que ya no es necesaria, puede cancelarla si especifica en este campo su identificador de operación. Puede recuperar el identificador de operación en la tabla de estado de solicitud (consulte la sección 8.1) o desde la salida del explorador web donde envió la solicitud.

## <a name="requirements-and-limitations"></a>Requisitos y limitaciones

La implementación actual del servicio División y combinación está sujeta a los siguientes requisitos y limitaciones:

- Las particiones deben existir y estar registradas en el mapa de particiones antes de que se pueda realizar una operación de división y combinación en estas particiones.
- El servicio no crea tablas ni ningún otro objeto de base de datos de manera automática como parte de sus operaciones. Esto significa que el esquema para todas las tablas particionadas y las tablas de referencia debe existir en la partición de destino antes de cualquier operación de división/combinación/desplazamiento. En especial, las tablas particionadas deben estar vacías en el intervalo donde una operación de división/combinación/desplazamiento agregará nuevos shardlets. De otro modo, la operación no pasará la comprobación de coherencia inicial en la partición de destino. Tenga en cuenta, además, que los datos de referencia solo se copian si la tabla de referencia está vacía y si no hay garantías de coherencia respecto de otras operaciones de escritura simultáneas en las tablas de referencia. Recomendamos que, cuando ejecute operaciones de división y combinación, ninguna otra operación de escritura realice cambios en las tablas de referencia.
- El servicio se basa en la identidad de las filas establecida por una clave o índice único que incluye la clave de particionamiento para mejorar el rendimiento y la confiabilidad de shardlets de gran tamaño. Esto permite que el servicio mueva datos en una granularidad incluso más fina que solo el valor de clave de particionamiento. De este modo se reduce la cantidad máxima de espacio de registro y bloqueos que se requieren durante la operación. Considere la posibilidad de crear un índice único o una clave principal que incluya la clave de particionamiento en una tabla determinada si desea usar esa tabla con solicitudes de división/combinación/desplazamiento. Por motivos de rendimiento, la clave de particionamiento debe ser la columna inicial de la clave o del índice.
- Durante el procesamiento de la solicitud, algunos datos de shardlets pueden estar presentes tanto en la partición de origen como en la de destino. Esto es necesario como protección contra errores durante el desplazamiento de los shardlets. La integración de división y combinación con el mapa de particiones garantiza que las conexiones mediante las API de enrutamiento dependiente de los datos que usan el método **OpenConnectionForKey** en el mapa de particiones no verán ningún estado intermedio incoherente. Sin embargo, cuando se conecten a la partición de origen o a la partición de destino sin usar el método **OpenConnectionForKey** , es posible que los estados intermedios incoherentes sean visibles cuando las solicitudes de división/combinación/desplazamiento estén en desarrollo. Estas conexiones pueden mostrar resultados parciales o duplicados, dependiendo del tiempo o de la partición subyacente a la conexión. Esta limitación incluye actualmente las conexiones realizadas por Consultas a través de particiones múltiples de Escalado elástico.
- No se debe compartir la base de datos de metadatos para el servicio División y combinación entre diferentes roles. Por ejemplo, un rol del servicio de División y combinación que se ejecuta en ensayo tiene que señalar a una base de datos de metadatos diferente que el rol de producción.

## <a name="billing"></a>Facturación

El servicio División y combinación se ejecuta como un servicio en la nube en la suscripción a Microsoft Azure. Por lo tanto, se aplican cargos por servicios en la nube a la instancia del servicio. A menos que realice operaciones de división/combinación/desplazamiento de manera frecuente, le recomendamos eliminar el servicio en la nube División y combinación. Esto ahorra costos relacionados con instancias de servicio en la nube en ejecución o implementadas. Puede volver a implementar e iniciar la configuración inmediatamente ejecutable cada vez que necesite realizar operaciones de división y combinación.

## <a name="monitoring"></a>Supervisión

### <a name="status-tables"></a>Tablas de estado

El servicio División y combinación proporciona la tabla **RequestStatus** en la base de datos de repositorio de metadatos para supervisar las solicitudes completadas y en curso. La tabla contiene una fila para cada solicitud de división y combinación que se ha enviado a esta instancia del servicio División y combinación. Brinda la siguiente información para cada solicitud:

- **Timestamp**

  Fecha y hora en que se inició la solicitud.

- **OperationId**

  GUID que identifica de manera única la solicitud. Esta solicitud también puede utilizarse para cancelar la operación mientras que todavía está en curso.

- **Estado**

  Estado actual de la solicitud. Para las solicitudes en curso, también muestra la fase actual en que está la solicitud.

- **CancelRequest**

  Marca que indica si la solicitud se ha cancelado.

- **Progress**

  Porcentaje estimativo de la finalización de la operación. Un valor de 50 indica que la operación está aproximadamente un 50% completada.

- **Detalles**

  Valor XML que proporciona un informe de progreso más detallado. El informe de progreso se actualiza periódicamente a medida que los conjuntos de filas se copian desde el origen al destino. En caso de errores o excepciones, esta columna también incluye información más detallada sobre el error.

### <a name="azure-diagnostics"></a>Diagnóstico de Azure

El servicio División y combinación usa Diagnósticos de Azure basado en el SDK de Azure 2.5 para supervisión y diagnóstico. Puede controlar la configuración de diagnósticos como se explica aquí: [Habilitación de diagnósticos en Azure Cloud Services y Virtual Machines](../../cloud-services/cloud-services-dotnet-diagnostics.md). El paquete de descarga incluye dos configuraciones de diagnóstico: una para el rol web y otra para rol de trabajo. Incluye las definiciones para registrar los contadores de rendimiento, registros IIS, registros de eventos de Windows y registros de eventos de la aplicación de división y combinación.

## <a name="deploy-diagnostics"></a>Implementar diagnósticos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Para habilitar la supervisión y el diagnóstico mediante el uso de la configuración de diagnóstico para el rol web y el rol de trabajo proporcionado por el paquete NuGet, ejecute los siguientes comandos con Azure PowerShell:

```powershell
$storageName = "<azureStorageAccount>"
$key = "<azureStorageAccountKey"
$storageContext = New-AzStorageContext -StorageAccountName $storageName -StorageAccountKey $key
$configPath = "<filePath>\SplitMergeWebContent.diagnostics.xml"
$serviceName = "<cloudServiceName>"

Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext `
    -DiagnosticsConfigurationPath $configPath -ServiceName $serviceName `
    -Slot Production -Role "SplitMergeWeb"

Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext `
    -DiagnosticsConfigurationPath $configPath -ServiceName $serviceName `
    -Slot Production -Role "SplitMergeWorker"
```

Puede encontrar más información sobre cómo configurar e implementar la configuración de diagnósticos aquí: [Habilitación de diagnósticos en Azure Cloud Services y Virtual Machines](../../cloud-services/cloud-services-dotnet-diagnostics.md).

## <a name="retrieve-diagnostics"></a>Recuperar diagnósticos

Se puede tener acceso fácilmente a los diagnósticos desde el Explorador de servidores de Visual Studio en la parte de Azure del árbol del Explorador de servidores. Abra una instancia de Visual Studio y, en la barra de menú, haga clic en Ver y en Explorador de servidores. Haga clic en el icono de Azure para conectarse a su suscripción a Azure. Luego vaya a Azure -> Storage -> `<your storage account>` -> Tablas -> WADLogsTable. Para más información, consulte [Explorador de servidores](/previous-versions/x603htbk(v=vs.140)).

![WADLogsTable][2]

WADLogsTable resaltado en la figura anterior contiene los eventos detallados del registro de aplicación del servicio División y combinación. Tenga en cuenta que la configuración predeterminada del paquete descargado está orientada a una implementación de producción. Por lo tanto, el intervalo en el que se extraen contadores y registros de las instancias de servicio es grande (5 minutos). Para pruebas y desarrollo, puede reducir el intervalo ajustando la configuración de diagnóstico del rol web o el rol de trabajo a sus necesidades. Haga clic con el botón derecho en el rol en el Explorador de servidores de Visual Studio (vea más arriba) y luego ajuste el período de transferencia en el cuadro de diálogo para las opciones de configuración de diagnósticos:

![Configuración][3]

## <a name="performance"></a>Rendimiento

En general, se espera un mejor rendimiento de los niveles de servicio más altos y con mayor rendimiento. Las asignaciones de memoria, CPU y E/S para los niveles de servicio más altos benefician las operaciones de copia y eliminación masiva que el servicio División y combinación usa. Por ese motivo, aumente el nivel de servicio para esas bases de datos durante un período definido y limitado.

El servicio también realiza consultas de validación como parte de su funcionamiento normal. Estas consultas de validación comprueban la presencia inesperada de datos en el intervalo de destino y se aseguran de que cualquier operación de división/combinación/desplazamiento comience desde un estado coherente. Estas consultas realizando todo el trabajo sobre los intervalos de clave de particionamiento definidos por el ámbito de la operación y el tamaño de lote que se proporcionan como parte de la definición de la solicitud. Estas consultas tienen un mejor rendimiento cuando existe un índice que tiene la clave de particionamiento como la columna inicial.

Además, una propiedad de unicidad con la clave de particionamiento como la columna inicial permitirá que el servicio utilice un enfoque optimizado que limite el consumo de recursos en términos de memoria y espacio de registro. Esta propiedad de unicidad es necesaria para mover los tamaños de datos de gran tamaño (normalmente más de 1 GB).

## <a name="how-to-upgrade"></a>Procedimiento de actualización

1. Siga los pasos en [Deploy a split-merge service](elastic-scale-configure-deploy-split-and-merge.md)(Implementar un servicio División y combinación).
2. Cambie el archivo de configuración de servicio en la nube para que la implementación de División y combinación refleje los nuevos parámetros de configuración. Un nuevo parámetro requerido es la información acerca del certificado utilizado para el cifrado. Una manera fácil de hacerlo consiste en comparar el nuevo archivo de plantilla de configuración de la descarga con la configuración existente. Asegúrese de agregar la configuración para "DataEncryptionPrimaryCertificateThumbprint" y "DataEncryptionPrimary" tanto para la web como para el rol de trabajo.
3. Antes de implementar la actualización en Azure, asegúrese de que se han terminado todas las operaciones de División y combinación actualmente en ejecución. Puede hacerlo fácilmente consultando las tablas RequestStatus y PendingWorkflows en la base de datos de metadatos de División y combinación para solicitudes en curso.
4. Actualice la implementación existente del servicio en la nube para División y combinación en la suscripción de Azure con el nuevo paquete y el archivo de configuración de servicio actualizado.

No es necesario aprovisionar una nueva base de datos de metadatos para que se actualice la División y combinación. La nueva versión actualizará automáticamente su base de datos de metadatos existente a la nueva versión.

## <a name="best-practices--troubleshooting"></a>Prácticas recomendadas y solución de problemas

- Defina un inquilino de prueba y ejecute las operaciones de división/combinación/desplazamiento más importantes con el inquilino de prueba entre varias particiones. Asegúrese de que todos los metadatos estén correctamente definidos en el mapa de particiones y que las operaciones no infrinjan las restricciones o las claves externas.
- Mantenga el tamaño del inquilino de prueba sobre el tamaño de datos máximo del inquilino de mayor tamaño para asegurarse de no encontrar problemas relacionados con el tamaño de los datos. Esto le permite evaluar un límite superior en el tiempo necesario para mover a un solo inquilino.
- Asegúrese de que el esquema permita las eliminaciones. El servicio División y combinación requiere la capacidad de quitar los datos de la partición de origen una vez que los datos se han copiado correctamente en el destino. Por ejemplo, los **desencadenadores de eliminación** pueden impedir que el servicio elimine los datos en el origen y pueden provocar errores en operaciones.
- La clave de particionamiento debe ser la columna inicial de la clave principal o la definición de índice único. Eso asegura el mejor rendimiento para las consultas de validación de división y combinación y para las operaciones de desplazamiento y eliminación de datos reales que siempre funcionan en intervalos de clave de particionamiento.
- Coloque el servicio División y combinación en la región y el centro de datos donde residen sus datos.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->
[1]:./media/elastic-scale-overview-split-and-merge/split-merge-overview.png
[2]:./media/elastic-scale-overview-split-and-merge/diagnostics.png
[3]:./media/elastic-scale-overview-split-and-merge/diagnostics-config.png
