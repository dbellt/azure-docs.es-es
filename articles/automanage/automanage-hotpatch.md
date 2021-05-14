---
title: Hotpatch para Windows Server Azure Edition (versión preliminar)
description: Conozca cómo funciona Hotpatch para Windows Server Azure Edition y cómo habilitarlo.
author: ju-shim
ms.service: virtual-machines
ms.subservice: hotpatch
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: jushiman
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1b3fc9f12dfa6ad4edcc120ac7c9592c9435a0e4
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107830185"
---
# <a name="hotpatch-for-new-virtual-machines-preview"></a>Revisión en caliente para nuevas máquinas virtuales (versión preliminar)

La aplicación de revisiones en caliente es una nueva manera de instalar actualizaciones en nuevas máquinas virtuales (VM) de Windows Server Azure Edition que no es necesario reiniciar después de la instalación. En este artículo se describen las máquinas virtuales de Hotpatch para Windows Server Azure Edition, que presentan las siguientes ventajas:
* Impacto reducido sobre la carga de trabajo con menos reinicios
* Implementación más rápida de actualizaciones ya que los paquetes son más pequeños, se instalan más rápido y tienen una orquestación de revisiones más sencilla con Azure Update Manager
* Mejor protección, ya que los paquetes de actualización de Hotpatch se limitan a las actualizaciones de seguridad de Windows que se instalan más rápido sin necesidad de reiniciar.

## <a name="how-hotpatch-works"></a>Cómo funciona la revisión en caliente

Lo que hace Hotpatch es establecer primero una línea de base con la última actualización acumulativa de Windows Update. Periódicamente, se lanzan revisiones en caliente (por ejemplo, el segundo martes de cada mes) que se basan en esa línea de base. Las revisiones en caliente contendrán actualizaciones que no requieran un reinicio. La línea de base se actualiza periódicamente (cada tres meses) con una nueva actualización acumulativa más reciente.

:::image type="content" source="media\automanage-hotpatch\hotpatch-sample-schedule.png" alt-text="Programación de ejemplo de Hotpatch.":::

Existen dos tipos de líneas de base: **líneas de base planeadas** y **líneas de base no planeadas**.
*  Las **líneas de base planeadas** se lanzan a un ritmo regular, intercaladas con versiones de revisiones en caliente.  Incluyen todas las actualizaciones de una _actualización acumulativa más reciente_ comparable de ese mes y requieren un reinicio.
    * En la programación de ejemplo anterior se ilustran cuatro versiones de línea de base planeadas en un año natural (cinco en total en el diagrama) y ocho versiones de revisiones en caliente.
* Las **líneas de base no planeadas** se lanzan cuando lo hace una actualización importante (por ejemplo, una corrección de día cero) y esa actualización en particular no se puede lanzar como una revisión en caliente.  Cuando se lancen líneas de base no planeadas, una versión de la revisión en caliente se reemplazará por una línea de base no planeada de ese mes.  Las líneas de base no planeadas también incluyen todas las actualizaciones de una _actualización acumulativa más reciente_ comparable de ese mes y también requieren un reinicio.
    * En la programación de ejemplo anterior se muestran dos líneas de base no planeadas que reemplazarían a las versiones de la revisión en caliente de esos meses (el número real de líneas de base no planeadas en un año no se conoce de antemano).

## <a name="regional-availability"></a>Disponibilidad regional
Hotpatch está disponible en todas las regiones globales de Azure en versión preliminar. En esta versión, no se admiten regiones de Azure Government.

## <a name="how-to-get-started"></a>Primeros pasos

> [!NOTE]
> Durante la fase de versión preliminar, solo puede empezar a trabajar en Azure Portal mediante [este vínculo](https://aka.ms/AzureAutomanageHotPatch).

Para empezar a usar Hotpatch en una nueva máquina virtual, siga estos pasos:
1.  Habilitación del acceso a la versión preliminar
    * Se requiere la habilitación puntual del acceso a la versión preliminar por cada suscripción.
    * El acceso a la versión preliminar se puede habilitar mediante API, PowerShell o la CLI, como se describe en la sección siguiente.
1.  Creación de una máquina virtual en Azure Portal
    * Durante la versión preliminar, deberá empezar a usar [este vínculo](https://aka.ms/AzureAutomanageHotPatch).
1.  Suministro de los detalles de la máquina virtual
    * Asegúrese de que _Windows Server 2019 Datacenter: Azure Edition_ esté seleccionado en la lista desplegable de imágenes.
    * En el paso de la pestaña Administración, desplácese hacia abajo hasta la sección "Guest OS updates" (Actualizaciones del SO invitado). Verá que Aplicar revisión en caliente está establecida en Activado y que Patch installation (Instalación de revisiones) tiene el valor predeterminado de Azure-orchestrated patching (Aplicación de revisiones orquestada por Azure).
    * De forma predeterminada, se habilitarán los procedimientos recomendados de administración automática de máquinas virtuales.
1. Creación de la máquina virtual

## <a name="enabling-preview-access"></a>Habilitación del acceso en versión preliminar

### <a name="rest-api"></a>API DE REST

En el ejemplo siguiente se describe cómo habilitar la versión preliminar para su suscripción:

```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/ InGuestHotPatchVMPreview/register?api-version=2015-12-01`
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestAutoPatchVMPreview/register?api-version=2015-12-01`
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestPatchVMPreview/register?api-version=2015-12-01`
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos:

```
GET on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestHotPatchVMPreview?api-version=2015-12-01`
GET on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestAutoPatchVMPreview?api-version=2015-12-01`
GET on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/InGuestPatchVMPreview?api-version=2015-12-01`
```

Una vez que la característica se ha registrado para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute.

```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Compute/register?api-version=2019-12-01`
```

### <a name="azure-powershell"></a>Azure PowerShell

Use el cmdlet ```Register-AzProviderFeature``` para habilitar la versión preliminar de su suscripción.

``` PowerShell
Register-AzProviderFeature -FeatureName InGuestHotPatchVMPreview -ProviderNamespace Microsoft.Compute
Register-AzProviderFeature -FeatureName InGuestAutoPatchVMPreview -ProviderNamespace Microsoft.Compute
Register-AzProviderFeature -FeatureName InGuestPatchVMPreview -ProviderNamespace Microsoft.Compute
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos:

``` PowerShell
Get-AzProviderFeature -FeatureName InGuestHotPatchVMPreview -ProviderNamespace Microsoft.Compute
Get-AzProviderFeature -FeatureName InGuestAutoPatchVMPreview -ProviderNamespace Microsoft.Compute
Get-AzProviderFeature -FeatureName InGuestPatchVMPreview -ProviderNamespace Microsoft.Compute
```

Una vez que la característica se ha registrado para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute.

``` PowerShell
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

### <a name="azure-cli"></a>Azure CLI

Utilice ```az feature register``` para habilitar la versión preliminar de su suscripción.

```
az feature register --namespace Microsoft.Compute --name InGuestHotPatchVMPreview
az feature register --namespace Microsoft.Compute --name InGuestAutoPatchVMPreview
az feature register --namespace Microsoft.Compute --name InGuestPatchVMPreview
```

El registro de la característica puede tardar hasta 15 minutos. Para comprobar el estado del registro, siga estos pasos:
```
az feature show --namespace Microsoft.Compute --name InGuestHotPatchVMPreview
az feature show --namespace Microsoft.Compute --name InGuestAutoPatchVMPreview
az feature show --namespace Microsoft.Compute --name InGuestPatchVMPreview
```

Una vez que la característica se ha registrado para su suscripción, complete el proceso de participación mediante la propagación del cambio en el proveedor de recursos de Compute.

```
az provider register --namespace Microsoft.Compute
```

## <a name="patch-installation"></a>Instalación de revisiones

Durante la versión preliminar, la opción [Automatic VM Guest Patching](../virtual-machines/automatic-vm-guest-patching.md) (Aplicación automática de revisiones a invitados de máquina virtual) se habilita automáticamente para todas las máquinas virtuales creadas con _Windows Server 2019 Datacenter: Azure Edition_. Con la revisión automática de invitado de máquina virtual habilitada:
* Las revisiones clasificadas de tipo Critico o de Seguridad se descargan y se aplican automáticamente en la máquina virtual.
* Las revisiones se aplican durante las horas valle de la zona horaria de la máquina virtual.
* Azure administra la orquestación de las revisiones y estas se aplican siguiendo el [principio de orden de disponibilidad](../virtual-machines/automatic-vm-guest-patching.md#availability-first-patching).
* Se supervisa el mantenimiento de la máquina virtual, determinado a través de las señales de mantenimiento de la plataforma, para detectar errores en la aplicación de revisiones.

### <a name="how-does-automatic-vm-guest-patching-work"></a>¿Cómo funciona la aplicación de revisiones a invitados de máquina virtual?

Si [Automatic VM Guest Patching](../virtual-machines/automatic-vm-guest-patching.md) (Aplicación automática de revisiones a invitados de máquina virtual) está habilitada en una máquina virtual, se descargan y aplican automáticamente las revisiones críticas y de seguridad. Este proceso se inicia automáticamente cada mes cuando se lanzan nuevas revisiones. La evaluación e instalación de las revisiones es un proceso automático, que incluye el reinicio de la máquina virtual si es necesario.

Con la revisión en caliente habilitada en las máquinas virtuales de _Windows Server 2019 Datacenter: Azure Edition_, la mayoría de las actualizaciones de seguridad mensuales se entregan como revisiones en caliente que no requieren reinicios. Las actualizaciones acumulativas más recientes enviadas en los meses de líneas de base planeadas o no planeadas requerirán que se reinicie la máquina virtual. También puede haber revisiones críticas o de seguridad adicionales disponibles periódicamente, lo que puede requerir que se reinicie la máquina virtual.

La máquina virtual se evalúa de forma automática cada pocos días y varias veces en un período de 30 días para determinar las revisiones que se le pueden aplicar. Esta evaluación automática garantiza que las revisiones que faltan se detecten lo antes posible.

Las revisiones de instalan a los 30 días de las versiones de revisiones mensuales, siguiendo los [principios de orden de disponibilidad](../virtual-machines/automatic-vm-guest-patching.md#availability-first-patching). Las revisiones solo se instalan durante las horas valle de la máquina virtual, en función de la zona horaria donde se encuentre. La máquina virtual debe estar en ejecución durante las horas valle para que las revisiones se instalen automáticamente. Si se apaga una máquina virtual durante una evaluación periódica, esta se evaluará y las revisiones aplicables se instalarán automáticamente durante la siguiente evaluación periódica, cuando la máquina virtual esté activada. La siguiente evaluación periódica suele tener lugar a los pocos días.

Las actualizaciones de definiciones y otras revisiones no clasificadas como críticas o de seguridad no se instalarán mediante la aplicación de revisiones automáticas a invitados de máquinas virtuales.

## <a name="understanding-the-patch-status-for-your-vm"></a>Descripción del estado de revisión de la máquina virtual

Para ver el estado de la revisión de la máquina virtual, vaya a la sección **Guest + host updates** (Actualizaciones de invitado y host) de la máquina virtual en Azure Portal. En la sección **Guest OS updates** (Actualizaciones del SO invitado), haga clic en "Go to Hotpatch (Preview)" (Ir a Hotpatch [versión preliminar]) para ver el estado de la revisión más reciente de la máquina virtual.

En esta pantalla, verá el estado de Hotpatch de la máquina virtual. También puede examinar si hay alguna revisión disponible para la máquina virtual que no se haya instalado. Tal y como se describe en la sección anterior "Instalación de revisiones", todas las actualizaciones críticas y de seguridad se instalarán automáticamente en la máquina virtual mediante [Automatic VM Guest Patching](../virtual-machines/automatic-vm-guest-patching.md) (Aplicación automática de revisiones a invitados de máquina virtual) y no se requieren más acciones. Las revisiones con otras clasificaciones de actualización no se instalan automáticamente. Por el contrario, son visibles en la lista de revisiones disponibles en la pestaña "Comprobación de actualizaciones". También puede ver el historial de implementaciones de actualizaciones en la máquina virtual mediante el "Historial de actualizaciones". Se muestra el historial de actualizaciones de los últimos 30 días, junto con los detalles de la instalación de revisiones.


:::image type="content" source="media\automanage-hotpatch\hotpatch-management-ui.png" alt-text="Administración de revisiones en caliente.":::

Con la aplicación automática de revisiones a invitados de máquinas virtuales, la máquina virtual se evalúa periódica y automáticamente para comprobar si tiene las actualizaciones disponibles. Estas evaluaciones periódicas garantizan que se detectan las revisiones que están disponibles. Puede ver los resultados de la evaluación en la pantalla "Actualizaciones" anterior, incluida la hora de la última evaluación. También puede optar por desencadenar una evaluación de revisiones a petición para la máquina virtual en cualquier momento mediante la opción "Assess now" (Evaluar ahora) y revisar los resultados una vez finalizada la operación.

De igual modo que con la evaluación a petición, también puede instalar revisiones a petición para la máquina virtual mediante la opción "Install updates now" (Instalar actualizaciones ahora). Aquí puede elegir instalar todas las actualizaciones dentro de las clasificaciones de revisiones específicas. También puede especificar que se incluyan o excluyan actualizaciones proporcionando una lista de artículos individuales de la base de conocimiento. Las revisiones instaladas a petición no se instalan siguiendo los principios de orden de disponibilidad y pueden requerir más reinicios y experimentar tiempo de inactividad de la máquina virtual durante la instalación de actualizaciones.

## <a name="supported-updates"></a>Actualizaciones admitidas

Hotpatch abarca las actualizaciones de seguridad de Windows y mantiene la paridad con el contenido de las actualizaciones de seguridad emitidas en el canal normal de Windows Update (sin Hotpatch).

Existen algunas consideraciones importantes a la hora de ejecutar una máquina virtual de Windows Server Azure Edition con Hotpatch habilitado. Para instalar actualizaciones que no se incluyen en el programa Hotpatch, los reinicios siguen siendo necesarios. Los reinicios también son necesarios periódicamente después de instalar una nueva línea de base. Estos reinicios mantienen la máquina virtual sincronizada con las revisiones que no son de seguridad incluidas en la actualización acumulativa más reciente.
* Las revisiones que actualmente no están incluidas en el programa Hotpatch incluyen las actualizaciones que no son de seguridad publicadas para Windows y las actualizaciones que no son de Windows (como las revisiones de .NET).  Estos tipos de revisiones deben instalarse durante el mes de una línea de base y requerirán un reinicio.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="what-is-hotpatching"></a>¿Qué es la aplicación de revisiones en caliente?

* La aplicación de revisiones en caliente es una nueva manera de instalar actualizaciones en máquinas virtuales (VM) de Windows Server 2019 Azure Edition en Azure en la que no es necesario reiniciar después la máquina virtual. Lo que hace es aplica revisiones al código en memoria de los procesos en ejecución sin necesidad de reiniciar el proceso.

### <a name="how-does-hotpatching-work"></a>¿Cómo funciona la aplicación de revisiones en caliente?

* La aplicación de revisiones en caliente lo que hace es establecer una línea de base con una actualización acumulativa más reciente de Windows y, luego, basar en ella las actualizaciones que no requieren un reinicio para surtir efecto.  La línea de base se actualiza periódicamente con una nueva actualización acumulativa. La actualización acumulativa incluye todas las actualizaciones de seguridad y de calidad y es necesario reiniciar.

### <a name="why-should-i-use-hotpatch"></a>¿Por qué habría que usar Hotpatch?

* Cuando se usa Hotpatch en Windows Server 2019 Datacenter: Azure Edition, la máquina virtual tiene mayor disponibilidad (menos reinicios) y actualizaciones más rápidas (paquetes más pequeños que se instalan más rápido sin necesidad de reiniciar los procesos). Como consecuencia, la máquina virtual siempre está actualizada y protegida.

### <a name="what-types-of-updates-are-covered-by-hotpatch"></a>¿Qué tipos de actualizaciones se incluyen en Hotpatch?

* Hotpatch actualmente abarca las actualizaciones de seguridad de Windows.

### <a name="when-will-i-receive-the-first-hotpatch-update"></a>¿Cuándo se recibirá la primera actualización de Hotpatch?

* Las actualizaciones de Hotpatch normalmente se lanzan el segundo martes de cada mes. Para obtener más información, consulte más adelante.

### <a name="what-will-the-hotpatch-schedule-look-like"></a>¿Cómo será la programación de Hotpatch?

* La aplicación de revisiones en caliente lo que hace es establecer una línea de base con una actualización acumulativa más reciente de Windows y, luego, basar en ella las actualizaciones de Hotpatch lanzadas mensualmente.  Durante la versión preliminar, las líneas de base se lanzan cada tres meses. Consulte la imagen siguiente para ver un ejemplo de una programación anual de tres meses (que incluye las líneas de base no planeadas debido a correcciones de día cero).

    :::image type="content" source="media\automanage-hotpatch\hotpatch-sample-schedule.png" alt-text="Programación de ejemplo de Hotpatch.":::

### <a name="are-reboots-still-needed-for-a-vm-enrolled-in-hotpatch"></a>¿Siguen siendo necesarios los reinicios en una máquina virtual inscrita en Hotpatch?

* Los reinicios siguen siendo necesarios para instalar las actualizaciones que no se incluyen en el programa Hotpatch y lo son periódicamente después de haber instalado una línea de base (actualización acumulativa más reciente de Windows Update). Con este reinicio, la máquina virtual se mantiene sincronizada con todas las revisiones incluidas en la actualización acumulativa. Las líneas de base (que requieren un reinicio) parten a un ritmo de tres meses y aumentan con el tiempo.

### <a name="are-my-applications-affected-when-a-hotpatch-update-is-installed"></a>¿Se ven afectadas las aplicaciones cuando se instala una actualización de Hotpatch?

* Como Hotpatch revisa el código en memoria de los procesos en ejecución sin necesidad de reiniciarlos, las aplicaciones no se ven afectadas por el proceso de aplicación de revisiones. Esto no quiere decir que no haya implicaciones para el rendimiento y la funcionalidad de la propia revisión.

### <a name="can-i-turn-off-hotpatch-on-my-vm"></a>¿Puedo desactivar Hotpatch en la máquina virtual?

* Puede desactivar Hotpatch en una máquina virtual mediante Azure Portal.  La desactivación de Hotpatch anula la inscripción de la máquina virtual en esta funcionalidad, lo que revierte la máquina virtual al comportamiento de actualización habitual de Windows Server.  Después de anular la inscripción de Hotpatch en una máquina virtual, puede volver a inscribir esa máquina virtual cuando se lance la siguiente línea de base de Hotpatch.

### <a name="can-i-upgrade-from-my-existing-windows-server-os"></a>¿Se pueden realizar actualizaciones desde el sistema operativo Windows Server existente?

* Actualmente no se admite la actualización desde versiones existentes de Windows Server (es decir, ediciones de Windows Server 2016 o 2019 que no son de Azure). Se admitirá la actualización a versiones futuras de Windows Server Azure Edition.

### <a name="can-i-use-hotpatch-for-production-workloads-during-the-preview"></a>¿Puedo usar Hotpatch con las cargas de trabajo de producción durante la versión preliminar?

* Las versiones preliminares están pensadas para realizar pruebas únicamente y no para su uso en entornos de producción.

### <a name="will-i-be-charged-during-the-preview"></a>¿Se cobrará durante la versión preliminar?

* La licencia de Windows Server Azure Edition es gratuita durante la versión preliminar. Sin embargo, el costo de cualquier infraestructura subyacente configurada para la máquina virtual (almacenamiento, proceso, redes, etc.) se le seguirá cargando en su suscripción.

### <a name="how-can-i-get-troubleshooting-support-for-hotpatching"></a>¿Cómo se puede obtener ayuda para solucionar problemas con la aplicación de revisiones en caliente?

* Se puede presentar una [incidencia de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). En la opción Servicio, busque y seleccione **Máquina virtual que ejecuta Windows** en Proceso. Seleccione **Características de Azure** como tipo de problema y **Automatic VM Guest Patching** (Aplicación automática de revisiones a invitados de máquina virtual) como subtipo de problema.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre Azure Update Management [aquí](../automation/update-management/overview.md).
* Más información sobre la revisión automática de invitados de máquina virtual [aquí](../virtual-machines/automatic-vm-guest-patching.md).
