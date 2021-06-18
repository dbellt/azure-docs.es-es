---
title: Aplicación de revisiones automática a invitados de las máquinas virtuales para máquinas virtuales de Azure
description: Aprenda a aplicar revisiones automáticamente a máquinas virtuales en Azure.
author: mayanknayar
ms.service: virtual-machines
ms.subservice: automatic-guest-patching
ms.collection: windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 02/17/2021
ms.author: manayar
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f59d43dfa4d952b29dbe16b6679a527c4fc0d50c
ms.sourcegitcommit: 89c889a9bdc2e72b6d26ef38ac28f7a6c5e40d27
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111565399"
---
# <a name="preview-automatic-vm-guest-patching-for-azure-vms"></a>Versión preliminar: Aplicación de revisiones automática a invitados de las máquinas virtuales para máquinas virtuales de Azure

La habilitación de la aplicación de revisiones automática a invitados de las máquinas virtuales de Azure facilita la administración de las actualizaciones mediante la aplicación segura y automática de revisiones a las máquinas virtuales para mantener el cumplimiento de la seguridad.

La aplicación de revisiones automáticas a invitados de máquina virtual tiene las siguientes características:
- Las revisiones clasificadas de tipo *Critico* o de *Seguridad* se descargan y se aplican automáticamente en la máquina virtual.
- Las revisiones se aplican durante las horas valle de la zona horaria de la máquina virtual.
- Azure administra la orquestación de las revisiones y estas se aplican siguiendo el [principio de orden de disponibilidad](#availability-first-patching).
- Se supervisa el mantenimiento de la máquina virtual, determinado a través de las señales de mantenimiento de la plataforma, para detectar errores en la aplicación de revisiones.
- Funciona con todos los tamaños de máquina virtual.

> [!IMPORTANT]
> La aplicación de revisiones automáticas a invitados de máquina virtual se encuentra actualmente en versión preliminar pública. No es necesario ningún procedimiento de participación para usar la funcionalidad de versión preliminar pública que se describe a continuación.
> Esta versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="how-does-automatic-vm-guest-patching-work"></a>¿Cómo funciona la aplicación de revisiones a invitados de máquina virtual?

Si la aplicación automática de revisiones a invitados de máquina virtual está habilitada en una máquina virtual, se descargan y se aplican automáticamente en la máquina virtual las revisiones disponibles de tipo *Crítico* y de *Seguridad*. Este proceso se inicia automáticamente cada mes cuando se lanzan nuevas revisiones. La evaluación e instalación de las revisiones es un proceso automático, que incluye el reinicio de la máquina virtual si es necesario.

La máquina virtual se evalúa de forma periódica cada pocos días y varias veces en cualquier período de 30 días para determinar las revisiones que se le pueden aplicar. Las revisiones se pueden instalar cualquier día en la máquina virtual, durante sus horas valle. Esta evaluación automática garantiza que las revisiones que faltan se detecten lo antes posible.

Las revisiones se instalan en un plazo de 30 días a partir del lanzamiento mensual de las revisiones, siguiendo el tipo de orquestación por orden de disponibilidad que se describe a continuación. Las revisiones solo se instalan durante las horas valle de la máquina virtual, en función de la zona horaria donde se encuentre. La máquina virtual debe estar en ejecución durante las horas valle para que las revisiones se instalen automáticamente. Si se apaga una máquina virtual durante una evaluación periódica, el proceso automático de evaluar la máquina virtual e instalar las revisiones aplicables se realizará durante la siguiente evaluación periódica (normalmente a los pocos días), cuando la máquina virtual esté encendida.

Las actualizaciones de definiciones y otras revisiones no clasificadas como *Críticas* o de *Seguridad* no se instalarán mediante la aplicación de revisiones automática a invitados de las máquinas virtuales. Para instalar revisiones con otras clasificaciones de revisión o programar la instalación de revisiones en su propia ventana de mantenimiento personalizada, puede usar [Update Management](./windows/tutorial-config-management.md#manage-windows-updates).

### <a name="availability-first-patching"></a>Aplicación de revisiones con el principio de primero la disponibilidad

Azure organiza el proceso de instalación de revisiones globalmente para todas las máquinas virtuales que tengan habilitada la aplicación de revisiones automáticas a invitados de máquina virtual. Esta orquestación sigue los principios de primero la disponibilidad entre los distintos niveles de disponibilidad proporcionados por Azure.

En un grupo de máquinas virtuales que se vayan a actualizar, la plataforma Azure orquestará las actualizaciones:

**Entre regiones:**
- Se orquesta una actualización mensual global a través de Azure, por fases, para evitar errores de implementación globales.
- Una fase puede tener una o más regiones y una actualización solo pasa a la siguiente fase si las máquinas virtuales válidas de una fase se actualizan correctamente.
- Las regiones emparejadas geográficamente no se actualizan simultáneamente y no pueden estar en la misma fase regional.
- El éxito de una actualización se mide realizando un seguimiento del mantenimiento de la máquina virtual posterior a la actualización. El seguimiento del mantenimiento de la máquina virtual se realiza a través de los indicadores de mantenimiento de la plataforma de la máquina virtual.

**Dentro de una región:**
- Las máquinas virtuales de diferentes Availability Zones no se actualizan simultáneamente.
- Las máquinas virtuales que no forman parte de un conjunto de disponibilidad se procesan por lotes, según la mejor opción, para evitar las actualizaciones simultáneas de todas las máquinas virtuales de una suscripción.

**Dentro de un conjunto de disponibilidad:**
- Las máquinas virtuales de un conjunto de disponibilidad común no se actualizan simultáneamente.
-   Las máquinas virtuales de un conjunto de disponibilidad común se actualizan dentro de los límites del dominio de actualización y las máquinas virtuales de varios dominios de actualización no se actualizan simultáneamente.

La fecha de instalación de la revisión de una máquina virtual determinada puede variar de un mes a otro, ya que una máquina virtual específica puede seleccionarse en un lote diferente entre ciclos de aplicación de revisiones mensuales.

### <a name="which-patches-are-installed"></a>¿Qué revisiones se instalan?
Las revisiones instaladas dependen de la fase de lanzamiento de la máquina virtual. Cada mes, se inicia un nuevo lanzamiento global en el que se instalan todas las revisiones críticas y de seguridad evaluadas para una máquina virtual individual para esa máquina virtual. El lanzamiento se organiza en todas las regiones de Azure en lotes (descritos en la sección anterior sobre revisiones con el principio de orden de disponibilidad).

El conjunto exacto de revisiones que se van a instalar varía en función de la configuración de la máquina virtual, incluido el tipo de sistema operativo y los tiempos de evaluación. Es posible que dos máquinas virtuales idénticas en regiones diferentes obtengan diferentes revisiones instaladas si hay más o menos revisiones disponibles cuando la orquestación de revisiones alcanza regiones diferentes en momentos diferentes. Del mismo modo, pero con menos frecuencia, las máquinas virtuales dentro de la misma región pero que se han evaluado en momentos diferentes (debido a distintos lotes de la zona de disponibilidad o del conjunto de disponibilidad) podrían obtener distintas revisiones.

Como la aplicación de revisiones automática del invitado de la máquina virtual no configura el origen de la revisión, dos máquinas virtuales similares configuradas para distintos orígenes de revisiones, como el repositorio público y el repositorio privado, también pueden ver una diferencia en el conjunto exacto de revisiones instaladas.

En el caso de los tipos de sistema operativo que publican revisiones en una cadencia fija, las máquinas virtuales configuradas en el repositorio público para el sistema operativo pueden esperar recibir el mismo conjunto de revisiones en las distintas fases de lanzamiento de un mes. Por ejemplo, las máquinas virtuales Windows configuradas para el repositorio de Windows Update público.

A medida que se desencadena un nuevo lanzamiento cada mes, una máquina virtual recibirá al menos una implementación de revisiones cada mes si la máquina virtual está encendida durante las horas de poca actividad. Esto garantiza que la máquina virtual se revisará con las revisiones críticas y de seguridad disponibles más recientes de manera mensual. Para garantizar la coherencia en el conjunto de revisiones instaladas, puede configurar las máquinas virtuales para evaluar y descargar las revisiones de sus propios repositorios privados.

## <a name="supported-os-images"></a>Imágenes de sistema operativo compatibles
Actualmente, en la versión preliminar, solo se admiten máquinas virtuales creadas a partir de determinadas imágenes de plataforma de sistema operativo. Las imágenes personalizadas no se admiten actualmente en la versión preliminar.

Las siguientes SKU de plataforma se admiten actualmente (y periódicamente se agregan más):

| Publicador               | Sistema operativo      |  SKU               |
|-------------------------|---------------|--------------------|
| Canonical  | UbuntuServer | 18.04-LTS |
| Redhat  | RHEL | 7.x |
| Microsoft Windows Server  | Windows Server | Centro de datos de 2012-R2 |
| Microsoft Windows Server  | Windows Server | 2016-Datacenter    |
| Microsoft Windows Server  | Windows Server | 2016-Datacenter-Server-Core |
| Microsoft Windows Server  | Windows Server | 2019-Datacenter |
| Microsoft Windows Server  | Windows Server | 2019-Datacenter-Core |

## <a name="patch-orchestration-modes"></a>Modos de orquestación de revisiones
Las máquinas virtuales que residen en Azure admiten ahora los siguientes modos de orquestación de revisiones:

**AutomaticByPlatform:**
- este modo es compatible con máquinas virtuales Linux y Windows.
- Este modo habilita la aplicación de revisiones automática a los invitados de la máquina virtual y Azure orquesta la instalación posterior de la revisión.
- Este modo es necesario para la revisión de la disponibilidad en primer lugar.
- Este modo solo se admite para las máquinas virtuales que se crean mediante las imágenes de plataforma de sistema operativo compatibles indicadas antes.
- En el caso de las máquinas virtuales Windows, al establecer este modo también se deshabilitan las actualizaciones automáticas nativas en la máquina virtual Windows para evitar la duplicación.
- Para usar este modo en máquinas virtuales Linux, establezca la propiedad `osProfile.linuxConfiguration.patchSettings.patchMode=AutomaticByPlatform` en la plantilla de máquina virtual.
- Para usar este modo en máquinas virtuales Windows, establezca la propiedad `osProfile.windowsConfiguration.enableAutomaticUpdates=true` y luego la propiedad `osProfile.windowsConfiguration.patchSettings.patchMode=AutomaticByPlatform` en la plantilla de máquina virtual.

**AutomaticByOS:**
- Este modo solo se admite para máquinas virtuales Windows.
- Este modo habilita las actualizaciones automáticas en la máquina virtual Windows y se instalan las revisiones en la máquina virtual a través de actualizaciones automáticas.
- Este modo no admite la revisión de la disponibilidad en primer lugar.
- Este modo se establece de manera predeterminada si no se especifica ningún otro modo de revisión para una máquina virtual Windows.
- Para usar este modo en máquinas virtuales Windows, establezca la propiedad `osProfile.windowsConfiguration.enableAutomaticUpdates=true` y luego la propiedad `osProfile.windowsConfiguration.patchSettings.patchMode=AutomaticByOS` en la plantilla de máquina virtual.

**Manual:**
- Este modo solo se admite para máquinas virtuales Windows.
- Este modo deshabilita las actualizaciones automáticas en la máquina virtual Windows.
- Este modo no admite la revisión de la disponibilidad en primer lugar.
- Este modo se debe establecer cuando se usan soluciones personalizadas de aplicación de revisiones.
- Para usar este modo en máquinas virtuales Windows, establezca la propiedad `osProfile.windowsConfiguration.enableAutomaticUpdates=false` y luego la propiedad `osProfile.windowsConfiguration.patchSettings.patchMode=Manual` en la plantilla de máquina virtual.

**ImageDefault:**
- este modo solo se admite para máquinas virtuales Linux.
- Este modo no admite la revisión de la disponibilidad en primer lugar.
- Este modo respeta la configuración de aplicación de revisiones predeterminada de la imagen que se usa para crear la máquina virtual.
- Este modo se establece de manera predeterminada si no se especifica ningún otro modo de revisión para una máquina virtual Linux.
- Para usar este modo en máquinas virtuales Linux, establezca la propiedad `osProfile.linuxConfiguration.patchSettings.patchMode=ImageDefault` en la plantilla de máquina virtual.

> [!NOTE]
>Para máquinas virtuales Windows, la propiedad `osProfile.windowsConfiguration.enableAutomaticUpdates` actualmente solo se puede establecer cuando se crea la máquina virtual por primera vez. Actualmente no se admite el cambio del modo Manual al Automático ni del Automático al Manual. Se admite el cambio del modo AutomaticByOS al modo AutomaticByPlatfom.

## <a name="requirements-for-enabling-automatic-vm-guest-patching"></a>Requisitos para habilitar la aplicación de revisiones automáticas a invitados de máquina virtual

- La máquina virtual debe tener instalado el agente de máquina virtual de Azure para [Windows](./extensions/agent-windows.md) o [Linux](./extensions/agent-linux.md).
- En el caso de las máquinas virtuales Linux, el agente de Linux de Azure debe ser la versión 2.2.53.1 o posterior. [Actualice el agente de Linux](./extensions/update-linux-agent.md) si la versión actual es anterior a la versión requerida.
- En el caso de las máquinas virtuales Windows, el servicio Windows Update debe estar en ejecución en la máquina virtual.
- La máquina virtual debe poder acceder a los puntos de conexión de actualización configurados. Si la máquina virtual está configurada para usar repositorios privados para Linux, o Windows Server Update Services (WSUS) para máquinas virtuales Windows, los puntos de conexión de actualización pertinentes deben ser accesibles.
- Use la versión 2020-12-01 o posterior de la API de Compute. La versión 2020-06-01 de la API de Compute se puede usar para máquinas virtuales Windows con funcionalidad limitada.

La habilitación de la funcionalidad en versión preliminar requiere solicitar la participación una sola vez para las características **InGuestAutoPatchVMPreview** y **InGuestPatchVMPreview** por suscripción, como se detalla en la sección siguiente.

### <a name="rest-api"></a>API DE REST
En el ejemplo siguiente se describe cómo habilitar la versión preliminar para su suscripción:

```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestAutoPatchVMPreview/register?api-version=2015-12-01`
```
```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestPatchVMPreview/register?api-version=2015-12-01`
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos:

```
GET on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestAutoPatchVMPreview?api-version=2015-12-01`
```
```
GET on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestPatchVMPreview?api-version=2015-12-01`
```
Una vez que se ha registrado la característica para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute.

```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Compute/register?api-version=2020-06-01`
```

### <a name="azure-powershell"></a>Azure PowerShell
Use el cmdlet [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature) para habilitar la versión preliminar de su suscripción.

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName InGuestAutoPatchVMPreview -ProviderNamespace Microsoft.Compute
Register-AzProviderFeature -FeatureName InGuestPatchVMPreview -ProviderNamespace Microsoft.Compute
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos:

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName InGuestAutoPatchVMPreview -ProviderNamespace Microsoft.Compute
Get-AzProviderFeature -FeatureName InGuestPatchVMPreview -ProviderNamespace Microsoft.Compute
```

Una vez que se ha registrado la característica para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

### <a name="azure-cli-20"></a>CLI de Azure 2.0
Utilice [az feature register](/cli/azure/feature#az_feature_register) para habilitar la versión preliminar de su suscripción.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name InGuestAutoPatchVMPreview
az feature register --namespace Microsoft.Compute --name InGuestPatchVMPreview
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name InGuestAutoPatchVMPreview
az feature show --namespace Microsoft.Compute --name InGuestPatchVMPreview
```

Una vez que se ha registrado la característica para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute.

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```

## <a name="enable-automatic-vm-guest-patching"></a>Habilitación de la aplicación de revisiones automáticas a invitados de máquina virtual
Para habilitar la aplicación de revisiones automática a invitados de máquina virtual en una máquina virtual Windows, asegúrese de que la propiedad *osProfile.windowsConfiguration.enableAutomaticUpdates* esté establecida en *true* en la definición de la plantilla de máquina virtual. Esta propiedad solo se puede establecer al crear la máquina virtual. Esta propiedad adicional no es aplicable a las máquinas virtuales Linux.

### <a name="rest-api-for-linux-vms"></a>API REST para máquinas virtuales Linux
En el ejemplo siguiente se describe cómo habilitar la aplicación de revisiones automáticas a invitados de máquina virtual:

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine?api-version=2020-12-01`
```

```json
{
  "properties": {
    "osProfile": {
      "linuxConfiguration": {
        "provisionVMAgent": true,
        "patchSettings": {
          "patchMode": "AutomaticByPlatform"
        }
      }
    }
  }
}
```

### <a name="rest-api-for-windows-vms"></a>API REST para máquinas virtuales Windows
En el ejemplo siguiente se describe cómo habilitar la aplicación de revisiones automáticas a invitados de máquina virtual:

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine?api-version=2020-06-01`
```

```json
{
  "properties": {
    "osProfile": {
      "windowsConfiguration": {
        "provisionVMAgent": true,
        "enableAutomaticUpdates": true,
        "patchSettings": {
          "patchMode": "AutomaticByPlatform"
        }
      }
    }
  }
}
```

### <a name="azure-powershell-for-windows-vms"></a>Azure PowerShell para máquinas virtuales Windows
Use el cmdlet [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) para habilitar la aplicación de revisiones automáticas a invitados de máquina virtual al crear o actualizar una máquina virtual.

```azurepowershell-interactive
Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate -PatchMode "AutomaticByPlatform"
```

### <a name="azure-cli-for-windows-vms"></a>CLI de Azure para máquinas virtuales Windows
Use [az vm create](/cli/azure/vm#az_vm_create) para habilitar la aplicación de revisiones automáticas a invitados de máquina virtual al crear una nueva máquina virtual. En el ejemplo siguiente se configura la aplicación de revisiones automáticas a invitados de una máquina virtual llamada *myVM* en el grupo de recursos llamado *myResourceGroup*:

```azurecli-interactive
az vm create --resource-group myResourceGroup --name myVM --image Win2019Datacenter --enable-agent --enable-auto-update --patch-mode AutomaticByPlatform
```

Para modificar una máquina virtual existente, use [az vm update](/cli/azure/vm#az_vm_update)

```azurecli-interactive
az vm update --resource-group myResourceGroup --name myVM --set osProfile.windowsConfiguration.enableAutomaticUpdates=true osProfile.windowsConfiguration.patchSettings.patchMode=AutomaticByPlatform
```

## <a name="enablement-and-assessment"></a>Habilitación y evaluación

> [!NOTE]
>Puede tardar más de tres horas en habilitar las actualizaciones automáticas de invitados en una máquina virtual, ya que la habilitación se realiza durante las horas valle de la máquina virtual. Como la evaluación y la instalación de revisiones solo se producen durante las horas valle, la máquina virtual también debe estar en ejecución durante esas horas valle para que se apliquen las revisiones.

Cuando la aplicación de revisiones automática de invitados de máquina virtual está habilitada para una máquina virtual, se instala una extensión de máquina virtual de tipo `Microsoft.CPlat.Core.LinuxPatchExtension` en una máquina virtual Linux o se instala una extensión de máquina virtual de tipo `Microsoft.CPlat.Core.WindowsPatchExtension` en una máquina virtual Windows. No es necesario que esta extensión se instale o actualice manualmente, ya que la plataforma Azure administra esta extensión como parte del proceso de aplicación de revisiones automáticas a invitados de máquina virtual.

Puede tardar más de tres horas en habilitar las actualizaciones automáticas de invitados en una máquina virtual, ya que la habilitación se realiza durante las horas valle de la máquina virtual. La extensión también se instala y actualiza durante las horas valle de la máquina virtual. Si las horas valle de la máquina virtual finalizan antes de que se pueda completar la habilitación, el proceso de habilitación se reanudará durante el siguiente período de horas valle disponible.

Las actualizaciones automáticas están deshabilitadas en la mayoría de los escenarios y la instalación de las revisiones en adelante se realiza mediante la extensión. Se aplican las siguientes condiciones.
- Si se había activado previamente Windows Update automático en la máquina virtual mediante el modo de revisión AutomaticByOS, se desactiva Windows Update automático para la máquina virtual cuando se instala la extensión.
- En el caso de las máquinas virtuales Ubuntu, las actualizaciones automáticas predeterminadas se deshabilitan automáticamente cuando se completa la habilitación de la aplicación de revisiones automática de invitados de máquina virtual.
- Para RHEL, las actualizaciones automáticas se deben deshabilitar manualmente (se trata de una limitación de la versión preliminar). Ejecute:

```
systemctl stop packagekit
```

```
systemctl mask packagekit
```

Para comprobar si se ha completado la aplicación de revisiones automáticas a invitados de máquina virtual y se ha instalado la extensión de aplicación de revisiones en la máquina virtual, puede revisar la vista de instancia de la máquina virtual. Si el proceso de habilitación se ha completado, se instalará la extensión y los resultados de la evaluación de la máquina virtual estarán disponibles en `patchStatus`. Se puede acceder a la vista de instancia de la máquina virtual de varias maneras, como se describe a continuación.

### <a name="rest-api"></a>API DE REST

```
GET on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine/instanceView?api-version=2020-12-01`
```
### <a name="azure-powershell"></a>Azure PowerShell
Use el cmdlet [Get-AzVM](/powershell/module/az.compute/get-azvm) con el parámetro `-Status` para acceder a la vista de instancia de la máquina virtual.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM" -Status
```

Actualmente, PowerShell solo proporciona información sobre la extensión de revisión. La información sobre `patchStatus` también estará disponible pronto a través de PowerShell.

### <a name="azure-cli"></a>CLI de Azure
Use [az vm get-instance-view](/cli/azure/vm#az_vm_get_instance_view) para acceder a la vista de instancia de la máquina virtual.

```azurecli-interactive
az vm get-instance-view --resource-group myResourceGroup --name myVM
```

### <a name="understanding-the-patch-status-for-your-vm"></a>Descripción del estado de revisión de la máquina virtual

En la sección `patchStatus` de la respuesta de la vista de instancia se proporcionan detalles sobre la evaluación más reciente y la instalación de la última revisión para la máquina virtual.

Los resultados de la evaluación de la máquina virtual se pueden revisar en la sección `availablePatchSummary`. Se realiza una evaluación periódica de una máquina virtual que tiene habilitada la aplicación de revisiones automáticas a invitados de máquina virtual. El número de revisiones disponibles después de una evaluación se proporciona en los resultados de `criticalAndSecurityPatchCount` y `otherPatchCount`. La aplicación de revisiones automáticas a invitados de máquina virtual instalará todas las revisiones que se hayan evaluado de las clasificaciones de revisión de tipo *Crítico* y de *Seguridad*. Se omite cualquier otra revisión evaluada.

Los resultados de la instalación de revisiones para la máquina virtual se pueden revisar en la sección `lastPatchInstallationSummary`. En esta sección se proporcionan detalles sobre el último intento de instalación de revisiones en la máquina virtual, incluido el número de revisiones instaladas, pendientes, con errores u omitidas. Las revisiones solo se instalan durante la ventana de mantenimiento de horas valle de la máquina virtual. Las revisiones pendientes y erróneas se reintentan automáticamente en la siguiente ventana de mantenimiento de horas valle.

## <a name="on-demand-patch-assessment"></a>Evaluación de la revisión a petición
Si la aplicación de revisiones automáticas a invitados de máquina virtual ya está habilitada para la máquina virtual, se realiza periódicamente una evaluación de las revisiones de la máquina virtual durante las horas valle de la máquina virtual. Este proceso es automático y los resultados de la evaluación más reciente se pueden revisar en la vista de instancia de la máquina virtual, tal y como se describió anteriormente en este documento. También puede desencadenar una evaluación de revisiones a petición para su máquina virtual en cualquier momento. La evaluación de revisiones puede tardar unos minutos en completarse y el estado de la evaluación más reciente se actualiza en la vista de instancia de la máquina virtual.

Para habilitar la funcionalidad en versión preliminar, hay que participar en la característica **InGuestPatchVMPreview** por suscripción. Esta vista previa de la característica es diferente de la inscripción de la característica de aplicación de revisiones automáticas a invitados de las máquinas virtuales realizada anteriormente para **InGuestAutoPatchVMPreview**. La habilitación de la vista previa de características adicionales es un requisito adicional e independiente. La versión preliminar de la característica para la evaluación de revisiones a petición se puede habilitar siguiendo el [proceso de habilitación de la versión preliminar](automatic-vm-guest-patching.md#requirements-for-enabling-automatic-vm-guest-patching) descrito anteriormente para la aplicación automática de revisiones de invitado de la máquina virtual.

> [!NOTE]
>La evaluación de revisiones a petición no desencadena automáticamente la instalación de revisiones. Si ha habilitado la aplicación de revisiones automáticas a invitados de las máquinas virtuales, las revisiones evaluadas y aplicables para la máquina virtual solo se instalarán durante las horas valle de la máquina virtual, siguiendo el proceso de aplicación de revisiones de primero la disponibilidad que se ha descrito anteriormente en este documento.

### <a name="rest-api"></a>API DE REST
```
POST on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine/assessPatches?api-version=2020-12-01`
```

### <a name="azure-powershell"></a>Azure PowerShell
Use el cmdlet [Invoke-AzVmPatchAssessment](/powershell/module/az.compute/invoke-azvmpatchassessment) para evaluar las revisiones disponibles para la máquina virtual.

```azurepowershell-interactive
Invoke-AzVmPatchAssessment -ResourceGroupName "myResourceGroup" -VMName "myVM"
```

### <a name="azure-cli"></a>CLI de Azure
Use [az vm assess-patches](/cli/azure/vm#az_vm_assess_patches) para evaluar las revisiones disponibles para la máquina virtual.

```azurecli-interactive
az vm assess-patches --resource-group myResourceGroup --name myVM
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Más información sobre la creación y administración de máquinas virtuales Windows](./windows/tutorial-manage-vm.md)
