---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/27/2020
ms.author: alkohli
ms.openlocfilehash: efa9e996cbfbc8954c294b4da8900b1d5bcbeeed
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96465822"
---
Para comprobar la red privada virtual, puede crear una cuenta de almacenamiento a la que solo pueda acceder la red virtual que creó. Siga estos pasos para crear y asociar esta cuenta de almacenamiento con la red virtual que creó:

1. Cree una cuenta de almacenamiento. Puede usar la cuenta de almacenamiento que planea usar con el dispositivo Azure Stack Edge. Intente acceder a la cuenta de almacenamiento desde una red externa (fuera de la red seleccionada). Se debería poder acceder a la red.
2. En Azure Portal, vaya a la cuenta de almacenamiento. 
3. Vaya a **Firewalls y redes virtuales**. En el panel de la derecha de **Permitir acceso desde**, elija **Redes seleccionadas**. En **Proteja su cuenta de almacenamiento con redes virtuales**, elija **+ Agregar red virtual existente**.

    ![Comprobación de VPN 1](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-1.png)

4. En la hoja **Agregar redes**:

    - Seleccione la suscripción. Se trata de la misma suscripción que está asociada con el recurso de Azure Stack Edge/Data Box Gateway. 
    - En la lista desplegable, elija la red virtual que se va a asociar con esta cuenta de almacenamiento. Seleccione la red virtual que creó en el paso anterior.
    - En **Subredes**, elija **_default_* _ y _GatewaySubnet*. Los puntos de conexión de servicio se habilitarán para las combinaciones de esta red virtual/subred. El proceso para habilitar el acceso tarda hasta 15 minutos en completarse.
    - Seleccione **Habilitar**.

    ![Comprobación de VPN 2](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-2.png)
    
4. Guarde la **configuración**.

    ![Comprobación de VPN 3](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-3.png)

5. Una vez guardada la configuración, puede ver las subredes para las que está habilitada la red virtual.

    ![Comprobación de VPN 4](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-4.png)

5. Para comprobar que los datos ahora solo van a través de la red privada virtual, siga estos pasos: 
    - Intente acceder a la cuenta de almacenamiento desde una red externa (fuera de la red seleccionada). No se debería poder acceder a la red. 
    - Intente acceder a la cuenta de almacenamiento desde la red virtual/subredes que habilitó en las redes seleccionadas. Se debería poder acceder a la red. 
 
Solo puede acceder a esta cuenta de almacenamiento si tiene habilitada la VPN. Si deshabilita la VPN, también tendrá que ajustar la configuración de red de la cuenta de almacenamiento. 

Para más información, vaya a [Configuración de Firewalls y redes virtuales de Azure Storage](../articles/storage/common/storage-network-security.md). 

