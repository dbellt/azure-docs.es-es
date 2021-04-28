---
title: Cree una oferta de máquina virtual de Azure en Azure Marketplace.
description: Obtenga información sobre cómo crear una oferta de máquina virtual en el marketplace comercial de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 04/08/2021
ms.openlocfilehash: f0c1d9d528ed4fbf61786042fb6fb34f05fec5d5
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107812600"
---
# <a name="how-to-create-a-virtual-machine-offer-on-azure-marketplace"></a>Creación de una oferta de máquina virtual de Azure en Azure Marketplace

En este artículo se explica cómo crear una oferta de máquina virtual de Azure para [Azure Marketplace](https://azuremarketplace.microsoft.com/). Se dirige a máquinas virtuales basadas en Windows y en Linux que contienen un sistema operativo, un disco duro virtual (VHD) y hasta 16 discos de datos.

Antes de comenzar, [cree una cuenta de marketplace comercial en el Centro de partners](create-account.md). Asegúrese de que la cuenta esté inscrita en el programa Marketplace comercial.

## <a name="before-you-begin"></a>Antes de empezar

Si todavía no lo ha hecho, revise [Planeación de una oferta de máquina virtual](marketplace-virtual-machines.md). Ahí se explican los requisitos técnicos de la máquina virtual y se enumera la información y los recursos que necesitará al crear la oferta.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
2. En el panel izquierdo, seleccione **Marketplace comercial** > **Información general**.
3. En la página **Información general**, seleccione **Nueva oferta** > **Máquina virtual de Azure**.

    ![Captura de pantalla que muestra las opciones de menú del panel izquierdo y el botón "Nueva oferta".](./media/create-vm/new-offer-azure-virtual-machine.png)

> [!NOTE]
> Una vez publicada la oferta, las modificaciones que realice en el Centro de partners aparecerán en Azure Marketplace solo después de volver a publicarla. Tras realizar cambios en la oferta, asegúrese siempre de volver a publicarla.

Escriba un **Identificador de oferta**. Se trata de un identificador único para cada oferta de su cuenta.

- Este identificador se muestra a los clientes en la dirección web de la oferta de Azure Marketplace y en Azure PowerShell y la CLI de Azure, si procede.
- Use solo letras minúsculas y números. El identificador puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres. Por ejemplo, si escribe **test-offer-1**, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

Escriba un **Alias de la oferta**. El alias de la oferta es el nombre que se usa para la oferta en el Centro de partners.

- Este nombre no se utiliza en Azure Marketplace. Es diferente del nombre de la oferta y de otros valores que se muestran a los clientes.

Seleccione **Crear** para generar la oferta y continuar. En el Centro de partners, abra la página **Configuración de la oferta**.

## <a name="enable-a-test-drive-optional"></a>Habilitación de una versión de prueba (opcional)

Una versión de prueba es una excelente manera de exhibir la oferta a posibles clientes al concederles acceso a un entorno preconfigurado durante un número fijo de horas. La oferta de una versión de prueba tiene como resultado una mayor tasa de conversión y genera clientes potenciales muy cualificados. Para más información sobre las versiones de prueba, consulte [¿Qué es una versión de prueba?](./what-is-test-drive.md).

> [!TIP]
> Una versión de prueba es diferente de una evaluación gratuita. Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

Para habilitar una versión de prueba, seleccione la casilla **Habilitar una versión de prueba**. La versión de prueba se configurará más adelante. Con la versión de prueba, es necesario configurar un CRM (vea la sección siguiente).

## <a name="configure-customer-leads-management"></a>Configuración de la administración de clientes potenciales

[!INCLUDE [Customer leads](includes/customer-leads.md)] 

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña del menú de navegación izquierdo, **Propiedades**.

## <a name="next-steps"></a>Pasos siguientes

- [Cómo configurar las propiedades de la oferta de máquinas virtuales](azure-vm-create-properties.md)
- [Procedimientos recomendados para la publicación de ofertas](gtm-offer-listing-best-practices.md)