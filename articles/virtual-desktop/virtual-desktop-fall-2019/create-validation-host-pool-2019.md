---
title: 'Actualizaciones de servicio para los grupos de host de Azure Virtual Desktop (clásico): Azure'
description: Aprenda a crear un grupo host de validación in Azure Virtual Desktop (clásico) para supervisar las actualizaciones de servicio antes de implementar las actualizaciones en producción.
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: bca4d55d0c5abb4e133a8eef8b51a2a5ae57b5d5
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111749838"
---
# <a name="tutorial-create-a-host-pool-to-validate-service-updates-in-azure-virtual-desktop-classic"></a>Tutorial: Creación de un grupo de hosts para validar actualizaciones de servicio en Azure Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Azure Virtual Desktop (clásico), que no admite objetos de Azure Virtual Desktop para Azure Resource Manager. Si está intentando administrar objetos de Azure Virtual Desktop para Azure Resource Manager, consulte [este artículo](../create-validation-host-pool.md).

Los grupos hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilino de Azure Virtual Desktop. Le recomendamos que cree un grupo de hosts de validación en el que se apliquen primero las actualizaciones del servicio. Esto permite supervisar las actualizaciones del servicio antes de que este las aplique a su entorno estándar o sin validación. Sin un grupo host de validación, es posible que no se detecten cambios que introducen errores, lo que podría dar lugar a tiempos de inactividad para los usuarios en el entorno de producción.

Para garantizar que las aplicaciones funcionan con las actualizaciones más recientes, el grupo de hosts de validación debe ser lo más parecido posible a los grupos de hosts del entorno sin validación. Los usuarios deben conectarse al grupo de hosts de validación con la misma frecuencia que lo hacen al estándar. Si ha automatizado las pruebas en su grupo host, debe incluir las pruebas automatizadas en el grupo host de validación.

Puede depurar incidencias en el grupo host de validación con [la característica de diagnóstico](diagnostics-role-service-2019.md) o con [artículos de solución de problemas de Azure Virtual Desktop](troubleshoot-set-up-overview-2019.md).

>[!NOTE]
> Se recomienda que mantenga el grupo host de validación para probar todas las actualizaciones futuras.

Antes de empezar, y si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Azure Virtual Desktop](/powershell/windows-virtual-desktop/overview/). Después, ejecute el siguiente cmdlet para iniciar sesión en su cuenta:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="create-your-host-pool"></a>Creación del grupo host

Puede crear un grupo host siguiendo las instrucciones de cualquiera de estos artículos:
- [Tutorial: Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace-2019.md)
- [Creación de un grupo host con una plantilla de Azure Resource Manager](create-host-pools-arm-template.md)
- [Creación de un grupo host con PowerShell](create-host-pools-powershell-2019.md)

## <a name="define-your-host-pool-as-a-validation-host-pool"></a>Definición del grupo host como grupo host de validación

Ejecute los siguientes cmdlets de PowerShell para definir el nuevo grupo host como grupo host de validación. Reemplace los valores entre comillas por los valores pertinentes para su sesión:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
Set-RdsHostPool -TenantName $myTenantName -Name "contosoHostPool" -ValidationEnv $true
```

Ejecute el siguiente cmdlet de PowerShell para confirmar que la propiedad de validación se ha definido. Reemplace los valores entre comillas por los valores pertinentes para su sesión.

```powershell
Get-RdsHostPool -TenantName $myTenantName -Name "contosoHostPool"
```

Los resultados del cmdlet deben ser similares a estos:

```
    TenantName          : contoso
    TenantGroupName     : Default Tenant Group
    HostPoolName        : contosoHostPool
    FriendlyName        :
    Description         :
    Persistent          : False
    CustomRdpProperty    : use multimon:i:0;
    MaxSessionLimit     : 10
    LoadBalancerType    : BreadthFirst
    ValidationEnv       : True
    Ring                :
```

## <a name="update-schedule"></a>Programación de actualizaciones

Las actualizaciones del servicio se realizan mensualmente. Si hay incidencias importantes, se proporcionarán actualizaciones críticas con mayor frecuencia.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado un grupo de hosts de validación, puede obtener información sobre cómo usar Azure Service Health para supervisar la implementación de Azure Virtual Desktop.

> [!div class="nextstepaction"]
> [Configuración de alertas de servicio](set-up-service-alerts-2019.md)
