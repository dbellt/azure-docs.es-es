---
title: Restablecimiento de las credenciales de vSphere para Azure VMware Solution
description: Aprenda a restablecer las credenciales de vSphere para la nube privada de Azure VMware Solution y asegúrese de que el conector de HCX tiene las credenciales de vSphere más recientes.
ms.topic: how-to
ms.date: 03/31/2021
ms.openlocfilehash: 3533de03cbea0014a8a50cd0103f96273f0680a7
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145310"
---
# <a name="reset-vsphere-credentials-for-azure-vmware-solution"></a>Restablecimiento de las credenciales de vSphere para Azure VMware Solution

En este artículo, se le guiará por los pasos necesarios para restablecer las credenciales de vCenter Server y NSX-T Manager para la nube privada de Azure VMware Solution. Esto le permitirá asegurarse de que el conector de HCX tiene las credenciales de vCenter Server más recientes.

Además de este procedimiento, también puede ver el vídeo para [restablecer la contraseña de administrador de vCenter CloudAdmin & NSX-T](https://youtu.be/cK1qY3knj88).

## <a name="reset-your-azure-vmware-solution-credentials"></a>Restablecimiento de las credenciales de Azure VMware Solution

 En primer lugar, vamos a restablecer las credenciales de los componentes de Azure VMware Solution. Las credenciales de administrador de vCenter Server CloudAdmin y NSX-T no expiran; sin embargo, puede seguir estos pasos para generar nuevas contraseñas para estas cuentas.

> [!NOTE]
> Si usa las credenciales de CloudAdmin para servicios conectados, como HCX, vRealize Orchestrator, vRealize Operations Manager o VMware Horizon, las conexiones dejarán de funcionar una vez que actualice la contraseña.  Estos servicios deben detenerse antes de iniciar la rotación de contraseñas.  Si no lo hace, pueden producirse bloqueos temporales en las cuentas de administrador de vCenter CloudAdmin y NSX-T, ya que estos servicios llamarán continuamente con sus credenciales antiguas.  Para obtener más información sobre cómo configurar cuentas independientes para los servicios conectados, vea [Conceptos de acceso e identidad](./concepts-identity.md).

1. En Azure Portal, abra una sesión de Azure Cloud Shell.

2. Ejecute el siguiente comando para actualizar la contraseña de vCenter CloudAdmin.  Tendrá que reemplazar {SubscriptionID}, {ResourceGroup} y {PrivateCloudName} por los valores reales de la nube privada a la que pertenece la cuenta de CloudAdmin.

```
az resource invoke-action --action rotateVcenterPassword --ids "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.AVS/privateClouds/{PrivateCloudName}" --api-version "2020-07-17-preview"
```
          
3. Ejecute el siguiente comando para actualizar la contraseña de administrador de NSX-T. Tendrá que reemplazar {SubscriptionID}, {ResourceGroup} y {PrivateCloudName} por los valores reales de la nube privada a la que pertenece la cuenta de administrador de NSX-T.

```
az resource invoke-action --action rotateNSXTPassword --ids "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.AVS/privateClouds/{PrivateCloudName}" --api-version "2020-07-17-preview"
```

## <a name="ensure-the-hcx-connector-has-your-latest-vcenter-server-credentials"></a>Comprobación de que el conector de HCX tiene las credenciales de vSphere más recientes

Ahora que ha restablecido sus credenciales, siga estos pasos para asegurarse de que el conector de HCX tiene las credenciales actualizadas.

1. Una vez cambiada la contraseña, vaya a la interfaz web del conector de HCX local con https://{ip of the HCX connector appliance}:443. Asegúrese de usar el puerto 443. Inicie sesión con las nuevas credenciales.

2. En el panel de VMware HCX, seleccione **Site Pairing** (Emparejamiento de sitios).
    
    :::image type="content" source="media/reset-vsphere-credentials/hcx-site-pairing.png" alt-text="Captura de pantalla del panel de VMware HCX con el emparejamiento de sitios resaltado.":::
 
3. Seleccione la conexión correcta a Azure VMware Solution (si hay más de una) y seleccione **Edit Connection** (Editar conexión).
 
4. Proporcione las nuevas credenciales de usuario de vCenter Server CloudAdmin y seleccione **Edit** (Editar), que guarda las credenciales. La operación de guardar debe mostrarse como correcta.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha examinado el restablecimiento de las credenciales de VCenter Server y NSX-T Manager para Azure VMware Solution, puede que quiera obtener información sobre lo siguiente:

- [Configuración de los componentes de red NSX en Azure VMware Solution](configure-nsx-network-components-azure-portal.md).
- La [administración del ciclo de vida de las máquinas virtuales de Azure VMware Solution](lifecycle-management-of-azure-vmware-solution-vms.md).
- [Implementación de una recuperación ante desastres de máquinas virtuales usando Azure VMware Solution](disaster-recovery-for-virtual-machines.md).