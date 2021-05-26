---
title: Rotación de las credenciales de administrador de la nube en Azure VMware Solution
description: Aprenda a rotar las credenciales de vCenter Server y NSX-T Manager para la nube privada de Azure VMware Solution.
ms.topic: how-to
ms.date: 05/11/2021
ms.openlocfilehash: fbfed495fd904b67ce283934791cee516349dfa6
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109814968"
---
# <a name="rotate-the-cloudadmin-credentials-for-azure-vmware-solution"></a>Rotación de las credenciales de administrador de la nube en Azure VMware Solution

Este artículo le guía por los pasos necesarios para rotar las credenciales de administrador de la nube para la nube privada de Azure VMware Solution.  Aunque las credenciales de administrador de vCenter Server CloudAdmin y NSX-T no expiran, puede generar nuevas contraseñas para estas cuentas. Después de rotar las credenciales, comprobará que el conector HCX tiene las credenciales de vCenter Server más recientes.

También puede ver un vídeo sobre cómo restablecer [vCenter CloudAdmin y contraseña de administrador de NSX-T](https://youtu.be/cK1qY3knj88). 

## <a name="prerequisites"></a>Requisitos previos

Si usa las credenciales de administrador de la nube para servicios conectados, como HCX, vRealize Orchestrator, vRealize Operations Manager o VMware Horizon, las conexiones dejarán de funcionar una vez que actualice la contraseña.  Detenga estos servicios antes de iniciar la rotación de contraseñas. Si no lo hace, sufrirá bloqueos temporales en las cuentas de administrador de vCenter CloudAdmin y NSX-T, ya que estos servicios llaman continuamente con sus credenciales anteriores.  Para obtener más información sobre cómo configurar cuentas independientes para los servicios conectados, vea [Conceptos de acceso e identidad](./concepts-identity.md).

## <a name="reset-your-azure-vmware-solution-credentials"></a>Restablecimiento de las credenciales de Azure VMware Solution

En este paso, rotará las credenciales de administrador de la nube para los componentes de Azure VMware Solution. 

>[!NOTE]
>No olvide reemplazar **{SubscriptionID},** **{ResourceGroup}** y **{PrivateCloudName}** por la información de su nube privada.

1. En Azure Portal, abra una sesión de Azure Cloud Shell.

2. Actualice la contraseña de vCenter CloudAdmin.  

   ```azurecli-interactive
   az resource invoke-action --action rotateVcenterPassword --ids "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.AVS/privateClouds/{PrivateCloudName}" --api-version "2020-07-17-preview"
   ```
          
3. Actualice la contraseña de administrador de NSX-T. 

   ```azurecli-interactive
   az resource invoke-action --action rotateNSXTPassword --ids "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.AVS/privateClouds/{PrivateCloudName}" --api-version "2020-07-17-preview"
   ```

## <a name="verify-hcx-connector-has-the-latest-credentials"></a>Comprobación de que el conector HCX tiene las credenciales más recientes

En este paso, comprobará que el conector HCX tiene las credenciales actualizadas.

1. Vaya al conector HCX local en https://{IP del dispositivo del conector HCX}:443 e inicie sesión con las nuevas credenciales.

   Asegúrese de usar el puerto 443. 

2. En el panel de VMware HCX, seleccione **Site Pairing** (Emparejamiento de sitios).
    
   :::image type="content" source="media/rotate-cloudadmin-credentials/hcx-site-pairing.png" alt-text="Captura de pantalla del panel de VMware HCX con el emparejamiento de sitios resaltado.":::
 
3. Seleccione la conexión correcta en Azure VMware Solution y seleccione **Edit Connection**.
 
4. Proporcione las nuevas credenciales de usuario de vCenter Server CloudAdmin y seleccione **Edit** (Editar), que guarda las credenciales. La operación de guardar debe mostrarse como correcta.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha examinado el restablecimiento de las credenciales de VCenter Server y NSX-T Manager para Azure VMware Solution, puede que quiera obtener información sobre lo siguiente:

- [Configuración de los componentes de red NSX en Azure VMware Solution](configure-nsx-network-components-azure-portal.md)
- [Supervisión y administración de máquinas virtuales de Azure VMware Solution](lifecycle-management-of-azure-vmware-solution-vms.md)
- [Implementación de una recuperación ante desastres de máquinas virtuales mediante Azure VMware Solution](disaster-recovery-for-virtual-machines.md)
