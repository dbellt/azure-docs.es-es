---
title: Implementación de hosts dedicados de Azure mediante Azure Portal
description: Implemente máquinas virtuales y conjuntos de escalado en hosts dedicados mediante Azure Portal.
author: cynthn
ms.service: virtual-machines
ms.subservice: dedicated-hosts
ms.topic: how-to
ms.workload: infrastructure
ms.date: 12/07/2020
ms.author: cynthn
ms.openlocfilehash: 8b10e281c20703f563fb2e0d3929ee936b094f26
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109753460"
---
# <a name="deploy-vms-and-scale-sets-to-dedicated-hosts-using-the-portal"></a>Implementación de máquinas virtuales y conjuntos de escalado en hosts dedicados mediante el portal

En este artículo se ofrecen instrucciones para crear un [host dedicado](dedicated-hosts.md) de Azure en el que se pueden hospedar máquinas virtuales (VM).


## <a name="limitations"></a>Limitaciones

- Los tamaños y tipos de hardware disponibles para hosts dedicados varían según la región. Para más información, consulte la [página de precios](https://aka.ms/ADHPricing) de hosts.

## <a name="create-a-host-group"></a>Creación de un grupo host

Un **grupo host** es un recurso que representa una colección de hosts dedicados. Puede crear un grupo host en una región y una zona de disponibilidad, y agregarle hosts. Al planear la alta disponibilidad, hay otras opciones. Puede usar una o ambas de las dos opciones siguientes con los hosts dedicados:
- Abarcar varias zonas de disponibilidad. En este caso, es necesario tener un grupo host en cada una de las zonas que quiera usar.
- Abarcar varios dominios de error que se asignan a bastidores físicos.

En cualquier caso, es necesario proporcionar el número de dominios de error del grupo host. Si no quiere abarcar dominios de error en el grupo, use un número de dominios de error de 1.

También puede usar zonas de disponibilidad y dominios de error a la vez.

En este ejemplo, se creará un grupo host con una zona de disponibilidad y dos dominios de error.


1. Abra el [portal](https://portal.azure.com).
1. Seleccione **Crear un recurso** en la esquina superior izquierda.
1. Busque el **grupo host** y, a continuación, seleccione los **grupos host** en los resultados.
1. En la página de los **grupos host**, seleccione **Crear**.
1. Seleccione la suscripción que quiere usar y, a continuación, seleccione **Crear nuevo** para crear un nuevo grupo de recursos.
1. Escriba *myDedicatedHostsRG* como **nombre** y seleccione **Aceptar**.
1. En el **nombre del grupo host**, escriba *myHostGroup*.
1. En **Ubicación**, seleccione **Este de EE.UU.**
1. Para la **zona de disponibilidad**, seleccione **1**.
1. En el **recuento de dominios de error**, seleccione **2**.
1. Seleccione **Selección de ubicación automática** para asignar automáticamente las instancias de máquinas virtuales y conjuntos de escalado a un host disponible de este grupo.
1. Seleccione **Revisar y crear** y, a continuación, espere a la confirmación.
1. Cuando reciba el mensaje de **Validación superada**, seleccione **Crear** para crear el grupo host.

Solo tardará unos minutos en crear el grupo host.


## <a name="create-a-dedicated-host"></a>Creación de un host dedicado

Ahora crearemos un host dedicado en el grupo host. Además de un nombre para el host, se le pedirá que proporcione el SKU del host. El SKU del host registra la serie de máquinas virtuales admitidas, así como la generación de hardware del host dedicado.

Para más información sobre los precios y los SKU de host, consulte [Precios de hosts dedicados de Azure](https://aka.ms/ADHPricing).

Si establece un número de dominios de error para el grupo host, se le pedirá que especifique el dominio de error para su host.

1. Seleccione **Crear un recurso** en la esquina superior izquierda.
1. Busque un **host dedicado** y, a continuación, seleccione los **hosts dedicados** en los resultados.
1. En la página de **hosts dedicados**, seleccione **Crear**.
1. Seleccione la suscripción que quiere usar.
1. Seleccione *myDedicatedHostsRG* como **grupo de recursos**.
1. En los **detalles de la instancia**, escriba *myHost* en el **nombre** y seleccione *Este de EE. UU.* como ubicación.
1. En **Perfil de hardware**, seleccione *Familia Standard Es3: tipo 1* para **Familia de tamaños**, *myHostGroup* para **Grupo host** y luego *1* para **Dominio de error**. En el resto de los campos, deje los valores predeterminados.
1. Cuando termine, seleccione **Revisar y crear** y, a continuación, espere a la confirmación.
1. Cuando reciba el mensaje de **Validación superada**, seleccione **Crear** para crear el host.

## <a name="create-a-vm"></a>Crear una VM

1. Elija **Crear un recurso** en la esquina superior izquierda de Azure Portal.
1. En el cuadro de búsqueda que está encima de la lista de recursos de Azure Marketplace, busque y seleccione la imagen que desea utilizar y, a continuación, elija **Crear**.
1. En la pestaña **Aspectos básicos**, en **Detalles del proyecto**, asegúrese de que esté seleccionada la suscripción correcta y luego seleccione *myDedicatedHostsRG* como **grupo de recursos**.
1. En **Detalles de instancia**, escriba *myVM* en **Nombre de máquina virtual** y elija *Este de EE. UU.* como **Ubicación**.
1. En **Opciones de disponibilidad**, seleccione **Zona de disponibilidad** y seleccione *1* en la lista desplegable.
1. En cuanto al tamaño, seleccione **Cambiar tamaño**. En la lista de tamaños disponibles, elija uno de la serie Esv3, como **Standard E2s v3**. Es posible que tenga que borrar el filtro para poder ver todos los tamaños disponibles.
1. Complete el resto de los campos de la pestaña **Básico** según sea necesario.
1. Si desea especificar qué host utilizar para la máquina virtual, en la parte superior de la página, seleccione la pestaña **Opciones avanzadas** y, en la sección **Host**, seleccione *myHostGroup* en **Grupo host** y *myHost* en **Host**. Si no lo hace, la máquina virtual se colocará automáticamente en un host con capacidad.
    ![Selección del grupo host y el host](./media/dedicated-hosts-portal/advanced.png)
1. Deje los valores predeterminados restantes y luego seleccione el botón **Revisar + crear** en la parte inferior de la página.
1. Cuando vea el mensaje en el que se indica que la validación se ha realizado correctamente, seleccione **Crear**.

La implementación de la máquina virtual tardará unos minutos.

## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado

Cuando se implementa un conjunto de escalado, se especifica el grupo host.

1. Busque *Conjunto de escalado* y seleccione **Conjuntos de escalado de máquinas virtuales** de la lista.
1. Seleccione **Agregar** para crear un nuevo conjunto de escalado.
1. Complete los campos de la pestaña **Básico** como lo haría normalmente, pero asegúrese de seleccionar un tamaño de máquina virtual que sea de la serie que eligió para el host dedicado, como **E2s v3 Estándar**.
1. En la pestaña **Avanzado**, para **Algoritmo de propagación**, seleccione **Propagación máxima**.
1. En **Grupo host**, seleccione el grupo host en el menú desplegable. Si ha creado el grupo recientemente, puede que tarde un minuto en agregarse a la lista.

## <a name="add-an-existing-vm"></a>Incorporación de una máquina virtual existente

Es posible incorporar una máquina virtual existente a un host dedicado, pero para ello antes es preciso detenerla o desasignarla. Antes de mover una máquina virtual a un host dedicado, asegúrese de que se admite su configuración:

- El tamaño de la máquina virtual debe estar en la misma familia de tamaños que el host dedicado. Por ejemplo, si el host dedicado es DSv3, el tamaño de la máquina virtual puede ser Standard_D4s_v3, pero no Standard_A4_v2.
- Es preciso que la máquina virtual se encuentre en la misma región que el host dedicado.
- La máquina virtual no puede formar parte de ningún grupo de ubicación por proximidad. Quite la máquina virtual del grupo de ubicación por proximidad antes de moverla a un host dedicado. Para más información, consulte [Traslado de una VM existente fuera de un grupo de selección de ubicación de proximidad](./windows/proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group)
- La máquina virtual no puede estar en un conjunto de disponibilidad.
- Si la máquina virtual está en una zona de disponibilidad, debe estar en la misma que el grupo host. Los valores de zona de disponibilidad de la máquina virtual y del grupo host deben coincidir.

Mueva la máquina virtual a un host dedicado desde el [portal](https://portal.azure.com).

1. Abra la página de la máquina virtual.
1. Seleccione **Stop** (Detener) para detener o desasignar la máquina virtual.
1. Seleccione **Configuration** (Configuración) en el menú de la izquierda.
1. Seleccione un grupo host y un host en los menús desplegables.
1. Cuando haya terminado, seleccione **Save** (Guardar) en la parte superior de la página.
1. Una vez que la máquina virtual se haya agregado al host, seleccione **Overview** (Información general) en el menú de la izquierda.
1. En la parte superior de la página, seleccione **Start** (Iniciar) para reiniciar la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más detalles, consulte la introducción a los [hosts dedicados](dedicated-hosts.md).

- [Aquí](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-dedicated-hosts/README.md) encontrará una plantilla de ejemplo en la que se usan zonas y dominios de error para obtener la máxima resistencia en una región.

- También puede implementar un host dedicado mediante la [CLI de Azure](./linux/dedicated-hosts-cli.md) o [PowerShell](./windows/dedicated-hosts-powershell.md).
