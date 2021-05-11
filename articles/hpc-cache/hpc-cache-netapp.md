---
title: Uso de Azure HPC Cache y Azure NetApp Files
description: Cómo usar Azure HPC Cache para mejorar el acceso a los datos almacenados con Azure NetApp Files
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 04/26/2021
ms.author: v-erkel
ms.openlocfilehash: 1358f3cfebcd5cdc6d9e402e9c487f80f1aa8bcd
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108322031"
---
# <a name="use-azure-hpc-cache-with-azure-netapp-files"></a>Uso de Azure HPC Cache con Azure NetApp Files

Puede usar [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) como destino de almacenamiento para su instancia de Azure HPC Cache. En este artículo se explica cómo los dos servicios pueden trabajar juntos y se ofrecen sugerencias para configurarlos.

Azure NetApp Files combina su sistema operativo ONTAP con la escalabilidad y la velocidad de Microsoft Azure. Esta combinación permite a los usuarios cambiar los flujos de trabajo establecidos a la nube sin necesidad de volver a escribir el código.

Agregar un componente de Azure HPC Cache puede mejorar el acceso a los archivos presentando varios volúmenes de Azure NetApp Files en un espacio de nombres agregado. Puede proporcionar almacenamiento en caché perimetral para los volúmenes ubicados en una región de servicio diferente. También puede mejorar el rendimiento a petición de los volúmenes que se crearon en niveles de servicio de nivel inferior para ahorrar costos.

## <a name="overview"></a>Información general

Para usar un sistema de Azure NetApp Files como almacenamiento de back-end con Azure HPC Cache, siga este proceso.

1. Cree los volúmenes y el sistema Azure NetApp Files según las instrucciones que se indican en [Planeación del sistema](#plan-your-azure-netapp-files-system), a continuación.
1. Cree la instancia de Azure HPC Cache en la región donde necesite acceso a archivos. (Siga las instrucciones existentes en [Creación de una instancia de Azure HPC Cache](hpc-cache-create.md)).
1. [Defina los destinos de almacenamiento](#create-storage-targets-in-the-cache) en caché que apuntan a los volúmenes de Azure NetApp Files. Cree un destino de almacenamiento en caché para cada dirección IP única utilizada para acceder a los volúmenes.
1. Haga que los clientes [monten Azure HPC Cache](#mount-storage-targets) en lugar de montar los volúmenes de Azure NetApp Files directamente.

## <a name="plan-your-azure-netapp-files-system"></a>Planeación del sistema Azure NetApp Files

Al planear el sistema Azure NetApp Files, preste atención a los elementos de esta sección para asegurarse de que puede integrarla sin problemas con Azure HPC Cache.

Lea también la [documentación de Azure NetApp Files](../azure-netapp-files/index.yml) antes de crear volúmenes para usarlos con Azure HPC Cache.

### <a name="nfs-client-access-only"></a>Acceso solo a clientes NFS

Azure HPC Cache actualmente solo admite el acceso a NFS. No se puede usar con la ACL SMB o volúmenes de bits de modo POSIX.

### <a name="exclusive-subnet-for-azure-netapp-files"></a>Subred exclusiva para Azure NetApp Files

Azure NetApp Files usa una única subred delegada para sus volúmenes. Ningún otro recurso puede usar esa subred. Además, solo se puede usar una subred de una red virtual para Azure NetApp Files. Obtenga más información en [Instrucciones para el planeamiento de red de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-network-topologies.md).

### <a name="delegated-subnet-size"></a>Tamaño de la subred delegada

Use el tamaño mínimo para la subred delegada al crear un sistema Azure NetApp Files para su uso con Azure HPC Cache.

El tamaño mínimo, que se especifica con la máscara de red /28, proporciona 16 direcciones IP. En la práctica, Azure NetApp Files usa solo tres de las direcciones IP disponibles para el acceso al volumen. Esto significa que solo tiene que crear tres destinos de almacenamiento en Azure HPC Cache para cubrir todos los volúmenes.

Si la subred delegada es demasiado grande, es posible que los volúmenes de Azure NetApp Files usen más direcciones IP de las que una única instancia de Azure HPC Cache puede controlar. Una sola caché tiene un límite de 10 destinos de almacenamiento para la mayoría de los tamaños de rendimiento de caché, o 20 destinos de almacenamiento para las configuraciones más grandes.

En el ejemplo de inicio rápido de la documentación de Azure NetApp Files se usa 10.7.0.0/16 para la subred delegada, que proporciona una subred que es demasiado grande.

### <a name="capacity-pool-service-level"></a>Nivel de servicio del grupo de capacidad

Al elegir el [nivel de servicio](../azure-netapp-files/azure-netapp-files-service-levels.md) para el grupo de capacidad, tenga en cuenta el flujo de trabajo. Si vuelve a escribe datos con frecuencia en el volumen de Azure NetApp Files, el rendimiento de la memoria caché se puede restringir si el tiempo de escritura diferida es lento. Elija un nivel de servicio alto para los volúmenes que tendrán escrituras frecuentes.

Los volúmenes con niveles de servicio bajos también pueden mostrar algún retraso al inicio de una tarea mientras la memoria caché rellena previamente el contenido. Después de que la memoria caché esté en funcionamiento con un buen espacio de trabajo de archivos, el retraso debería pasar a ser inapreciable.

Es importante planear el nivel de servicio del grupo de capacidad con anterioridad, ya que no se puede cambiar después de la creación. Sería necesario crear un nuevo volumen en un grupo de capacidad diferente y copiar los datos.

Tenga en cuenta que puede cambiar la cuota de almacenamiento de un volumen y el tamaño del grupo de capacidad sin interrumpir el acceso.

## <a name="create-storage-targets-in-the-cache"></a>Creación de destinos de almacenamiento en la memoria caché

Una vez que se haya configurado el sistema Azure NetApp Files y se haya creado la instancia de Azure HPC Cache, defina los destinos de almacenamiento en la memoria caché que apuntan a los volúmenes del sistema de archivos.

Cree un destino de almacenamiento para cada dirección IP utilizada por los volúmenes de Azure NetApp Files. La dirección IP se muestra en la página de instrucciones de montaje del volumen.

Si hay varios volúmenes que comparten la misma dirección IP, puede usar un destino de almacenamiento para todos ellos.  

Siga las [instrucciones de montaje de la documentación de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md) para buscar las direcciones IP que se van a utilizar.

También puede encontrar direcciones IP con la CLI de Azure:

```azurecli
az netappfiles volume list -g ${RESOURCE_GROUP} --account-name ${ANF_ACCOUNT} --pool-name ${POOL} --query "[].mountTargets[].ipAddress" | grep -Ee '[0-9]+[.][0-9]+[.][0-9]+[.][0-9]+' | tr -d '"' | tr -d , | sort | uniq
```

Los nombres de exportación del sistema Azure NetApp Files tienen un único componente de ruta de acceso. No intente crear un destino de almacenamiento para la exportación raíz ``/`` en Azure NetApp Files, porque esa exportación no proporciona acceso a archivos.

No hay restricciones especiales en las rutas de acceso del espacio de nombres virtual para estos destinos de almacenamiento.

## <a name="mount-storage-targets"></a>Montaje de los destinos de almacenamiento

Las máquinas cliente deben montar la memoria caché en lugar de montar los volúmenes de Azure NetApp Files directamente. Siga las instrucciones de [Montaje de la instancia de Azure HPC Cache](hpc-cache-mount.md).

## <a name="next-steps"></a>Pasos siguientes

* Obtener más información sobre la configuración y el uso de [Azure NetApp Files](../azure-netapp-files/index.yml)
* Para ayudar a planear y configurar el sistema Azure HPC Cache para usar Azure NetApp Files, [póngase en contacto con el soporte técnico](hpc-cache-support-ticket.md).
