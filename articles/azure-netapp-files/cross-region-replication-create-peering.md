---
title: Creación de una replicación de volúmenes para Azure NetApp Files | Microsoft Docs
description: Se describe cómo crear el emparejamiento de replicación de volumen para Azure NetApp Files, a fin de configurar la replicación entre regiones.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/10/2021
ms.author: b-juche
ms.openlocfilehash: 84b10fefbfae4ca42fa96cc96f357dbfe0d25800
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108794779"
---
# <a name="create-volume-replication-for-azure-netapp-files"></a>Creación de una replicación de volúmenes para Azure NetApp Files

> [!IMPORTANT]
> La característica de replicación entre regiones se encuentra actualmente en versión preliminar. Debe enviar una solicitud de lista de espera para acceder a la característica a través de la [página de envío de lista de espera de replicación entre regiones de Azure NetApp Files](https://aka.ms/anfcrrpreviewsignup). Antes de utilizar la característica de replicación entre regiones, debe esperar un correo electrónico de confirmación oficial del equipo de Azure NetApp Files.

En este artículo se muestra cómo configurar la replicación entre regiones mediante la creación del emparejamiento de replicación. 

La configuración del emparejamiento de replicación le permite replicar datos de forma asincrónica desde un volumen de Azure NetApp Files (origen) a otro volumen Azure NetApp Files (destino). El volumen de origen y el de destino deben implementarse en regiones independientes. El nivel de servicio del grupo de capacidad de destino puede coincidir con el del grupo de capacidad de origen, o bien puede seleccionar un nivel de servicio diferente.   

La replicación de Azure NetApp Files no admite actualmente varias suscripciones; todas las replicaciones deben realizarse en una sola suscripción.

Antes de comenzar, asegúrese de revisar los [requisitos y las consideraciones sobre el uso de la replicación entre regiones](cross-region-replication-requirements-considerations.md).  

## <a name="locate-the-source-volume-resource-id"></a>Búsqueda del identificador de recurso del volumen de origen  

Debe obtener el identificador de recurso del volumen de origen que desea replicar. 

1. Vaya al volumen de origen y seleccione **Propiedades** en Configuración para mostrar el identificador de recurso del volumen de origen.   
    ![Búsqueda del identificador de recurso del volumen de origen](../media/azure-netapp-files/cross-region-replication-source-volume-resource-id.png)
 
2. Copie el identificador de recurso en el Portapapeles.  Lo necesitará más adelante.

## <a name="create-the-data-replication-volume-the-destination-volume"></a>Creación del volumen de replicación de datos (volumen de destino)

Debe crear un volumen de destino en el que desee que se repliquen los datos del volumen de origen.  Para poder crear un volumen de destino, debe tener una cuenta de NetApp y un grupo de capacidad en la región de destino. 

1. La cuenta de destino debe estar en una región distinta de la región del volumen de origen. Si es necesario, cree una cuenta de NetApp en la región de Azure que se usará para la replicación siguiendo los pasos descritos en [Creación de una cuenta de NetApp](azure-netapp-files-create-netapp-account.md).   
También puede seleccionar una cuenta de NetApp existente en otra región.  

2. Si es necesario, cree un grupo de capacidad en la cuenta de NetApp recién creada siguiendo los pasos descritos en [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md).   

    También puede seleccionar un grupo de capacidad existente para hospedar el volumen de destino de replicación.  

    El nivel de servicio del grupo de capacidad de destino puede coincidir con el del grupo de capacidad de origen, o bien puede seleccionar un nivel de servicio diferente.

3. Delegue una subred en la región que se va a usar para la replicación siguiendo los pasos descritos en [Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

4. Cree el volumen de replicación de datos seleccionando **Volúmenes** en Servicio de almacenamiento en la cuenta de NetApp de destino. Después, haga clic en el botón **+ Add data replication** (+ Agregar replicación de datos).  

    ![Agregar replicación de datos](../media/azure-netapp-files/cross-region-replication-add-data-replication.png)
 
5. En la página Crear un volumen que aparece, complete los siguientes campos en la pestaña **Aspectos básicos**:
    * Nombre del volumen
    * Grupo de capacidad
    * Cuota de volumen
        > [!NOTE] 
        > La cuota de volumen (tamaño) del volumen de destino debe reflejar la del volumen de origen. Si especifica un tamaño menor que el volumen de origen, el tamaño del volumen de destino cambia automáticamente al tamaño del volumen de origen. 
    * Virtual network 
    * Subnet

    Para obtener más información sobre los campos, vea [Creación de un volumen NFS](azure-netapp-files-create-volumes.md#create-an-nfs-volume). 

6. En la pestaña **Protocolo**, seleccione el mismo protocolo que el volumen de origen.  
En el caso del protocolo NFS, asegúrese de que las reglas de la directiva de exportación cumplan los requisitos de los hosts de la red remota que tendrán acceso a la exportación.  

7. En la pestaña **Etiquetas**, cree pares clave-valor según sea necesario.  

8. En la pestaña **Replicación**, pegue el identificador de recurso del volumen de origen que obtuvo en [Búsqueda del identificador de recurso del volumen de origen](#locate-the-source-volume-resource-id) y, después, seleccione la programación de replicación deseada. Entre las opciones de programación de replicación se incluyen: cada 10 minutos, cada hora, diariamente, semanalmente y mensualmente.  

    ![Creación de replicación de volumen](../media/azure-netapp-files/cross-region-replication-create-volume-replication.png)

9. Haga clic en **Revisar y crear** y, después, haga clic en **Crear** para crear el volumen de replicación de datos.   

    ![Revisión y creación de la replicación](../media/azure-netapp-files/cross-region-replication-review-create-replication.png)

## <a name="authorize-replication-from-the-source-volume"></a>Autorización de la replicación desde el volumen de origen  

Para autorizar la replicación, debe obtener el identificador de recurso del volumen de destino de replicación y pegarlo en el campo Autorizar del volumen de origen de replicación. 

1. En Azure Portal, vaya a Azure NetApp Files.

2. Vaya a la cuenta de NetApp de destino y al grupo de capacidad de destino en el que se encuentra el volumen de destino de replicación.

3. Seleccione el volumen de destino de replicación, vaya a **Propiedades** en Configuración y busque el **identificador de recurso** del volumen de destino. Copie el identificador de recurso del volumen de destino en el Portapapeles.

    ![Identificador de recurso de propiedades](../media/azure-netapp-files/cross-region-replication-properties-resource-id.png) 
 
4. En Azure NetApp Files, vaya a la cuenta de origen de replicación y al grupo de capacidad de origen. 

5. Busque el volumen de origen de replicación y selecciónelo. Vaya a **Replicación** en Servicio de almacenamiento y haga clic en **Autorizar**.

    ![Autorizar replicación](../media/azure-netapp-files/cross-region-replication-authorize.png) 

6. En el campo Autorizar, pegue el identificador de recurso del volumen de replicación de destino que obtuvo en el paso 3 y, después, haga clic en **Aceptar**.

## <a name="next-steps"></a>Pasos siguientes  

* [Replicación entre regiones](cross-region-replication-introduction.md)
* [Requisitos y consideraciones del uso de la replicación entre regiones](cross-region-replication-requirements-considerations.md)
* [Visualización del estado de mantenimiento de la relación de la replicación](cross-region-replication-display-health-status.md)
* [Métricas de replicación de volúmenes](azure-netapp-files-metrics.md#replication)
* [Administración de la recuperación ante desastres](cross-region-replication-manage-disaster-recovery.md)
* [Eliminación de volúmenes o replicaciones de volúmenes](cross-region-replication-delete.md)
* [Solución de problemas de replicación entre regiones](troubleshoot-cross-region-replication.md)

