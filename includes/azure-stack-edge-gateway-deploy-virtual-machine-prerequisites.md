---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 01/15/2021
ms.author: alkohli
ms.openlocfilehash: f166413507afb9aff814eaddaade099d2e34ae68
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106554683"
---
Para poder implementar VM en el dispositivo Azure Stack Edge, debe configurar el cliente para que se conecte al dispositivo mediante Azure Resource Manager con Azure PowerShell. Para consultar las instrucciones detalladas, vaya a [Conexión a Azure Resource Manager en un dispositivo Azure Stack Edge Pro](../articles/databox-online/azure-stack-edge-gpu-connect-resource-manager.md).

Asegúrese de que se pueden usar los pasos siguientes para acceder al dispositivo desde el cliente. Ya ha realizado esta configuración cuando se conectó a Azure Resource Manager y ahora está comprobando que la configuración se ha realizado correctamente. 

1. Ejecute el siguiente comando para comprobar que la comunicación de Azure Resource Manager funciona:     

    ```powershell
    Add-AzureRmEnvironment -Name <Environment Name> -ARMEndpoint "https://management.<appliance name>.<DNSDomain>"
    ```

1. Para llamar a las API de dispositivo local para autenticarse, escriba: 

    `login-AzureRMAccount -EnvironmentName <Environment Name> -TenantId c0257de7-538f-415c-993a-1b87a031879d`

    Para conectarse mediante Azure Resource Manager, proporcione el nombre de usuario *EdgeArmUser* y la contraseña.

1. Si configuró el proceso para Kubernetes, puede omitir este paso. De lo contrario, asegúrese de que ha habilitado una interfaz de red para el proceso de la siguiente manera: 

   a. En la interfaz de usuario local, vaya la configuración de **Proceso**.  
   b. Seleccione la interfaz de red que desea usar para crear un conmutador virtual. Las máquinas virtuales que cree se adjuntarán a un conmutador virtual conectado a este puerto y a la red asociada. Asegúrese de elegir una red que coincida con la dirección IP que usará para la máquina virtual.  

    ![Captura de pantalla del panel de configuración de red de la configuración de proceso.](../articles/databox-online/media/azure-stack-edge-gpu-deploy-virtual-machine-templates/enable-compute-setting.png)

   c. En **Enable for compute** (Habilitar para proceso) en la interfaz de red, seleccione **Yes** (Sí). Azure Stack Edge creará y administrará un conmutador virtual correspondiente a esa interfaz de red. No especifique direcciones IP específicas para Kubernetes en este momento. La habilitación del proceso puede tardar varios minutos.

    > [!NOTE]
    > Si va a crear máquinas virtuales de GPU, seleccione una interfaz de red conectada a Internet, ya que ello le permite instalar una extensión de GPU en el dispositivo.