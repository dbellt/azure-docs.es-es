---
title: Preguntas frecuentes de computación confidencial de Azure
description: Respuestas a las preguntas frecuentes sobre la computación confidencial de Azure.
author: JBCook
ms.topic: troubleshooting
ms.workload: infrastructure
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.date: 4/17/2020
ms.author: jencook
ms.openlocfilehash: aa78d6495eeffd0ea733451e029f07413602ce0d
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107812739"
---
# <a name="frequently-asked-questions-for-azure-confidential-computing"></a>Preguntas frecuentes de computación confidencial de Azure

En este artículo se proporcionan respuestas a algunas de las preguntas más comunes sobre la ejecución de [cargas de trabajo de computación confidencial en máquinas virtuales de Azure](overview.md).

Si su problema con Azure no se trata en este artículo, visite los foros de Azure en [MSDN y Stack Overflow](https://azure.microsoft.com/support/forums/). Puede publicar su problema en ellos o [@AzureSupport en Twitter](https://twitter.com/AzureSupport). También puede enviar una solicitud de soporte técnico de Azure. Para enviar una solicitud de soporte técnico, en la [página de soporte técnico de Azure](https://azure.microsoft.com/support/options/), seleccione Obtener soporte técnico.

## <a name="confidential-computing-virtual-machines"></a>Máquinas virtuales de computación confidencial<a id="vm-faq"></a>

**¿Cómo puedo implementar VM de la serie DCsv2 en Azure?**

Estas son algunas formas de implementar una máquina virtual DCsv2:
   - Uso de una [plantilla de Azure Resource Manager](../virtual-machines/windows/template-description.md)
   - En [Azure Portal](https://portal.azure.com/#create/hub)
   - En la plantilla de solución de Marketplace [Azure Confidential Computing (Virtual Machine)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.acc-virtual-machine-v2?tab=overview). La plantilla de solución de Marketplace le ayudará a restringir un cliente a los escenarios admitidos (regiones, imágenes, disponibilidad, cifrado de disco). 

**¿Funcionarán todas las imágenes de sistema operativo con la computación confidencial de Azure?**

No. Las máquinas virtuales solo se pueden implementar en máquinas operativas de segunda generación con Ubuntu Server 18.04, Ubuntu Server 20.04, Windows Server 2019 Datacenter y Windows Server 2016 Datacenter. Obtenga más información sobre las máquinas virtuales de generación 2 en [Linux](../virtual-machines/generation-2.md) y [Windows](../virtual-machines/generation-2.md)

**Las máquinas virtuales de DCsv2 aparecen atenuadas en el portal y no puedo seleccionar ninguna**.

En función de la burbuja de información que se encuentra junto a la máquina virtual, hay que realizar diferentes acciones:
   -    **UnsupportedGeneration**: cambie la generación de la imagen de máquina virtual a "Gen2".
   -    **NotAvailableForSubscription**: la región todavía no está disponible para su suscripción. Seleccione una región disponible.
   -    **InsufficientQuota**: [cree una solicitud de soporte técnico para aumentar la cuota](../azure-portal/supportability/per-vm-quota-requests.md). Las suscripciones de evaluación gratuita no tienen cuota de máquinas virtuales de computación confidencial. 

**Las máquinas virtuales de DCsv2 no aparecen cuando intento buscarlas en el selector de tamaño del portal**.

Asegúrese de haber seleccionado una [región disponible](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines). Asegúrese también de seleccionar "Borrar todos los filtros" en el selector de tamaño. 

**¿Puedo habilitar las redes aceleradas con la computación confidencial de Azure?**

 No. Las redes aceleradas no se admiten en máquinas virtuales de las series DC o DCsv2. Las redes aceleradas no se pueden habilitar para ninguna implementación de máquina virtual de computación confidencial o implementación de clúster de Azure Kubernetes Service que se ejecute en computación confidencial.

**¿Puedo usar Azure Dedicated Host con estas máquinas?**

Sí. Azure Dedicated Host admite máquinas virtuales de la serie DCsv2. Azure Dedicated Host proporciona un servidor físico de un solo inquilino en el que puede ejecutar las máquinas virtuales. Los usuarios suelen usar Azure Dedicated Host para abordar los requisitos de cumplimiento en torno a la seguridad física, la integridad de los datos y la supervisión. 

**Recibo un error de implementación de plantilla de Azure Resource Manager: "No se pudo completar la operación porque se superó la cuota de Núcleos de familia DCsv2 estándar aprobada"**

[cree una solicitud de soporte técnico para aumentar la cuota](../azure-portal/supportability/per-vm-quota-requests.md). Las suscripciones de evaluación gratuita no tienen cuota de máquinas virtuales de computación confidencial. 

**¿Cuál es la diferencia entre las máquinas virtuales de la serie DCsv2 y la serie DC?**

Las VM de la serie DC se ejecutan en procesadores Intel de 6 núcleos anteriores con Intel SGX y tienen menos memoria total, menos memoria caché de páginas de enclave (EPC), y solo están disponibles en dos regiones (Este de EE. UU. y Europa Occidental en tamaños Standard_DC2s y Standard_DC4s). No hay planes para que estas VM estén disponibles con carácter general y no se recomiendan para su uso en producción. Para implementar estas VM, use la instancia de Marketplace [Confidential Compute DC-Series VM [Preview]](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.confidentialcompute?tab=Overview).

**¿Las máquinas virtuales de DCsv2 están disponibles globalmente?**

No. En este momento, estas máquinas virtuales solo están disponibles en regiones seleccionadas. Active la [página de productos por regiones](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines) para ver las últimas regiones disponibles. 

**¿La tecnología Hyper-Threading está desactivada en estas máquinas?**

Hyper-Threading está deshabilitada en todos los clústeres de procesos confidenciales de Azure.

**Cómo instalar el SDK Open Enclave en las máquinas virtuales DCsv2?**
   
Para obtener instrucciones sobre cómo instalar el SDK OE en una máquina en Azure o local, siga las instrucciones del [SDK Open Enclave en GitHub](https://github.com/openenclave/openenclave).
     
También puede consultar el SDK Open Enclave en GitHub para obtener instrucciones de instalación específicas del sistema operativo:
   - [Instalación del SDK de OE en Windows](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Windows.md)
   - [Instalación del SDK de OE en Ubuntu 18.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_18.04.md)