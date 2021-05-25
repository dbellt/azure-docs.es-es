---
title: Implementación de una aplicación en un clúster administrado de Service Fabric mediante PowerShell
description: En este tutorial, se conectará a un clúster administrado de Service Fabric e implementará una aplicación mediante PowerShell.
ms.topic: tutorial
ms.date: 5/10/2021
ms.openlocfilehash: c51d753f3d0aa5d3c05c66fc45ae653f0bfa8e71
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685430"
---
# <a name="tutorial-deploy-an-app-to-a-service-fabric-managed-cluster"></a>Tutorial: Implementación de una aplicación en un clúster administrado de Service Fabric

En esta serie de tutoriales, trataremos de lo siguiente:

> [!div class="checklist"]
> * [Implementación de un clúster administrado de Service Fabric](tutorial-managed-cluster-deploy.md)
> * [Escalado horizontal de un clúster administrado de Service Fabric](tutorial-managed-cluster-scale.md)
> * [Incorporación y eliminación de nodos de un clúster administrado de Service Fabric](tutorial-managed-cluster-add-remove-node-type.md)
> * Implementación de una aplicación en un clúster administrado de Service Fabric

En esta parte de la serie se explica lo siguiente:

> [!div class="checklist"]
> * Conexión a un clúster administrado de Service Fabric
> * Implementación de una aplicación en un clúster
> * Creación de instancias de una aplicación en un clúster
> * Eliminación de una aplicación de un clúster

## <a name="prerequisites"></a>Requisitos previos

* Un clúster administrado de Service Fabric (consulte [*Implementación de un clúster administrado*](tutorial-managed-cluster-deploy.md)).

## <a name="connect-to-your-cluster"></a>Conexión al clúster

Para conectarse al clúster, necesitará la huella digital del certificado del clúster. Puede encontrar este valor en la salida de las propiedades del clúster de la implementación de recursos o consultando las propiedades del clúster en un recurso existente.

El siguiente comando se puede utilizar para consultar la huella digital del certificado del clúster en el recurso del clúster.

```powershell
$serverThumbprint = (Get-AzResource -ResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ServiceFabric/managedclusters/mysfcluster).Properties.clusterCertificateThumbprints
```

Con esta huella, ya está preparado para conectarse al clúster.

```powershell
$connectionEndpoint = "mysfcluster.eastus2.cloudapp.azure.com:19000"
Connect-ServiceFabricCluster -ConnectionEndpoint $connectionEndpoint -KeepAliveIntervalInSec 10 `
      -X509Credential `
      -ServerCertThumbprint $serverThumbprint  `
      -FindType FindByThumbprint `
      -FindValue $clientThumbprint `
      -StoreLocation CurrentUser `
      -StoreName My

```

### <a name="upload-an-application-package"></a>Carga de un paquete de aplicación

En este tutorial, utilizaremos el ejemplo de la [aplicación Voting de Service Fabric](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/tree/voting-sample-no-reverse-proxy). Para más información sobre la implementación de la aplicación Service Fabric mediante PowerShell, consulte [Implementación y eliminación de aplicaciones con PowerShell](service-fabric-deploy-remove-applications.md).

> [!NOTE]
> En la versión preliminar del clúster administrado de Service Fabric no podrá publicar aplicaciones directamente desde Visual Studio.

Primero deberá [empaquetar la aplicación para la implementación](service-fabric-package-apps.md). Para este tutorial, siga los pasos para empaquetar una aplicación desde Visual Studio. Es importante tener en cuenta la ruta de acceso donde se ha empaquetado la aplicación, ya que se utilizará para la ruta de acceso siguiente.

Una vez creado el paquete de aplicación, puede cargarlo en el clúster. Actualice el valor `$path` para que represente la ruta de acceso donde está el paquete de aplicación y ejecute lo siguiente:

```powershell
$path = "C:\Users\<user>\Documents\service-fabric-dotnet-quickstart\Voting\pkg\Debug"
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -CompressPackage
Register-ServiceFabricApplicationType -ApplicationPathInImageStore Debug
```

### <a name="create-an-application"></a>Crear una aplicación

Se pueden crear instancias de una aplicación a partir de cualquier versión del tipo de aplicación que se haya registrado correctamente mediante el cmdlet New-ServiceFabricApplication. El nombre de cada aplicación debe empezar con el esquema "fabric:" y ser único para cada instancia de la aplicación. También se crean los servicios predeterminados que se hayan definido en el manifiesto de aplicación del tipo de aplicación de destino.

```powershell
New-ServiceFabricApplication fabric:/Voting VotingType 1.0.0
```

Una vez completada esta operación, debería ver que las instancias de la aplicación se ejecutan en Service Fabric Explorer.

### <a name="remove-an-application"></a>Eliminación de una aplicación

Cuando ya no se necesite una instancia de aplicación, puede eliminarla de forma permanente por su nombre mediante el cmdlet `Remove-ServiceFabricApplication`, que también suprime automáticamente todos los servicios que pertenecen a la aplicación, lo que elimina de forma permanente todos los estados de servicio.

```powershell
Remove-ServiceFabricApplication fabric:/Voting
```

## <a name="next-steps"></a>Pasos siguientes

En este paso, hemos implementado una aplicación en un clúster administrado de Service Fabric. Para más información sobre las opciones de implementación de aplicaciones, consulte:

* [Implementación de secretos de aplicación de un clúster administrado](how-to-managed-cluster-application-secrets.md)
* [Implementación de aplicaciones de clúster administrado mediante plantillas de Resource Manager](how-to-managed-cluster-app-deployment-template.md)
* [Implementación de aplicaciones de clúster administrado con una identidad administrada](how-to-managed-cluster-application-managed-identity.md)

Para más información sobre las opciones de configuración del clúster administrado, consulte:

* [Configuración de un clúster administrado](how-to-managed-cluster-configuration.md)