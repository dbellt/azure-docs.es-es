---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 05/27/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c5741ab0207d5631aab4abd594f5ff557ebfe23b
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110657992"
---
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En **Buscar recursos, servicios y documentos (G +/)** , escriba *red virtual*.

   :::image type="content" source="./media/vpn-gateway-basic-vnet-rm-portal-include/marketplace.png" alt-text="La captura de pantalla muestra la barra de búsqueda de Azure Portal." border="false":::
1. Seleccione **Red virtual** en los resultados de **Marketplace**.

   :::image type="content" source="./media/vpn-gateway-basic-vnet-rm-portal-include/marketplace-results.png" alt-text="La captura de pantalla muestra los resultados de la barra de búsqueda de Azure Portal y la selección de Red virtual en Marketplace." border="false":::
1. En la página **Red virtual**, seleccione **Crear**.

   :::image type="content" source="./media/vpn-gateway-basic-vnet-rm-portal-include/vnet-click-create.png" alt-text="La captura de pantalla muestra la página Red virtual y la selección del botón Crear." border="false":::
1. Una vez que haya seleccionado **Crear**, se abrirá la página **Crear red virtual**.
1. En la pestaña **Aspectos básicos**, configure las opciones de configuración de la red virtual **Detalles del proyecto** y **Detalles de la instancia**.

   :::image type="content" source="./media/vpn-gateway-basic-vnet-rm-portal-include/basics.png" alt-text="La captura de pantalla que muestra la pestaña Datos básicos." border="false":::

   Tras rellenar los campos, verá una marca de verificación verde cuando se validen los caracteres que escribe en el campo. Algunos valores se rellenan automáticamente, que puede sustituir por sus propios valores:

   - **Suscripción**: compruebe que la suscripción que aparece en la lista es la correcta. Puede cambiar las suscripciones mediante la lista desplegable.
   - **Grupo de recursos**: seleccione uno existente o haga clic en **Crear** para crear un grupo de recursos nuevo. Para más información sobre los grupos de recursos, consulte [Información general de Azure Resource Manager](../articles/azure-resource-manager/management/overview.md#resource-groups).
   - **Name**: escriba el nombre de la red virtual.
   - **Región**: seleccione la ubicación de la red virtual. La ubicación determina dónde van a residir los recursos que se implementen en esta red virtual.

1. Configure los valores en la pestaña **Direcciones IP**. Los valores que se muestran en los ejemplos siguientes son para fines de demostración. Ajuste estos valores según las opciones de configuración que necesite.

   :::image type="content" source="./media/vpn-gateway-basic-vnet-rm-portal-include/addresses.png" alt-text="La captura de pantalla muestra la pestaña Direcciones IP." border="false"::: 
   - **Espacio de direcciones IPv4**: de manera predeterminada, se crea automáticamente un espacio de direcciones. Puede hacer clic en el espacio de direcciones para modificarlo a fin de que refleje sus valores. También puede agregar espacios de direcciones adicionales.
   - **Subred**: si usa el espacio de direcciones predeterminado, se crea automáticamente una subred predeterminada. Si cambia el espacio de direcciones, debe agregar una subred. Seleccione **+ Agregar una subred** para abrir la ventana **Agregar subred**. Configure las siguientes opciones y, a continuación, seleccione **Agregar** para agregar los valores:
      - **Nombre de subred**: en este ejemplo, asignamos a la subred el nombre "FrontEnd".
      - **Rango de direcciones de subred**: intervalo de direcciones para esta subred.

1. En la pestaña **Seguridad**, en este momento, deje los valores predeterminados:

   - **Protección contra DDos**: Básico
   - **Firewall**: Disabled
1. Seleccione **Revisar y crear** para validar la configuración de la red virtual.
1. Una vez validada la configuración, seleccione **Crear**.
