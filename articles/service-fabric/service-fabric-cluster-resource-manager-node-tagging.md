---
title: Etiquetas de nodo dinámicas de Azure Service Fabric
description: Azure Service Fabric le permite agregar y quitar etiquetas de nodo dinámicamente.
author: yu-supersonic
ms.topic: conceptual
ms.date: 04/05/2021
ms.author: branim
ms.custom: devx-track-csharp
ms.openlocfilehash: 712e6422060619e5567a74d6335320eff9ed8e66
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107741439"
---
# <a name="introduction-to-dynamic-node-tags"></a>Introducción a las etiquetas de nodo dinámicas
Las etiquetas de nodo le permiten agregar y quitar etiquetas dinámicamente de los nodos para influir en la selección de ubicación de los servicios. El etiquetado de nodos es muy flexible y permite cambios en la selección de ubicación del servicio sin actualizaciones de aplicaciones ni de clústeres. Las etiquetas se pueden agregar o quitar de los nodos en cualquier momento, y los servicios pueden especificar requisitos para determinadas etiquetas cuando se crean. También es posible actualizar dinámicamente los requisitos de las etiquetas de un servicio mientras está en ejecución.

El etiquetado de nodos es similar a las [restricciones de selección de ubicación](service-fabric-cluster-resource-manager-configure-services.md), y normalmente se usa para controlar en qué nodos se ejecuta un servicio. Cada servicio de Service Fabric se puede configurar para requerir que se coloque la etiqueta o para seguir ejecutándose.

El etiquetado de nodos es compatible con todos los tipos de servicios hospedados de Service Fabric (Reliable Services, ejecutables de invitado y contenedores). Para usar el etiquetado de nodos, tiene que ejecutar la versión 8.0 o posterior del entorno en tiempo de ejecución de Service Fabric.

En lo que queda de este artículo, se describen las maneras de habilitar o deshabilitar el etiquetado de nodos, y se dan ejemplos sobre cómo usar esta característica.


## <a name="describing-dynamic-node-tags"></a>Descripción de etiquetas de nodo dinámicas
Los servicios pueden especificar las etiquetas que requieren. Existen dos tipos de etiquetas:
* Las **etiquetas necesarias para la selección de ubicación** describen un conjunto de etiquetas, que solo son necesarias para la selección de ubicación del servicio. Una vez que se coloca la réplica, estas etiquetas se pueden quitar sin interrumpir el servicio. Si alguna de estas etiquetas se quita del nodo, la réplica de servicio seguirá funcionando, y Service Fabric no quitará el servicio.

* Las **etiquetas necesarias para la ejecución** describen un conjunto de etiquetas que son necesarias tanto para la selección de ubicación como para la ejecución del servicio. Si se quita cualquiera de las etiquetas en ejecución necesarias, Service Fabric moverá el servicio a otro nodo donde se hayan especificado esas etiquetas.

Ejemplo: Las etiquetas necesarias para la selección de ubicación se pueden usar cuando se usa algún tipo de servicio de activador de contenedor, y se necesita ese servicio para que se ubique el contenedor y, en cuanto se active el contenedor, ya no necesitará el activador, y puede quitar la etiqueta asociada a él, pero el contenedor debe seguir ejecutándose.
Las etiquetas necesarias para la ejecución se pueden usar cuando se tiene un servicio de facturación, lo que resulta útil para colocarse con un servicio orientado al usuario. Cuando se produce un error en el servicio de facturación en el nodo, usted quita la etiqueta asociada a él, y el servicio orientado al usuario se mueve a otro nodo, que tiene el servicio de facturación y su etiqueta presentes.

Una etiqueta o un conjunto de etiquetas se pueden agregar, actualizar o quitar de un nodo único mediante mecanismos de interfaz estándar de Service Fabric, como las API de C#, las API REST o los comandos de PowerShell.

> [!NOTE]
> Service Fabric no mantiene distribuciones UD/FD cuando se usan etiquetas de nodo. Administre las etiquetas de nodo correctamente, para no obtener infracciones de FD/UD debido a una distribución incorrecta de las etiquetas entre los dominios.

## <a name="enabling-dynamic-node-tags"></a>Habilitación de etiquetas de nodo dinámicas
Para que esta característica funcione, tendrá que habilitar la configuración nodeTaggingEnabled en la sección PlacementAndLoadBalancing del manifiesto de clúster mediante XML o JSON:

``` xml
<Section Name="PlacementAndLoadBalancing">
     <Parameter Name="NodeTaggingEnabled" Value="true" />
</Section>
```

a través de ClusterConfig.json para las implementaciones independientes o Template.json para los clústeres hospedados en Azure:

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": " NodeTaggingEnabled ",
          "value": "true"
      }
    ]
  }
]
```

## <a name="setting-dynamic-node-tags"></a>Definición de etiquetas de nodo dinámicas

### <a name="using-powershell"></a>Usar PowerShell

Agregue etiquetas de nodo al nodo:

```posh
Add-ServiceFabricNodeTags -NodeName "DB.1" -NodeTags @("SampleTag1", "SampleTag2")
```
Este comando agregará las etiquetas "SampleTag1" y "SampleTag2" al nodo DB.1.

Quite las etiquetas de nodo del nodo:

```posh
Remove-ServiceFabricNodeTags -NodeName "DB.1" -NodeTags @("SampleTag1", "SampleTag2")
```
Este comando quitará las etiquetas "SampleTag1" y "SampleTag2" del nodo DB.1.

### <a name="using-c-apis"></a>Uso de las API de C#

Agregue etiquetas de nodo al nodo:

```csharp
FabricClient fabricClient = new FabricClient();
List<string> nodeTagsList = new List<string>();
nodeTagsList.Add("SampleTag1");
nodeTagsList.Add("SampleTag2");
await fabricClient.ClusterManager.AddNodeTagsAsync("DB.1", nodeTagsList);
```

Quite las etiquetas de nodo del nodo:

```csharp
FabricClient fabricClient = new FabricClient();
List<string> nodeTagsList = new List<string>();
nodeTagsList.Add("SampleTag1");
nodeTagsList.Add("SampleTag2");
await fabricClient.ClusterManager.RemoveNodeTagsAsync("DB.1", nodeTagsList);
```

## <a name="setting-required-tags-for-services"></a>Definición de las etiquetas necesarias para los servicios

### <a name="using-powershell"></a>Usar PowerShell

Cree un nuevo servicio:

```posh
New-ServiceFabricService -ApplicationName fabric:/HelloWorld -ServiceName fabric:/HelloWorld/svc1 -ServiceTypeName HelloWorldStateful -Stateful -PartitionSchemeSingleton -TargetReplicaSetSize 5 -MinReplicaSetSize 3 -TagsRequiredToRun @("SampleTag1") - TagsRequiredToPlace @("SampleTag2")
```
Este comando crea un servicio, que requiere que "SampleTag2" esté presente en un nodo para que el servicio se coloque allí, y que "SampleTag1" esté presente para que el servicio siga ejecutándose en ese nodo.

Actualice el servicio existente:

```posh
Update-ServiceFabricService -Stateful -ServiceName fabric:/myapp/test -TagsRequiredToRun @("SampleTag1") -TagsRequiredToPlace @("SampleTag2")
```
Este comando actualiza un servicio, que requiere que "SampleTag2" esté presente en un nodo para que el servicio se coloque allí, y que "SampleTag1" esté presente para que el servicio siga ejecutándose en ese nodo.

### <a name="using-c-apis"></a>Uso de las API de C#

Cree un nuevo servicio:

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
//set up the rest of the ServiceDescription
ServiceTags serviceTags = new ServiceTags();
serviceTags.TagsRequiredToPlace.Add("SampleTag1");
serviceTags.TagsRequiredToRun.Add("SampleTag2");
serviceDescription.ServiceTags = serviceTags;
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Actualice el servicio existente:

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceUpdateDescription serviceUpdate = new StatefulServiceUpdateDescription();
ServiceTags serviceTags = new ServiceTags();
serviceTags.TagsRequiredToPlace.Add("SampleTag1");
serviceTags.TagsRequiredToRun.Add("SampleTag2");
serviceUpdate.ServiceTags = serviceTags;
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), serviceUpdate);
```

Todos estos comandos se aplican igualmente a los servicios sin estado.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre las [restricciones de selección de ubicación](service-fabric-cluster-resource-manager-configure-services.md).
