---
title: 'Tutorial: Implementación de una nube privada de Azure VMware Solution'
description: Aprenda a crear y administrar una nube privada de Azure VMware Solution.
ms.topic: tutorial
ms.date: 02/22/2021
ms.openlocfilehash: 89a44ce7e5910609068f72c321971ced2e3646b4
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107374852"
---
# <a name="tutorial-deploy-an-azure-vmware-solution-private-cloud"></a>Tutorial: Implementación de una nube privada de Azure VMware Solution

Azure VMware Solution permite implementar un clúster de vSphere en Azure. La implementación inicial mínima es de tres hosts. Se pueden agregar más hosts de uno en uno, hasta un máximo de 16 hosts por clúster.

Dado que Azure VMware Solution no permite administrar la nube privada con la instancia local de vCenter en el inicio, se necesita una configuración adicional. Estos procedimientos y los requisitos previos relacionados se tratan en este tutorial.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una nube privada de Azure VMware Solution
> * Comprobar la nube privada implementada

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Permisos y derechos administrativos adecuados para crear una nube privada. Debe tener como mínimo un nivel de colaborador en la suscripción.
- Consulte la información recopilada en el artículo de [planeación](production-ready-deployment-steps.md) para implementar Azure VMware Solution.
- Asegúrese de que tiene las opciones de red correctamente configuradas como se describe en [Tutorial: Lista de comprobación de red](tutorial-network-checklist.md).
- Los hosts se han aprovisionado y el proveedor de recursos Microsoft.AVS se ha registrado como se describe en el artículo sobre [solicitud de hosts y habilitación del proveedor de recursos de Microsoft.AVS](enable-azure-vmware-solution.md).

## <a name="create-a-private-cloud"></a>Creación de una nube privada

Puede crear una nube privada de Azure VMware Solution mediante [Azure Portal](#azure-portal) o con la [CLI de Azure](#azure-cli).

### <a name="azure-portal"></a>Azure Portal

[!INCLUDE [create-avs-private-cloud-azure-portal](includes/create-private-cloud-azure-portal-steps.md)]

### <a name="azure-cli"></a>Azure CLI

En lugar de usar Azure Portal para crear una nube privada de Azure VMware Solution, puede usar la CLI de Azure con Azure Cloud Shell.  Para conocer la lista de los comandos que puede usar con Azure VMware Solution, consulte [Comandos de Azure VMware](/cli/azure/ext/vmware/vmware).

#### <a name="open-azure-cloud-shell"></a>Apertura de Azure Cloud Shell

Seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione **Entrar** para ejecutarlos.

#### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando ["az group create"](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive

az group create --name myResourceGroup --location eastus
```

#### <a name="create-a-private-cloud"></a>Creación de una nube privada

Proporcione un nombre para el grupo de recursos y la nube privada, una ubicación y el tamaño del clúster.

| Propiedad  | Descripción  |
| --------- | ------------ |
| **-g** (nombre del grupo de recursos)     | Nombre del grupo de recursos para los recursos de nube privada.        |
| **-n** (nombre de nube privada)     | Nombre de la nube privada de Azure VMware Solution.        |
| **--location**     | Ubicación que se usó para la nube privada.         |
| **--cluster-size**     | El tamaño del clúster. El valor mínimo es 3.         |
| **--network-block**     | Bloque de red de ñas direcciones IP de CIDR que se usará para la nube privada. El bloque de direcciones no debe superponerse a los bloques de direcciones que se hayan usado en otras redes virtuales ubicadas en las redes locales y de la suscripción.        |
| **--sku** | Valor de la SKU: AV36 |

```azurecli-interactive
az vmware private-cloud create -g myResourceGroup -n myPrivateCloudName --location eastus --cluster-size 3 --network-block xx.xx.xx.xx/22 --sku AV36
```

## <a name="azure-vmware-commands"></a>Comandos de Azure VMware

Para conocer la lista de los comandos que puede usar con Azure VMware Solution, consulte [Comandos de Azure VMware](/cli/azure/ext/vmware/vmware).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido cómo:

> [!div class="checklist"]
> * Creación de una nube privada de Azure VMware Solution
> * Comprobar la nube privada implementada
> * Eliminación de una nube privada de Azure VMware Solution

Continúe con el siguiente tutorial para aprender a crear un jumpbox. Use el jumpbox para conectarse a su entorno para que pueda administrar la nube privada localmente.


> [!div class="nextstepaction"]
> [Acceso a una nube privada de Azure VMware Solution](tutorial-access-private-cloud.md)
