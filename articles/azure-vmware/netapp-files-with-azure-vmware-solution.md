---
title: Integración de Azure NetApp Files con Azure VMware Solution
description: Use Azure NetApp Files con máquinas virtuales de Azure VMware Solution para migrar y sincronizar datos entre servidores locales, máquinas virtuales de Azure VMware Solution e infraestructuras en la nube.
ms.topic: how-to
ms.date: 06/08/2021
ms.openlocfilehash: 3383ee2afe271fbf50def125bd1fd4366b8b6165
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111755724"
---
# <a name="integrate-azure-netapp-files-with-azure-vmware-solution"></a>Integración de Azure NetApp Files con Azure VMware Solution

[Azure NetApp Files](../azure-netapp-files/azure-netapp-files-introduction.md) es un servicio de Azure para la migración y ejecución de las cargas de trabajo de archivos empresariales más exigentes en la nube: bases de datos, SAP y aplicaciones informáticas de alto rendimiento, sin cambios de código. En este artículo, configurará, probará y comprobará el volumen de Azure NetApp Files como un recurso compartido de archivos para cargas de trabajo de Azure VMware Solution mediante el protocolo Network File System (NFS). El sistema operativo invitado se ejecuta en las máquinas virtuales (VM) que accedan a volúmenes de Azure NetApp Files. 

Azure NetApp Files y Azure VMware Solution se crean en la misma región de Azure. Azure NetApp Files está disponible en muchas [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp,azure-vmware&regions=all) y admite la replicación entre regiones. Para información sobre los métodos de configuración de Azure NetApp Files, consulte [Jerarquía de almacenamiento de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-understand-storage-hierarchy.md).

Servicios en los que se usa Azure NetApp Files:

- **Conexiones de Active Directory**: Azure NetApp Files admite [Active Directory Domain Services y Azure Active Directory Domain Services](../azure-netapp-files/create-active-directory-connections.md#decide-which-domain-services-to-use).

- **Protocolo de recurso compartido**: Azure NetApp Files admite los protocolos Bloque de mensajes del servidor (SMB) y Network File System (NFS). Esta compatibilidad significa que los volúmenes se pueden montar en el cliente Linux y se pueden asignar en el cliente Windows.

- **Azure VMware Solution**: Los recursos compartidos de Azure NetApp Files se pueden montar desde las máquinas virtuales que se crean en el entorno de Azure VMware Solution.


En el diagrama se muestra una conexión a través de Azure ExpressRoute a una nube privada de Azure VMware Solution. El entorno de Azure VMware Solution accede al recurso compartido de Azure NetApp Files montado en las VM de Azure VMware Solution.

![Diagrama que muestra NetApp Files para la arquitectura de Azure VMware Solution.](media/net-app-files/net-app-files-topology.png)


## <a name="prerequisites"></a>Requisitos previos 

> [!div class="checklist"]
> * Suscripción a Azure con Azure NetApp Files habilitado
> * Subred para Azure NetApp Files
> * Máquina virtual Linux en Azure VMware Solution
> * Máquinas virtuales Windows en Azure VMware Solution


## <a name="create-and-mount-azure-netapp-files-volumes"></a>Creación y montaje de volúmenes de Azure NetApp Files

Creará y montará volúmenes de Azure NetApp Files en máquinas virtuales de Azure VMware Solution.

1. [Creación de una cuenta de NetApp](../azure-netapp-files/azure-netapp-files-create-netapp-account.md).

1. [Configuración de un grupo de capacidad](../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).

1. [Creación de un volumen SMB para Azure NetApp Files](../azure-netapp-files/azure-netapp-files-create-volumes-smb.md).

1. [Creación de un volumen de NFS para Azure NetApp Files](../azure-netapp-files/azure-netapp-files-create-volumes.md).

1. [Delegación de una subred en Azure NetApp Files](../azure-netapp-files/azure-netapp-files-delegate-subnet.md).


## <a name="verify-pre-configured-azure-netapp-files"></a>Comprobación del servicio Azure NetApp Files preconfigurado 

Comprobará el servicio Azure NetApp Files preconfigurado creado en Azure en el nivel de servicio Prémium de Azure NetApp Files.

1. En Azure Portal, en **ALMACENAMIENTO**, seleccione **Azure NetApp Files**. Se mostrará una lista de instancias configuradas de Azure NetApp Files. 

   :::image type="content" source="media/net-app-files/azure-net-app-files-list.png" alt-text="Captura de pantalla que muestra la lista del servicio Azure NetApp Files preconfigurado."::: 

2. Seleccione una cuenta de NetApp Files configurada para ver su configuración. En este ejemplo, seleccione **Contoso-anf2**. 

3. Seleccione **Grupos de capacidad** para comprobar el grupo configurado. 

   :::image type="content" source="media/net-app-files/net-app-settings.png" alt-text="Captura de pantalla que muestra opciones para ver los grupos de capacidad y los volúmenes de una cuenta de NetApp Files configurada.":::

   Se abre la página de grupos de capacidad que muestra la capacidad y el nivel de servicio. En este ejemplo, el grupo de almacenamiento se configura como de 4 TiB con un nivel de servicio Prémium.

4. Seleccione **Volúmenes** para ver los volúmenes creados en el grupo de capacidad. (Consulte la captura de pantalla anterior).

5. Seleccione un volumen para ver su configuración.  

   :::image type="content" source="media/net-app-files/azure-net-app-volumes.png" alt-text="Captura de pantalla que muestra los volúmenes creados en el grupo de capacidad.":::

   Se abre una ventana que muestra los detalles de configuración del volumen.

   :::image type="content" source="media/net-app-files/configuration-of-volume.png" alt-text="Captura de pantalla que muestra los detalles de configuración de un volumen.":::

   Puede notar que el volumen anfvolume tiene un tamaño de 200 GiB y se encuentra en el grupo de capacidad anfpool1. Se exporta como un recurso compartido de archivos NFS a través de 10.22.3.4:/ANFVOLUME. Se creó una IP privada de Azure Virtual Network (VNet) para Azure NetApp Files y la ruta de acceso de NFS para montar en la máquina virtual.

   Para más información acerca del rendimiento de los volúmenes de Azure NetApp Files por su tamaño o "cuota", consulte [Consideraciones sobre el rendimiento de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-performance-considerations.md). 

## <a name="verify-pre-configured-azure-vmware-solution-vm-share-mapping"></a>Comprobación de la asignación preconfigurada de recursos compartidos de máquina virtual de Azure VMware Solution

Para que una VM de Azure VMware Solution pueda acceder a un recurso compartido de Azure NetApp Files, será necesario que comprenda la asignación de recursos compartidos SMB y NFS. Solo después de configurar los volúmenes SMB o NFS, se pueden montar como se documenta aquí.

- **Recurso compartido SMB:** cree una conexión de Active Directory antes de implementar un volumen SMB. Para que la conexión tenga éxito, la subred delegada de Azure NetApp Files debe tener acceso a los controladores de dominio especificados. Una vez que se haya configurado Active Directory dentro de la cuenta de Azure NetApp Files, aparecerá como un elemento seleccionable al crear volúmenes SMB.

- **Recurso compartido NFS:** Azure NetApp Files colabora en la creación de volúmenes con NFS o un protocolo dual (NFS y SMB). El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo. NFS se puede montar en el servidor Linux mediante las líneas de comandos o las entradas /etc/fstab.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha leído sobre la integración de Azure NetApp Files con las cargas de trabajo de Azure VMware Solution, puede obtener más información acerca de:

- [Limitaciones de recursos para Azure NetApp Files](../azure-netapp-files/azure-netapp-files-resource-limits.md#resource-limits).
- [Instrucciones para el planeamiento de red de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-network-topologies.md)
- [Replicación entre regiones de volúmenes de Azure NetApp Files](../azure-netapp-files/cross-region-replication-introduction.md) 
- [Preguntas más frecuentes acerca de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-faqs.md)
