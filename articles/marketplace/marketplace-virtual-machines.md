---
title: 'Planeación de una oferta de máquina virtual: marketplace comercial de Microsoft'
description: En este artículo se describen los requisitos para publicar una oferta de máquina virtual en Azure Marketplace.
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 10/19/2020
ms.openlocfilehash: 25cfca98857d33a90b2c1f11692d83ae30911bb7
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110480879"
---
# <a name="plan-a-virtual-machine-offer"></a>Planificación de una oferta de máquina virtual

En este artículo se explican los distintos requisitos y opciones para publicar una oferta de máquina virtual en el marketplace comercial. Las ofertas de máquinas virtuales son ofertas de transacciones que se implementan y facturan mediante Azure Marketplace.

Antes de empezar, [cree una cuenta de marketplace comercial en el Centro de partners](create-account.md) y asegúrese de que su cuenta esté inscrita en el programa de Marketplace comercial.

### <a name="technical-fundamentals"></a>Aspectos fundamentales técnicos

El proceso de diseño, compilación y prueba de estos recursos lleva tiempo y requiere experiencia tanto en la plataforma de Azure como en las tecnologías que se usan para crear la oferta. El equipo de ingeniería debe tener conocimientos prácticos de [Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/), [Azure Storage](https://azure.microsoft.com/services/?filter=storage#storage) y [Redes de Azure](https://azure.microsoft.com/services/?filter=networking#networking), así como destreza en el [diseño y la arquitectura de aplicaciones de Azure](https://azure.microsoft.com/solutions/architecture/). Vea los recursos técnicos adicionales: 

- Tutoriales
  - [Máquinas virtuales Linux](../virtual-machines/linux/tutorial-manage-vm.md)
  - [Máquinas virtuales Windows](../virtual-machines/windows/tutorial-manage-vm.md)

- Ejemplos
  - [Ejemplos de la CLI de Azure para máquinas virtuales Linux](https://github.com/Azure-Samples/azure-cli-samples/tree/master/virtual-machine)
  - [Azure PowerShell para máquinas virtuales Linux](https://github.com/Azure/azure-docs-powershell-samples/tree/master/virtual-machine)
  - [Ejemplos de la CLI de Azure para máquinas virtuales Windows](https://github.com/Azure-Samples/azure-cli-samples/tree/master/virtual-machine)
  - [Azure PowerShell para máquinas virtuales Windows](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm-quick)

## <a name="technical-requirements"></a>Requisitos técnicos

Las ofertas de máquina virtual tienen los siguientes requisitos técnicos:

- Debe preparar un disco duro virtual (VHD) del sistema operativo. Los VHD de los discos de datos son opcionales. Esto se explica con más detalle a continuación.
- El cliente puede cancelar su oferta en cualquier momento.
- Debe crear al menos un plan para la oferta. El precio del plan se basa en la [opción de licencia](#licensing-options) que seleccione.
   > [!IMPORTANT]
   > Cada imagen de máquina virtual de un plan debe tener el mismo número de discos de datos.

Una VM contiene dos componentes:

- **Disco duro virtual del sistema operativo**: contiene el sistema operativo y la solución que se implementa con la oferta. El proceso de preparación del disco duro virtual es diferente en función de si se basa en Linux, en Windows o una máquina virtual personalizada.
- **Discos duros virtuales de discos de datos** (opcional): el almacenamiento persistente dedicado para una máquina virtual. No use el disco duro virtual del sistema operativo (por ejemplo, la unidad C:) para almacenar la información persistente. 
    - Puede incluir hasta 16 discos de datos.
    - Use un VHD por cada disco de datos, incluso si el disco está vacío.

    > [!NOTE]
    > Independientemente del sistema operativo que use, agregue solo el número mínimo de discos de datos necesarios para la solución. Los usuarios no pueden quitar los discos que formen parte de una imagen durante la implementación, pero siempre pueden agregar discos durante o después de la implementación.

Para obtener instrucciones detalladas sobre la preparación de los activos técnicos, consulte [Creación de una máquina virtual con una base aprobada](azure-vm-create-using-approved-base.md) o [Creación de una máquina virtual con su propia imagen](azure-vm-create-using-own-image.md).

## <a name="preview-audience"></a>Público preliminar

[!INCLUDE [Test drives section](includes/preview-audience.md)]

## <a name="plans-and-pricing"></a>Planes y precios

Las ofertas de VM requieren al menos un plan. Un plan define el ámbito y los límites de la solución, así como los precios asociados. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de licencias, así como evaluaciones gratuitas. Consulte [Planes y precios de ofertas de marketplace comercial](plans-pricing.md) para obtener instrucciones generales sobre los planes, incluidos los modelos de precios, las evaluaciones gratuitas y los planes privados. 

Las VM se pueden comercializar íntegramente con los modelos de licencia de pago por uso o BYOL (traiga su propia licencia). Microsoft hospeda la transacción comercial y factura al cliente en su nombre. Usted obtiene la ventaja de usar la relación de pago preferida entre el cliente y Microsoft, incluidos los contratos empresariales existentes. Para obtener más información, consulte [Capacidades de transacción de marketplace comercial](./marketplace-commercial-transaction-capabilities-and-considerations.md).

> [!NOTE]
> El pago por adelantado de Azure (que anteriormente se denominaba compromisos monetarios) asociado con un Contrato Enterprise se puede aplicar al uso que Azure hace de su máquina virtual, pero no a las tarifas de licencias de software.

### <a name="licensing-options"></a>Opciones de licencia

Cuando prepare la publicación de una nueva oferta, debe decidir la opción de licencia. Esta opción determina la información adicional que deberá proporcionar más adelante cuando cree la oferta en el Centro de partners.

Estas son las opciones de licencia disponibles para las ofertas de máquina virtual:

| Opción de obtención de licencias | Proceso de transacción |
| --- | --- |
| Evaluación gratuita | Ofrezca a sus clientes una evaluación gratuita de uno, tres o seis meses. |
| Versión de prueba | Esta opción permite a los clientes evaluar las VM sin ningún costo adicional. No tienen que ser un cliente existente de Azure para poder participar en la experiencia de prueba. Para obtener más información, vea [¿Qué es una versión de prueba?](./what-is-test-drive.md) |
| BYOL | La opción de incorporación de licencias propias permite a los clientes aportar licencias de software existentes a Azure.\* |
| Basado en uso | También conocida como pago por uso, esta opción permite a los clientes pagar por hora. |
| Demostración interactiva  | Proporcione a los clientes una experiencia guiada de su solución mediante una demostración interactiva. La ventaja es que puede ofrecer una experiencia de prueba sin tener que proporcionar una configuración complicada de la solución compleja. |
|

\* Como editor, debe brindar soporte a todos los aspectos de la transacción de las licencias de software, incluidas, entre otras cosas, el pedido, la entrega, la medición, la facturación, el pago y el cobro.

En el ejemplo siguiente se muestra una oferta de máquina virtual en Azure Marketplace que tiene precios basados en el uso.

:::image type="content" source="media/vm/sample-offer-screen.png" alt-text="Pantalla de oferta de VM de ejemplo.":::

### <a name="private-plans"></a>Planes privados

Puede restringir la detección y la implementación de su VM para un conjunto de clientes específico. Para ello, publique la imagen y los precios como un plan privado. Los planes privados le ofrecen la posibilidad de crear ofertas exclusivas para sus clientes más próximos y proporcionan software y términos personalizados. Los términos personalizados le permiten resaltar una gran variedad de escenarios, incluidas las ofertas sobre el terreno con precios y términos especializados, así como acceso anticipado a software de versión limitada. Los planes privados permiten ofrecer precios o productos específicos a un conjunto limitado de clientes.

Para obtener más información, vea [Planes y precios de las ofertas del marketplace comercial](plans-pricing.md) y [Ofertas privadas en el marketplace comercial de Microsoft](private-offers.md).

## <a name="test-drive"></a>Versión de prueba

Se puede elegir habilitar una versión de prueba para la VM. Las versiones de prueba proporcionan a los clientes acceso a un entorno preconfigurado durante un número fijo de horas. Aunque puede habilitar las opciones de publicación en cualquier versión de prueba, esta característica conlleva requisitos adicionales. Para más información sobre las versiones de prueba, consulte [¿Qué es una versión de prueba?](what-is-test-drive.md). Para información sobre la configuración de diferentes tipos de versiones de prueba, consulte [Configuración técnica de la versión de prueba](test-drive-technical-configuration.md).

[!INCLUDE [Test drives section](includes/test-drives.md)]

## <a name="customer-leads"></a>Clientes potenciales

[!INCLUDE [Customer leads section](includes/customer-leads.md)]

## <a name="legal-contracts"></a>Contratos legales

[!INCLUDE [Legal contracts section](includes/legal-contracts-intro.md)]
[!INCLUDE [Legal section for Standard contracts](includes/legal-contracts-standard.md)]

## <a name="cloud-solution-providers"></a>Proveedores de soluciones en la nube

Al crear la oferta en el Centro de partners, verá la pestaña **Revender mediante los CSP**. Esta opción permite que los asociados que forman parte del programa de proveedores de soluciones en la nube (CSP) de Microsoft revendan la VM como parte de una oferta agrupada. Todos los planes Traiga su propia licencia (BYOL) se incorporan automáticamente al programa. También puede optar por participar en los planes que no sean BYOL. Para más información, consulte [Programa del Proveedor de soluciones en la nube](cloud-solution-providers.md). 

> [!NOTE]
> La participación en el canal de asociados de Proveedores de soluciones en la nube (CSP) ya está disponible. Para más información sobre cómo comercializar su oferta mediante los canales de asociados de CSP de Microsoft, consulte [**Proveedores de soluciones en la nube**](./cloud-solution-providers.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de máquina virtual en Azure Marketplace](azure-vm-create.md)
- [Creación de una máquina virtual con una base aprobada](azure-vm-create-using-approved-base.md) o [Creación de una máquina virtual con su propia imagen](azure-vm-create-using-own-image.md).
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)
