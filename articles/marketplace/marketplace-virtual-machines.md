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
ms.openlocfilehash: 6e391ca79c9dcf9ab533a7e67961c654b5f8630b
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107103375"
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

Una audiencia preliminar puede tener acceso a la oferta de la VM antes de publicarla en Azure Marketplace a fin de probar la funcionalidad completa. En la página **Audiencia preliminar**, puede definir una audiencia preliminar limitada. 

> [!NOTE]
> Una audiencia preliminar no es lo mismo que un plan privado. Un plan privado es el que solo está disponible para una audiencia determinada de su elección. Esto le permite negociar un plan personalizado con clientes específicos. Para obtener más información, vea la próxima sección: Planes

Puede enviar invitaciones a direcciones de correo electrónico de cuentas de Microsoft (MSA) o de Azure Active Directory (Azure AD). Agregue hasta 10 direcciones de correo electrónico manualmente o importe hasta 20 con un archivo .csv. Aunque la oferta ya esté publicada, puede definir una audiencia preliminar para probar los cambios o las actualizaciones que realice en ella.

## <a name="plans-and-pricing"></a>Planes y precios

Las ofertas de VM requieren al menos un plan. Un plan define el ámbito y los límites de la solución, así como los precios asociados. Puede crear varios planes para que su oferta brinde a sus clientes diferentes opciones técnicas y de licencias, así como evaluaciones gratuitas. Consulte [Planes y precios de ofertas de marketplace comercial](plans-pricing.md) para obtener instrucciones generales sobre los planes, incluidos los modelos de precios, las evaluaciones gratuitas y los planes privados. 

Las VM se pueden comercializar íntegramente con los modelos de licencia de pago por uso o BYOL (traiga su propia licencia). Microsoft hospeda la transacción comercial y factura al cliente en su nombre. Usted obtiene la ventaja de usar la relación de pago preferida entre el cliente y Microsoft, incluidos los contratos empresariales existentes. Para obtener más información, consulte [Capacidades de transacción de marketplace comercial](./marketplace-commercial-transaction-capabilities-and-considerations.md).

> [!NOTE]
> El pago por adelantado de Azure (que anteriormente se denominaba compromisos monetarios) asociado con un Contrato Enterprise se puede aplicar al uso que Azure hace de su máquina virtual, pero no a las tarifas de licencias de software.

### <a name="licensing-options"></a>Opciones de licencia

Cuando prepare la publicación de una nueva oferta de VM, debe decidir la opción de licencia. Esta opción determina la información adicional que deberá proporcionar más adelante cuando cree la oferta en el Centro de partners.

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

> [!TIP]
> Una versión de prueba es diferente de una [evaluación gratuita](plans-pricing.md#free-trials). Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

## <a name="customer-leads"></a>Clientes potenciales

Debe conectar su oferta al sistema de administración de relaciones con clientes (CRM) para recopilar información del cliente. El cliente le pedirá permiso para compartir su información. Estos datos del cliente, junto con el nombre y el identificador de la oferta, además de la tienda en línea donde la encontró, se envían al sistema CRM que haya configurado. El marketplace comercial admite una gran variedad de sistemas CRM, junto con la opción de usar una tabla de Azure o de configurar un punto de conexión HTTPS con Power Automate.

Puede agregar o modificar una conexión CRM en cualquier momento durante o después de la creación de la oferta. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

## <a name="legal-contracts"></a>Contratos legales

Para simplificar el proceso de adquisición para los clientes y reducir la complejidad legal para los proveedores de software, Microsoft ofrece un contrato estándar que se puede usar con las ofertas del marketplace comercial. Al ofrecer el software bajo el contrato estándar, los clientes solo tienen que leerlo y aceptarlo, sin necesidad de crear términos y condiciones personalizados.

Si decide usar el contrato estándar, tiene la opción de agregarle términos de modificación universales y hasta 10 modificaciones personalizadas. También puede usar sus propios términos y condiciones en lugar del contrato estándar. Estos detalles se administrarán en la página **Propiedades**. Para más información, consulte [Contrato estándar para el marketplace comercial de Microsoft](standard-contract.md).

> [!NOTE]
> Después de publicar una oferta con el contrato estándar en el marketplace comercial, no puede usar sus propios términos y condiciones personalizados. Solo puede elegir una de las dos opciones. O bien ofrecer su solución bajo el contrato estándar o según sus propios términos y condiciones. Si quiere modificar los términos del contrato estándar, puede hacerlo a través de las modificaciones al contrato estándar.

## <a name="cloud-solution-providers"></a>Proveedores de soluciones en la nube

Al crear la oferta en el Centro de partners, verá la pestaña **Revender mediante los CSP**. Esta opción permite que los asociados que forman parte del programa de proveedores de soluciones en la nube (CSP) de Microsoft revendan la VM como parte de una oferta agrupada. Todos los planes Traiga su propia licencia (BYOL) se incorporan automáticamente al programa. También puede optar por participar en los planes que no sean BYOL. Para más información, consulte [Programa del Proveedor de soluciones en la nube](cloud-solution-providers.md). 

> [!NOTE]
> La participación en el canal de asociados de Proveedores de soluciones en la nube (CSP) ya está disponible. Para más información sobre cómo comercializar su oferta mediante los canales de asociados de CSP de Microsoft, consulte [**Proveedores de soluciones en la nube**](./cloud-solution-providers.md).

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de máquina virtual en Azure Marketplace](azure-vm-create.md)
- [Creación de una máquina virtual con una base aprobada](azure-vm-create-using-approved-base.md) o [Creación de una máquina virtual con su propia imagen](azure-vm-create-using-own-image.md).
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)
