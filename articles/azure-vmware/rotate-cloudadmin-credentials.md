---
title: Rotación de las credenciales de administrador de la nube en Azure VMware Solution
description: Aprenda a rotar las credenciales de vCenter Server y NSX-T Manager para la nube privada de Azure VMware Solution.
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: 305447f8eac40a08564b9b57d82709f223f6086d
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112021508"
---
# <a name="rotate-the-cloudadmin-credentials-for-azure-vmware-solution"></a>Rotación de las credenciales de administrador de la nube en Azure VMware Solution

En este artículo, rotará las credenciales de cloudadmin (credenciales de vCenter y NSX-T) para la nube privada de Azure VMware Solution.  Aunque las contraseñas de estas cuentas no expiran, puede generar otras. Después de generar contraseñas nuevas, debe actualizar VMware HCX Connector con las credenciales más recientes aplicadas.

También puede ver un vídeo sobre cómo restablecer [vCenter CloudAdmin y contraseña de administrador de NSX-T](https://youtu.be/cK1qY3knj88). 

## <a name="prerequisites"></a>Requisitos previos

Si usa las credenciales de administrador de la nube para servicios conectados, como HCX, vRealize Orchestrator, vRealize Operations Manager o VMware Horizon, las conexiones dejarán de funcionar una vez que actualice la contraseña.  Detenga estos servicios antes de iniciar la rotación de contraseñas. Si no lo hace, sufrirá bloqueos temporales en las cuentas de administrador de vCenter CloudAdmin y NSX-T, ya que estos servicios llaman continuamente con sus credenciales anteriores.  Para obtener más información sobre cómo configurar cuentas independientes para los servicios conectados, vea [Conceptos de acceso e identidad](./concepts-identity.md).

## <a name="reset-your-azure-vmware-solution-cloudadmin-credentials"></a>Restablecimiento de las credenciales de Azure VMware Solution

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

## <a name="update-hcx-connector-with-the-latest-cloudadmin-credentials"></a>Actualización de HCX Connector con las credenciales de cloudadmin más recientes

En este paso, actualizará el conector HCX con las credenciales actualizadas.

1. Vaya al conector HCX local en https://{IP del dispositivo del conector HCX}:443 e inicie sesión con las nuevas credenciales.

   Asegúrese de usar el puerto 443. 

2. En el panel de VMware HCX, seleccione **Site Pairing** (Emparejamiento de sitios).
    
   :::image type="content" source="media/rotate-cloudadmin-credentials/hcx-site-pairing.png" alt-text="Captura de pantalla del panel de VMware HCX con el emparejamiento de sitios resaltado.":::
 
3. Seleccione la conexión correcta en Azure VMware Solution y seleccione **Edit Connection**.
 
4. Proporcione las nuevas credenciales de usuario de vCenter Server CloudAdmin y seleccione **Edit** (Editar), que guarda las credenciales. La operación de guardar debe mostrarse como correcta.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha examinado el restablecimiento de las credenciales de VCenter Server y NSX-T Manager para Azure VMware Solution, puede que quiera obtener información sobre lo siguiente:

- [Configuración de los componentes de red NSX en Azure VMware Solution](configure-nsx-network-components-azure-portal.md)
- [Integración de servicios nativos de Azure en Azure VMware Solution](integrate-azure-native-services.md)
- [Implementación de la recuperación ante desastres para las cargas de trabajo de Azure VMware Solution con VMware HCX](deploy-disaster-recovery-using-vmware-hcx.md)
