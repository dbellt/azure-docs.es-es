---
title: Instalación de Azure Firewall en un centro de Virtual WAN
titleSuffix: Azure Virtual WAN
description: Aprenda a configurar Azure Firewall en un centro de Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 05/26/2021
ms.author: cherylmc
ms.openlocfilehash: 90d1233a1374d4ad65410c818ab6e42b36a4e6f9
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579656"
---
# <a name="configure-azure-firewall-in-a-virtual-wan-hub"></a>Configuración de Azure Firewall en un centro de Virtual WAN

Un **centro de conectividad protegido** es un centro de Azure Virtual WAN con Azure Firewall. Este artículo le guía por los pasos necesarios para convertir un centro de conectividad de WAN virtual en un centro protegido mediante la instalación de Azure Firewall directamente desde las páginas del portal de Azure Virtual WAN.

## <a name="before-you-begin"></a>Antes de empezar

En los pasos de este artículo se da por hecho que ya ha implementado una WAN virtual con uno o más centros.

Para crear una nueva WAN virtual y un nuevo centro, siga los pasos que se describen en los siguientes artículos:

* [Creación de una instancia de Virtual WAN](virtual-wan-site-to-site-portal.md#openvwan)
* [Creación de un centro de conectividad](virtual-wan-site-to-site-portal.md#hub)

## <a name="view-virtual-hubs"></a>Visualización de centros de conectividad virtuales

En la página **Información general** de la WAN virtual se muestra una lista de los centros de conectividad virtuales y los centros protegidos. En la ilustración siguiente se muestra una WAN virtual sin centros protegidos.

:::image type="content" source="./media/howto-firewall/overview.png" alt-text="Captura de pantalla que muestra la página de información general de una instancia de Virtual WAN con una lista de centros virtuales." lightbox="./media/howto-firewall/overview.png":::

## <a name="convert-to-secured-hub"></a>Convertir en centro protegido

1. En la página **Información general** de la WAN virtual, seleccione el centro que desea convertir en un centro protegido. En la página del centro virtual, verá dos opciones para implementar Azure Firewall en este centro. Seleccione cualquiera de las opciones.

   :::image type="content" source="./media/howto-firewall/security.png" alt-text="Captura de pantalla que muestra la página de información general de la instancia de Virtual WAN, donde puede seleccionar Convertir en centro seguro o Azure Firewall." lightbox="./media/howto-firewall/security.png":::

1. Después de seleccionar una de las opciones, verá la página **Convertir en centro seguro**. Seleccione el centro que desea convertir y, a continuación, seleccione **Siguiente: Azure Firewall** en la parte inferior de la página.

   :::image type="content" source="./media/howto-firewall/select-hub.png" alt-text="Captura de pantalla de Convert to secure hub (Convertir en centro seguro) con un centro seleccionado." lightbox="./media/howto-firewall/select-hub.png":::
1. Después de completar el flujo de trabajo, seleccione **Confirmar**.

   :::image type="content" source="./media/howto-firewall/confirm.png" alt-text="Captura de pantalla que muestra el panel Convertir en centro seguro con la opción Confirmar seleccionada." lightbox="./media/howto-firewall/confirm.png":::
1. Una vez que el centro se ha convertido en un centro protegido, puede verlo en la página de **Información general** de la WAN virtual.

   :::image type="content" source="./media/howto-firewall/secured-hub.png" alt-text="Captura de pantalla de la vista del centro protegido." lightbox="./media/howto-firewall/secured-hub.png":::

## <a name="view-hub-resources"></a>Visualización de recursos de centro

En la página de **Información general** de la WAN virtual, seleccione el centro protegido. En la página del centro, puede ver todos los recursos del centro virtual, incluido Azure Firewall.

Para ver la configuración de Azure Firewall desde el centro protegido, en **Seguridad** seleccione **Configuración del centro virtual protegido**.

:::image type="content" source="./media/howto-firewall/hub-settings.png" alt-text="Captura de pantalla de la configuración del centro virtual protegido." lightbox="./media/howto-firewall/hub-settings.png":::

## <a name="configure-additional-settings"></a>Configuración de valores adicionales

Para configurar valores adicionales de Azure Firewall para el centro virtual, seleccione el vínculo a **Azure Firewall Manager**. Para obtener información acerca de las directivas de firewall, consulte [Azure Firewall Manager](../firewall-manager/secure-cloud-network.md#create-a-firewall-policy-and-secure-your-hub).

:::image type="content" source="./media/howto-firewall/additional-settings.png" alt-text="Captura de pantalla de información general, donde se ha seleccionado la opción para administrar la ruta del proveedor de seguridad de este centro virtual protegido en Azure Firewall Manager." lightbox="./media/howto-firewall/additional-settings.png":::

Para volver a la página **Información general** del centro, puede volver haciendo clic en la ruta de acceso, tal como muestra la flecha de la ilustración a continuación.

:::image type="content" source="./media/howto-firewall/arrow.png" alt-text="Captura de pantalla que muestra cómo volver a la página de información general." lightbox="./media/howto-firewall/arrow.png":::

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Virtual WAN, consulte las [preguntas más frecuentes](virtual-wan-faq.md).
