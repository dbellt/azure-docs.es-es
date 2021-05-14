---
title: Migración a Azure Cloud Services (soporte extendido) mediante PowerShell
description: Migración desde Azure Cloud Services (clásico) a Azure Cloud Services (soporte extendido) mediante PowerShell
author: tanmaygore
ms.service: cloud-services-extended-support
ms.subservice: classic-to-arm-migration
ms.reviwer: mimckitt
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: aab67914b1317bc0cc443f333932ecef924176b6
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108293033"
---
# <a name="migrate-to-azure-cloud-services-extended-support-using-powershell"></a>Migración a Azure Cloud Services (soporte extendido) mediante PowerShell

En estos pasos se muestra cómo usar los comandos de Azure PowerShell para migrar de [Cloud Services (clásico)](../cloud-services/cloud-services-choose-me.md) a [Cloud Services (soporte extendido)](overview.md).

> [!IMPORTANT]
> La migración de Cloud Services (clásico) a Cloud Services (soporte extendido) mediante la herramienta de migración se encuentra actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="1-plan-for-migration"></a>1) Planear la migración
La planificación es el paso más importante para una experiencia de migración satisfactoria. Revise la [información general de Cloud Services (soporte extendido)](overview.md) y [Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager](../virtual-machines/migration-classic-resource-manager-plan.md) antes de comenzar con los pasos de migración. 

## <a name="2-install-the-latest-version-of-powershell"></a>2) Instalar la versión más reciente de PowerShell
Hay dos opciones principales para instalar Azure PowerShell: [Galería de PowerShell](https://www.powershellgallery.com/profiles/azure-sdk/) o [Instalador de plataforma web (WebPI)](https://aka.ms/webpi-azps). WebPI recibe actualizaciones mensuales. La galería de PowerShell recibe actualizaciones de forma continua. Este artículo se basa en los cmdlets de la versión 2.1.0 de Azure PowerShell.

Para ver las instrucciones de instalación, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/servicemanagement/install-azure-ps?preserve-view=true&view=azuresmps-4.0.0).

## <a name="3-ensure-admin-permissions"></a>3) Garantizar los permisos de administrador
Para realizar esta migración, debe estar agregado como coadministrador de la suscripción en [Azure Portal](https://portal.azure.com).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú de **Central**, seleccione **Suscripción**. Si no lo ve, haga clic en **Todos los servicios**.
3. Busque la entrada de la suscripción adecuada y después examine el campo **MI ROL**. En el caso de un coadministrador, el valor debe ser *Administrador de cuenta*.

Si no puede agregar un coadministrador, póngase en contacto con un administrador del servicio o coadministrador de la suscripción para que le agreguen.

## <a name="4-register-the-classic-provider-and-cloudservice-feature"></a>4) Registrar el proveedor clásico y la característica CloudService
En primer lugar, inicie un símbolo del sistema de PowerShell. Para la migración, configure el entorno para el modelo clásico y el de Resource Manager.

Inicie sesión en su cuenta para el modelo de Resource Manager.

```powershell
Connect-AzAccount
```

Puede encontrar las suscripciones disponibles ejecutando el siguiente comando:

```powershell
Get-AzSubscription | Sort Name | Select Name
```

Establezca la suscripción de Azure para la sesión actual. En este ejemplo se establece el nombre de la suscripción predeterminado en **My Azure Subscription** (Mi suscripción de Azure). Reemplace el nombre de la suscripción de ejemplo por el suyo propio.

```powershell
Select-AzSubscription –SubscriptionName "My Azure Subscription"
```

Regístrese con el proveedor de recursos de migración ejecutando el comando siguiente:

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```
> [!NOTE]
> El registro es un paso que solo se realiza una vez, pero debe hacerlo antes de intentar la migración. Si no se registra, recibirá el siguiente mensaje de error:
>
> *BadRequest: La suscripción no está registrada para la migración.*

Registre la característica CloudServices de la suscripción. Los registros pueden tardar varios minutos en completarse. 

```powershell
Register-AzProviderFeature -FeatureName CloudServices -ProviderNamespace Microsoft.Compute
```

Espere cinco minutos a que finalice el registro. 

Compruebe el estado de la aprobación del proveedor clásico con el comando siguiente:

```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Compruebe el estado del registro mediante lo siguiente:  
```powershell
Get-AzProviderFeature -FeatureName CloudServices
```

Asegúrese de que RegistrationState sea `Registered` para ambos antes de continuar.

Antes de cambiar al modelo de implementación clásico, asegúrese de tener una cuota suficiente de vCPU de Azure Resource Manager en la región de Azure de su red virtual o implementación actuales. Puede usar el siguiente comando de PowerShell para comprobar la cantidad de vCPU que tiene actualmente en Azure Resource Manager. Para obtener más información sobre las cuotas de vCPU, vea [Límites y Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

En este ejemplo se comprueba la disponibilidad en la región del **oeste de EE. UU.** Reemplace el nombre de la región de ejemplo por el suyo propio.

```powershell
Get-AzVMUsage -Location "West US"
```

Ahora inicie sesión en su cuenta para el modelo de implementación clásico.

```powershell
Add-AzureAccount
```

Puede encontrar las suscripciones disponibles ejecutando el siguiente comando:

```powershell
Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName
```

Establezca la suscripción de Azure para la sesión actual. En este ejemplo se establece la suscripción predeterminada en **My Azure Subscription** (Mi suscripción de Azure). Reemplace el nombre de la suscripción de ejemplo por el suyo propio.

```powershell
Select-AzureSubscription –SubscriptionName "My Azure Subscription"
```


## <a name="5-migrate-your-cloud-services"></a>5) Migrar Cloud Services 
* [Migrar una instancia de Cloud Services no en una red virtual](#51-option-1---migrate-a-cloud-service-not-in-a-virtual-network)
* [Migrar una instancia de Cloud Services en una red virtual](#51-option-2---migrate-a-cloud-service-in-a-virtual-network)

> [!NOTE]
> Todas las operaciones que se describen aquí son idempotentes. Si tiene un problema diferente de una función no admitida o un error de configuración, se recomienda que vuelva a intentar la operación de preparación, anulación o confirmación. La plataforma intenta nuevamente la acción.


### <a name="51-option-1---migrate-a-cloud-service-not-in-a-virtual-network"></a>5.1) Opción 1: Migrar una instancia de Cloud Services no en una red virtual
Obtenga la lista de servicios en la nube con el comando siguiente. Después, seleccione el servicio en la nube que se quiere migrar.

```powershell
Get-AzureService | ft Servicename
```

Obtenga el nombre de la implementación de Cloud Services. En este ejemplo, el nombre de servicio es **My Service** (Mi servicio). Reemplace el nombre del servicio de ejemplo por el suyo propio.

```powershell
$serviceName = "My Service"
$deployment = Get-AzureDeployment -ServiceName $serviceName
$deploymentName = $deployment.DeploymentName
```

Primero, valide que puede migrar la instancia de Cloud Services con los comandos siguientes:

```powershell
$validate = Move-AzureService -Validate -ServiceName $serviceName -DeploymentName $deploymentName -CreateNewVirtualNetwork
$validate.ValidationMessages
 ```

El comando siguiente muestra cualquier advertencia y error que bloquee la migración. Si la validación se realiza correctamente, podrá pasar al paso de preparación siguiente.

```powershell
Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName -CreateNewVirtualNetwork
```

### <a name="51-option-2---migrate-a-cloud-service-in-a-virtual-network"></a>5.1) Opción 2: Migrar una instancia de Cloud Services en una red virtual

Para migrar una instancia de Cloud Services en una red virtual, migre la red virtual. La instancia de Cloud Services se migra automáticamente con la red virtual.

> [!NOTE]
> El nombre de red virtual podría ser diferente del que se muestra en el portal nuevo. El nuevo Azure Portal muestra el nombre como `[vnet-name]` pero el nombre de red virtual real es de tipo `Group [resource-group-name] [vnet-name]`. Antes de empezar la migración, busque el nombre real de la red virtual con el comando `Get-AzureVnetSite | Select -Property Name` o consúltelo en el antiguo Azure Portal. 

En este ejemplo se establece el nombre de red virtual en **myVnet**. Reemplace el nombre de la red virtual de ejemplo por el suyo propio.

```powershell
$vnetName = "myVnet"
```

En primer lugar, valide que puede migrar la red virtual con el comando siguiente:

```powershell
Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
```

El comando siguiente muestra cualquier advertencia y error que bloquee la migración. Si la validación se realiza correctamente, podrá continuar al paso de preparación siguiente:

```powershell
Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
```

Compruebe la configuración de la instancia de Cloud Services preparada mediante Azure PowerShell o Azure Portal. Si no está preparado para la migración y quiere volver al estado anterior, utilice el comando siguiente:

```powershell
Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName
```

Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando:

```powershell
Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
```


## <a name="next-steps"></a>Pasos siguientes
Revise la sección [Cambios posteriores a la migración](in-place-migration-overview.md#post-migration-changes) para ver los cambios en los archivos de implementación, la automatización y otros atributos de la nueva implementación de Cloud Services (soporte extendido).