---
title: Planeamiento de una plantilla de solución para una oferta de Aplicación de Azure
description: Conozca lo que es necesario para crear un plan de plantilla de solución para una nueva oferta de Aplicación de Azure usando el portal del marketplace comercial en el Centro de partners de Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 25feef60e6b417d835f06be9641207ce56aa53a6
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111536432"
---
# <a name="plan-a-solution-template-for-an-azure-application-offer"></a>Planeamiento de una plantilla de solución para una oferta de Aplicación de Azure

En este artículo se explican los requisitos para publicar un plan de plantilla de solución para una oferta de Aplicación de Azure. Un plan de plantilla de solución es uno de los dos tipos de planes que admiten las ofertas de Aplicación de Azure. Para información sobre la diferencia entre estos dos tipos de planes, consulte [Tipos de planes](plan-azure-application-offer.md#plans). Si aún no lo ha hecho, lea [Planeamiento de una oferta de Aplicación de Azure](plan-azure-application-offer.md).

El tipo de plan de plantilla de solución requiere una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) para implementar automáticamente la infraestructura de la solución.

## <a name="solution-template-requirements"></a>Requisitos de la plantilla de solución

| Requisitos | Detalles |
| ------------ | ------------- |
| Facturación y medición | Aunque no se pueden realizar transacciones con los planes de plantilla de solución, se pueden usar para implementar ofertas de máquina virtual de pago que se facturan mediante el marketplace comercial de Microsoft. Los recursos que implementa la plantilla de ARM de la solución se configuran en la suscripción de Azure del cliente. Las transacciones de máquinas virtuales de pago por uso se realizan con el cliente mediante Microsoft y se facturan con la suscripción de Azure del cliente. <br><br> En el caso de la facturación denominada "traiga su propia licencia" (BYOL), aunque Microsoft factura los costos de infraestructura derivados de la suscripción del cliente, usted realizará la transacción de las tarifas de licencia de software directamente con el cliente. |
| Disco duro virtual (VHD) compatible con Azure | Las máquinas virtuales deben estar basadas en Windows o Linux. Para más información, consulte:<ul><li>[Creación de un recurso técnico de máquina virtual de Azure](./azure-vm-create-certification-faq.md#address-a-vulnerability-or-an-exploit-in-a-vm-offer) (para discos duros virtuales de Windows)</li><li>[Distribuciones de Linux aprobadas en Azure](../virtual-machines/linux/endorsed-distros.md) (para discos duros virtuales de Linux).</li></ul> |
| Atribución de uso del cliente | Se requiere habilitar la atribución de uso del cliente en todas las plantillas de solución que se publican en Azure Marketplace. Para obtener más información sobre la atribución de uso del cliente y cómo habilitarla, consulte [Atribución de uso del cliente para asociados de Azure](azure-partner-customer-usage-attribution.md). |
| Uso de discos administrados | Los [discos administrados](../virtual-machines/managed-disks-overview.md) son la opción predeterminada para los discos persistentes de VM de infraestructura como servicio (IaaS) en Azure. Debe usar discos administrados en las plantillas de solución.<ul><li>Para actualizar las plantillas de solución, siga las instrucciones de [Uso de discos administrados en plantillas de Azure Resource Manager](../virtual-machines/using-managed-disks-template-deployments.md) y use [las muestras](https://github.com/Azure/azure-quickstart-templates) proporcionadas.</li><li>Para publicar el disco duro virtual como una imagen en Azure Marketplace, importe el disco duro virtual subyacente de los discos administrados en una cuenta de almacenamiento mediante [Azure PowerShell](/previous-versions/azure/virtual-machines/scripts/virtual-machines-powershell-sample-copy-managed-disks-vhd) o la [CLI de Azure](/previous-versions/azure/virtual-machines/scripts/virtual-machines-cli-sample-copy-managed-disks-vhd).</ul> |
| Paquete de implementación | Necesitará un paquete de implementación que permita a los clientes implementar el plan. Si crea varios planes que requieren la misma configuración técnica, puede usar el mismo paquete de plan. Para más información, consulte la siguiente sección: Paquete de implementación. |
|||

## <a name="deployment-package"></a>Paquete de implementación

El paquete de implementación contiene todos los archivos de plantilla necesarios para este plan, así como recursos adicionales, empaquetados en un archivo .zip.

Todas las aplicaciones de Azure deben incluir estos dos archivos en la carpeta raíz de un archivo .zip:

- Un archivo de plantilla de Resource Manager llamado [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template). Esta plantilla define los recursos que se van a implementar en la suscripción de Azure del cliente. Para ver ejemplos de plantillas de Resource Manager, consulte la [galería de plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/) o el repositorio [GitHub: Plantillas de inicio rápido de Azure Resource Manager](https://github.com/azure/azure-quickstart-templates) correspondiente.
- Una definición de interfaz de usuario para la experiencia de creación de aplicaciones de Azure llamada [createUiDefinition.json](../azure-resource-manager/managed-applications/create-uidefinition-overview.md). En la interfaz de usuario, puede especificar los elementos que permiten a los consumidores proporcionar los valores de los parámetros.

Los tamaños de archivo máximos admitidos son:

- Hasta 1 GB para el tamaño de archivo .zip comprimido total.
- Hasta 1 GB para cada archivo individual descomprimido dentro del archivo .zip.

Todas las nuevas ofertas de la aplicación de Azure deben incluir un GUID de [atribución de uso del cliente para asociados de Azure](azure-partner-customer-usage-attribution.md).

## <a name="azure-regions"></a>Regiones de Azure

Puede publicar su plan en la región pública de Azure, en la región de Azure Government, o en ambas. Antes de publicar en [Azure Government](../azure-government/documentation-government-manage-marketplace-partners.md), pruebe y valide el plan en el entorno, ya que algunos puntos de conexión pueden ser diferentes. Para configurar y probar el plan, solicite una cuenta de prueba desde [Prueba de Microsoft Azure Government](https://azure.microsoft.com/global-infrastructure/government/request/).

Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos).

Para ver una lista de países y regiones que se admiten en el marketplace comercial, consulte [Disponibilidad geográfica y compatibilidad con monedas](marketplace-geo-availability-currencies.md).

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS. Para dar a conocer sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a su anuncio en el programa directamente o a descripciones de su cumplimiento de estos en sus propios sitios web. Estos vínculos solo son visibles para los clientes de Azure Government.

## <a name="choose-who-can-see-your-plan"></a>Selección de quién puede ver el plan

Puede configurar cada plan para que sea visible para todo el mundo o solo para una audiencia concreta (privada). Puede crear hasta 100 planes, y 45 de ellos pueden ser privados. Puede que quiera crear un plan privado para ofrecer diferentes opciones de precios o configuraciones técnicas a clientes específicos.

Puede conceder acceso a un plan privado con los identificadores de la suscripción de Azure con la opción de incluir una descripción de cada identificador de suscripción que asigne. Puede agregar hasta 10 identificadores de suscripción manualmente o hasta 10 000 mediante un archivo .csv. Los identificadores de suscripción de Azure se representan como GUID y las letras deben estar en minúsculas.

> [!NOTE]
> Si publica un plan privado, puede cambiar su visibilidad a público más adelante. Sin embargo, una vez que se publica un plan público, no se puede cambiar su visibilidad a privado.

En el caso de los planes de plantilla de solución, también puede ocultar el plan para Azure Marketplace. Puede que quiera hacerlo si el plan solo se implementa indirectamente mediante otra plantilla de solución o aplicación administrada.

> [!NOTE]
> Los planes privados no son compatibles con las suscripciones de Azure establecidas mediante un revendedor del programa de proveedores de soluciones en la nube (CSP).

Para más información, consulte [Ofertas privadas en el marketplace comercial de Azure](private-offers.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de aplicación de Azure](azure-app-offer-setup.md)