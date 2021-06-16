---
title: Actualización del entorno de ejecución de Service Fabric en Azure
description: En este tutorial, aprenderá a utilizar PowerShell para actualizar el runtime de un clúster de Service Fabric hospedado en Azure.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 7745a4c1cf316a1c8ed38f6a54ca765550c9c81c
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110664952"
---
# <a name="tutorial-upgrade-the-runtime-of-a-service-fabric-cluster-in-azure"></a>Tutorial: Actualización del runtime de un clúster de Service Fabric en Azure

Este tutorial es la cuarta parte de una serie, y muestra cómo actualizar el runtime de Service Fabric en un clúster de Azure Service Fabric. Esta parte del tutorial se escribe para los clústeres de Service Fabric que se ejecutan en Azure y no se aplica a los clústeres de Service Fabric independientes.

> [!WARNING]
> Esta parte del tutorial requiere PowerShell. La actualización del runtime del clúster no se admite todavía con las herramientas de la CLI de Azure. Como alternativa, un clúster se puede actualizar en el portal. Para obtener más información, consulte [Actualización de un clúster de Azure Service Fabric](service-fabric-cluster-upgrade.md).

Si el clúster ya está ejecutando el runtime más reciente de Service Fabric, no es necesario realizar este paso. Sin embargo, este artículo puede usarse para instalar cualquier runtime compatible en un clúster de Azure Service Fabric.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Leer la versión del clúster
> * Establecer la versión del clúster

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * Creación de un [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) en Azure mediante una plantilla
> * [Supervisión de un clúster](service-fabric-tutorial-monitor-cluster.md)
> * [Escalado o reducción horizontal](service-fabric-tutorial-scale-cluster.md)
> * Actualización del entorno en tiempo de ejecución de un clúster
> * [Eliminación de un clúster](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este tutorial:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Instale [Azure Powershell](/powershell/azure/install-az-ps) o la [CLI de Azure](/cli/azure/install-azure-cli).
* Creación de un [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) seguro en Azure
* Configure un entorno de desarrollo de Windows. Instale [Visual Studio 2019](https://www.visualstudio.com) y las cargas de trabajo de **desarrollo Azure**, **desarrollo web y ASP.NET** y **desarrollo a través de plataformas .NET Core**.  Después, configure un [entorno de desarrollo .NET](service-fabric-get-started.md).

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en su cuenta de Azure y seleccione su suscripción antes de ejecutar comandos de Azure.

```powershell
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>
```

## <a name="get-the-runtime-version"></a>Obtención de la versión del runtime

Después de haberse conectado a Azure y seleccionado la suscripción que contiene el clúster de Service Fabric, puede obtener la versión del runtime del clúster.

```powershell
Get-AzServiceFabricCluster -ResourceGroupName SFCLUSTERTUTORIALGROUP -Name aztestcluster `
    | Select-Object ClusterCodeVersion
```

O bien, puede obtener simplemente una lista de todos los clústeres de su suscripción con el ejemplo siguiente:

```powershell
Get-AzServiceFabricCluster | Select-Object Name, ClusterCodeVersion
```

Anote el valor de **ClusterCodeVersion**. Este valor se usará en la sección siguiente.

## <a name="upgrade-the-runtime"></a>Actualización del runtime

Utilice el valor de **ClusterCodeVersion** de la sección anterior con el cmdlet `Get-ServiceFabricRuntimeUpgradeVersion` para conocer a qué versiones es posible actualizar. Solo se puede ejecutar este cmdlet desde un equipo conectado a Internet. Por ejemplo, si deseara ver a qué versiones de runtime puede actualizar desde la versión `5.7.198.9494`, usaría el siguiente comando:

```powershell
Get-ServiceFabricRuntimeUpgradeVersion -BaseVersion "5.7.198.9494"
```

Con una lista de versiones, puede indicar al clúster de Azure Service Fabric que se actualice a un runtime más reciente. Por ejemplo, si está disponible la versión `6.0.219.9494` como actualización, utilice el siguiente comando para actualizar el clúster.

```powershell
Set-AzServiceFabricUpgradeType -ResourceGroupName SFCLUSTERTUTORIALGROUP `
                                    -Name aztestcluster `
                                    -UpgradeMode Manual `
                                    -Version "6.0.219.9494"
```

> [!IMPORTANT]
> La actualización del runtime del clúster puede tardar mucho tiempo en completarse. PowerShell se bloquea mientras se ejecuta la actualización. Puede usar otra sesión de PowerShell para comprobar el estado de la actualización.

El estado de la actualización se puede supervisar con PowerShell o la CLI de Azure Service Fabric (sfctl).

En primer lugar, conéctese al clúster con el certificado TLS/SSL que se creó en la primera parte del tutorial. Utilice el cmdlet `Connect-ServiceFabricCluster` o `sfctl cluster upgrade-status`.

```powershell
$endpoint = "<mycluster>.southcentralus.cloudapp.azure.com:19000"
$thumbprint = "63EB5BA4BC2A3BADC42CA6F93D6F45E5AD98A1E4"

Connect-ServiceFabricCluster -ConnectionEndpoint $endpoint `
                             -KeepAliveIntervalInSec 10 `
                             -X509Credential -ServerCertThumbprint $thumbprint `
                             -FindType FindByThumbprint -FindValue $thumbprint `
                             -StoreLocation CurrentUser -StoreName My
```

```console
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

A continuación, use `Get-ServiceFabricClusterUpgrade` o `sfctl cluster upgrade-status` para mostrar el estado. Se mostrará algo similar al resultado siguiente.

```powershell
Get-ServiceFabricClusterUpgrade

TargetCodeVersion                          : 6.0.219.9494
TargetConfigVersion                        : 3
StartTimestampUtc                          : 11/28/2017 3:09:48 AM
UpgradeState                               : RollingForwardPending
UpgradeDuration                            : 00:09:00
CurrentUpgradeDomainDuration               : 00:09:00
NextUpgradeDomain                          : 1
UpgradeDomainsStatus                       : { "0" = "Completed";
                                             "1" = "Pending";
                                             "2" = "Pending";
                                             "3" = "Pending";
                                             "4" = "Pending" }
UpgradeKind                                : Rolling
RollingUpgradeMode                         : Monitored
FailureAction                              : Rollback
ForceRestart                               : False
UpgradeReplicaSetCheckTimeout              : 37201.09:59:01
HealthCheckWaitDuration                    : 00:05:00
HealthCheckStableDuration                  : 00:05:00
HealthCheckRetryTimeout                    : 00:45:00
UpgradeDomainTimeout                       : 02:00:00
UpgradeTimeout                             : 12:00:00
ConsiderWarningAsError                     : False
MaxPercentUnhealthyApplications            : 0
MaxPercentUnhealthyNodes                   : 100
ApplicationTypeHealthPolicyMap             : {}
EnableDeltaHealthEvaluation                : True
MaxPercentDeltaUnhealthyNodes              : 0
MaxPercentUpgradeDomainDeltaUnhealthyNodes : 0
ApplicationHealthPolicyMap                 : {}
```

```console
sfctl cluster upgrade-status

{
  "codeVersion": "6.0.219.9494",
  "configVersion": "3",

... item cut to save space ...

  },
  "upgradeDomains": [
    {
      "name": "0",
      "state": "Completed"
    },
    {
      "name": "1",
      "state": "Pending"
    },
    {
      "name": "2",
      "state": "Pending"
    },
    {
      "name": "3",
      "state": "Pending"
    },
    {
      "name": "4",
      "state": "Pending"
    }
  ],
  "upgradeDurationInMilliseconds": "PT1H2M4.63889S",
  "upgradeState": "RollingForwardPending"
}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Obtener la versión del runtime del clúster
> * Actualizar el runtime del clúster
> * Supervisar la actualización

Avance hasta el siguiente tutorial:

> [!div class="nextstepaction"]
> [Eliminación de un clúster](service-fabric-tutorial-delete-cluster.md)
