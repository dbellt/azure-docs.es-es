---
title: 'Actualizaciones de servicio para los grupos de host de Azure Virtual Desktop: Azure'
description: Cómo crear un grupo host de validación para supervisar actualizaciones de servicio antes de implementar las actualizaciones en producción.
author: Heidilohr
ms.topic: tutorial
ms.date: 12/15/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: femila
ms.openlocfilehash: 2c944d1068ae74a97c8a6315e98a1348f9378b8c
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111749136"
---
# <a name="tutorial-create-a-host-pool-to-validate-service-updates"></a>Tutorial: Creación de un grupo host para validar las actualizaciones del servicio

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop con objetos de Azure Resource Manager. Si usa Azure Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/create-validation-host-pool-2019.md).

Los grupos de hosts son una colección de una o varias máquinas virtuales idénticas en un entorno de Azure Virtual Desktop. Le recomendamos encarecidamente que cree un grupo de hosts de validación en el que se apliquen primero las actualizaciones del servicio. Esto permite supervisar las actualizaciones del servicio antes de que este las aplique a su entorno estándar o sin validación. Sin un grupo de hosts de validación, es posible que no se detecten cambios que introduzcan errores, lo que podría dar lugar a tiempos de inactividad para los usuarios en el entorno estándar.

Para garantizar que las aplicaciones funcionan con las actualizaciones más recientes, el grupo de hosts de validación debe ser lo más parecido posible a los grupos de hosts del entorno sin validación. Los usuarios deben conectarse al grupo de hosts de validación con la misma frecuencia que lo hacen al estándar. Si ha automatizado las pruebas en su grupo host, debe incluir las pruebas automatizadas en el grupo host de validación.

Puede depurar incidencias en el grupo host de validación con [la característica de diagnóstico](diagnostics-role-service.md) o con [artículos de solución de problemas de Azure Virtual Desktop](troubleshoot-set-up-overview.md).

>[!NOTE]
> Se recomienda que mantenga el grupo host de validación para probar todas las actualizaciones futuras.

>[!IMPORTANT]
>Azure Virtual Desktop con la integración de Administración de recursos de Azure actualmente tiene problemas para habilitar y deshabilitar los entornos de validación. Este artículo se actualizará cuando se haya resuelto el problema.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, siga las instrucciones que se indican en [Configuración del módulo de PowerShell para Azure Virtual Desktop](powershell-module.md) para configurar el módulo de PowerShell e iniciar sesión en Azure.

## <a name="create-your-host-pool"></a>Creación del grupo host

Puede crear un grupo host siguiendo las instrucciones de cualquiera de estos artículos:
- [Tutorial: Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace.md)
- [Creación de un grupo host con PowerShell](create-host-pools-powershell.md)

## <a name="define-your-host-pool-as-a-validation-host-pool"></a>Definición del grupo host como grupo host de validación

Ejecute los siguientes cmdlets de PowerShell para definir el nuevo grupo host como grupo host de validación. Reemplace los valores entre corchetes por los valores pertinentes para su sesión:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -ValidationEnvironment:$true
```

Ejecute el siguiente cmdlet de PowerShell para confirmar que la propiedad de validación se ha definido. Reemplace los valores entre corchetes por los valores pertinentes para su sesión.

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | Format-List
```

Los resultados del cmdlet deben ser similares a estos:

```powershell
    HostPoolName        : hostpoolname
    FriendlyName        :
    Description         :
    Persistent          : False
    CustomRdpProperty   : use multimon:i:0;
    MaxSessionLimit     : 10
    LoadBalancerType    : BreadthFirst
    ValidationEnvironment : True
```

## <a name="enable-your-validation-environment-with-the-azure-portal"></a>Habilitación de un entorno de validación con Azure Portal

Para habilitar un entorno de validación también se puede usar Azure Portal.

Para usar Azure Portal para configurar un grupo de hosts de validación:

1. Inicie sesión en Azure Portal en <https://portal.azure.com>.
2. Busque y seleccione **Azure Virtual Desktop**.
3. En la página de Azure Virtual Desktop, seleccione **Grupos de hosts**.
4. Seleccione el nombre del grupo de hosts que quiera editar.
5. Seleccione **Propiedades**.
6. En el campo del entorno de validación, seleccione **Sí** para habilitar el entorno de validación.
7. Seleccione **Guardar**. De esta forma se aplicará la nueva configuración.

## <a name="update-schedule"></a>Programación de actualizaciones

Las actualizaciones del servicio se realizan mensualmente. Si hay incidencias importantes, se proporcionarán actualizaciones críticas con mayor frecuencia.

Si hay alguna actualización del servicio, asegúrese de que tenga al menos un pequeño grupo de usuarios que inician sesión cada día para validar el entorno. Le recomendamos que visite periódicamente nuestro [sitio de TechCommunity](https://techcommunity.microsoft.com/t5/forums/searchpage/tab/message?filter=location&q=wvdupdate&location=forum-board:WindowsVirtualDesktop&sort_by=-topicPostDate&collapse_discussion=true) y siga cualquier publicación con WVDUPdate para mantenerse informado sobre las actualizaciones del servicio.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado un grupo de hosts de validación, puede obtener información sobre cómo usar Azure Service Health para supervisar la implementación de Azure Virtual Desktop.

> [!div class="nextstepaction"]
> [Configuración de alertas de servicio](./set-up-service-alerts.md)
