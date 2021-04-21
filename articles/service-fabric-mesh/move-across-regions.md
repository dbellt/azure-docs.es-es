---
title: Traslado de una aplicación de Service Fabric Mesh a otra región
description: Para mover los recursos de Service Fabric Mesh, implemente una copia de la plantilla actual en una nueva región de Azure.
author: erikadoyle
ms.author: edoyle
ms.topic: how-to
ms.date: 01/14/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 9f3fcdc56b4e8d7873872212cb62f57a7669b459
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726635"
---
# <a name="move-a-service-fabric-mesh-application-to-another-azure-region"></a>Traslado de una aplicación de Service Fabric Mesh a otra región de Azure

> [!IMPORTANT]
> Se ha retirado la versión preliminar de Azure Service Fabric Mesh. Ya no se permitirán nuevas implementaciones a través de la API de Service Fabric Mesh. La compatibilidad con las implementaciones existentes continuará hasta el 28 de abril de 2021.
> 
> Para obtener más información, consulte [Retirada de la versión preliminar de Azure Service Fabric Mesh](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

En este artículo se describe cómo mover una aplicación de Service Fabric Mesh y sus recursos a otra región de Azure. Puede mover sus recursos a otra región por varios motivos. Por ejemplo, en respuesta a interrupciones, para obtener las características o los servicios disponibles solo en regiones concretas, para cumplir los requisitos de gobernanza y directivas internas o en respuesta a los requisitos de planeamiento de capacidad.

 [Service Fabric Mesh no admite](../azure-resource-manager/management/move-support-resources.md#microsoftservicefabricmesh) la capacidad para mover directamente los recursos entre regiones de Azure. Sin embargo, los recursos se pueden mover indirectamente. Para ello, es preciso implementar una copia de la plantilla de Azure Resource Manager actual en la nueva región de destino y después redireccionar el tráfico de entrada y las dependencias a la aplicación de Service Fabric Mesh recién creada.

## <a name="prerequisites"></a>Requisitos previos

* Un controlador de entrada (como [Application Gateway](../application-gateway/index.yml)) que sirva como intermediario para enrutar el tráfico entre clientes y la aplicación de Service Fabric Mesh
* Disponibilidad de Service Fabric Mesh (versión preliminar) en la región de Azure de destino (`westus`, `eastus` o `westeurope`)

## <a name="prepare"></a>Preparación

1. Tome una "instantánea" del estado actual de la aplicación de Service Fabric Mesh, para lo que debe exportar la plantilla de Azure Resource Manager y los parámetros de la implementación más reciente. Para ello, siga los pasos de [Exportación de la plantilla después de la implementación](../azure-resource-manager/templates/export-template-portal.md#export-template-after-deployment) desde Azure Portal. También puede usar la [CLI de Azure](../azure-resource-manager/management/manage-resource-groups-cli.md#export-resource-groups-to-templates), [Azure PowerShell](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates) o la [API REST](/rest/api/resources/resourcegroups/exporttemplate).

2. Si corresponde, [exporte otros recursos del mismo grupo](../azure-resource-manager/templates/export-template-portal.md#export-template-from-a-resource-group) para volver a implementarlos en la región de destino.

3. Examine (y edite si fuera necesario) la plantilla exportada para asegurarse de que los valores de la propiedad existentes son los que desea usar en la región de destino. El nuevo valor de `location` (región de Azure) es un parámetro que se especificará al volver a realizar la implementación.

## <a name="move"></a>Move

1. Cree un grupo de recursos en la región de destino (o use uno existente).

2. Con la plantilla exportada, siga los pasos que se indican en [Implementación de recursos desde una plantilla personalizada](../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template) mediante Azure Portal. También puede usar la [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md), [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md) o la [API REST](../azure-resource-manager/templates/deploy-rest.md).

3. Para obtener las instrucciones necesarias para mover los recursos relacionados como las [cuentas de Azure Storage](../storage/common/storage-account-move.md), consulte las instrucciones para los servicios individuales que se enumeran en el tema [Movimiento de recursos de Azure entre regiones](../azure-resource-manager/management/move-resources-overview.md#move-resources-across-regions).

## <a name="verify"></a>Verify

1. Una vez completada la implementación, pruebe los puntos de conexión de la aplicación para comprobar la funcionalidad de la misma.

2. También puede comprobar el estado de la aplicación comprobando el estado de la aplicación ([az mesh app show](/cli/azure/ext/mesh/mesh/app#ext-mesh-az-mesh-app-show)) y examinando los registros de la aplicación y ([az mesh code-package-log](/cli/azure/ext/mesh/mesh/code-package-log)) los comandos mediante la [CLI de Azure Service Fabric Mesh](./service-fabric-mesh-quickstart-deploy-container.md#set-up-service-fabric-mesh-cli).

## <a name="commit"></a>Commit

Una vez que haya confirmado la funcionalidad equivalente de su aplicación de Service Fabric Mesh en la región de destino, configure su controlador de entrada (por ejemplo, [Application Gateway](../application-gateway/redirect-overview.md)) para redirigir el tráfico a la aplicación nueva.

## <a name="clean-up-source-resources"></a>Limpieza de los recursos de origen

Para completar el traslado de la aplicación de Service Fabric Mesh, [elimine la aplicación de origen o el grupo de recursos principal](../azure-resource-manager/management/delete-resource-group.md).

## <a name="next-steps"></a>Pasos siguientes

* [Traslado de recursos de Azure entre regiones](../azure-resource-manager/management/move-resources-overview.md#move-resources-across-regions)
* [Compatibilidad con el traslado de recursos de Azure entre regiones](../azure-resource-manager/management/move-support-resources.md)
* [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
* [Compatibilidad con la operación de traslado para recursos](../azure-resource-manager/management/move-support-resources.md
)
