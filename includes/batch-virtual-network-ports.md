---
title: archivo de inclusión
description: archivo de inclusión
services: batch
documentationcenter: ''
author: JnHs
manager: evansma
editor: ''
ms.service: batch
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.date: 04/23/2021
ms.author: jenhayes
ms.custom: include file
ms.openlocfilehash: 3100e84255f95b2154f7e8d0ce47024546383da4
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107990479"
---
### <a name="general-requirements"></a>Requisitos generales

* La red virtual debe estar en la misma región y suscripción que la cuenta de Batch que se utiliza para crear el grupo.

* La subred especificada para el grupo debe tener suficientes direcciones IP sin asignar para acoger el número de VM destinadas al grupo; esto es, la suma de las propiedades `targetDedicatedNodes` y `targetLowPriorityNodes` del grupo. Si la subred no tiene suficientes direcciones IP sin asignar, el grupo asigna parcialmente los nodos de proceso y se produce un error de cambio de tamaño.

* Todos los servidores DNS personalizados que dan servicio a la red virtual deben resolver el punto de conexión de Azure Storage. En concreto, las direcciones URL del tipo `<account>.table.core.windows.net`, `<account>.queue.core.windows.net` y `<account>.blob.core.windows.net` deben poderse resolver.

* Se pueden crear varios grupos en la misma red virtual o en la misma subred (siempre que tenga suficiente espacio de direcciones). No puede existir un solo grupo en varias redes virtuales o subredes.

Los requisitos adicionales de red virtual difieren en función de si el grupo de Batch se encuentra en la configuración de máquina virtual o en la de servicios en la nube. Para las nuevas implementaciones de grupo en una red virtual, se recomienda la configuración de máquina virtual.

### <a name="pools-in-the-virtual-machine-configuration"></a>Grupos en la configuración de máquina virtual

**Redes virtuales admitidas**: solo redes virtuales basadas en Azure Resource Manager

**Id. de subred**: al especificar la subred mediante las API de Batch, utilice el *identificador de recurso* de la subred. El identificador de subred tiene el formato:

`/subscriptions/{subscription}/resourceGroups/{group}/providers/Microsoft.Network/virtualNetworks/{network}/subnets/{subnet}`

**Permisos**: compruebe si sus directivas de seguridad o bloqueos del grupo de recursos o la suscripción de la red virtual restringen los permisos de un usuario para administrar la red virtual.

**Recursos de red adicionales**: Batch crea automáticamente recursos de red adicionales en el grupo de recursos que contiene la red virtual.

> [!IMPORTANT]
> Por cada cien nodos dedicados o de prioridad baja, Batch crea: un grupo de seguridad de red, una dirección IP pública y un equilibrador de carga. Estos recursos están limitados por las [cuotas de recursos](../articles/azure-resource-manager/management/azure-subscription-service-limits.md) de la suscripción. En el caso de los grupos grandes, es posible que deba solicitar un aumento de la cuota de uno o varios de estos recursos.

#### <a name="network-security-groups-batch-default"></a>Grupos de seguridad de red: valor predeterminado de Batch

La subred debe permitir la comunicación de entrada desde el servicio Batch para poder programar tareas en los nodos de proceso y la de salida para comunicarse con Azure Storage u otros recursos, en función de lo que necesite la carga de trabajo. En cuanto a los grupos que hay en la configuración de la máquina virtual, Batch agrega grupos de seguridad de red a nivel de las interfaces de red (NIC) conectadas a los nodos de proceso. Estos grupos se configuran con las siguientes reglas adicionales:

* Tráfico TCP de entrada en los puertos 29876 y 29877 desde las direcciones IP del servicio Batch que se corresponden con la etiqueta de servicio `BatchNodeManagement`.
* Tráfico TCP de entrada en el puerto 22 (nodos de Linux) o el puerto 3389 (nodos de Windows) para permitir el acceso remoto. Para determinados tipos de tareas de instancias múltiples en Linux (como MPI), tendrá que permitir también el tráfico del puerto 22 de SSH para las direcciones IP en la subred que contiene los nodos de ejecución de Batch. Se puede bloquear por regla de grupo de seguridad de red del nivel de subred (véase a continuación).
* Tráfico saliente en cualquier puerto para la red virtual. Se puede modificar por regla de grupo de seguridad de red del nivel de subred (véase a continuación).
* Tráfico saliente en cualquier puerto para Internet. Se puede modificar por regla de grupo de seguridad de red del nivel de subred (véase a continuación).

> [!IMPORTANT]
> Tenga cuidado si modifica reglas de entrada o salida en los grupos de seguridad de red configurados para Batch o las agrega a ellos. Si un grupo de seguridad de red deniega la comunicación con los nodos de ejecución en la subred especificada, el servicio Batch establecerá el estado de dichos nodos en **No utilizable**. Además, no se deben aplicar bloqueos de recursos a ningún recurso creado por Batch, ya que así se puede impedir la limpieza de recursos como consecuencia de las acciones iniciadas por el usuario, como la eliminación de un grupo.

#### <a name="network-security-groups-specifying-subnet-level-rules"></a>Grupos de seguridad de red: especificación de grupos de nivel de subred

No es preciso que especifique grupos de seguridad de red en el nivel de subred de la red virtual, ya que Batch configura los suyos propios (véase a continuación). Si tiene un grupo de seguridad de red asociado a la subred en la que se implementan los nodos de ejecución de Batch o si desea aplicar reglas de NSG personalizadas que reemplacen a las reglas aplicadas de forma predeterminada, debe configurar el grupo de seguridad de red, como mínimo, con las reglas de seguridad de entrada y de salida que se muestran en las tablas siguientes.

Configure el tráfico de entrada en los puertos 3389 (Windows) o 22 (Linux) solo si necesita permitir el acceso remoto a los nodos de ejecución del grupo desde orígenes externos. Si requiere que las tareas con varias instancias sean compatibles con ciertos runtimes de MPI, es posible que necesite habilitar las reglas del puerto 22 en Linux. Para que se puedan usar los nodos de ejecución del grupo no es obligatorio permitir el tráfico en estos puertos.

> [!WARNING]
> Las direcciones IP del servicio Batch pueden cambiar con el tiempo. Por lo tanto, se recomienda encarecidamente usar la etiqueta de servicio `BatchNodeManagement` (o una variante regional) para las reglas de NSG que se indican en las tablas siguientes. Evite rellenar las reglas de NSG con direcciones IP específicas del servicio Batch.

**Reglas de seguridad de entrada**

| Direcciones IP de origen | Etiqueta de servicio de origen | Puertos de origen | Destination | Puertos de destino | Protocolo | Acción |
| --- | --- | --- | --- | --- | --- | --- |
| N/D | [Etiqueta de servicio](../articles/virtual-network/network-security-groups-overview.md#service-tags) `BatchNodeManagement` (si usa una variante regional, en la misma región que su cuenta de Batch) | * | Any | 29876-29877 | TCP | Allow |
| Las direcciones IP de origen de usuario para tener acceso remoto a los nodos de ejecución o la subred de nodo de ejecución para tareas de instancias múltiples de Linux, si es necesario. | N/D | * | Any | 3389 (Windows), 22 (Linux) | TCP | Allow |

**Reglas de seguridad de salida**

| Source | Puertos de origen | Destination | Etiqueta de servicio de destino | Puertos de destino | Protocolo | Acción |
| --- | --- | --- | --- | --- | --- | --- |
| Any | * | [Etiqueta de servicio](../articles/virtual-network/network-security-groups-overview.md#service-tags) | `Storage` (si se usa una variante regional, en la misma región que la cuenta de Batch) | 443 | TCP | Allow |
| Any | * | [Etiqueta de servicio](../articles/virtual-network/network-security-groups-overview.md#service-tags) | `BatchNodeManagement` (si se usa una variante regional, en la misma región que la cuenta de Batch) | 443 | TCP | Allow |

La salida a `BatchNodeManagement` es necesaria para ponerse en contacto con el servicio Batch desde los nodos de ejecución como para las tareas del Administrador de trabajos.

### <a name="pools-in-the-cloud-services-configuration"></a>Grupos de la configuración de servicios en la nube

> [!WARNING]
> Los grupos de configuración de Cloud Services están [en desuso](https://azure.microsoft.com/updates/azure-batch-cloudserviceconfiguration-pools-will-be-retired-on-29-february-2024/). En su lugar, utilice los grupos de configuración de máquina virtual.

**Redes virtuales admitidas**: solo redes virtuales clásicas

**Id. de subred**: al especificar la subred mediante las API de Batch, utilice el *identificador de recurso* de la subred. El identificador de subred tiene el formato:

`/subscriptions/{subscription}/resourceGroups/{group}/providers/Microsoft.ClassicNetwork /virtualNetworks/{network}/subnets/{subnet}`

**Permisos**: la entidad de servicio `Microsoft Azure Batch` debe tener el rol `Classic Virtual Machine Contributor` de Azure para la red virtual especificada.

#### <a name="network-security-groups"></a>Grupos de seguridad de red

La subred debe permitir las comunicaciones entrantes desde el servicio Batch para poder programar tareas en los nodos de proceso y la comunicación saliente para comunicarse con Azure Storage u otros recursos.

No es necesario especificar un grupo de seguridad de red, ya que Batch configura la comunicación de entrada únicamente de las direcciones IP de Batch a los nodos del grupo. Sin embargo, si la subred especificada tiene asociados grupos de seguridad de red o un firewall, configure las reglas de seguridad de entrada y salida como se muestra en las tablas siguientes. Si un grupo de seguridad de red deniega la comunicación con los nodos de ejecución en la subred especificada, el servicio Batch establece el estado de dichos nodos en **No utilizable**.

Configure el tráfico entrante en el puerto 3389 para Windows si tiene que permitir el RDP a los nodos del grupo. Esta acción no se requiere para que se puedan usar los nodos del grupo.

**Reglas de seguridad de entrada**

| Direcciones IP de origen | Puertos de origen | Destination | Puertos de destino | Protocolo | Acción |
| --- | --- | --- | --- | --- | --- |
Any <br /><br />Aunque esto requiere "permitir todo" realmente, el servicio Batch aplica una regla de lista de control de acceso a nivel de cada nodo que filtra todas las direcciones IP de servicio que no sean de Batch. | * | Any | 10100, 20100, 30100 | TCP | Allow |
| Opcional, para permitir el acceso RDP en nodos de ejecución. | * | Any | 3389 | TCP | Allow |

**Reglas de seguridad de salida**

| Source | Puertos de origen | Destination | Puertos de destino | Protocolo | Acción |
| --- | --- | --- | --- | --- | --- |
| Any | * | Any | 443  | Any | Allow |
