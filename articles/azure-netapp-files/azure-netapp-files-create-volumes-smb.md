---
title: Creación de un volumen SMB para Azure NetApp Files | Microsoft Docs
description: En este artículo se muestra cómo crear un volumen SMB3 en Azure NetApp Files. Obtenga información acerca de los requisitos para las conexiones de Active Directory y Domain Services.
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
ms.date: 03/29/2021
ms.author: b-juche
ms.openlocfilehash: eeeaf01dd20e5b309884a01f954ceca576cbcbb9
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259632"
---
# <a name="create-an-smb-volume-for-azure-netapp-files"></a>Creación de un volumen de SMB para Azure NetApp Files

Azure NetApp Files admite la creación de volúmenes con NFS (NFSv3 y NFSv4.1), SMB3 o el protocolo dual (NFSv3 y SMB). El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo. 

En este artículo se muestra cómo crear un volumen SMB3. Para los volúmenes NFS, consulte [Creación de un volumen NFS](azure-netapp-files-create-volumes.md). Para volúmenes de protocolo dual, consulte [Creación de un volumen de protocolo dual](create-volumes-dual-protocol.md).

## <a name="before-you-begin"></a>Antes de empezar 

* Debe haber configurado un grupo de capacidad. Consulte [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md).     
* Debe haber una subred delegada en Azure NetApp Files. Consulte [Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

## <a name="configure-active-directory-connections"></a>Configuración de conexiones de Active Directory 

Debe crear una conexión de Active Directory antes de crear un volumen SMB. Si no ha configurado conexiones de Active Directory para Azure NetApp Files, siga las instrucciones descritas en [Creación y administración de conexiones de Active Directory para Azure NetApp Files](create-active-directory-connections.md).

## <a name="add-an-smb-volume"></a>Incorporación de un volumen SMB

1. Haga clic en la hoja **Volúmenes** desde la hoja Grupos de capacidad. 

    ![Vaya a Volúmenes](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png)

2. Haga clic en **+ Agregar volumen** para crear un volumen.  
    Aparece la ventana Crear un volumen.

3. En la ventana Crear un volumen, haga clic en **Crear** y proporcione la información para los campos siguientes en la pestaña Aspectos básicos:   
    * **Nombre del volumen**      
        Especifique el nombre para el volumen que va a crear.   

        Un nombre de volumen debe ser único dentro de cada grupo de capacidad. Debe tener tres caracteres de longitud, como mínimo. Puede usar cualquier carácter alfanumérico.   

        No se puede usar `default` o `bin` como nombre del volumen.

    * **Grupo de capacidad**  
        Especifique el grupo de capacidad en el que desee que el volumen se cree.

    * **Cuota**  
        Especifique la cantidad de almacenamiento lógico que se asigna al volumen.  

        El campo **Cuota disponible** muestra la cantidad de espacio no utilizado en el grupo de capacidad elegido que puede usar para crear un nuevo volumen. El tamaño del volumen nuevo no debe superar la cuota disponible.  

    * **Rendimiento (MiB/S)**    
        Si el volumen se crea en un grupo de capacidad con QoS manual, especifique el rendimiento que desea para el volumen.   

        Si el volumen se crea en un grupo de capacidad con QoS automático, el valor que se muestra en este campo es (rendimiento de cuota x nivel de servicio).   

    * **Red virtual**  
        Especifique la red virtual de Azure desde la que desea tener acceso al volumen.  

        La red virtual que especifique debe tener una subred delegada en Azure NetApp Files. Solo puede tener acceso al servicio Azure NetApp Files desde la misma red virtual o desde una red virtual que se encuentre en la misma ubicación que el volumen mediante el emparejamiento de redes virtuales. También puede acceder al volumen desde la red local mediante ExpressRoute.   

    * **Subred**  
        Especifique la subred que desea usar para el volumen.  
        La red virtual que especifique debe estar delegada en Azure NetApp Files. 
        
        Si no ha delegado una subred, puede hacer clic en **Crear nuevo** en la página Crear un volumen. A continuación, en la página de creación de la subred, especifique la información de la subred y seleccione **Microsoft.NetApp/volumes** para delegar la subred para Azure NetApp Files. En cada red virtual, solo puede delegarse una subred a Azure NetApp Files.   
 
        ![Crear un volumen](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Creación de una subred](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Si desea aplicar una directiva de instantáneas existente al volumen, haga clic en **Mostrar la sección avanzada** para expandirla, especifique si quiere ocultar la ruta de acceso de la instantánea y seleccione una directiva de instantáneas en el menú desplegable. 

        Para obtener información sobre cómo crear una directiva de instantáneas, consulte [Administración de directivas de instantánea](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Mostrar la sección avanzada](../media/azure-netapp-files/volume-create-advanced-selection.png)

4. Haga clic en **Protocolo** y complete la siguiente información:  
    * Seleccione **SMB** como tipo de protocolo para el volumen. 
    * Seleccione la conexión de **Active Directory** en la lista desplegable.
    * Especifique el nombre del volumen compartido en **Nombre del recurso compartido**.
    * Si desea facilitar la disponibilidad continua para el volumen SMB, seleccione **Habilitar la disponibilidad continua**.    

        > [!IMPORTANT]   
        > La característica de disponibilidad continua de SMB se encuentra actualmente en versión preliminar pública. Para acceder a esta característica, debe enviar una solicitud de lista de espera desde la **[página de envío de solicitudes de lista de espera de la versión preliminar pública de recursos compartidos con disponibilidad continua de SMB para Azure NetApp Files](https://aka.ms/anfsmbcasharespreviewsignup)** . Antes de utilizar la característica de disponibilidad continua, espere a recibir el correo electrónico de confirmación oficial del equipo de Azure NetApp Files.   
        > 
        > Habilite la disponibilidad continua solo con cargas de trabajo de SQL. No se admite el uso de recursos compartidos de disponibilidad continua de SMB con cargas de trabajo que *no* sean de SQL Server. Esta característica se admite actualmente en Windows SQL Server. Linux SQL Server no se admite actualmente. Si usa una cuenta que no sea de administrador (dominio) para instalar SQL Server, asegúrese de que la cuenta tiene asignado el privilegio de seguridad necesario. Si la cuenta de dominio no tiene el privilegio de seguridad necesario (`SeSecurityPrivilege`) y el privilegio no se puede establecer en el nivel de dominio, puede conceder el privilegio a la cuenta mediante el campo **Security privilege users** (Usuarios con privilegios de seguridad) de conexiones de Active Directory. Consulte la sección [Creación de una conexión de Active Directory](create-active-directory-connections.md#create-an-active-directory-connection).

    <!-- [1/13/21] Commenting out command-based steps below, because the plan is to use form-based (URL) registration, similar to CRR feature registration -->
    <!-- 
        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSMBCAShare
        ```

        Check the status of the feature registration: 

        > [!NOTE]
        > The **RegistrationState** may be in the `Registering` state for up to 60 minutes before changing to`Registered`. Wait until the status is `Registered` before continuing.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFSMBCAShare
        ```
        
        You can also use [Azure CLI commands](/cli/azure/feature?preserve-view=true&view=azure-cli-latest) `az feature register` and `az feature show` to register the feature and display the registration status. 
    --> 

    ![Captura de pantalla que describe la pestaña de protocolo de creación de un volumen SMB.](../media/azure-netapp-files/azure-netapp-files-protocol-smb.png)

5. Haga clic en **Revisar y crear** para revisar los detalles del volumen.  Haga clic en **Crear** para crear el volumen SMB.

    El volumen que creó aparece en la hoja Volúmenes. 
 
    Un volumen hereda los atributos de ubicación, la suscripción y el grupo de recursos de su grupo de capacidad. Para supervisar el estado de la implementación del volumen, puede usar la pestaña Notificaciones.

## <a name="control-access-to-an-smb-volume"></a>Control del acceso a un volumen SMB  

El acceso a un volumen SMB se administra mediante permisos.  

### <a name="share-permissions"></a>Permisos de recursos compartidos  

De forma predeterminada, un nuevo volumen tiene los permisos de recursos compartidos **Todos/Control total**. Los miembros del grupo Administradores de dominio pueden cambiar los permisos de los recursos compartidos de la siguiente manera:  

1. Asigne el recurso compartido a una unidad.  
2. Haga clic con el botón derecho en la unidad, seleccione **Propiedades** y, luego, vaya a la pestaña **Seguridad**.

[ ![Establecimiento de permisos de recursos compartidos](../media/azure-netapp-files/set-share-permissions.png)](../media/azure-netapp-files/set-share-permissions.png#lightbox)

### <a name="ntfs-file-and-folder-permissions"></a>Permisos de carpetas y archivos NTFS  

Puede establecer permisos para un archivo o carpeta con la pestaña **Seguridad** de las propiedades del objeto en el cliente SMB de Windows.
 
![Establecimiento de permisos de carpetas y archivos](../media/azure-netapp-files/set-file-folder-permissions.png) 

## <a name="next-steps"></a>Pasos siguientes  

* [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Preguntas más frecuentes de SMB](./azure-netapp-files-faqs.md#smb-faqs)
* [Solución de problemas de SMB o de volúmenes de dos protocolos](troubleshoot-dual-protocol-volumes.md)
* [Obtenga información sobre la integración de redes virtuales para los servicios de Azure](../virtual-network/virtual-network-for-azure-services.md)
* [Install a new Active Directory forest using Azure CLI](/windows-server/identity/ad-ds/deploy/virtual-dc/adds-on-azure-vm) (Instalación de un nuevo bosque de Active Directory en la CLI de Azure)
