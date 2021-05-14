---
title: Alta disponibilidad en Azure Cosmos DB
description: En este artículo se describe cómo Azure Cosmos DB ofrece una alta disponibilidad
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/05/2021
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: ac1e77d99707cdaa34ef42eb9b327a62f4e864c0
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107365372"
---
# <a name="how-does-azure-cosmos-db-provide-high-availability"></a>¿Cómo proporciona Azure Cosmos DB la alta disponibilidad?
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB proporciona la alta disponibilidad de dos maneras principales. En primer lugar, Azure Cosmos DB replica los datos entre las regiones configuradas dentro de una cuenta de Cosmos. En segundo lugar, Azure Cosmos DB mantiene 4 réplicas de datos dentro de una región.

Azure Cosmos DB es un servicio básico de base de datos distribuido globalmente disponible en [todas las regiones en las que Azure está disponible](https://azure.microsoft.com/global-infrastructure/services/?products=cosmos-db&regions=all). Puede asociar cualquier número de regiones de Azure con su cuenta de Azure Cosmos y los datos se replican de forma automática y transparente. Puede agregar o quitar una región a la cuenta de Azure Cosmos en cualquier momento. Cosmos DB está disponible en los cinco entornos en la nube de Azure disponibles para los clientes:

* La nube **pública de Azure**, que está disponible globalmente.

* **Azure China 21Vianet** está disponible gracias a una exclusiva asociación entre Microsoft y 21Vianet, uno de los proveedores de acceso a Internet más importantes del país en China.

* **Azure Alemania** proporciona servicios bajo un modelo de administrador de datos que garantiza que los datos de los clientes permanecen en Alemania bajo el control de T-Systems International GmbH, una subsidiaria de Deutsche Telekom, que actúa como administrador de datos para Alemania.

* **Azure Government** está disponible en cuatro regiones en los Estados Unidos para agencias gubernamentales de EE. UU. y sus asociados.

* **Azure Government para el Departamento de Defensa (DoD)** está disponible en dos regiones de Estados Unidos para el Departamento de Defensa de EE. UU.

Dentro de una región, Azure Cosmos DB mantiene cuatro copias de los datos como réplicas en particiones físicas, tal como se muestra en la siguiente imagen:

:::image type="content" source="./media/high-availability/cosmosdb-data-redundancy.png" alt-text="Creación de particiones físicas" border="false":::

* Los datos de los contenedores de Azure Cosmos se [dividen horizontalmente en particiones](partitioning-overview.md).

* Un conjunto de particiones es una colección de varios conjuntos de réplicas. En cada una de las regiones, cada partición está protegida por un conjunto de réplicas con todas las escrituras replicadas y confirmadas de forma duradera por la mayoría de las réplicas. Las réplicas se distribuyen entre unos 10 o 20 dominios de error como máximo.

* Se replica cada partición de todas las regiones. Cada región contiene todas las particiones de datos de un contenedor de Azure Cosmos y puede atender operaciones tanto de lectura como de escritura cuando están habilitadas las operaciones de escritura en varias regiones.  

Si la cuenta de Azure Cosmos se distribuye entre *N* regiones de Azure, habrá al menos *N* x 4 copias de todos los datos. Tener una cuenta de Azure Cosmos en más de dos regiones mejora la disponibilidad de la aplicación y proporciona una latencia baja en las regiones asociadas.

## <a name="slas-for-availability"></a>SLA para disponibilidad

Azure Cosmos DB proporciona Acuerdos de Nivel de Servicio completos que abarcan rendimiento, latencia en el percentil 99, coherencia y alta disponibilidad. En la tabla siguiente se muestran las garantías de alta disponibilidad proporcionadas por Azure Cosmos DB para cuentas de una y varias regiones. Para lograr una disponibilidad mayor de las operaciones de escritura, configure las cuentas de Azure Cosmos para que tengan operaciones de escritura en varias regiones.

|Tipo de operación  | Región única |Varias regiones (operaciones de escritura en una única región)|Varias regiones (operaciones de escritura de varias regiones) |
|---------|---------|---------|-------|
|Escrituras    | 99,99    |99,99   |99,999|
|Lecturas     | 99,99    |99,999  |99,999|

> [!NOTE]
> En la práctica, la disponibilidad de escritura real para modelos de posible coherencia, prefijo coherente, sesión y obsolescencia limitada es significativamente mayor que los SLA publicados. La disponibilidad de lectura real para todos los niveles de coherencia es significativamente mayor que los SLA publicados.

## <a name="high-availability-with-azure-cosmos-db-in-the-event-of-regional-outages"></a>Alta disponibilidad con Azure Cosmos DB en caso de interrupciones regionales

En los infrecuentes casos en que se produce una interrupción regional, Azure Cosmos DB garantiza que la base de datos siempre tenga una alta disponibilidad. Los siguientes detalles capturan el comportamiento de Azure Cosmos DB durante una interrupción, en función de la configuración de la cuenta de Azure Cosmos:

* Con Azure Cosmos DB, antes de que el cliente confirme una operación de escritura, un cuórum de réplicas dentro de la región que acepta las operaciones de escritura confirma los datos de forma permanente. Para obtener más detalles, consulte [Niveles de coherencia y rendimiento](./consistency-levels.md#consistency-levels-and-throughput).

* Las cuentas de varias regiones configuradas con varias regiones de escritura seguirán teniendo una alta disponibilidad para las escrituras y lecturas. Las conmutaciones por error regionales se detectan y controlan en el cliente de Azure Cosmos DB. También son instantáneas y no requieren cambios de la aplicación.

* Las cuentas de una sola región pueden perder disponibilidad después de una interrupción regional. Se recomienda configurar siempre **al menos dos regiones** (si es posible, al menos dos regiones de escritura) con su cuenta de Azure Cosmos para garantizar una alta disponibilidad en todo momento.

> [!IMPORTANT]
> Al usar las API de SQL, es necesario configurar el SDK de Cosmos DB para utilizar todas las regiones de lectura especificadas y aprovechar la disponibilidad mejorada. Para obtener más información, consulte [este artículo](troubleshoot-sdk-availability.md).

### <a name="multi-region-accounts-with-a-single-write-region-write-region-outage"></a>Cuentas de varias regiones con una sola región de escritura (interrupción de la región de escritura)

* Durante una interrupción en la región de escritura, la cuenta de Azure Cosmos promoverá automáticamente una región secundaria para que sea la nueva región de escritura principal cuando la opción para **habilitar la conmutación por error automática** está configurada en la cuenta de Azure Cosmos. Cuando se habilita, la conmutación por error se producirá en otra región según el orden de prioridad de regiones que especificó.

* Tenga en cuenta que la conmutación por error manual no debería desencadenarse y no se realizará correctamente en presencia de una interrupción en la región de origen o de destino. Esto se debe a la comprobación de coherencia que requiere el procedimiento de conmutación por error, la cual requiere conectividad entre las regiones.

* Cuando la región afectada previamente vuelva a estar en línea, los datos de escritura cuya replicación se anuló al producirse el error en la región se ponen a disposición a través de la [fuente de conflictos](how-to-manage-conflicts.md#read-from-conflict-feed). Las aplicaciones pueden leer la fuente de conflictos, resolver los conflictos de acuerdo con la lógica específica de la aplicación y escribir los datos actualizados de nuevo en el contenedor de Azure Cosmos según corresponda.

* Una vez que se recupera la región de escritura previamente afectada, se convierte en disponible automáticamente como una región de lectura. Puede volver a la región recuperada como la región de escritura. Puede cambiar las regiones con [PowerShell, la CLI de Azure o Azure Portal](how-to-manage-database-account.md#manual-failover). No se produce **ninguna pérdida de datos ni de disponibilidad** antes, durante ni después de cambiar la región de escritura, y la aplicación sigue siendo de alta disponibilidad.

> [!IMPORTANT]
> Se recomienda encarecidamente configurar las cuentas de Azure Cosmos usadas para que las cargas de trabajo de producción **habiliten la conmutación por error automática**. Esto permite a Cosmos DB conmutar por error las bases de datos de la cuenta a las regiones disponibles automáticamente. En ausencia de esta configuración, la cuenta experimentará la pérdida de la disponibilidad de escritura durante toda la interrupción de la región de escritura, ya que la conmutación por error manual no se realizará correctamente debido a la falta de conectividad con la región.

### <a name="multi-region-accounts-with-a-single-write-region-read-region-outage"></a>Cuentas de varias regiones con una sola región de escritura (interrupción de la región de lectura)

* Durante una interrupción de la región de lectura, las cuentas de Azure Cosmos mediante un nivel de coherencia o coherencia alta con tres o más regiones de lectura seguirán teniendo una alta disponibilidad para lecturas y escrituras.

* Las cuentas de Azure Cosmos que usen una coherencia alta con tres regiones (una de escritura y dos de lectura), mantendrán la disponibilidad de escritura durante las interrupciones de la región de lectura. En el caso de las cuentas con dos regiones y la conmutación automática por error habilitada, la cuenta dejará de aceptar escrituras hasta que la región se marque como con errores y se produzca la conmutación automática por error.

* La región afectada se desconecta automáticamente y se marcará como sin conexión. Los [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md) redirigirán las llamadas de lectura a la siguiente región disponible en la lista de regiones preferidas.

* Si ninguna de las regiones de la lista de las regiones preferidas está disponible, las llamadas se devuelven automáticamente a la región actual de escritura.

* No es necesario realizar ningún cambio en el código de su aplicación para gestionar la interrupción de la región de lectura. Cuando la región de lectura afectada se vuelve a conectar, se sincronizará automáticamente con la región de escritura actual y estará disponible de nuevo para atender las solicitudes de lectura.

* Las siguientes lecturas se redirigen a la región recuperada sin necesidad de realizar cambios en el código de la aplicación. Durante la conmutación por error y cuando se vuelva a unir una región previamente errónea, Azure Cosmos DB seguirá cumpliendo las garantías de coherencia de lectura.

* Incluso en un caso poco frecuente y desafortunado en que la región de Azure sea irrecuperable de forma permanente, no se produce pérdida de datos si se configura la cuenta de Azure Cosmos de varias regiones con coherencia *fuerte*. En el caso de una región de escritura irrecuperable de forma permanente, una cuenta de Azure Cosmos de varias regiones configurada con coherencia de obsolescencia limitada, la posible ventana de pérdida de datos se restringe a la ventana de obsolescencia (*K* o *T*) donde K = 100 000 actualizaciones o T = 5 minutos, lo que antes suceda. En cuanto a los niveles de posible coherencia y prefijo coherente por sesión, el período de pérdida potencial de datos se restringe a un máximo de 15 minutos. Para obtener más información sobre los objetivos de RPO y RTO de Azure Cosmos DB, consulte [Durabilidad de datos y los niveles de coherencia](./consistency-levels.md#rto).

## <a name="availability-zone-support"></a>Compatibilidad de zonas de disponibilidad

Además de la resistencia entre regiones, Azure Cosmos DB también admite la **redundancia de zona** en las regiones que se admiten al seleccionar la región que se desea asociar a una base de datos de Azure Cosmos.

Gracias a la compatibilidad de zona de disponibilidad, Azure Cosmos DB garantizará que las réplicas se coloquen en varias zonas de una región determinada para proporcionar alta disponibilidad y resistencia en los errores zonales. Availability Zones proporciona un Acuerdo de Nivel de Servicio de disponibilidad del 99,995 % sin cambios en la latencia. En el caso de que haya un error de una sola zona, la redundancia de zona proporciona durabilidad total de los datos con el RPO=0 y la disponibilidad con RTO=0. La redundancia de zona es una funcionalidad complementaria a la replicación regional. No se puede confiar en la redundancia de zona por sí sola para lograr la resistencia regional.

La redundancia de zona solo se puede configurar cuando se agrega una nueva región a una cuenta de Azure Cosmos. En el caso de las regiones existentes, para habilitar la redundancia de zona es preciso quitar la región y, después, volver a agregarla con la redundancia de zona habilitada. En el caso de una cuenta de una sola región, es preciso agregar una región adicional a la que poder realizar la conmutación por error de forma temporal y, después, quitar y agregar la región deseada con la redundancia de zona habilitada.

Al configurar las escrituras en varias regiones para la cuenta de Azure Cosmos, puede optar por recibir la redundancia de zona sin ningún costo adicional. En caso contrario, consulte la tabla siguiente sobre los precios de la compatibilidad de la redundancia de zona. Para obtener una lista de las regiones en que las zonas de disponibilidad están disponibles, consulte el artículo sobre [zonas de disponibilidad](../availability-zones/az-region.md).

En la tabla siguiente se resume la funcionalidad de alta disponibilidad de varias configuraciones de cuenta:

|KPI|Una sola región sin zonas de disponibilidad|Una sola región con zonas de disponibilidad|Varias regiones, operaciones de escritura en una sola región con zonas de disponibilidad|Varias regiones, operaciones de escritura en varias regiones con zonas de disponibilidad|
|---------|---------|---------|---------|---------|
|Contrato de Nivel de Servicio de disponibilidad de escritura | 99,99% | 99,995 % | 99,995 % | 99,999 % |
|Contrato de Nivel de Servicio de disponibilidad de lectura  | 99,99% | 99,995 % | 99,995 % | 99,999 % |
|Errores de zona: pérdida de datos | Pérdida de datos | No se produce pérdida de datos | No se produce pérdida de datos | No se produce pérdida de datos |
|Errores de zona: disponibilidad | Pérdida de disponibilidad | Sin pérdida de disponibilidad | Sin pérdida de disponibilidad | Sin pérdida de disponibilidad |
|Interrupción regional: pérdida de datos | Pérdida de datos |  Pérdida de datos | Depende del nivel de coherencia. Para más información, consulte el artículo que explica el [equilibrio entre coherencia, disponibilidad y rendimiento](./consistency-levels.md). | Depende del nivel de coherencia. Para más información, consulte el artículo que explica el [equilibrio entre coherencia, disponibilidad y rendimiento](./consistency-levels.md).
|Interrupción regional: disponibilidad | Pérdida de disponibilidad | Pérdida de disponibilidad | No hay pérdida de disponibilidad en caso de error de la región de lectura y temporal en caso de error de la región de escritura | Sin pérdida de disponibilidad |
|Precio (***1** _) | N/D | Unidades de solicitud aprovisionadas x 1,25 | Unidades de solicitud aprovisionadas x 1,25 (_*_2_**) | Velocidad de operaciones de escritura en varias regiones |

***1*** En el caso de las cuentas sin servidor, las unidades de solicitud se multiplican por un factor de 1,25.

***2*** La velocidad 1,25 solo se aplica a las regiones en que está habilitada la zona de disponibilidad.

Availability Zones se puede habilitar mediante:

* [Azure Portal](how-to-manage-database-account.md#addremove-regions-from-your-database-account)

* [Azure PowerShell](manage-with-powershell.md#create-account)

* [CLI de Azure](manage-with-cli.md#add-or-remove-regions)

* [Plantillas del Administrador de recursos de Azure](./manage-with-templates.md)

## <a name="building-highly-available-applications"></a>Compilación de aplicaciones de alta disponibilidad

* Revise el [comportamiento esperado de los SDK de Azure Cosmos](troubleshoot-sdk-availability.md) durante estos eventos y qué configuraciones influyen en este.

* Para garantizar una alta disponibilidad de escritura y lectura, configure la cuenta de Azure Cosmos para abarcar al menos dos regiones con varias regiones de escritura. Esta configuración proporcionará la disponibilidad más alta, la latencia más baja y la mejor escalabilidad para lecturas y escrituras respaldadas por los SLA. Para más información, vea cómo [configurar la cuenta de Azure Cosmos con varias regiones de escritura](tutorial-global-distribution-sql-api.md).

* Para cuentas de Azure Cosmos de varias regiones que estén configuradas con una región única de escritura, [habilite la conmutación por error automática mediante la CLI de Azure o Azure Portal](how-to-manage-database-account.md#automatic-failover). Después de habilitar la conmutación automática por error, siempre que se produzca un desastre regional, Cosmos DB conmutarán por error automáticamente su cuenta.  

* Incluso si su cuenta de Azure Cosmos es de alta disponibilidad, es posible que la aplicación no se haya diseñado correctamente para seguir teniendo alta disponibilidad. Para probar la alta disponibilidad de un extremo a otro de la aplicación, como parte de las pruebas de la aplicación o las exploraciones de recuperación ante desastres (DR), deshabilite temporalmente la conmutación automática por error de la cuenta, invoque la [conmutación por error manual mediante PowerShell, la CLI de Azure o Azure Portal](how-to-manage-database-account.md#manual-failover) y, luego, supervise la conmutación por error de la aplicación. Cuando haya terminado, puede realizar la conmutación por recuperación a la región primaria y restaurar la conmutación automática por error en la cuenta.

> [!IMPORTANT]
> No invoque la conmutación por error manual durante una interrupción de Cosmos DB en las regiones de origen o de destino, ya que requiere conectividad con las regiones para mantener la coherencia de los datos y no se realizará correctamente.

* En un entorno de base de datos distribuida de forma global, existe una relación directa entre el nivel de coherencia y la durabilidad de los datos si se produce una interrupción en toda la región. A medida que desarrolle el plan de continuidad empresarial, tendrá que saber el tiempo máximo aceptable para que la aplicación se recupere por completo tras un evento de interrupción. El tiempo necesario para que una aplicación se recupere totalmente se conoce como "objetivo de tiempo de recuperación (RTO)". También debe conocer el período máximo de actualizaciones de datos recientes que la aplicación puede tolerar perder al recuperarse después de un evento de interrupción. El período de tiempo de las actualizaciones que se puede permitir perder se conoce como objetivo de punto de recuperación (RPO). Para ver el RPO y el RTO de Azure Cosmos DB, consulte [Durabilidad de datos y los niveles de coherencia](./consistency-levels.md#rto).

## <a name="what-to-expect-during-a-cosmos-db-region-outage"></a>Qué cabe esperar durante una interrupción de región de Cosmos DB

En el caso de las cuentas de una sola región, los clientes experimentarán la pérdida de disponibilidad de la lectura y escritura.

Las cuentas de varias regiones experimentarán diferentes comportamientos según la tabla siguiente.

| Regiones de escritura | Conmutación por error automática | Qué esperar | Qué hacer |
| -- | -- | -- | -- |
| Una región de escritura | no habilitado. | En caso de interrupción en una región de lectura, todos los clientes se redirigirán a otras regiones. No se produce pérdida de la disponibilidad de lectura o escritura. No se produce pérdida de datos. <p/> En caso de que se produzca una interrupción en la región de escritura, los clientes experimentarán la pérdida de la disponibilidad de escritura. Si no se ha seleccionado el nivel de coherencia seguro, es posible que algunos datos no se hayan replicado en las regiones activas restantes. Esto depende del nivel de coherencia seleccionado, como se describe en [esta sección](consistency-levels.md#rto). Si la región afectada sufre una pérdida de datos definitiva, es posible que los datos no replicados se pierdan. <p/> Cosmos DB restaurará la disponibilidad de escritura automáticamente cuando termine la interrupción. | Durante la interrupción, asegúrese de que haya suficientes unidades de solicitud aprovisionadas en las regiones restantes para admitir el tráfico de lectura. <p/> *No* desencadene una conmutación por error manual durante la interrupción, ya que no se realizará correctamente. <p/> Cuando termine la interrupción, reajuste las unidades de solicitud aprovisionadas según corresponda. |
| Una región de escritura | habilitado | En caso de interrupción en una región de lectura, todos los clientes se redirigirán a otras regiones. No se produce pérdida de la disponibilidad de lectura o escritura. No se produce pérdida de datos. <p/> En caso de que se produzca una interrupción en la región de escritura, los clientes experimentarán la pérdida de la disponibilidad de escritura hasta que Cosmos DB elija automáticamente una nueva región como la región de escritura según las preferencias establecidas. Si no se ha seleccionado el nivel de coherencia seguro, es posible que algunos datos no se hayan replicado en las regiones activas restantes. Esto depende del nivel de coherencia seleccionado, como se describe en [esta sección](consistency-levels.md#rto). Si la región afectada sufre una pérdida de datos definitiva, es posible que los datos no replicados se pierdan. | Durante la interrupción, asegúrese de que haya suficientes unidades de solicitud aprovisionadas en las regiones restantes para admitir el tráfico de lectura. <p/> *No* desencadene una conmutación por error manual durante la interrupción, ya que no se realizará correctamente. <p/> Cuando termine la interrupción, puede volver a trasladar la región de escritura a la región original y reajustar las unidades de solicitud aprovisionadas según corresponda. Las cuentas que usan las API de SQL también pueden recuperar los datos no replicados en la región con errores desde la [fuente de conflictos](how-to-manage-conflicts.md#read-from-conflict-feed). |
| Varias regiones de escritura | No aplicable | No se produce pérdida de la disponibilidad de lectura o escritura. <p/> Es posible que los datos actualizados recientemente en la región con errores no estén disponibles en el resto de regiones activas. Los niveles de coherencia de sesión y prefijo eventuales garantizan una obsolescencia de <15 minutos. La obsolescencia limitada garantiza menos de K actualizaciones o de T segundos, en función de la configuración. Si la región afectada sufre una pérdida de datos definitiva, es posible que los datos no replicados se pierdan. | Durante la interrupción, asegúrese de que haya suficientes unidades de solicitud aprovisionadas en las regiones restantes para admitir el tráfico adicional. <p/> Cuando termine la interrupción, puede reajustar las unidades de solicitud aprovisionadas según corresponda. Si es posible, Cosmos DB recuperará de forma automática los datos no replicados en la región con errores con el método de resolución de conflictos configurado para las cuentas de la API de SQL y con la directiva de Últimos casos de escritura correcta para las cuentas que usan otras API. |

## <a name="next-steps"></a>Pasos siguientes

Ahora puede pasar a la lectura de los artículos siguientes:

* [Availability and performance tradeoffs for various consistency levels](./consistency-levels.md) (Compromisos entre rendimiento y disponibilidad en los distintos niveles de coherencia)

* [Escalado del rendimiento aprovisionado globalmente](./request-units.md)

* [Distribución global en segundo plano](global-dist-under-the-hood.md)

* [Niveles de coherencia en Azure Cosmos DB](consistency-levels.md)

* [Procedimiento para configurar la cuenta de Cosmos con varias regiones de escritura](how-to-multi-master.md)

* [Comportamiento del SDK en entornos de varias regiones](troubleshoot-sdk-availability.md)