---
title: Configuración de la preferencia de enrutamiento de una VM - Azure Portal
description: Obtenga información sobre cómo crear una VM con una dirección IP pública con preferencia de enrutamiento mediante Azure Portal.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2021
ms.author: mnayak
ms.openlocfilehash: 555fd1d3132ae44ed9abb158dee76af3f5761146
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082161"
---
# <a name="configure-routing-preference-for-a-vm-using-the-azure-portal"></a>Configuración de la preferencia de enrutamiento de una VM mediante Azure Portal

En este artículo se muestra cómo configurar las preferencias de enrutamiento para una máquina virtual. El tráfico de Internet enlazado desde la VM se enrutará a través de la red del ISP cuando elija **Internet** como opción de preferencia de enrutamiento. El enrutamiento predeterminado se hará a través de la red global de Microsoft.

En este artículo se muestra cómo crear una máquina virtual con una dirección IP pública que esté configurada para enrutar el tráfico a través de la red pública de Internet mediante Azure Portal.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Proceso** y, luego, **Máquina virtual de Windows Server 2016** u otro sistema operativo de su elección.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    |Configuración|Valor|
    |---|---|
    |Nombre|myVM|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.yml?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm-).|
    |Subscription| Seleccione su suscripción.|
    |Resource group| Seleccione **Usar existente** y, a continuación, **myResourceGroup**.|
    |Location| Seleccione **Este de EE. UU**.|

4. Seleccione un tamaño para la máquina virtual y luego **Seleccionar**.
5. En la pestaña de **redes**, haga clic en **Crear nueva** para crear una **dirección IP pública**.
6. Escriba *myPublicIpAddress*, seleccione una SKU como **estándar** y, a continuación, seleccione **Internet** como preferencia de enrutamiento; una vez hecho esto, presione **Aceptar**, tal como se muestra en la siguiente imagen:

   ![Selección de estática](./media/tutorial-routing-preference-virtual-machine-portal/routing-preference-internet-new.png)

6. Seleccione un puerto o ningún puerto en **Seleccionar puertos de entrada públicos**. Se selecciona el puerto 3389 para permitir el acceso remoto a la máquina virtual Windows Server desde Internet. No se recomienda abrir el puerto 3389 desde Internet con cargas de trabajo de producción.

   ![Selección de un puerto](./media/tutorial-routing-preference-virtual-machine-portal/pip-ports-new.png)

7. Acepte los valores predeterminados restantes y seleccione **Aceptar**.
8. En la página **Resumen**, seleccione **Crear**. La máquina virtual tarda minutos en crearse.
9. Una vez implementada la máquina virtual, escriba *myPublicIpAddress* en el cuadro de búsqueda de la parte superior del portal. Cuando **myPublicIpAddress** aparezca en los resultados de búsqueda, selecciónela.
10. Puede ver la dirección IP pública asignada, y que la dirección se asigna a la máquina virtual **myVM**, como se muestra en la siguiente imagen:

    ![Captura de pantalla que muestra la IP pública de la NIC para la interfaz de red mynic.](./media/tutorial-routing-preference-virtual-machine-portal/pip-properties-new.png)

11. Seleccione **Redes** y haga clic en el valor NIC **mynic**; a continuación, seleccione la dirección IP pública para confirmar que la preferencia de enrutamiento está asignada como **Internet**.

    ![En la captura de pantalla se muestra la dirección IP y la preferencia de enrutamiento de una dirección IP pública.](./media/tutorial-routing-preference-virtual-machine-portal/pip-routing-internet-new.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba *myResourceGroup* en el cuadro **Buscar** que se encuentra en la parte superior del portal. Seleccione **myResourceGroup** cuando lo vea en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba *myResourceGroup* para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre las [direcciones IP públicas con preferencia de enrutamiento](routing-preference-overview.md).
- Obtenga más información acerca de las [direcciones IP públicas](./public-ip-addresses.md#public-ip-addresses) en Azure.
- Obtenga más información acerca de toda la [configuración de direcciones IP públicas](virtual-network-public-ip-address.md#create-a-public-ip-address).