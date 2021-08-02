---
title: Modelo de compra de núcleo virtual
description: El modelo de compra de núcleo virtual permite escalar los recursos de proceso y de almacenamiento de manera independiente, igualar el rendimiento local y optimizar el precio para Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sashan, moslake
ms.date: 06/02/2021
ms.custom: references_regions
ms.openlocfilehash: a78622aaccfaeb372d67ef4575e2bd923ab6cb58
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111415465"
---
# <a name="vcore-purchase-model-overview---azure-sql-database"></a>Información general sobre el modelo de compra de núcleo virtual: Azure SQL Database 
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se revisa el modelo de compra de núcleo virtual para [Azure SQL Database](sql-database-paas-overview.md). Para más información sobre cómo elegir entre los modelos de compra de núcleos virtuales y DTU, consulte [Elección entre los modelos de compra de núcleo virtual y DTU: Azure SQL Database y SQL Managed Instance](purchasing-models.md).

El modelo de compra de núcleo virtual que Azure SQL Database usa proporciona varias ventajas con respecto al modelo de compra DTU:

- Mayores límites de proceso, memoria, E/S y almacenamiento.
- Control sobre la generación de hardware para satisfacer mejor los requisitos de proceso y memoria de la carga de trabajo.
- Descuentos de precios para [Ventaja híbrida de Azure (AHB)](../azure-hybrid-benefit.md).
- Mayor transparencia en los detalles de hardware que potencian el proceso, lo que facilita la planeación de las migraciones desde implementaciones locales.
- [Los precios de las instancias reservadas](reserved-capacity-overview.md) solo están disponibles para el modelo de compra de núcleo virtual. 

## <a name="service-tiers"></a>Niveles de servicio

Entre las opciones de nivel de servicio del modelo de compra de núcleo virtual se incluyen Uso general, Crítico para la empresa e Hiperescala. El nivel de servicio define generalmente la arquitectura de almacenamiento, los límites de espacio y de E/S y las opciones de continuidad empresarial relacionadas con la disponibilidad y la recuperación ante desastres.

|**Caso de uso**|**Uso general**|**Crítico para la empresa**|**Hiperescala**|
|---|---|---|---|
|Más adecuado para|La mayoría de las cargas de trabajo empresariales. Ofrece opciones de proceso y almacenamiento equilibradas y escalables pensando en el presupuesto. |Ofrece a las aplicaciones empresariales la mayor resistencia a los errores mediante el uso de varias réplicas aisladas y proporciona el mayor rendimiento de E/S por réplica de base de datos.|La mayoría de las cargas de trabajo de una empresa que tengan requisitos altamente escalables de almacenamiento y escalado de lectura.  Ofrece mayor resistencia a los errores al permitir la configuración de más de una réplica de base de datos aislada. |
|Storage|Usa el almacenamiento remoto.<br/>**Proceso aprovisionado de SQL Database**:<br/>5 GB – 4 TB<br/>**Proceso sin servidor**:<br/>5 GB - 3 TB|Usa almacenamiento local de SSD.<br/>**Proceso aprovisionado de SQL Database**:<br/>5 GB – 4 TB|Crecimiento automático flexible de almacenamiento según sea necesario. Admite hasta 100 TB de almacenamiento. Utiliza almacenamiento SSD local para la caché del grupo de búferes local y almacenamiento de datos local. Utiliza almacenamiento remoto de Azure como almacén de datos final a largo plazo. |
|IOPS y rendimiento (aproximado)|**SQL Database**: vea los límites de recursos para [bases de datos únicas](resource-limits-vcore-single-databases.md) y [grupos elásticos](resource-limits-vcore-elastic-pools.md).|Vea los límites de recursos para [bases de datos únicas](resource-limits-vcore-single-databases.md) y [grupos elásticos](resource-limits-vcore-elastic-pools.md).|Hiperescala es una arquitectura de varios niveles con almacenamiento en caché en varios niveles. IOPS y el rendimiento efectivos dependen de la carga de trabajo.|
|Disponibilidad|1 réplica, sin réplicas de escalado de lectura|3 réplicas, 1 [réplica de escalado de lectura](read-scale-out.md),<br/>Alta disponibilidad (HA) con redundancia de zona|1 réplica de lectura y escritura, además de 0 a 4 [réplicas de escalado de lectura](read-scale-out.md)|
|Copias de seguridad|[Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)](../../storage/common/geo-redundant-design.md), de 1 a 35 días (7 días de forma predeterminada)|[RA-GRS](../..//storage/common/geo-redundant-design.md), de 1 a 35 días (7 días de forma predeterminada)|Copias de seguridad basadas en instantáneas en el almacenamiento remoto de Azure. Los procesos de restauración usan estas instantáneas para conseguir una recuperación rápida. Las copias de seguridad son instantáneas y no afectan al rendimiento de E/S del proceso. Las restauraciones son rápidas y no son operaciones relacionadas con el tamaño de los datos (tardan minutos en lugar de horas o días).|
|En memoria|No compatible|Compatible|La compatibilidad es parcial. Se admiten tipos de tablas optimizadas para memoria, variables de tabla y módulos compilados de forma nativa.|
|||


### <a name="choosing-a-service-tier"></a>Selección de un nivel de servicio

Para obtener información sobre cómo seleccionar un nivel de servicio para una carga de trabajo determinada, consulte los siguientes artículos:

- [Cuándo elegir el nivel de servicio De uso general](service-tier-general-purpose.md#when-to-choose-this-service-tier)
- [Cuándo elegir el nivel de servicio Crítico para la empresa](service-tier-business-critical.md#when-to-choose-this-service-tier)
- [Cuándo elegir el nivel de servicio Hiperescala](service-tier-hyperscale.md#who-should-consider-the-hyperscale-service-tier)


## <a name="compute-tiers"></a>Niveles de proceso

Entre las opciones de nivel de proceso del modelo de núcleo virtual se incluyen los niveles de proceso aprovisionados y sin servidor.


### <a name="provisioned-compute"></a>Proceso aprovisionado

El nivel de proceso aprovisionado proporciona una cantidad específica de recursos de proceso que se aprovisionan continuamente con independencia de la actividad de carga de trabajo y factura la cantidad de proceso aprovisionado a un precio fijo por hora.


### <a name="serverless-compute"></a>Proceso sin servidor

El [nivel de proceso sin servidor](serverless-tier-overview.md) escala automáticamente los recursos de proceso en función de la actividad de la carga de trabajo y se factura según la cantidad de proceso usado por segundo.



## <a name="hardware-generations"></a>Generaciones de hardware

Entre las opciones de generación de hardware del modelo de núcleo virtual se incluyen Gen 4/5, la serie M, la serie Fsv2 y la serie DC. En general, la generación de hardware define los límites de proceso y de memoria, así como otras características que afectan al rendimiento de la carga de trabajo.

### <a name="gen4gen5"></a>Gen4/Gen5

- El hardware Gen4/Gen5 proporciona recursos equilibrados de proceso y memoria, y es adecuado para la mayoría de las cargas de trabajo de base de datos que no tienen una memoria mayor, un núcleo virtual superior o unos requisitos de núcleo virtual único más rápidos como los proporcionados por la serie Fsv2 o la serie M.

Para ver las regiones en las que Gen4/Gen5 está disponible, vea la [disponibilidad de Gen4/Gen5](#gen4gen5-1).

### <a name="fsv2-series"></a>Serie Fsv2

- La serie Fsv2 es una opción de hardware optimizado para proceso que ofrece una latencia de CPU baja y una velocidad de reloj elevada para las cargas de trabajo más exigentes de CPU.
- En función de la carga de trabajo, la serie Fsv2 puede ofrecer más rendimiento de CPU por núcleo virtual que Gen5, y el tamaño de 72 núcleos virtuales puede proporcionar más rendimiento de CPU con menos costos que 80 núcleos virtuales en Gen5. 
- Fsv2 proporciona menos memoria y tempdb por núcleo virtual que otro hardware, por lo que se pueden tomar en consideración las series Gen5 y M para las cargas de trabajo sensibles a esos límites.  

La serie Fsv2 solo se admite en el nivel De uso general. Para las regiones en las que está disponible la serie Fsv2, consulte la [disponibilidad de la serie Fsv2](#fsv2-series-1).

### <a name="m-series"></a>Serie M

- La serie M es una opción de hardware optimizado para memoria para las cargas de trabajo que exigen más memoria y mayores límites de proceso que los proporcionados por Gen5.
- La serie M cuenta con 29 GB por cada núcleo virtual y hasta 128 núcleos virtuales, lo que multiplica por ocho el límite de memoria con respecto a Gen5 hasta alcanzar prácticamente los 4 TB.

La serie M solo se admite en el nivel Crítico para la empresa y no es compatible con la redundancia de zona.  Para ver las regiones en las que la serie M está disponible, consulte la [disponibilidad de la serie M](#m-series-1).

#### <a name="azure-offer-types-supported-by-m-series"></a>Tipos de oferta de Azure disponibles en la serie M

Para acceder a la serie M, la suscripción debe ser un tipo de oferta de pago, como Pago por uso o Contrato Enterprise (EA).  Para ver una lista completa de los tipos de oferta de Azure disponibles en la serie M, consulte las [ofertas actuales sin límites de gasto](https://azure.microsoft.com/support/legal/offer-details).

<!--
To enable M-series hardware for a subscription and region, a support request must be opened. The subscription must be a paid offer type including Pay-As-You-Go or Enterprise Agreement (EA).  If the support request is approved, then the selection and provisioning experience of M-series follows the same pattern as for other hardware generations. For regions where M-series is available, see [M-series availability](#m-series).
-->

### <a name="dc-series"></a>Serie DC

> [!NOTE]
> Actualmente, la serie DC está en **versión preliminar pública**.

- El hardware de la serie DC utiliza procesadores Intel con tecnología Software Guard Extensions (Intel SGX).
- La serie DC es necesaria para [Always Encrypted con enclaves seguros](/sql/relational-databases/security/encryption/always-encrypted-enclaves), que no es compatible con otras configuraciones de hardware.
- La serie DC está diseñada para cargas de trabajo que procesan datos confidenciales y requieren funcionalidades de procesamiento de consultas confidenciales, proporcionadas por Always Encrypted con enclaves seguros.
- El hardware de la serie DC proporciona recursos de proceso y memoria equilibrados.

La serie DC solo es compatible con el proceso aprovisionado (no se admite sin servidor) y no admite la redundancia de zona. Para ver las regiones en las que la serie DC está disponible, consulte la [disponibilidad de la serie DC](#dc-series-1).

#### <a name="azure-offer-types-supported-by-dc-series"></a>Tipos de oferta de Azure disponibles en la serie DC

Para acceder a la serie DC, la suscripción debe ser un tipo de oferta de pago, como Pago por uso o Contrato Enterprise (EA).  Para ver una lista completa de los tipos de oferta de Azure disponibles en la serie DC, consulte las [ofertas actuales sin límites de gasto](https://azure.microsoft.com/support/legal/offer-details).

### <a name="compute-and-memory-specifications"></a>Especificaciones de memoria y proceso


|Generación de hardware  |Proceso  |Memoria  |
|:---------|:---------|:---------|
|Gen4     |- Procesadores Intel&reg; E5-2673 v3 (Haswell) de 2,4 GHz<br>- Aprovisionamiento de hasta 24 núcleos virtuales (1 núcleo virtual = 1 núcleo físico)  |- 7 GB por núcleo virtual<br>- Aprovisionamiento de hasta 168 GB|
|Gen5     |**Proceso aprovisionado**<br>- Procesadores Intel&reg; E5-2673 v4 (Broadwell) de 2,3 GHz, Intel&reg; SP-8160 (Skylake)\* e Intel&reg; 8272CL (Cascade Lake) de 2,5 GHz\*<br>- Aprovisionamiento de hasta 80 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)<br><br>**Proceso sin servidor**<br>- Procesadores Intel&reg; E5-2673 v4 (Broadwell) de 2,3 GHz e Intel&reg; SP-8160 (Skylake)*<br>- Escalado vertical automático de hasta 40 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)|**Proceso aprovisionado**<br>- 5,1 GB por núcleo virtual<br>- Aprovisionamiento de hasta 408 GB<br><br>**Proceso sin servidor**<br>- Escalado vertical automático de hasta 24 GB por núcleo virtual<br>- Escalado vertical automático de hasta 120 GB máx.|
|Serie Fsv2     |- Procesadores Intel&reg; 8168 (Skylake)<br>- Presentación de una velocidad de reloj turbo sostenida de todos los núcleos de hasta 3,4 GHz y una velocidad de reloj turbo de un solo núcleo máxima de 3,7 GHz.<br>- Aprovisionamiento de hasta 72 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)|1,9 GB por núcleo virtual<br>- Aprovisionamiento de hasta 136 GB|
|Serie M     |- Procesadores Intel&reg; E7-8890 v3 de 2,5 GHz e Intel&reg; 8280M de 2,7 GHz (Cascade Lake)<br>- Aprovisionamiento de hasta 128 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)|29 GB por núcleo virtual<br>- Aprovisionamiento de hasta 3,7 TB|
|Serie DC     | - Procesadores Intel XEON E-2288G<br>- Presentación de Software Guard Extensions de Intel (Intel SGX)<br>- Aprovisionamiento de hasta 8 núcleos virtuales (1 núcleo virtual = 1 núcleo físico) | 4,5 GB por núcleo virtual |

\* En la vista de administración dinámica [sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database), la generación de hardware para bases de datos que usan procesadores Intel&reg; SP-8160 (Skylake) aparece como Gen6, mientras que la generación de hardware para bases de datos que usan procesadores Intel&reg; 8272CL (Cascade Lake) aparece como Gen7. Los límites de recursos en todas las bases de datos Gen5 son los mismos, independientemente del tipo de procesador (Broadwell, Skylake o Cascade Lake).

Para obtener más información sobre los límites de recursos, vea [Límites de recursos para bases de datos únicas (núcleo virtual)](resource-limits-vcore-single-databases.md) o [Límites de recursos para grupos elásticos (núcleo virtual)](resource-limits-vcore-elastic-pools.md).

### <a name="selecting-a-hardware-generation"></a>Selección de una generación de hardware

En Azure Portal, puede seleccionar la generación de hardware para una base de datos o un grupo de SQL Database en el momento de la creación o puede cambiar la generación de hardware de una base de datos o grupo existente.

**Seleccionar una generación de hardware al crear una instancia de SQL Database o un grupo SQL**

Para obtener información detallada, consulte [Creación de una instancia de SQL Database](single-database-create-quickstart.md).

En la pestaña **Básico**, seleccione el vínculo **Configurar base de datos** en la sección **Compute + storage** (Proceso + almacenamiento) y, a continuación, seleccione el vínculo **Cambiar configuración**:

:::image type="content" source="./media/service-tiers-vcore/configure-sql-database.png" alt-text="configuración de SQL Database" loc-scope="azure-portal":::

Seleccione la generación de hardware deseada:

:::image type="content" source="./media/service-tiers-vcore/select-hardware.png" alt-text="selección de hardware para SQL Database" loc-scope="azure-portal":::

**Cambiar la generación de hardware de una instancia de SQL Database o un grupo SQL existente**

En el caso de una base de datos, en la página de información general, seleccione el vínculo **Plan de tarifa**:

:::image type="content" source="./media/service-tiers-vcore/change-hardware.png" alt-text="cambio de hardware para SQL Database" loc-scope="azure-portal":::

En la página de información general, seleccione **Configurar**.

Siga los pasos para cambiar la configuración y seleccione la generación de hardware como se describe en los pasos anteriores.

### <a name="hardware-availability"></a>Disponibilidad de hardware

#### <a name="gen4gen5"></a><a id="gen4gen5-1"></a> Gen4/Gen5

El hardware de Gen4 está [en proceso de eliminación gradual](https://azure.microsoft.com/updates/gen-4-hardware-on-azure-sql-database-approaching-end-of-life-in-2020/) y ya no está disponible para las nuevas implementaciones. Todas las nuevas bases de datos deben implementarse en hardware de Gen5.

Gen5 está disponible en todas las regiones públicas de todo el mundo.

#### <a name="fsv2-series"></a>Serie Fsv2

La serie Fsv2 está disponible en las siguientes regiones: Centro de Australia, Centro de Australia 2, Este de Australia, Sudeste de Australia, Sur de Brasil, Centro de Canadá, Este de Asia, Asia Pacífico, Centro de Francia, Centro de la India, Centro de Corea del Sur, Sur de Corea del Sur, Norte de Europa, Norte de Sudáfrica, Sudeste de Asia, Sur de Reino Unido, Oeste de Reino Unido, Oeste de Europa, Oeste de Europa 2.

#### <a name="m-series"></a>Serie M

La serie M está disponible en las siguientes regiones: Este de EE. UU., Norte de Europa, Oeste de Europa, Oeste de EE. UU. 2.
<!--
M-series may also have limited availability in additional regions. You can request a different region than listed here, but fulfillment in a different region may not be possible.

To enable M-series availability in a subscription, access must be requested by [filing a new support request](#create-a-support-request-to-enable-m-series).


##### Create a support request to enable M-series: 

1. Select **Help + support** in the portal.
2. Select **New support request**.

On the **Basics** page, provide the following:

1. For **Issue type**, select **Service and subscription limits (quotas)**.
2. For **Subscription** = select the subscription to enable M-series.
3. For **Quota type**, select **SQL database**.
4. Select **Next** to go to the **Details** page.

On the **Details** page, provide the following:

1. In the **PROBLEM DETAILS** section select the **Provide details** link. 
2. For **SQL Database quota type** select **M-series**.
3. For **Region**, select the region to enable M-series.
    For regions where M-series is available, see [M-series availability](#m-series).

Approved support requests are typically fulfilled within 5 business days.
-->

#### <a name="dc-series"></a>Serie DC

> [!NOTE]
> Actualmente, la serie DC está en **versión preliminar pública**.

La serie DC está disponible en las siguientes regiones: Centro de Canadá, Este de Canadá, Este de EE. UU., Norte de Europa, Sur de Reino Unido, Oeste de Europa y Oeste de EE. UU.

Si necesita la serie DC en una región no admitida actualmente, [envíe una incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) siguiendo las instrucciones de [Solicitud de aumentos de cuota para Azure SQL Database y la instancia administrada de SQL](quota-increase-request.md).

## <a name="next-steps"></a>Pasos siguientes

- Para comenzar, consulte [Creación de una instancia de SQL Database mediante Azure Portal](single-database-create-quickstart.md)
- Para obtener información detallada acerca de los precios, consulte la [página de precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/).
- Para obtener detalles sobre los tamaños de almacenamiento y proceso específicos disponibles, consulte:
    - [Límites de recursos basados en núcleos virtuales de Azure SQL Database](resource-limits-vcore-single-databases.md)
    - [Límites de recursos basados en núcleos virtuales de Azure SQL Database agrupada](resource-limits-vcore-elastic-pools.md)
