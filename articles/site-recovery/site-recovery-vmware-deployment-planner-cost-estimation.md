---
title: Revisar las estimaciones de costes en Azure Site Recovery Deployment Planner
description: En este artículo se describe cómo revisar el informe de estimación de coste en Azure Site Recovery Deployment Planner para la recuperación ante desastres de VMware.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: 92d1c5babc2daacdd0b0687a5ff779fcd4272e71
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110577076"
---
# <a name="review-cost-estimations-in-the-vmware-deployment-planner"></a>Revisión de las estimaciones de costes en VMware Deployment Planner 

El informe de Deployment Planner ofrece un resumen de la estimación de costos incluida en las hojas de [Recomendaciones](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations) y del análisis detallado de los costos en la hoja Estimación de costos. Incluye el análisis detallado del costo por máquina virtual. 

>[!Note]
>La versión actual de la herramienta Deployment Planner v2.5 proporciona la estimación de costos de las VM que se replican en Managed Disks.

### <a name="cost-estimation-summary"></a>Resumen de la estimación de costos 
En el gráfico siguiente se muestra la vista de resumen del costo total estimado de la recuperación ante desastres en Azure de la región de destino elegida y en la moneda que ha especificado para generar el informe.
Resumen de la estimación de costos

![Resumen de la estimación de costos](media/site-recovery-vmware-deployment-planner-analyze-report/cost-estimation-summary-v2a.png)

El resumen le ayuda a comprender lo que debe pagar por los servicios de almacenamiento, proceso, red y licencia cuando protege todas las máquinas virtuales compatibles en Azure mediante Azure Site Recovery. El costo se calcula para las máquinas virtuales compatibles y no para todas las máquinas virtuales para las que se han generado perfiles.  
 
Puede ver el costo mensual o anual. Obtenga más información sobre las [regiones de destino admitidas](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions) y las [monedas admitidas](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies).

**Cost by components** (Costo por componentes): el costo total de recuperación ante desastres se divide en cuatro componentes: costo de proceso, almacenamiento, red y licencia de Azure Site Recovery. El costo se calcula en función del consumo en el que se incurrirá durante la replicación y en el transcurso del simulacro de recuperación ante desastres por los servicios de proceso, almacenamiento (premium y estándar), circuito ExpressRoute o VPN configurados entre el sitio local y Azure, y la licencia de Azure Site Recovery.

**Cost by states**: el costo total de recuperación ante desastres se clasifica según dos estados diferentes: replicación y simulacro de recuperación ante desastres. 

**Replication cost** (Costo de replicación):  costo en el que se incurrirá durante la replicación. Incluye el costo de almacenamiento, red y licencia de Azure Site Recovery. 

**DR-Drill cost** (Costo del simulacro de recuperación ante desastres): costo en el que se incurrirá durante las conmutaciones por error de prueba. Azure Site Recovery pone en marcha las máquinas virtuales durante la conmutación por error de prueba. El costo del simulacro de recuperación ante desastres incluye el costo de los servicios de proceso y almacenamiento de las máquinas virtuales en ejecución. 

**Azure storage cost per Month/Year**: muestra el costo total en el que se incurrirá por el almacenamiento premium y estándar por la replicación y el simulacro de recuperación ante desastres.

## <a name="detailed-cost-analysis"></a>Análisis detallado del costo
Los precios de Azure para los recursos de proceso, almacenamiento o red, entre otros, varía en las distintas regiones de Azure. Puede generar un informe de estimación de costos con los precios de Azure más recientes en función de su suscripción, la oferta que está asociada a su suscripción y la región de destino de Azure en la moneda especificada. De forma predeterminada, la herramienta usa la región Oeste de EE. UU. 2 de Azure y la moneda dólares estadounidenses (USD). Si ha usado otra región y moneda, la próxima vez que genere un informe sin identificador de suscripción, identificador de oferta, región de destino y moneda, usará los precios de la última región de destino usada y la última moneda usada para la estimación de costos.
En esta sección se muestra el identificador de suscripción y el identificador de oferta que ha usado para generar el informe.  Si no se usa, está en blanco.

En todo el informe, las celdas marcadas en color gris son de solo lectura. Las celdas en blanco se pueden modificar según sus necesidades.

![Detalles de la estimación de costos 1](media/site-recovery-hyper-v-deployment-planner-cost-estimation/cost-estimation1-h2a.png)

### <a name="overall-dr-cost-by-components"></a>Overall DR cost by components (Costo total de la recuperación ante desastres por componente)
La primera sección muestra el costo total de la recuperación ante desastres por componente y por estado. 

**Proceso**: costo de las máquinas virtuales IaaS que se ejecutan en Azure para satisfacer las necesidades de recuperación ante desastres. Incluye las máquinas virtuales creadas con Azure Site Recovery durante los simulacros de recuperación ante desastres (conmutaciones por error de prueba) y las máquinas virtuales que se ejecutan en Azure como SQL Server con grupos de disponibilidad AlwaysOn y controladores de dominio o servidores de nombres de dominio.

**Almacenamiento**: costo del consumo de almacenamiento de Azure para satisfacer las necesidades de recuperación ante desastres. Incluye el consumo de almacenamiento para la replicación y durante los simulacros de recuperación ante desastres.
Red: costo de ExpressRoute y VPN de sitio a sitio para satisfacer las necesidades de recuperación ante desastres. 

**ASR license** (Licencia de ASR): costo de la licencia de Azure Site Recovery para todas las máquinas virtuales compatibles. Si ha insertado manualmente una máquina virtual en la tabla de análisis detallado del costo, el costo de la licencia de Azure Site Recovery también está incluido para esa máquina virtual.

### <a name="overall-dr-cost-by-states"></a>Overall DR cost by states (Costo total de la recuperación ante desastres por estado)
El costo total de la recuperación ante desastres se clasifica según dos estados diferentes: replicación y simulacro de recuperación ante desastres.

**Replication cost** (Costo de replicación): se incurre en el costo en el momento de la replicación. Incluye el costo de almacenamiento, red y licencia de Azure Site Recovery. 

**DR-Drill cost** (Costo del simulacro de recuperación ante desastres): se incurre en el costo en el momento del simulacro de recuperación ante desastres. Azure Site Recovery pone en marcha las máquinas virtuales durante los simulacros de recuperación ante desastres. El costo del simulacro de recuperación ante desastres incluye el costo de los servicios de proceso y almacenamiento de las máquinas virtuales en ejecución.
Duración total de los simulacros de recuperación ante desastres en un año = Número de simulacros de recuperación ante desastres x Duración de cada simulacro de recuperación ante desastres (días) Costo promedio del simulacro de recuperación ante desastres (por mes) = Costo total de los simulacros de recuperación ante desastres/12

### <a name="storage-cost-table"></a>Tabla Storage cost (Costo de almacenamiento):
Esta tabla muestra el costo del almacenamiento premium y estándar en el que se incurre por la replicación y los simulacros de recuperación ante desastres, con y sin descuento.

### <a name="site-to-azure-network"></a>Site to Azure network (Red de sitio a Azure)
Seleccione la opción adecuada según sus requisitos. 

**ExpressRoute**: de forma predeterminada, la herramienta selecciona el plan de ExpressRoute que más se aproxime al ancho de banda de red necesario para la replicación diferencial. Puede cambiar el plan según sus requisitos.

**VPN Gateway**: seleccione VPN Gateway si tiene alguna instancia en su entorno. De forma predeterminada, es NA.

**Target Region** (Región de destino): región de Azure especificada para la recuperación ante desastres. El precio usado en el informe para proceso, almacenamiento, red y licencia se basa en los precios de Azure para esa región. 

### <a name="vm-running-on-azure"></a>VM running on Azure (Máquina virtual que se ejecuta en Azure)
Si se está ejecutando algún controlador de dominio, máquina virtual con DNS o máquina virtual con SQL Server con grupos de disponibilidad AlwaysOn en Azure para recuperación ante desastres, puede proporcionar el número de máquinas virtuales y su tamaño para valorar el costo del proceso en el costo total de la recuperación ante desastres. 

### <a name="apply-overall-discount-if-applicable"></a>Apply overall discount if applicable (Aplicar el descuento total si es aplicable)
Si es un asociado o un cliente de Azure y tiene derecho a un descuento sobre el precio general de Azure, puede usar este campo. La herramienta aplica el descuento (en %) en todos los componentes.

### <a name="number-of-virtual-machines-type-and-compute-cost-per-year"></a>Number of virtual machines type and compute cost (per year) [Número de tipo de máquinas virtuales y el cálculo de costo (anual)]
Esta tabla muestra el número de máquinas virtuales Windows y no Windows, y el costo de los servicios de proceso para la recuperación ante desastres de las mismas.

### <a name="settings"></a>Configuración 

**Moneda**: la moneda en la que se genera el informe. Duración del costo:  puede ver todos los costos del mes o de todo el año. 

## <a name="detailed-cost-analysis-table"></a>Tabla Detailed cost analysis (Análisis detallado del costo)
La tabla ![Detailed cost analysis](media/site-recovery-hyper-v-deployment-planner-cost-estimation/detailed-cost-analysis-h2a.png) (Análisis detallado del costo) muestra el desglose del costo de cada máquina virtual compatible. Para usar esta tabla para obtener el costo estimado de la recuperación ante desastres en Azure de máquinas virtuales sin perfiles, puede agregar las máquinas virtuales manualmente. Esto resulta útil cuando es necesario calcular los costos en Azure de una nueva implementación de recuperación ante desastres sin realizar una generación de perfiles detallada.
Para agregar las máquinas virtuales manualmente: 
1.  Haga clic en el botón "Insert row" (Insertar fila) para insertar una fila nueva entre las filas inicial y final.

2.  Rellene las columnas siguientes en función del tamaño aproximado de máquina virtual y del número de máquinas virtuales que coincidan con esta configuración: 

* Number of VMs (Número de máquinas virtuales), IaaS size (Tamaño de IaaS): valor que elija
* Storage Type (Standard/Premium) [Tipo de almacenamiento (Estándar/Premium)]
* Tamaño total de almacenamiento de VM (GB) de la máquina de origen
* Number of DR drills in a year (Número de simulacros de recuperación ante desastres en un año) 
* Each DR drill duration (Days) [Duración de cada simulacro de recuperación ante desastres (días)] 
* Tipo de SO
* Redundancia de datos 
* Ventaja híbrida de Azure

1. Haga clic en el botón "Apply to all" (Aplicar a todas) para aplicar a todas las máquinas virtuales de la tabla el mismo valor de número de simulacros de recuperación ante desastres en un año, duración de cada simulacro de recuperación ante desastres y ventaja para uso híbrido de Azure.

1. Haga clic en "Re-calculate cost" (Volver a calcular el costo) para actualizar el costo.

**VM Name** (Nombre de la máquina virtual): el nombre de la máquina virtual.

**Number of VMs** (Número de máquinas virtuales): número de máquinas virtuales que coinciden con la configuración. Puede actualizar el número de máquinas virtuales existentes si no se va a generar el perfil de las máquinas virtuales con una configuración similar y se protegerán.

**IaaS size (Recommendation)** (Tamaño de IaaS [recomendado]): el tamaño de rol de máquina virtual de la máquina virtual compatible que recomienda la herramienta. 

**IaaS size (Your selection)** (Tamaño de IaaS [su selección]): de forma predeterminada, es el mismo que el tamaño recomendado de rol de máquina virtual. Puede cambiar el rol para ajustarlo a sus requisitos. El costo de los servicios de proceso se basa en el tamaño de rol de máquina virtual seleccionado.

**Storage type** (Tipo de almacenamiento): tipo del almacenamiento que usa la máquina virtual. Puede ser almacenamiento estándar o premium.

**VM total storage size (GB)** (Tamaño de almacenamiento total de máquina virtual [GB]): Almacenamiento total de la VM de origen.

**Number of DR-Drills in a year** (Número de simulacros de recuperación ante desastres en un año): el número de veces que se realizan los simulacros de recuperación ante desastres en un año. De forma predeterminada, es 4 veces en un año. Puede modificar el período para máquinas virtuales específicas o aplicar el nuevo valor a todas las máquinas virtuales. Para ello, escriba el nuevo valor en la fila superior y haga clic en el botón "Apply to all" (Aplicar a todas). El costo total de los simulacros de recuperación ante desastres se calcula en función del número de simulacros de recuperación ante desastres realizados en un año y de la duración de cada uno de ellos.  

**Each DR-Drill duration (Days)** (Duración de cada simulacro de recuperación ante desastres [días]): la duración de cada simulacro de recuperación ante desastres. De forma predeterminada, es de 7 días cada 90 días según la [ventaja Disaster Recovery Software Assurance](https://azure.microsoft.com/pricing/details/site-recovery). Puede modificar el período para máquinas virtuales específicas o puede aplicar un nuevo valor a todas las máquinas virtuales. Para ello, escriba el nuevo valor en la fila superior y haga clic en el botón "Apply to all" (Aplicar a todas). El costo total de los simulacros de recuperación ante desastres se calcula en función del número de simulacros de recuperación ante desastres en un año y la duración de cada uno de ellos.
  
**OS Type** (Tipo de sistema operativo): el tipo de sistema operativo de la máquina virtual. Es Windows o Linux. Si el sistema operativo es Windows, se puede aplicar la ventaja para uso híbrido de Azure a esa máquina virtual. 

**Data redundancy** (Redundancia de datos): puede ser almacenamiento con redundancia local (LRS), almacenamiento con redundancia geográfica (GRS) o almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS). El valor predeterminado es LRS. Puede cambiar el tipo en función de su cuenta de almacenamiento para máquinas virtuales específicas o puede aplicar el nuevo tipo a todas las máquinas virtuales. Para ello, cambie el tipo de la fila superior y haga clic en el botón "Apply to all" (Aplicar a todas).  El costo del almacenamiento para la replicación se calcula en función del precio de la redundancia de datos que ha seleccionado. 

**Ventaja híbrida de Azure**: puede aplicar Ventaja híbrida de Azure a máquinas virtuales Windows, si es aplicable.  El valor predeterminado es Yes. Puede cambiar la configuración de máquinas virtuales específicas o actualizar todas las máquinas virtuales. Para ello, haga clic en el botón "Apply to all" (Aplicar a todas).

**Total Azure consumption** (Consumo total de Azure): incluye los costos de licencia de Compute, Storage y Azure Site Recovery para la recuperación ante desastres. En función de los valores seleccionados, mostrará el costo mensual o anual.

**Steady state replication cost** (Costo de replicación de estado estable): incluye el costo de almacenamiento para la replicación.

**Total DR-Drill cost (average)** (Costo total de simulacro de recuperación ante desastres [medio]): incluye el costo de los servicios de proceso y almacenamiento de los simulacros de recuperación ante desastres.

**ASR license cost** (Costo de licencia de ASR): el costo de la licencia de Azure Site Recovery.

## <a name="supported-target-regions"></a>Regiones de destino admitidas
Azure Site Recovery Deployment Planner proporciona una estimación de los costos para las siguientes regiones de Azure. Si su región no aparece en la lista, puede usar cualquiera de las siguientes regiones cuyo precio sea más parecido al de su región.

eastus, eastus2, westus, centralus, northcentralus, southcentralus, northeurope, westeurope, eastasia, southeastasia, japaneast, japanwest, australiaeast, australiasoutheast, brazilsouth, southindia, centralindia, westindia, canadacentral, canadaeast, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth 

## <a name="supported-currencies"></a>Monedas admitidas
Azure Site Recovery Deployment Planner puede generar el informe de costos en cualquiera de las siguientes monedas.

|Moneda|Nombre|Moneda|Nombre|Moneda|Nombre|
|---|---|---|---|---|---|---|---|
|ARS|Peso argentino ($)|AUD|Dólar australiano ($)|BRL|Real brasileño (R$)|
|CAD|Dólar canadiense ($)|CHF|Franco suizo (chf)|DKK|Corona danesa (kr)|
|EUR|Euro (€)|GBP|Libra británica (£)|HKD|Dólar de Hong Kong (HK$)|
|IDR|Rupia indonesia (Rp)|INR|Rupia india (₹)|JPY|Yen japonés (¥)|
|KRW|Won coreano (₩)|MXN|Peso mexicano (MX$)|MYR|Ringgit malayo (RM$)|
|NOK|Corona noruega (kr)|NZD|Dólar de Nueva Zelanda ($)|RUB|Rublo ruso (руб)|
|SAR|Saudi Riyal (SR)|SEK|Corona sueca (kr)|TWD|Dólar taiwanés (NT$)|
|TRY|Lira turca (TL)|USD| Dólar estadounidense (USD)|ZAR|Rand sudafricano (R)|

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre cómo proteger las [máquinas virtuales de VMware en Azure con Azure Site Recovery](./vmware-azure-tutorial.md).
