---
title: Aprovisionamiento de un grupo en una red virtual
description: Cómo crear un grupo de Batch en una red virtual de Azure para que los nodos de proceso puedan comunicarse de manera segura con otras máquinas virtuales en la red, como un servidor de archivos.
ms.topic: how-to
ms.date: 06/09/2021
ms.custom: seodec18
ms.openlocfilehash: 5279e9efb426c327761f14188b436e47387ea1eb
ms.sourcegitcommit: f9e368733d7fca2877d9013ae73a8a63911cb88f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111903098"
---
# <a name="create-an-azure-batch-pool-in-a-virtual-network"></a>Creación de un grupo de Azure Batch en una red virtual

Al crear un grupo de Azure Batch, puede aprovisionarlo en una subred de una [red virtual de Azure](../virtual-network/virtual-networks-overview.md) que especifique. En este artículo se explica cómo configurar un grupo de Batch en una red virtual.

## <a name="why-use-a-vnet"></a>¿Por qué se utiliza una red virtual?

Los nodos de proceso de un grupo pueden comunicarse entre sí, por ejemplo, para ejecutar tareas de instancias múltiples, sin necesidad de una red virtual independiente. Sin embargo, de forma predeterminada, los nodos de un grupo no se pueden comunicar con las máquinas virtuales que están fuera del grupo, como los servidores de licencias o los servidores de archivos.

Para permitir que los nodos de proceso se comuniquen de manera segura con otras máquinas virtuales o con una red local, puede aprovisionar el grupo en una subred de una red virtual de Azure.

## <a name="prerequisites"></a>Requisitos previos

- **Autenticación**. Para usar una red virtual de Azure, la API de cliente de Batch debe usar la autenticación de Azure Active Directory (AD). La compatibilidad de Azure Batch con Azure AD se documenta en [Autenticación de soluciones de servicio de Batch con Active Directory](batch-aad-auth.md).

- **Una red virtual de Azure**. Consulte la sección siguiente para conocer los requisitos de la red virtual y cómo configurarla. Para preparar una red virtual con una o varias subredes previamente, puede usar Azure Portal, Azure PowerShell, la interfaz de línea de comandos (CLI) de Azure u otros métodos.
  - Para crear una red virtual basada en Azure Resource Manager, consulte [Creación de una red virtual](../virtual-network/manage-virtual-network.md#create-a-virtual-network). Una red virtual basada en Resource Manager se recomienda para las implementaciones nuevas y solo se admite en los grupos que usen Configuración de máquina virtual.
  - Para crear una red virtual clásica, consulte [Creación de una red virtual (clásica) con varias subredes](/previous-versions/azure/virtual-network/create-virtual-network-classic). Una red virtual clásica solo se admite en los grupos que usen la configuración de Cloud Services.

## <a name="vnet-requirements"></a>Requisitos de la red virtual

[!INCLUDE [batch-virtual-network-ports](../../includes/batch-virtual-network-ports.md)]

## <a name="create-a-pool-with-a-vnet-in-the-azure-portal"></a>Creación de un grupo con una red virtual en Azure Portal

Una vez que haya creado la red virtual y le haya asignado una subred, puede crear un grupo de Batch a esa red virtual. Siga estos pasos para crear un grupo en Azure Portal: 

1. Vaya a la cuenta de Batch en Azure Portal. Esta cuenta debe estar en la misma suscripción y región que el grupo de recursos que contiene la red virtual que pretende utilizar.
2. En la ventana **Configuración** que aparece a la izquierda, seleccione el elemento de menú **Grupos**.
3. En la ventana **Grupos**, seleccione **Agregar**.
4. En la ventana **Agregar grupo**, seleccione la opción que desea utilizar en el menú desplegable **Tipo de imagen**.
5. Seleccione el valor correcto de **publicador/oferta/SKU** para la imagen personalizada.
6. Especifique los valores requeridos restantes, incluido el **tamaño de nodo**, los **nodos dedicados de destino** y los **nodos de baja prioridad**, además de cualquier otro valor opcional que desee.
7. En **Virtual Network**, seleccione la red virtual y la subred que desea usar.

   ![Adición de un grupo con red virtual](./media/batch-virtual-network/add-vnet-pool.png)

## <a name="user-defined-routes-for-forced-tunneling"></a>Rutas definidas por el usuario para la tunelización forzada

Es posible que en su organización haya que redirigir (exigir) el tráfico vinculado con Internet desde la subred a la ubicación local para llevar a cabo la inspección y el registro. Además, es posible que haya habilitado la tunelización forzada para las subredes de la red virtual.

Si desea asegurarse de que los nodos del grupo funcionen en una red virtual con la tunelización forzada habilitada, debe agregar las siguientes [rutas definidas por el usuario](../virtual-network/virtual-networks-udr-overview.md) (UDR) para esa subred:

- El servicio Batch debe comunicarse con los nodos para programar tareas. Para habilitar esta comunicación, agregue una ruta definida por el usuario para cada dirección IP usada por el servicio Batch en la región donde existe la cuenta de Batch. Las direcciones IP del servicio Batch se encuentran en la etiqueta de servicio `BatchNodeManagement.<region>`. Para obtener la lista de direcciones IP, consulte [Etiquetas de servicio en un entorno local](../virtual-network/service-tags-overview.md).

- Asegúrese de que la red local no bloquee el tráfico TCP saliente al servicio Azure Batch en el puerto de destino 443. Estas direcciones IP de destino del servicio Azure Batch son las mismas que se encuentran en la etiqueta de servicio `BatchNodeManagement.<region>` que se usa para las rutas anteriores.

- Asegúrese de que la red local no bloquee el tráfico TCP saliente a Azure Storage en el puerto de destino 443 (en concreto, las direcciones URL con el formato `*.table.core.windows.net`, `*.queue.core.windows.net` y `*.blob.core.windows.net`).

- Si usa montajes de archivos virtuales, revise los [requisitos de red](virtual-file-mount.md#networking-requirements) y asegúrese de que no se bloquee el tráfico necesario.

Cuando agregue una ruta definida por el usuario, defina la ruta de cada prefijo de dirección IP de Batch relacionado y establezca **Tipo del próximo salto** en **Internet**.

![Ruta definida por el usuario](./media/batch-virtual-network/user-defined-route.png)

> [!WARNING]
> Las direcciones IP del servicio Batch pueden cambiar con el tiempo. Para evitar las interrupciones debidas a un cambio en las direcciones IP, cree un proceso para actualizar automáticamente las direcciones IP del servicio Batch y mantenerlas actualizadas en la tabla de enrutamiento.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el [flujo de trabajo y los recursos principales del servicio Batch](batch-service-workflow-features.md), como grupos, nodos, trabajos y tareas.
- Obtenga información sobre cómo [crear una ruta definida por el usuario en Azure Portal](../virtual-network/tutorial-create-route-table-portal.md).
