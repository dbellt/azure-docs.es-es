---
title: Preguntas más frecuentes acerca de Azure NetApp Files | Microsoft Docs
description: Revise las preguntas más frecuentes sobre Azure NetApp Files, como redes, seguridad, rendimiento, administración de capacidad y migración o protección de datos.
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
ms.topic: conceptual
ms.date: 04/06/2021
ms.author: b-juche
ms.openlocfilehash: d63587eec1f7e6d24ae1638e8365b85fd1ec2c94
ms.sourcegitcommit: c2a41648315a95aa6340e67e600a52801af69ec7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106504998"
---
# <a name="faqs-about-azure-netapp-files"></a>Preguntas más frecuentes acerca de Azure NetApp Files

En este artículo se responden algunas preguntas frecuentes sobre Azure NetApp Files. 

## <a name="networking-faqs"></a>Preguntas frecuentes sobre redes

### <a name="does-the-nfs-data-path-go-over-the-internet"></a>¿La ruta de acceso de datos de NFS pasa a través de Internet?  

No. La ruta de acceso de NFS no pasa por Internet. Azure NetApp Files es un servicio nativo de Azure que está implementado en Azure Virtual Network (VNet) en donde el servicio está disponible. Azure NetApp Files usa una subred delegada y aprovisiona una interfaz de red directamente en la red virtual (VNet). 

Consulte [Directrices para el planeamiento de red de Azure NetApp Files](./azure-netapp-files-network-topologies.md) para una información más detallada.  

### <a name="can-i-connect-a-vnet-that-i-already-created-to-the-azure-netapp-files-service"></a>¿Puedo conectar una red virtual que ya haya creado con el servicio Azure NetApp Files?

Sí, puede conectar redes virtuales que haya creado al servicio. 

Consulte [Directrices para el planeamiento de red de Azure NetApp Files](./azure-netapp-files-network-topologies.md) para una información más detallada.  

### <a name="can-i-mount-an-nfs-volume-of-azure-netapp-files-using-dns-fqdn-name"></a>¿Puedo montar un volumen NFS de Azure NetApp Files mediante el nombre FQDN de DNS?

Sí que puede, si crea las entradas DNS necesarias. Azure NetApp Files proporciona la dirección IP de servicio para el volumen aprovisionado. 

> [!NOTE] 
> Azure NetApp Files puede implementar direcciones IP adicionales para el servicio según sea necesario.  Puede que sea necesario actualizar las entradas DNS periódicamente.

### <a name="can-i-set-or-select-my-own-ip-address-for-an-azure-netapp-files-volume"></a>¿Puedo establecer o seleccionar mi propia dirección IP para un volumen de Azure NetApp Files?  

No. La asignación de direcciones IP a volúmenes de Azure NetApp Files es dinámica. No se admiten las asignaciones de IP estáticas. 

### <a name="does-azure-netapp-files-support-dual-stack-ipv4-and-ipv6-vnet"></a>¿Admite Azure NetApp Files la red virtual de pila dual (IPv4 e IPv6)?

No, actualmente Azure NetApp Files no admite la red virtual de pila dual (IPv4 e IPv6).  
 
## <a name="security-faqs"></a>Preguntas más frecuentes de seguridad

### <a name="can-the-network-traffic-between-the-azure-vm-and-the-storage-be-encrypted"></a>¿Se puede cifrar el tráfico de red entre la máquina virtual de Azure y el almacenamiento?

El tráfico de datos entre los clientes de NFSv 4.1 y los volúmenes de Azure NetApp Files se puede cifrar mediante Kerberos con el cifrado AES-256. Para más información, consulte [Configuración del cifrado Kerberos de NFSv4.1 para Azure NetApp Files](configure-kerberos-encryption.md).   

El tráfico de datos entre los clientes de NFSv3 o SMB3 y los volúmenes de Azure NetApp Files no se cifra. Sin embargo, el tráfico desde una máquina virtual de Azure (que ejecuta un cliente SMB o NFS) a Azure NetApp Files es tan seguro como cualquier otro tráfico de máquina virtual a máquina virtual de Azure. Se trata de tráfico local a la red del centro de datos de Azure. 

### <a name="can-the-storage-be-encrypted-at-rest"></a>¿Se puede cifrar el almacenamiento en reposo?

Todos los volúmenes de Azure NetApp Files se cifran mediante el estándar FIPS 140-2. El servicio Azure NetApp Files administra todas las claves. 

### <a name="how-are-encryption-keys-managed"></a>¿Cómo se administran las claves de cifrado? 

La administración de claves para Azure NetApp Files se controla mediante el servicio. Se genera una clave de cifrado de datos XTS-AES-256 única para cada volumen. Una jerarquía de claves de cifrado se usa para cifrar y proteger todas las claves de volumen. Estas claves de cifrado nunca se muestran ni se notifican en un formato sin cifrar. Las claves de cifrado se eliminan inmediatamente cuando se suprime un volumen.

La compatibilidad con las claves administradas por el cliente (Bring Your Own Key) mediante Azure Dedicated HSM está disponible de manera controlada en las regiones Este de EE. UU., Centro-sur de EE. UU., Oeste de EE. UU 2. y Centro y US Gov Virginia. Puede solicitar acceso en [anffeedback@microsoft.com](mailto:anffeedback@microsoft.com) . Las solicitudes se aprueban a medida que la capacidad va estando disponible.

### <a name="can-i-configure-the-nfs-export-policy-rules-to-control-access-to-the-azure-netapp-files-service-mount-target"></a>¿Puedo configurar las reglas de directivas de exportación NFS para controlar el acceso al destino de montaje del servicio Azure NetApp Files?

Sí, puede configurar hasta cinco reglas en una sola directiva de exportación NFS.

### <a name="does-azure-netapp-files-support-network-security-groups"></a>¿Azure NetApp Files admite grupos de seguridad de red?

No, actualmente no se pueden aplicar grupos de seguridad de red a la subred delegada de Azure NetApp Files o las interfaces de red creadas por el servicio.

### <a name="can-i-use-azure-rbac-with-azure-netapp-files"></a>¿Puedo usar RBAC de Azure con Azure NetApp Files?

Sí, Azure NetApp Files admite las características de RBAC de Azure. Junto con los roles de Azure integrados, puede [crear roles personalizados](../role-based-access-control/custom-roles.md) para Azure NetApp Files. 

Para obtener una lista completa de los permisos de Azure NetApp Files, consulte las operaciones del proveedor de recursos de Azure para [`Microsoft.NetApp`](../role-based-access-control/resource-provider-operations.md#microsoftnetapp).

### <a name="are-azure-activity-logs-supported-on-azure-netapp-files"></a>¿Se admiten los registros de actividad de Azure en Azure NetApp Files?

Azure NetApp Files es un servicio nativo de Azure. Todas las API PUT, POST y DELETE se registran en Azure NetApp Files. Por ejemplo, los registros muestran actividades como quién creó la instantánea, quién modificó el volumen, etc.

Para obtener una lista completa de las operaciones de API, consulte [API REST de Azure NetApp Files](/rest/api/netapp/).

### <a name="can-i-use-azure-policies-with-azure-netapp-files"></a>¿Puedo usar directivas de Azure con Azure NetApp Files?

Sí, puede crear [directivas de Azure personalizadas](../governance/policy/tutorials/create-custom-policy-definition.md). 

Sin embargo, no puede crear directivas de Azure (directivas de nomenclatura personalizadas) en la interfaz de Azure NetApp Files. Consulte [Instrucciones para el planeamiento de red de Azure NetApp Files](azure-netapp-files-network-topologies.md#considerations).

## <a name="performance-faqs"></a>Preguntas más frecuentes sobre rendimiento

### <a name="what-should-i-do-to-optimize-or-tune-azure-netapp-files-performance"></a>¿Qué debo hacer para optimizar o ajustar el rendimiento de Azure NetApp Files?

Puede realizar las siguientes acciones según los requisitos de rendimiento: 
- Asegúrese de que la máquina virtual tiene el tamaño apropiado.
- Habilite redes aceleradas para la máquina virtual.
- Seleccione el nivel de servicio deseado y el tamaño de grupo de capacidad.
- Cree un volumen con el tamaño de cuota deseada para la capacidad y el rendimiento.

### <a name="how-do-i-convert-throughput-based-service-levels-of-azure-netapp-files-to-iops"></a>¿Cómo puedo convertir los niveles de servicio basado en el rendimiento de Azure NetApp Files a IOPS?

Puede convertir MB/s a IOPS mediante el uso de la siguiente fórmula:  

`IOPS = (MBps Throughput / KB per IO) * 1024`

### <a name="how-do-i-change-the-service-level-of-a-volume"></a>¿Cómo puedo cambiar el nivel de servicio de un volumen?

Puede cambiar el nivel de servicio de un volumen existente al moverlo a otro grupo de capacidad que use el [nivel de servicio](azure-netapp-files-service-levels.md) que quiere para dicho volumen. Consulte [Cambio dinámico del nivel de servicio de un volumen](dynamic-change-volume-service-level.md). 

### <a name="how-do-i-monitor-azure-netapp-files-performance"></a>¿Cómo se puede supervisar el rendimiento de Azure NetApp Files?

Azure NetApp Files proporciona métricas de rendimiento del volumen. También puede usar Azure Monitor para supervisar las métricas de uso de Azure NetApp Files.  Consulte [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md) para obtener la lista de métricas de rendimiento de Azure NetApp Files.

### <a name="whats-the-performance-impact-of-kerberos-on-nfsv41"></a>¿Cuál es el impacto en el rendimiento de Kerberos en NFSv4.1?

Consulte [Impacto en el rendimiento de Kerberos en los volúmenes de NFSv4.1](performance-impact-kerberos.md) para obtener información sobre las opciones de seguridad de NFSv4.1, los vectores de rendimiento que se hayan probado y el impacto esperado en el rendimiento. 

## <a name="nfs-faqs"></a>Preguntas más frecuentes sobre NFS

### <a name="i-want-to-have-a-volume-mounted-automatically-when-an-azure-vm-is-started-or-rebooted--how-do-i-configure-my-host-for-persistent-nfs-volumes"></a>Quiero tener un volumen montado automáticamente cuando se inicia o se reinicia una máquina virtual de Azure.  ¿Cómo tengo que configurar mi host para los volúmenes persistentes de NFS?

Para que un volumen NFS se monte automáticamente al inicio o reinicio de una máquina virtual, agregue una entrada al archivo `/etc/fstab` en el host. 

Consulte [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md) para consultar los detalles.  

### <a name="why-does-the-df-command-on-nfs-client-not-show-the-provisioned-volume-size"></a>¿Por qué el comando DF en el cliente NFS no muestra el tamaño del volumen aprovisionado?

El tamaño del volumen notificado en DF es el tamaño máximo que puede alcanzar el volumen de Azure NetApp Files. El tamaño del volumen de Azure NetApp Files en el comando DF no es reflejo de la cuota o el tamaño del volumen.  Puede obtener el tamaño del volumen o la cuota de Azure NetApp Files mediante Azure Portal o la API.

### <a name="what-nfs-version-does-azure-netapp-files-support"></a>¿Qué versión de NFS admite Azure NetApp Files?

Azure NetApp Files admite NFSv3 y NFSv4.1. Se pueden [crear volúmenes](azure-netapp-files-create-volumes.md) con cualquier versión de NFS. 

### <a name="how-do-i-enable-root-squashing"></a>¿Cómo se puede habilitar root squashing?

El usuario puede especificar si la cuenta raíz tendrá o no acceso al volumen por medio de la directiva de exportación del volumen. Para más información, consulte [Configuración de la directiva de exportación para un volumen NFS](azure-netapp-files-configure-export-policy.md).

### <a name="can-i-use-the-same-file-path-volume-creation-token-for-multiple-volumes"></a>¿Puedo usar la misma ruta de acceso de archivo (token de creación de volumen) para varios volúmenes?

Sí, puede hacerlo. Sin embargo, la ruta de acceso de archivo debe usarse en una suscripción o región distinta.   

Por ejemplo, puede crear un volumen llamado denominado `vol1`. A continuación, crea otro volumen también llamado `vol1` en un grupo de capacidad distinto, pero en la misma suscripción y región. En este caso, el uso del mismo nombre de volumen `vol1` producirá un error. Para usar la misma ruta de acceso de archivo, el nombre debe estar en otra región o suscripción.

### <a name="when-i-try-to-access-nfs-volumes-through-a-windows-client-why-does-the-client-take-a-long-time-to-search-folders-and-subfolders"></a>Cuando intento acceder a volúmenes NFS a través de un cliente de Windows, ¿por qué el cliente tarda mucho tiempo en buscar carpetas y subcarpetas?

Asegúrese de que `CaseSensitiveLookup` está habilitado en el cliente de Windows para acelerar la búsqueda de carpetas y subcarpetas:

1. Use el siguiente comando de PowerShell para habilitar CaseSensitiveLookup:   
    `Set-NfsClientConfiguration -CaseSensitiveLookup 1`    
2. Monte el volumen en el servidor de Windows.   
    Ejemplo:   
    `Mount -o rsize=1024 -o wsize=1024 -o mtype=hard \\10.x.x.x\testvol X:*`

### <a name="how-does-azure-netapp-files-support-nfsv41-file-locking"></a>¿Cómo admite Azure NetApp Files el bloqueo de archivos de NFSv4.1? 

En el caso de los clientes de NFSv4.1, Azure NetApp Files admite el mecanismo de bloqueo de archivos de NFSv4.1 que mantiene el estado de todos los bloqueos de archivos en un modelo basado en concesión. 

Por RFC 3530, Azure NetApp Files define un período de concesión único para todos los estados de un cliente NFS. Si el cliente no renueva su concesión dentro del período definido, el servidor liberará todos los estados asociados a la concesión del cliente.  

Por ejemplo, si un cliente que monta un volumen deja de responder o se bloquea más allá de los tiempos de espera, se liberarán los bloqueos. El cliente puede renovar su concesión de forma explícita o implícita realizando operaciones como leer un archivo.   

Un período de gracia define un período de procesamiento especial en el que los clientes pueden intentar reclamar su estado de bloqueo durante una recuperación del servidor. El tiempo de espera predeterminado para las concesiones es de 30 segundos con un período de gracia de 45 segundos. Transcurrido ese tiempo, se liberará la concesión del cliente.   

## <a name="smb-faqs"></a>Preguntas más frecuentes de SMB

### <a name="which-smb-versions-are-supported-by-azure-netapp-files"></a>¿Qué versiones de SMB son compatibles con Azure NetApp Files?

Azure NetApp Files admite SMB 2.1 y SMB 3.1 (que incluye compatibilidad con SMB 3.0).    

### <a name="is-an-active-directory-connection-required-for-smb-access"></a>¿Es necesaria una conexión de Active Directory para el acceso SMB? 

Sí, tiene que crear una conexión de Active Directory antes de implementar un volumen de SMB. Para que la conexión tenga éxito la subred delegada de Azure NetApp Files tiene que tener acceso a los controladores de dominio especificados.  Consulte [Crear un volumen de SMB](./azure-netapp-files-create-volumes-smb.md) para más información. 

### <a name="how-many-active-directory-connections-are-supported"></a>¿Cuántas conexiones de Active Directory se admiten?

Azure NetApp Files no admite varias conexiones de Active Directory (AD) en una única *región*, incluso si las conexiones de AD se encuentran en distintas cuentas de NetApp. Sin embargo, puede tener varias conexiones de AD en una única *suscripción*, siempre que las conexiones de AD se encuentren en regiones diferentes. Si necesita varias conexiones de AD en una sola región, puede usar suscripciones independientes para ello. 

Se configura una conexión de AD por cuenta de NetApp; la conexión de AD solo es visible a través de la cuenta de NetApp en la que se crea.

### <a name="does-azure-netapp-files-support-azure-active-directory"></a>¿ Azure NetApp Files admite Azure Active Directory? 

Se admite tanto [Azure Active Directory (AD) Domain Services](../active-directory-domain-services/overview.md) como [Active Directory Domain Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview). Puede usar controladores de dominio de Active Directory existentes con Azure NetApp Files. Los controladores de dominio pueden residir en Azure como máquinas virtuales o en el entorno local mediante ExpressRoute o VPN S2S. En este momento, Azure NetApp Files no admite la unión a AD para [Azure Active Directory](https://azure.microsoft.com/resources/videos/azure-active-directory-overview/).

Si usa Azure NetApp Files con Azure Active Directory Domain Services, la ruta de acceso de la unidad organizativa es `OU=AADDC Computers` cuando configura Active Directory para su cuenta de NetApp.

### <a name="what-versions-of-windows-server-active-directory-are-supported"></a>¿Qué versiones de Windows Server Active Directory se admiten?

Azure NetApp Files admite las versiones 2008r2SP1-2019 de Windows Server de Active Directory Domain Services.

### <a name="why-does-the-available-space-on-my-smb-client-not-show-the-provisioned-size"></a>¿Por qué el espacio disponible en el cliente SMB no muestra el tamaño aprovisionado?

El tamaño de volumen que SMB indica es el tamaño máximo que puede alcanzar el volumen de Azure NetApp Files. El tamaño del volumen de Azure NetApp Files que se muestra en el cliente SMB no es reflejo de la cuota o el tamaño del volumen. Puede obtener el tamaño del volumen o la cuota de Azure NetApp Files mediante Azure Portal o la API.

### <a name="im-having-issues-connecting-to-my-smb-share-what-should-i-do"></a>Tengo problemas para conectarme al recurso compartido de SMB. ¿Cuál debo hacer?

Como procedimiento recomendado, establezca la tolerancia máxima para la sincronización del reloj del equipo en cinco minutos. Para más información, consulte [Tolerancia máxima para la sincronización del reloj del equipo](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj852172(v=ws.11)). 

### <a name="can-i-manage-smb-shares-sessions-and-open-files-through-computer-management-console-mmc"></a>¿Puedo administrar `SMB Shares`, `Sessions` y `Open Files` a través de la consola de administración de equipos (MMC)?

Actualmente no se admite la administración de `SMB Shares`, `Sessions` y `Open Files` a través de la consola de administración de equipos (MMC).

### <a name="how-can-i-obtain-the-ip-address-of-an-smb-volume-via-the-portal"></a>¿Cómo puedo obtener la dirección IP de un volumen SMB a través del portal?

Use el vínculo de la **vista JSON** en el panel Información general del volumen y busque el identificador **startIP** en **properties** -> **mountTargets**.

## <a name="capacity-management-faqs"></a>Preguntas más frecuentes sobre la administración de la capacidad

### <a name="how-do-i-monitor-usage-for-capacity-pool-and-volume-of-azure-netapp-files"></a>¿Cómo puedo supervisar el uso de grupo de capacidad y volumen de Azure NetApp Files? 

Azure NetApp Files proporciona métricas de uso de grupo de capacidad y volumen. También puede usar Azure Monitor para supervisar el uso de Azure NetApp Files. Consulte [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md) para más información. 

### <a name="can-i-manage-azure-netapp-files-through-azure-storage-explorer"></a>¿Puedo administrar Azure NetApp Files mediante Explorador de Azure Storage?

No. Explorador de Azure Storage no es compatible con Azure NetApp Files.

### <a name="how-do-i-determine-if-a-directory-is-approaching-the-limit-size"></a>¿Cómo puedo determinar si un directorio está llegando al tamaño límite?

Puede usar el comando `stat` desde un cliente para ver si un directorio está llegando al límite de tamaño máximo de los metadatos del directorio (320 MB).   

En un directorio de 320 MB, el número de bloques es 655 360 y el tamaño de cada bloque es de 512 bytes.  (Es decir, 320 x 1024 x 1024/512). Este número se traduce en aproximadamente 4 millones archivos como máximo para un directorio de 320 MB. Sin embargo, el número máximo real de archivos puede ser menor, en función de factores como el número de archivos que contienen caracteres que no son ASCII en el directorio. Por lo tanto, debe usar el comando `stat` como se indica a continuación para determinar si el directorio está llegando a su límite.  

Ejemplos:

```console
[makam@cycrh6rtp07 ~]$ stat bin
File: 'bin'
Size: 4096            Blocks: 8          IO Block: 65536  directory

[makam@cycrh6rtp07 ~]$ stat tmp
File: 'tmp'
Size: 12288           Blocks: 24         IO Block: 65536  directory
 
[makam@cycrh6rtp07 ~]$ stat tmp1
File: 'tmp1'
Size: 4096            Blocks: 8          IO Block: 65536  directory
```


## <a name="data-migration-and-protection-faqs"></a>Preguntas frecuentes sobre migración y protección de datos

### <a name="how-do-i-migrate-data-to-azure-netapp-files"></a>¿Cómo se pueden migrar los datos a Azure NetApp Files?
Azure NetApp Files proporciona volúmenes SMB y NFS.  Puede usar cualquier herramienta de copia basada en archivos para migrar datos al servicio. 

NetApp ofrece una solución basada en SaaS, [NetApp Cloud Sync](https://cloud.netapp.com/cloud-sync-service).  La solución le permite replicar datos de NFS o SMB en los recursos compartidos de SMB o exportaciones de NFS de Azure NetApp Files. 

También puede utilizar una amplia gama de herramientas gratuitas para copiar los datos. Para NFS, puede usar las herramientas de cargas de trabajo, como [rsync](https://rsync.samba.org/examples.html) para copiar y sincronizar datos de origen en un volumen de Azure NetApp Files. Para SMB, puede usar las cargas de trabajo [robocopy](/windows-server/administration/windows-commands/robocopy) de la misma manera.  Estas herramientas también pueden replicar los permisos de archivo o carpeta. 

Los requisitos para la migración de datos desde un entorno local a Azure NetApp Files son los siguientes: 

- Asegúrese de que Azure NetApp Files está disponible en la región de Azure de destino.
- Valide la conectividad de red entre el origen y la dirección IP del volumen de destino de Azure NetApp Files. La transferencia de datos entre el entorno local y el servicio Azure NetApp Files se admite a través de ExpressRoute.
- Cree el volumen de destino de Azure NetApp Files.
- Transfiera los datos de origen al volumen de destino mediante la herramienta de copia de archivo que prefiera.

### <a name="how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region"></a>¿Cómo puedo crear una copia de Azure NetApp Files en otra región de Azure?
    
Azure NetApp Files proporciona volúmenes SMB y NFS.  Cualquier herramienta de copia basada en archivos puede usarse para replicar datos entre regiones de Azure. 

NetApp ofrece una solución basada en SaaS, [NetApp Cloud Sync](https://cloud.netapp.com/cloud-sync-service).  La solución le permite replicar datos de NFS o SMB en los recursos compartidos de SMB o exportaciones de NFS de Azure NetApp Files. 

También puede utilizar una amplia gama de herramientas gratuitas para copiar los datos. Para NFS, puede usar las herramientas de cargas de trabajo, como [rsync](https://rsync.samba.org/examples.html) para copiar y sincronizar datos de origen en un volumen de Azure NetApp Files. Para SMB, puede usar las cargas de trabajo [robocopy](/windows-server/administration/windows-commands/robocopy) de la misma manera.  Estas herramientas también pueden replicar los permisos de archivo o carpeta. 

Los requisitos para la replicación de un volumen de Azure NetApp Files en otra región de Azure son los siguientes: 
- Asegúrese de que Azure NetApp Files está disponible en la región de Azure de destino.
- Valide la conectividad de red entre redes virtuales en cada región. Actualmente, no se admite el emparejamiento global entre redes virtuales.  Puede establecer la conectividad entre redes virtuales mediante la vinculación con un circuito de ExpressRoute o el uso de una conexión VPN de sitio a sitio. 
- Cree el volumen de destino de Azure NetApp Files.
- Transfiera los datos de origen al volumen de destino mediante la herramienta de copia de archivo que prefiera.

### <a name="is-migration-with-azure-data-box-supported"></a>¿Se admite la migración con Azure Data Box?

No. En la actualidad, Azure Data Box no es compatible con Azure NetApp Files. 

### <a name="is-migration-with-azure-importexport-service-supported"></a>¿Se admite la migración con el servicio Azure Import/Export?

No. En la actualidad, el servicio Azure Import/Export no es compatible con Azure NetApp Files.

## <a name="product-faqs"></a>Preguntas más frecuentes sobre productos

### <a name="can-i-use-azure-netapp-files-nfs-or-smb-volumes-with-azure-vmware-solution-avs"></a>¿Puedo usar volúmenes NFS o SMB de Azure NetApp Files con Azure VMware Solution (AVS)?

Puede montar volúmenes NFS de Azure NetApp Files en máquinas virtuales Windows o Linux de AVS. Puede asignar recursos compartidos de SMB de Azure NetApp Files en máquinas virtuales Windows de AVS. Para obtener más información, consulte [Azure NetApp Files con Azure VMware Solution]( ../azure-vmware/netapp-files-with-azure-vmware-solution.md).  

### <a name="what-regions-are-supported-for-using-azure-netapp-files-nfs-or-smb-volumes-with-azure-vmware-solution-avs"></a>¿Qué regiones se admiten para usar volúmenes NFS o SMB de Azure NetApp Files con Azure VMware Solution (AVS)?

El uso de volúmenes NFS o SMB de Azure NetApp Files con AVS se admite en las siguientes regiones: Este de EE. UU., Oeste de EE. UU., Oeste de Europa y Este de Australia.

## <a name="next-steps"></a>Pasos siguientes  

- [Preguntas más frecuentes acerca de Microsoft Azure ExpressRoute](../expressroute/expressroute-faqs.md)
- [Preguntas más frecuentes acerca de Azure Virtual Network](../virtual-network/virtual-networks-faq.md)
- [Creación de una solicitud de soporte técnico de Azure](../azure-portal/supportability/how-to-create-azure-support-request.md)
- [Azure Data Box](../databox/index.yml)
- [Preguntas más frecuentes sobre el rendimiento de SMB para Azure NetApp Files](azure-netapp-files-smb-performance.md)
