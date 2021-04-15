---
title: Revisión del sistema operativo Windows en el clúster de Service Fabric
description: En este artículo se explica cómo automatizar la aplicación de revisiones de sistema operativo en un clúster de Service Fabric mediante la aplicación de orquestación de revisiones.
services: service-fabric
documentationcenter: .net
author: athinanthny
manager: chackdan
editor: ''
ms.assetid: de7dacf5-4038-434a-a265-5d0de80a9b1d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/01/2019
ms.author: atsenthi
ms.openlocfilehash: e94b809513bda8edc7a51baf79ec05a2c9c77489
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448561"
---
# <a name="patch-the-windows-operating-system-in-your-service-fabric-cluster"></a>Revisión del sistema operativo Windows en el clúster de Service Fabric

## <a name="automatic-os-image-upgrades"></a>Actualizaciones automáticas de las imágenes del sistema operativo

Las obtención de [actualizaciones de imágenes de sistema operativo automáticas en el conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) es el procedimiento recomendado para mantener el sistema operativo revisado en Azure. Las actualizaciones automáticas de imágenes del sistema operativo basadas en un conjunto de escalado de máquinas virtuales requieren una durabilidad Silver o superior en un conjunto de escalado.

Requisitos para las actualizaciones automáticas de imágenes del sistema operativo por Virtual Machine Scale Sets
-   El [nivel de durabilidad](../service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) de Service Fabric es Plata u Oro, no Bronce.
-   La extensión de Service Fabric en la definición del modelo de conjunto de escalado debe tener la versión 1.1 o posterior de TypeHandlerVersion.
-   El nivel de durabilidad debe ser el mismo en el clúster de Service Fabric y la extensión Service Fabric de la definición del modelo de conjunto de escalado.
- No es necesario realizar un sondeo de estado adicional o el uso de la extensión de estado de aplicación para Virtual Machine Scale Sets.

Asegúrese de que la configuración de durabilidad coincida con la del clúster y la extensión de Service Fabric, ya que la falta de coincidencia produce errores de actualización. Los niveles de durabilidad se pueden modificar según las directrices que se describen en [esta página](../service-fabric/service-fabric-cluster-capacity.md#changing-durability-levels).

La actualización automática de la imagen del sistema operativo no está disponible con durabilidad Bronce. Si bien la [Aplicación de orquestación de parches](#patch-orchestration-application ) (pensada solo para clústeres alojados que no sean de Azure) *no se recomienda* para niveles de durabilidad Plata o mayores, es su única opción para automatizar las actualizaciones de Windows con respecto a la actualización de dominios de Service Fabric.

> [!IMPORTANT]
> Las actualizaciones de VM, donde "Windows Update" aplica las revisiones del sistema operativo sin reemplazar el disco de este, no se admiten en Azure Service Fabric.

Hay dos pasos necesarios para habilitar correctamente la característica con Windows Update deshabilitado en el sistema operativo.

1. Habilitar la actualización automática de la imagen del sistema operativo y deshabilitar la ARM de Windows Update 
    ```json
    "virtualMachineProfile": { 
        "properties": {
          "upgradePolicy": {
            "automaticOSUpgradePolicy": {
              "enableAutomaticOSUpgrade":  true
            }
          }
        }
      }
    ```
    
    ```json
    "virtualMachineProfile": { 
        "osProfile": { 
            "windowsConfiguration": { 
                "enableAutomaticUpdates": false 
            }
        }
    }
    ```

    Azure PowerShell
    ```azurepowershell-interactive
    Update-AzVmss -ResourceGroupName $resourceGroupName -VMScaleSetName $scaleSetName -AutomaticOSUpgrade $true -EnableAutomaticUpdate $false
    ``` 
    
1. Actualizar el modelo del conjunto de escalado: después de cambiar esta configuración, se necesita restablecer la imagen inicial para actualizar el modelo del conjunto de escalado para que el cambio surta efecto.
    
    Azure PowerShell
    ```azurepowershell-interactive
    $scaleSet = Get-AzVmssVM -ResourceGroupName $resourceGroupName -VMScaleSetName $scaleSetName
    $instances = foreach($vm in $scaleSet)
    {
        Set-AzVmssVM -ResourceGroupName $resourceGroupName -VMScaleSetName $scaleSetName -InstanceId $vm.InstanceID -Reimage
    }
    ``` 
    
Eche un vistazo a las [actualizaciones automáticas de la imagen del sistema operativo de Virtual Machine Scale Sets](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) para obtener más instrucciones.

## <a name="patch-orchestration-application"></a>Aplicación de la orquestación de revisiones

> [!IMPORTANT]
> A partir del 30 de abril de 2019, ya no se admite la versión 1.2.* de la aplicación de orquestación de revisiones. Asegúrese de haber actualizado a la última versión.

La aplicación de orquestación de revisiones (POA) es un contenedor alrededor del servicio Administrador de reparaciones de Azure Service Fabric, que habilita la programación de revisiones del sistema operativo basada en la configuración para clústeres que no están hospedados en Azure. Aunque no se necesita POA para clústeres hospedados en ubicaciones distintas de Azure, es necesario programar la instalación de revisiones mediante dominio de actualización para aplicar revisiones en los hosts de clúster de Service Fabric sin incurrir en tiempo de inactividad.

POA es una aplicación de Service Fabric que automatiza la instalación de revisiones de sistema operativo en un clúster de Service Fabric sin instalación en tiempo de inactividad.

POA ofrece las siguientes características:

- **Instalación automática de actualizaciones de sistema operativo**. Las actualizaciones de sistema operativo se descargan e instalan automáticamente. Los nodos de clúster se reinician según sea necesario sin inactividad en tiempo de inactividad del clúster.

- **Integración de la aplicación de revisiones y el estado del clúster**. Mientras POA aplica actualizaciones, monitorea la salud de los nodos del clúster. Los nodos de clúster se actualizan de uno en uno o por dominio de actualización. Si el estado del clúster deja de funcionar debido al proceso de aplicación de revisiones, la aplicación de revisiones se detiene para evitar agravar el problema.

## <a name="internal-details-of-poa"></a>Detalles internos de POA

POA consta de los siguientes subcomponentes:

- **Coordinator Service**: este servicio con estado es responsable de:
    - Coordinar el trabajo de Windows Update en todo el clúster.
    - Almacenar el resultado de las operaciones completadas de Windows Update.

- **Node Agent Service**: es un servicio sin estado que se ejecuta en todos los nodos del clúster de Service Fabric. El servicio es responsable de:
    - Arrancar el servicio Node Agent NTService.
    - Supervisar el servicio Node Agent NTService.

- **Node Agent NTService**: este servicio de Windows NT se ejecuta con privilegios elevados (SYSTEM). En cambio, Node Agent Service y Coordinator Service se ejecutan con privilegios de nivel inferior (NETWORK SERVICE). El servicio es responsable de realizar las siguientes tareas de Windows Update en todos los nodos de clúster:
    - Deshabilitar las actualizaciones automáticas de Windows en el nodo.
    - Descargar e instalar Windows Update según la directiva que el usuario ha proporcionado.
    - Reiniciar el equipo después de la instalación de Windows Update.
    - Realizar la carga de los resultados de las actualizaciones de Windows al servicio Coordinator Service.
    - Realizar un informe de estado si se produce un error en la operación después de agotar todos los reintentos.

> [!NOTE]
> POA usa el servicio de administrador de reparaciones de Service Fabric para habilitar o deshabilitar el nodo, y llevar a cabo comprobaciones de estado. La tarea de reparación creada por POA sigue el progreso de Windows Update en cada nodo.

## <a name="prerequisites"></a>Requisitos previos

> [!NOTE]
> La versión mínima requerida de .NET framework es 4.6.

### <a name="enable-the-repair-manager-service-if-its-not-running-already"></a>Habilitar el servicio de administrador de reparaciones (si no se está ejecutando ya)

POA requiere que el servicio del administrador de reparaciones esté habilitado en el clúster.

#### <a name="azure-clusters"></a>Clústeres de Azure

Los clústeres de Azure en el nivel de durabilidad Plata tendrán habilitado el administrador de reparaciones de forma predeterminada. Es posible que los clústeres de Azure en el nivel de durabilidad Oro tengan habilitado o no el servicio de administrador de reparaciones, dependiendo de cuándo se crearon esos clústeres. Los clústeres de Azure en el plan de durabilidad Bronce no tienen habilitado el servicio de administrador de reparaciones. Si el servicio ya está habilitado, puede ver que se ejecuta en la sección de servicios del sistema en Service Fabric Explorer.

##### <a name="the-azure-portal"></a>El Portal de Azure
Puede habilitar Administrador de reparaciones desde Azure Portal al configurar el clúster. Cuando configure el clúster, seleccione la opción **Incluir administrador de reparaciones** en **Características complementarias**.

![Imagen de la habilitación del administrador de reparaciones de Azure Portal](media/service-fabric-patch-orchestration-application/EnableRepairManager.png)

##### <a name="the-azure-resource-manager-deployment-model"></a>El modelo de implementación de Azure Resource Manager
También puede usar el [modelo de implementación de Azure Resource Manager](./service-fabric-cluster-creation-via-arm.md) para habilitar el servicio del administrador de reparaciones en clústeres de Service Fabric nuevos y existentes. Obtenga la plantilla del clúster que desea implementar. Puede usar las plantillas de ejemplo o crear una plantilla del modelo de implementación de Azure Resource Manager personalizada. 

Para habilitar el servicio del administrador de reparaciones mediante la [plantilla del modelo de implementación de Azure Resource Manager](./service-fabric-cluster-creation-via-arm.md), haga lo siguiente:

1. Asegúrese de que `apiVersion` esté establecido en *2017-07-01-preview* para el recurso *Microsoft.ServiceFabric/clusters*. Si es diferente, debe actualizar `apiVersion` a *2017-07-01-preview* o posterior:

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

1. Habilite el servicio de administrador de reparaciones agregando la sección `addonFeatures` siguiente después de la sección `fabricSettings`:

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

3. Una vez actualizada la plantilla del clúster con estos cambios, aplíquelos para completar la actualización. Ahora puede ver el servicio del administrador de reparaciones en ejecución en el clúster. Se denomina *fabric:/System/RepairManagerService* en la sección servicios del sistema de Service Fabric Explorer. 

### <a name="standalone-on-premises-clusters"></a>Clústeres locales independientes

Puede usar las [Opciones de configuración de clústeres de Windows independientes](./service-fabric-cluster-manifest.md) para habilitar el servicio de administrador de reparaciones en clústeres de Service Fabric nuevos o existentes.

Para habilitar el servicio de administrador de reparaciones:

1. Asegúrese de que `apiVersion` en las [configuraciones generales del clúster](./service-fabric-cluster-manifest.md#general-cluster-configurations) esté establecido en *04-2017* o posterior, como se muestra aquí:

    ```json
    {
        "name": "SampleCluster",
        "clusterConfigurationVersion": "1.0.0",
        "apiVersion": "04-2017",
        ...
    }
    ```

1. Habilite el servicio de administrador de reparaciones agregando la sección `addonFeatures` después de la sección `fabricSettings`, tal y como se muestra a continuación:

    ```json
    "fabricSettings": [
        ...      
    ],
    "addonFeatures": [
        "RepairManager"
    ],
    ```

1. Actualice el manifiesto de clúster con estos cambios, con el manifiesto de clúster actualizado [crear un nuevo clúster](./service-fabric-cluster-creation-for-windows-server.md) o [actualizar la configuración del clúster](./service-fabric-cluster-upgrade-windows-server.md). 

   Una vez que el clúster se ejecuta con un manifiesto de clúster actualizado, puede ver el servicio Administrador de reparaciones que se ejecuta en el clúster. Se denomina *fabric:/System/RepairManagerService*, y está en la sección servicios del sistema de Service Fabric Explorer.

### <a name="configure-windows-updates-for-all-nodes"></a>Configurar las actualizaciones de Windows para todos los nodos

Es posible que las actualizaciones automáticas de Windows provoquen la pérdida de disponibilidad, ya que varios nodos de clúster pueden reiniciarse al mismo tiempo. POA intenta deshabilitar las actualizaciones automáticas de Windows en cada nodo del clúster de forma predeterminada. En caso de que la configuración se administre mediante directivas de administrador o de grupo, se recomienda configurar explícitamente la directiva de Windows Update en "Notificar antes de descargar".

## <a name="download-the-application-package"></a>Descargar del paquete de aplicación

Para descargar el paquete de la aplicación, vaya a la página de la [versión de la aplicación de orquestación de revisiones](https://github.com/microsoft/Service-Fabric-POA/releases/latest/) en GitHub.

## <a name="configure-poa-behavior"></a>Configurar el comportamiento de POA

Puede configurar el comportamiento de POA para satisfacer sus necesidades. Invalide los valores predeterminados al pasar el parámetro de la aplicación durante la creación o actualización de la aplicación. Puede proporcionar los parámetros de la aplicación especificando `ApplicationParameter` en los cmdlets `Start-ServiceFabricApplicationUpgrade` o `New-ServiceFabricApplication`.

| Parámetro        | Tipo                          | Detalles |
|:-|-|-|
|MaxResultsToCache    |long                              | El número máximo de resultados de Windows Update que deben almacenarse en caché. <br><br>El valor predeterminado es 3000 suponiendo que: <br> &nbsp;&nbsp;- El número de nodos es 20. <br> &nbsp;&nbsp;- El número de actualizaciones en un nodo al mes es 5. <br> &nbsp;&nbsp;- El número de resultados por cada operación es 10. <br> &nbsp;&nbsp;- Deben almacenarse los resultados de los últimos tres meses. |
|TaskApprovalPolicy   |Enum <br> { NodeWise, UpgradeDomainWise }                          |TaskApprovalPolicy indica la directiva que usará Coordinator Service para instalar las actualizaciones de Windows en todos los nodos del clúster de Service Fabric.<br><br>Los valores permitidos son: <br>*NodeWise*: Las actualizaciones de Windows se instalan en un nodo cada vez. <br> *UpgradeDomainWise*: Las actualizaciones de Windows se instalan en un dominio de actualización cada vez. (Como máximo, todos los nodos que pertenecen a un dominio de actualización son aptos para una actualización de Windows).<br><br> Para ayudar a decidir qué directiva es la más adecuada para el clúster, consulte la sección [P+F](#frequently-asked-questions).
|LogsDiskQuotaInMB   |long  <br> (Valor predeterminado: *1024*)               | El tamaño máximo de los registros de la aplicación de orquestación de revisiones en MB que se pueden almacenar de forma persistente y local en un nodo.
| WUQuery               | string<br>(Valor predeterminado: *IsInstalled=0*)                | Consulta para obtener las actualizaciones de Windows. Para más información, vea [WuQuery](/windows/win32/api/wuapi/nf-wuapi-iupdatesearcher-search).
| InstallWindowsOSOnlyUpdates | *Boolean* <br> (valor predeterminado: false)                 | Use esta marca para controlar qué actualizaciones deben descargarse e instalarse. Se permiten los siguientes valores <br>True: instala solo actualizaciones del sistema operativo Windows.<br>False: instala todas las actualizaciones disponibles en la máquina.          |
| WUOperationTimeOutInMinutes | Int <br>(Valor predeterminado: *90*)                   | Especifica el tiempo de espera para cualquier operación de Windows Update (buscar, descargar o instalar). Si la operación no se realiza en el tiempo de espera especificado, se anula.       |
| WURescheduleCount     | Int <br> (Valor predeterminado: *5*)                  | El número máximo de veces que el servicio vuelve a programar la actualización de Windows si se produce un error de la operación de forma persistente.          |
| WURescheduleTimeInMinutes | Int <br>(Valor predeterminado: *30*) | El intervalo en el que el servicio vuelve a programar la actualización de Windows si el error persiste. |
| WUFrequency           | Cadena separada por comas (valor predeterminado: *Weekly, Wednesday, 7:00:00*)     | Frecuencia para la instalación de actualizaciones de Windows. El formato y los valores posibles son: <br>- Monthly, DD, HH:MM:SS (ejemplo: *Monthly, 5, 12:22:32*). Los valores permitidos para el campo _DD_ (día) son números comprendidos entre 1 y 28, y _last_. <br>- Weekly, día, HH:MM:SS (ejemplo: *Weekly, martes, 12:22:32*)  <br>- Daily, HH:MM:SS (ejemplo: *Daily, 12:22:32*)  <br>- MonthlyByWeekAndDay, semana, día, HH:MM:SS (ejemplo: *MonthlyByWeekAndDay, 2, viernes, 21:00:00* indica 9:00 PM UTC del viernes de la segunda semana de cada mes) <br>- *None* indica que no deben aplicarse actualizaciones de Windows.  <br><br> Las horas se muestran en UTC.|
| AcceptWindowsUpdateEula | Boolean <br>(Valor predeterminado: *true*) | Al establecer esta marca, la aplicación acepta el contrato de licencia del usuario final para Windows Update en nombre del propietario del equipo.              |

> [!TIP]
> Si quieres que las actualizaciones de Windows se ejecuten de forma inmediata, establezca `WUFrequency` en relación con la hora de implementación de la aplicación. Por ejemplo, suponga que tiene un clúster de prueba de cinco nodos y planea implementar la aplicación aproximadamente a las 17:00 UTC. Si supone que la implementación o actualización de la aplicación tarda 30 minutos como máximo, establezca WUFrequency como *Daily, 17:30:00*.

## <a name="deploy-poa"></a>Implementar POA

1. Finalice todos los pasos de requisitos previos para preparar el clúster.
1. Implemente POA como cualquier otra aplicación de Service Fabric. Para implementarla mediante PowerShell, consulte [Implementación y eliminación de aplicaciones con PowerShell](./service-fabric-deploy-remove-applications.md).
1. Para configurar la aplicación en el momento de la implementación, pase el `ApplicationParameter` al cmdlet `New-ServiceFabricApplication`. Para su comodidad, hemos proporcionado el script Deploy.ps1 junto con la aplicación. Para usar el script:

    - Conéctese a un clúster de Service Fabric mediante `Connect-ServiceFabricCluster`.
    - Ejecute el script de PowerShell Deploy.ps1 con el valor adecuado de `ApplicationParameter`.

> [!NOTE]
> Mantenga el script y la carpeta *PatchOrchestrationApplication* de la aplicación en el mismo directorio.

## <a name="upgrade-poa"></a>Actualizar POA

Para actualizar su versión de POA mediante PowerShell, siga las instrucciones descritas en [Actualización de aplicaciones de Service Fabric con PowerShell](./service-fabric-application-upgrade-tutorial-powershell.md).

## <a name="remove-poa"></a>Quitar POA

Para quitar la aplicación, siga las instrucciones descritas en [Implementación y eliminación de aplicaciones con PowerShell](./service-fabric-deploy-remove-applications.md).

Para su comodidad, hemos proporcionado el script Undeploy.ps1 junto con la aplicación. Para usar el script:

  - Conéctese a un clúster de Service Fabric mediante ```Connect-ServiceFabricCluster```.
  - Ejecute el script de PowerShell Undeploy.ps1.

> [!NOTE]
> Mantenga el script y la carpeta *PatchOrchestrationApplication* de la aplicación en el mismo directorio.

## <a name="view-the-windows-update-results"></a>Ver los resultados de Windows Update

POA expone API de REST para mostrar los resultados históricos a los usuarios. Un ejemplo del resultado JSON:

```json
[
  {
    "NodeName": "_stg1vm_1",
    "WindowsUpdateOperationResults": [
      {
        "OperationResult": 0,
        "NodeName": "_stg1vm_1",
        "OperationTime": "2019-05-13T08:44:56.4836889Z",
        "OperationStartTime": "2019-05-13T08:44:33.5285601Z",
        "UpdateDetails": [
          {
            "UpdateId": "7392acaf-6a85-427c-8a8d-058c25beb0d6",
            "Title": "Cumulative Security Update for Internet Explorer 11 for Windows Server 2012 R2 (KB3185319)",
            "Description": "A security issue has been identified in a Microsoft software product that could affect your system. You can help protect your system by installing this update from Microsoft. For a complete listing of the issues that are included in this update, see the associated Microsoft Knowledge Base article. After you install this update, you may have to restart your system.",
            "ResultCode": 0,
            "HResult": 0
          }
        ],
        "OperationType": 1,
        "WindowsUpdateQuery": "IsInstalled=0",
        "WindowsUpdateFrequency": "Daily,10:00:00",
        "RebootRequired": false
      }
    ]
  },
  ...
]
```

Los campos JSON se describen en la tabla siguiente:

Campo | Valores | Detalles
-- | -- | --
OperationResult | 0: se realizó correctamente<br> 1: se realizó correctamente con errores<br> 2: con error<br> 3: anulada<br> 4: anulada con tiempo de espera | Indica el resultado de la operación global (que normalmente implica la instalación de una o más actualizaciones).
ResultCode | Igual que OperationResult | Este campo indica el resultado de la operación de instalación para una actualización individual.
OperationType | 1: instalación<br> 0: buscar y descargar| De forma predeterminada, la instalación es el único valor de OperationType que se muestra en los resultados.
WindowsUpdateQuery | El valor predeterminado es "IsInstalled=0" | La consulta de Windows Update que se utilizó para buscar actualizaciones. Para más información, vea [WuQuery](/windows/win32/api/wuapi/nf-wuapi-iupdatesearcher-search).
RebootRequired | true: se requiere reinicio<br> false: no se requiere reinicio | Indica si se requiere reiniciar para completar la instalación de actualizaciones.
operationStartTime | DateTime | Indica la hora a la que se inició la operación (descarga/instalación).
OperationTime | DateTime | Indica la hora a la que se completó la operación (descarga/instalación).
HResult | 0: correcto<br> otro: error| Indica el motivo del error de la actualización de Windows con updateID "7392acaf-6a85-427c-8a8d-058c25beb0d6".

Si todavía no hay ninguna actualización programada, el resultado JSON está vacío.

Inicie sesión en el clúster para consultar los resultados de Windows Update. Obtenga la dirección IP de la réplica principal del servicio Coordinador y abra la siguiente dirección URL desde el explorador: http://&lt;IP DE RÉPLICA&gt;:&lt;PuertoDeAplicación&gt;/PatchOrchestrationApplication/v1/GetWindowsUpdateResults.

El punto de conexión REST para el servicio Coordinator Service tiene un puerto dinámico. Para comprobar la dirección URL exacta, consulte Service Fabric Explorer. Por ejemplo, los resultados están disponibles en *http://10.0.0.7:20000/PatchOrchestrationApplication/v1/GetWindowsUpdateResults* .

![Imagen del punto de conexión REST](media/service-fabric-patch-orchestration-application/Rest_Endpoint.png)

Si el proxy inverso está habilitado en el clúster, también puede tener acceso a la dirección URL desde fuera del clúster.

El punto de conexión al que debe dirigirse es *http://&lt;SERVERURL&gt;:&lt;REVERSEPROXYPORT&gt;/PatchOrchestrationApplication/CoordinatorService/v1/GetWindowsUpdateResults*.

Para habilitar el proxy inverso en el clúster, siga las instrucciones descritas en [Proxy inverso en Azure Service Fabric](./service-fabric-reverseproxy.md). 

> 
> [!WARNING]
> Después de configurar el proxy inverso, se podrá acceder de manera externa a todos los microservicios del clúster que exponen un punto de conexión HTTP.

## <a name="diagnostics-and-health-events"></a>Diagnóstico y eventos de mantenimiento

En esta sección se describe cómo depurar o diagnosticar problemas con actualizaciones de revisiones a través de POA en clústeres de Service Fabric.

> [!NOTE]
> Para obtener muchas de las mejoras de autodiagnóstico que se mencionan a continuación, debe tener instalada la versión v1.1.4.0 de POA.

NodeAgentNTService crea [tareas de reparación](/dotnet/api/system.fabric.repair.repairtask) para instalar actualizaciones en los nodos. Cada tarea la prepara el servicio Coordinator Service de acuerdo con la directiva de aprobación de tareas. Finalmente, las tareas preparadas están aprobadas por el Administrador de reparaciones, que no aprueba ninguna tarea si el clúster se encuentra en un estado incorrecto. 

Vayamos paso a paso para comprender cómo funcionan las actualizaciones en un nodo:

1. NodeAgentNTService, que se ejecuta en cada nodo, busca las actualizaciones de Windows disponibles a la hora programada. Si hay actualizaciones disponibles, las descarga en el nodo.

1. Una vez que se descargan las actualizaciones, NodeAgentNTService crea una tarea de reparación correspondiente para el nodo con el nombre *POS___\<unique_id>* . Puede ver estas tareas de reparación con el cmdlet [Get-ServiceFabricRepairTask](/powershell/module/servicefabric/get-servicefabricrepairtask) o con SFX en la sección de detalles del nodo. Una vez que se crea la tarea de reparación, esta se mueve rápidamente a un [estado *reclamado*](/dotnet/api/system.fabric.repair.repairtaskstate).

1. El servicio del coordinador busca periódicamente las tareas de reparación que tengan un estado *reclamado* y las actualiza al estado *en preparación* en función de TaskApprovalPolicy. Si TaskApprovalPolicy se configura para ser NodeWise, una tarea de reparación correspondiente a un nodo se prepara solo si no hay otra tarea de reparación actualmente en el estado *En preparación*, *Aprobada*, *En ejecución*, o *En restauración*. 

   De manera similar, en el caso de UpgradeWise TaskApprovalPolicy, hay tareas en los estados anteriores solo para los nodos que pertenecen al mismo dominio de actualización. Una vez que una tarea de reparación tiene el estado *En preparación*, el nodo correspondiente de Service Fabric se [deshabilita](/powershell/module/servicefabric/disable-servicefabricnode) con la intención de *Reiniciar*.

   Las versiones de POA 1.4.0 y posteriores publican eventos con la propiedad ClusterPatchingStatus en CoordinatorService para mostrar los nodos que se van a revisar. Las actualizaciones se instalan en _poanode_0, tal como se muestra en la siguiente imagen:

    [![Imagen del estado de revisión del clúster](media/service-fabric-patch-orchestration-application/clusterpatchingstatus.png)](media/service-fabric-patch-orchestration-application/clusterpatchingstatus.png#lightbox)

1. Una vez que el nodo está deshabilitado, la tarea de reparación se mueve al estado de *En ejecución*. 
   
   > [!NOTE]
   > Un nodo bloqueado en un estado *deshabilitado* puede bloquear una nueva tarea de reparación, lo cual detiene la operación de aplicación de revisiones en el clúster.

1. Una vez que la tarea de reparación está en estado de *En ejecución*, comienza la instalación de la revisión de ese nodo. Una vez instalada la revisión, el nodo puede o no reiniciarse dependiendo de la misma. A continuación, la tarea de reparación se mueve al estado *En restauración*, lo que permite volver a habilitar el nodo. A continuación, la tarea de reparación se marca como completada.

   En las versiones de POA 1.4.0 y versiones posteriores, puede encontrar el estado de la actualización viendo los eventos de estado en NodeAgentService con la propiedad WUOperationStatus-\<NodeName>. Las secciones resaltadas en las imágenes que tiene a continuación muestran el estado de la actualización de Windows en el nodo *poanode_0* y *poanode_2*:

   [![Captura de pantalla que muestra la ventana de la consola de estado de la operación de Windows Update con poanode_0 resaltado.](media/service-fabric-patch-orchestration-application/wuoperationstatusa.png)](media/service-fabric-patch-orchestration-application/wuoperationstatusa.png#lightbox)

   [![Captura de pantalla que muestra la ventana de la consola de estado de la operación de Windows Update con poanode_1 resaltado.](media/service-fabric-patch-orchestration-application/wuoperationstatusb.png)](media/service-fabric-patch-orchestration-application/wuoperationstatusb.png#lightbox)

   También puede obtener los detalles mediante el uso de PowerShell. Para ello, conéctese al clúster y recupere el estado de la tarea de reparación mediante [Get-ServiceFabricRepairTask](/powershell/module/servicefabric/get-servicefabricrepairtask). 
   
   En el siguiente ejemplo, la tarea "POS__poanode_2_125f2969-933c-4774-85d1-ebdf85e79f15" está en estado *DownloadComplete*. Esto significa que las actualizaciones se han descargado en el nodo *poanode_2* y que la instalación se intentará una vez que la tarea pase al estado de *En ejecución*.

   ``` powershell
    D:\service-fabric-poa-bin\service-fabric-poa-bin\Release> $k = Get-ServiceFabricRepairTask -TaskId "POS__poanode_2_125f2969-933c-4774-85d1-ebdf85e79f15"

    D:\service-fabric-poa-bin\service-fabric-poa-bin\Release> $k.ExecutorData
    {"ExecutorSubState":2,"ExecutorTimeoutInMinutes":90,"RestartRequestedTime":"0001-01-01T00:00:00"}
    ```

   Si quedan más problemas, inicie sesión en la máquina virtual (VM) o en las máquinas virtuales y obtenga información sobre ellas mediante el uso de registros de eventos de Windows. La tarea de reparación mencionada anteriormente solo puede existir en los siguientes subestados de ejecutor:

      ExecutorSubState | Descripción
    -- | -- 
      None=1 |  Implica que no hubo una operación en curso en el nodo. El estado puede estar en transición.
      DownloadCompleted=2 | Implica que la operación de descarga se completó correctamente, con un error parcial o con error.
      InstallationApproved=3 | Implica que la operación de descarga se completó anteriormente y que el Administrador de reparaciones aprobó la instalación.
      InstallationInProgress=4 | Corresponde al estado de ejecución de la tarea de reparación.
      InstallationCompleted=5 | Implica que la operación de instalación se completó correctamente, correctamente de forma parcial o con error.
      RestartRequested=6 | Implica la instalación de revisiones completada y que hay una acción de reinicio pendiente en el nodo.
      RestartNotNeeded=7 |  Implica que el reinicio no era necesario una vez completada la instalación de la revisión.
      RestartCompleted=8 | Implica que el reinicio se completó correctamente.
      OperationCompleted=9 | La operación de Windows Update finalizó correctamente.
      OperationAborted=10 | Implica que la operación de Windows Update se anuló.

1. En las versiones de POA 1.4.0 y versiones posteriores, cuando finaliza un intento de actualización de nodo, se publica un evento con la propiedad "WUOperationStatus-[NodeName]" en NodeAgentService para recibir una notificación cuando se inicie el siguiente intento de descargar e instalar las actualizaciones de Windows. Esto se muestra en la siguiente imagen:

     [![Captura de pantalla que muestra la ventana de la consola de estado de la operación de Windows Update con NodeAgentService.](media/service-fabric-patch-orchestration-application/wuoperationstatusc.png)](media/service-fabric-patch-orchestration-application/wuoperationstatusc.png#lightbox)

### <a name="diagnostics-logs"></a>Registros de diagnóstico

Los registros de aplicación de orquestación de revisiones se recopilan como parte de los registros en tiempo de ejecución de Service Fabric.

Puede capturar registros mediante la herramienta de diagnóstico o la canalización que prefiera. POA usa los siguientes identificadores de proveedor fijos para registrar eventos a través de [origen del evento](/dotnet/api/system.diagnostics.tracing.eventsource):

- e39b723c-590c-4090-abb0-11e3e6616346
- fc0028ff-bfdc-499f-80dc-ed922c52c5e9
- 24afa313-0d3b-4c7c-b485-1047fd964b60
- 05dc046c-60e9-4ef7-965e-91660adffa68

### <a name="health-reports"></a>Informes de mantenimiento

POA también publica los informes de estado en el servicio Coordinator Service o el servicio Node Agent Service en los siguientes casos:

* El servicio Node Agent Service está inactivo

   Si el servicio Node Agent NTService está inactivo en un nodo, se genera un informe de estado de nivel de advertencia en el servicio Node Agent Service.

* El servicio de administrador de reparaciones no está habilitado

   Si el servicio de administrador de reparaciones no se encuentra en el clúster, se genera un informe de estado de nivel de advertencia para el servicio Coordinator Service.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**P: ¿Por qué veo el clúster en estado de error cuando se ejecuta POA?**

A. Durante el proceso de instalación, POA deshabilita o reinicia los nodos, lo que puede provocar temporalmente un clúster incorrecto.

En función de la directiva de la aplicación, un nodo puede dejar de estar activo durante una operación de aplicación de revisiones, *o bien* un dominio de actualización completo puede dejar de estar activo.

Al final de la instalación de las actualizaciones de Windows, los nodos se vuelven a habilitar después del reinicio.

En el ejemplo siguiente, el clúster entró en estado de error temporalmente porque dos nodos dejaron de estar activos y se infringió la directiva MaxPercentageUnhealthyNodes. El error es temporal hasta que se puede iniciar la operación de aplicación de revisiones.

![Imagen de clúster en mal estado](media/service-fabric-patch-orchestration-application/MaxPercentage_causing_unhealthy_cluster.png)

Si el problema persiste, consulte la sección de solución de problemas.

**P: ¿Qué puedo hacer si POA está en un estado de advertencia?**

A. Compruebe si un informe de estado publicado a la aplicación indica la causa principal. Normalmente la advertencia contiene los detalles del problema. Si el problema es transitorio, se espera que la aplicación se recupere automáticamente.

**P: ¿Qué puedo hacer si el clúster está en mal estado y tengo que realizar una actualización urgente del sistema operativo?**

A. POA no instala actualizaciones cuando el clúster está en mal estado. Intente devolver el clúster a un estado correcto para desbloquear el flujo de trabajo de POA.

**P: ¿Debo definir TaskApprovalPolicy como "NodeWise" o "UpgradeDomainWise" para el clúster?**

A. La configuración "UpgradeDomainWise" acelera la reparación general del clúster mediante la revisión en paralelo de todos los nodos que pertenecen a un dominio de actualización. Durante el proceso, los nodos que pertenecen a un dominio de actualización completo no están disponibles (en [estado *deshabilitado*](/dotnet/api/system.fabric.query.nodestatus#System_Fabric_Query_NodeStatus_Disabled)).

En cambio, la configuración "NodeWise" solo revisa un nodo a la vez, lo que implicaría que la revisión del clúster general podría tardar más tiempo. Sin embargo, como máximo, solo habría un nodo no disponible (en estado *deshabilitado*) durante el proceso de aplicación de revisiones.

Si el clúster puede tolerar la ejecución en un número N-1 de dominios de actualización durante el ciclo de aplicación de revisiones (donde N es el número total de dominios de actualización del clúster), puede establecer la directiva como "UpgradeDomainWise". En caso contrario, establézcala en "NodeWise".

**P: ¿Cuánto se tarda en aplicar revisiones a un nodo?**

A. La aplicación de revisiones a un nodo puede tardar minutos (por ejemplo, [actualizaciones de definiciones de Windows Defender](https://www.microsoft.com/en-us/wdsi/definitions)) u horas (por ejemplo: [actualizaciones acumulativas de Windows](https://www.catalog.update.microsoft.com/Search.aspx?q=windows%20server%20cumulative%20update)). El tiempo necesario para aplicar revisiones en un nodo depende, principalmente, de: 
 - El tamaño de las actualizaciones.
 - El número de actualizaciones, que deben aplicarse en un periodo de revisión.
 - El tiempo necesario para instalar las actualizaciones, reiniciar el nodo (si es necesario) y completar los pasos de instalación posteriores al reinicio.
 - El rendimiento de la máquina o VM y las condiciones de red.

**P: ¿Cuánto se tarda en aplicar revisiones a todo un clúster?**

A. El tiempo necesario para aplicar una revisión a un clúster completo depende de:

- El tiempo necesario para aplicar revisiones a un nodo.

- La directiva del servicio Coordinator Service. La directiva predeterminada, "NodeWise", da como resultado la revisión de un solo nodo cada vez, un enfoque más lento que el uso de "UpgradeDomainWise". 

   Por ejemplo: si la aplicación de revisiones a un nodo tarda, aproximadamente, una hora, para aplicar revisiones en un clúster de 20 nodos (mismo tipo de nodos) con 5 dominios de actualización que contienen, cada uno, 4 nodos, requiere lo siguiente:
    - Para "NodeWise": aproximadamente 20 horas.
    - Para "UpgradeDomainWise": aproximadamente 5 horas.

- La carga del clúster. Cada operación de revisión requiere cambiar la ubicación de la carga de trabajo del cliente a otros nodos disponibles en el clúster. El nodo al que se aplicaría la revisión tendría el [estado *deshabilitado*](/dotnet/api/system.fabric.query.nodestatus#System_Fabric_Query_NodeStatus_Disabling) durante este tiempo. Si el clúster se ejecuta cerca de la carga máxima, el proceso de deshabilitación podría tardar más. Por lo tanto, el proceso general de aplicación de revisiones puede parecer lento en estas condiciones.

- Errores de estado del clúster durante la aplicación de revisiones. Cualquier [degradación](/dotnet/api/system.fabric.health.healthstate#System_Fabric_Health_HealthState_Error) en el [estado del clúster](./service-fabric-health-introduction.md) interrumpiría el proceso de revisión. Esta incidencia se agregaría al tiempo general necesario para aplicar revisiones a todo el clúster.

**P: ¿Por qué veo algunas actualizaciones en los resultados de Windows Update obtenidos a través de la API de REST, pero no en el historial de Windows Update en la máquina?**

A. Algunas actualizaciones del producto solo aparecen en su propio historial de actualizaciones o revisiones. Por ejemplo, puede que se muestren o que no se muestren las actualizaciones de Windows Defender en el historial de Windows Update en Windows Server 2016.

**P: ¿POA se puede utilizar para aplicar revisiones al clúster de desarrollo (clúster con un solo nodo)?**

A. No, POA no se puede utilizar para aplicar revisiones a un clúster de un nodo. Esta limitación se produce por diseño, ya que los [servicios del sistema de Service Fabric](./service-fabric-technical-overview.md#system-services) u otras aplicaciones de cliente incurrirían en tiempo de inactividad. Por lo tanto, la revisión de los trabajos de reparación nunca se aprobó por el Administrador de reparaciones.

**P: ¿Cómo se pueden revisar los nodos del clúster en Linux?**

A. Para aprender a para organizar las actualizaciones en Linux, consulte [Azure virtual machine scale set automatic OS image upgrades](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) (Actualizaciones de imagen automáticas del sistema operativo del conjunto de escalado de máquinas virtuales de Azure).

**P: ¿Por qué está tardando tanto el ciclo de actualización?**

A. Consulte el resultado de JSON, y vaya a la entrada del ciclo de actualizaciones de todos los nodos; a continuación, averigüe el tiempo que tarda la instalación de la actualización en cada nodo mediante OperationStartTime y OperationTime (OperationCompletionTime). 

Si hay un período de tiempo grande en la que no se realiza ninguna actualización, el clúster puede estar en un estado de error y, por consiguiente, Administrador de reparaciones no puede aprobar las tareas de reparación de POA. Si la instalación de la actualización está tardando mucho en un nodo, es posible que ese nodo no se haya actualizado durante un tiempo. Es posible que haya una gran cantidad de actualizaciones pendientes de instalación, lo que puede dar lugar a retrasos. 

También es posible que se bloquee la revisión del nodo porque está atascado en el estado *deshabilitado*. Esto suele ocurrir porque deshabilitar el nodo podría provocar situaciones de pérdida de datos o cuórum.

**P: ¿Por qué debe deshabilitarse el nodo cuando POA lo está aplicando?**

A. POA inhabilita el nodo con el intento de *reinicio* que detiene o reasigna todos los servicios de la estructura de Service Fabric que se ejecutan en el nodo. POA hace esto para garantizar que las aplicaciones no terminen usando una combinación de archivos DLL nuevos y antiguos, por lo que no recomendamos revisar un nodo sin desactivarlo antes.

**P: ¿Cuál es el número máximo de nodos que se puede actualizar con POA?**

A. POA usa el Administrador de reparaciones de Service Fabric para crear tareas de reparación de nodos para las actualizaciones. No obstante, no puede haber más de 250 tareas de reparación al mismo tiempo. Actualmente, POA crea tareas de reparación para cada nodo al mismo tiempo, de modo que no puede actualizar más de 250 nodos en un clúster. 

## <a name="disclaimers"></a>Declinación de responsabilidades

- POA acepta el acuerdo de licencia del usuario final de Windows Update en nombre del usuario. Opcionalmente se puede desactivar la configuración en la configuración de la aplicación.

- POA recopila datos de telemetría para realizar el seguimiento de uso y rendimiento. Los datos de telemetría de la aplicación siguen la configuración de telemetría del runtime de Service Fabric (que se activa de forma predeterminada).

## <a name="troubleshooting"></a>Solución de problemas

En esta sección se proporcionan posibles soluciones para solucionar problemas relacionados con la revisión de nodos.

### <a name="a-node-is-not-coming-back-to-up-state"></a>El nodo no vuelve a su estado activo

* Es posible que el nodo esté bloqueado en un estado *deshabilitado* porque:

  - Hay una comprobación de seguridad pendiente. Para solucionar este problema, asegúrese de que hay suficientes nodos disponibles en estado correcto.

* Es posible que el nodo esté bloqueado en un estado *deshabilitado* porque:

  - Se ha deshabilitado manualmente.
  - Se ha deshabilitado por causa de un trabajo de infraestructura de Azure en curso.
  - POA lo deshabilitó temporalmente para aplicar revisiones al nodo.

* Es posible que el nodo esté bloqueado en un estado inactivo porque:

  - Se ha desactivado manualmente.
  - Se está llevando a cabo un reinicio (que puede desencadenar POA).
  - Tiene un problema en la máquina virtual o tiene problemas de conectividad de red.

### <a name="updates-were-skipped-on-some-nodes"></a>Las actualizaciones se omitieron en algunos nodos

POA intenta instalar las actualizaciones de Windows según la directiva de reprogramación. El servicio puede intentar recuperar el nodo y omitir la actualización en función de la directiva de la aplicación.

En ese caso, se genera un informe de estado de nivel de advertencia en el servicio Node Agent Service. El resultado de Windows Update también contiene el motivo posible del error.

### <a name="the-health-of-the-cluster-goes-to-error-while-the-update-is-being-installed"></a>El estado del clúster es de error mientras se instala la actualización

Una actualización de Windows fallida puede desactivar el estado de una aplicación o un clúster en un nodo concreto o un dominio de actualización. POA interrumpe las operaciones posteriores de Windows Update hasta que el estado del clúster vuelva a ser correcto.

Un administrador debe intervenir y determinar por qué la aplicación o el clúster pasó a ser incorrecto debido a Windows Update.

## <a name="poa-release-notes"></a>Notas de la versión de POA

>[!NOTE]
> En el caso de las versiones de POA 1.4.0 y versiones posteriores, puede encontrar notas de la versión y versiones en la [página de la versión de la aplicación de orquestación de revisiones](https://github.com/microsoft/Service-Fabric-POA/releases/) en GitHub.

### <a name="version-110"></a>Versión 1.1.0
- Versión pública

### <a name="version-111"></a>Versión 1.1.1
- Se ha corregido un error en SetupEntryPoint de NodeAgentService que impedía la instalación de NodeAgentNTService.

### <a name="version-120"></a>Versión 1.2.0

- Correcciones de errores en el flujo de trabajo de reinicio del sistema.
- Corrección de errores en la creación de tareas de RM debido a las cuales la comprobación del estado durante la preparación de las tareas de reparación no ocurría según lo previsto.
- Cambio en el modo de inicio del servicio de Windows POANodeSvc de automático a automático con retraso.

### <a name="version-121"></a>Versión 1.2.1

- Corrija los errores en el flujo de trabajo de reducción vertical de clúster. Introdujo la lógica de recolección de elementos no utilizados para las tareas de reparación POA pertenecientes a nodos inexistentes.

### <a name="version-122"></a>Versión 1.2.2

- Correcciones de errores varios.
- Ahora se firman los archivos binarios.
- Vínculo sfpkg agregado para la aplicación.

### <a name="version-130"></a>Versión 1.3.0

- Si establece InstallWindowsOSOnlyUpdates en false se instalarán todas las actualizaciones disponibles.
- Se ha cambiado la lógica de deshabilitación de las actualizaciones automáticas. Esto permite solucionar un problema por el que las actualizaciones automáticas no se deshabilitaban en Server 2016 o versiones posteriores.
- Restricción de colocación con parámetros para ambos microservicios de POA para casos de uso avanzados.

### <a name="version-131"></a>Versión 1.3.1
- Corrección de una regresión en la que POA 1.3.0 no funcionará en Windows Server 2012 R2 o versión anterior debido a un error al deshabilitar las actualizaciones automáticas. 
- Corrección de un error donde la configuración InstallWindowsOSOnlyUpdates siempre se elige como True.
- Cambio del valor predeterminado de InstallWindowsOSOnlyUpdates a False.

### <a name="version-132"></a>Versión 1.3.2
- Corregir un problema que afecta al ciclo de vida de aplicación de revisiones en un nodo, si hay nodos con nombre que sea subconjunto del nombre de nodo actual. Para esos nodos, es posible, que falte la aplicación de revisiones o que el reinicio esté pendiente.
