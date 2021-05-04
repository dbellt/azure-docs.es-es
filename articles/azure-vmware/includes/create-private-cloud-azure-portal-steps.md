---
title: Creación de una nube privada de Azure VMware Solution
description: Pasos para crear una nube privada de Azure VMware Solution mediante Azure Portal.
ms.topic: include
ms.date: 04/23/2021
ms.openlocfilehash: 40bd1880511f22d9518d0c4526bc697a3693a518
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107945847"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-create-private-cloud.md -->

Puede crear una nube privada de Azure VMware Solution mediante Azure Portal o con la CLI de Azure.


### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Crear un nuevo recurso**. 

1. En el cuadro de texto **Buscar en el Marketplace**, escriba `Azure VMware Solution` y seleccione **Azure VMware Solution** en la lista. 

1. En la ventana **Azure VMware Solution**, seleccione **Crear**.

1. En la pestaña **Básica**, escriba valores en los campos. 

   >[!TIP]
   >La información se recopiló durante la [fase de planeamiento](../production-ready-deployment-steps.md) de esta inicio rápido.

   | Campo   | Valor  |
   | ---| --- |
   | **Suscripción** | Seleccione la suscripción que va a usar para la implementación.|
   | **Grupos de recursos** | Seleccione el grupo para los recursos de nube privada. |
   | **Ubicación** | Seleccione una ubicación, como **Este de EE. UU.** Esta es la *región* que definió en la fase de planeamiento. |
   | **Nombre del recurso** | Facilite el nombre de la nube privada de Azure VMware Solution. |
   | **SKU** | Seleccione **AV36**. |
   | **Hosts** | Indica el número de hosts asignados para el clúster de nube privada. El valor predeterminado es 3, que se puede aumentar o reducir después de la implementación.  |
   | **Bloque de direcciones** | Escriba un bloque de direcciones IP para la red CIDR para la nube privada; por ejemplo, 10.175.0.0/22. |
   | **Virtual Network** | Mantenga este campo en blanco porque el circuito de ExpressRoute de Azure VMware Solution se establece en un paso posterior a la implementación.   |

   :::image type="content" source="../media/tutorial-create-private-cloud/create-private-cloud.png" alt-text="En la pestaña Básico, escriba los valores de los campos." border="true":::

1. Cuando termine, seleccione **Revisar y crear**. En la siguiente pantalla, compruebe la información especificada. Si toda la información es correcta, seleccione **Crear**.

   > [!NOTE]
   > Este paso tarda aproximadamente entre 3 y 4 horas. La incorporación de un nodo individual a un clúster existente o al mismo clúster tarda entre 30 y 45 minutos.

1. Comprobación de que la implementación se ha realizado correctamente. Navegue hasta el grupo de recursos que creó y seleccione la nube privada.  Una vez que se haya completado la implementación, se mostrará el estado **Correcto**. 

   :::image type="content" source="../media/tutorial-create-private-cloud/validate-deployment.png" alt-text="Comprobación de que la implementación se ha realizado correctamente." border="true":::


### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
En lugar de usar Azure Portal para crear una nube privada de Azure VMware Solution, puede usar la CLI de Azure con Azure Cloud Shell. Para conocer la lista de los comandos que puede usar con Azure VMware Solution, consulte [Comandos de Azure VMware](/cli/azure/ext/vmware/vmware).

Para empezar a usar la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header](../../../includes/azure-cli-prepare-your-environment-no-header.md)]


1. Para crear un grupo de recursos, use el comando ["az group create"](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

   ```azurecli-interactive
   
   az group create --name myResourceGroup --location eastus
   ```

2. Proporcione un nombre para el grupo de recursos y la nube privada, una ubicación y el tamaño del clúster.

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
