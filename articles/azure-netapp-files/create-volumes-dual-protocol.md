---
title: Creación de un volumen de protocolo dual (NFSv3 y SMB) para Azure NetApp Files | Microsoft Docs
description: Describe cómo crear un volumen que usa el protocolo dual NFSv3 y SMB con compatibilidad con la asignación de usuarios LDAP.
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
ms.date: 04/05/2021
ms.author: b-juche
ms.openlocfilehash: b6a2d7ad92c209a93d740d60808c2cbd2f90c6b4
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107258425"
---
# <a name="create-a-dual-protocol-nfsv3-and-smb-volume-for-azure-netapp-files"></a>Creación de un volumen de protocolo dual (NFSv3 y SMB) para Azure NetApp Files

Azure NetApp Files admite la creación de volúmenes con NFS (NFSv3 y NFSv4.1), SMB3 o el protocolo dual. En este artículo se muestra cómo crear un volumen que use el protocolo dual de NFSv3 y SMB con compatibilidad con la asignación de usuarios LDAP. 

Para crear volúmenes NFS, consulte [Creación de un volumen NFS](azure-netapp-files-create-volumes.md). Para crear volúmenes SMB, consulte [Creación de un volumen SMB](azure-netapp-files-create-volumes-smb.md). 

## <a name="before-you-begin"></a>Antes de empezar 

* Debe haber creado ya un grupo de capacidad.  
    Consulte [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md).   
* Debe haber una subred delegada en Azure NetApp Files.  
    Consulte [Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

## <a name="considerations"></a>Consideraciones

* Asegúrese de cumplir los [requisitos para las conexiones de Active Directory](create-active-directory-connections.md#requirements-for-active-directory-connections). 
* Cree una zona de búsqueda inversa en el servidor DNS y, a continuación, agregue un registro de puntero (PTR) de la máquina host de AD en esa zona de búsqueda inversa. De lo contrario, se producirá un error en la creación del volumen de dos protocolos.
* Asegúrese de que el cliente NFS esté actualizado y ejecute las actualizaciones más recientes del sistema operativo.
* Asegúrese de que el servidor LDAP de Active Directory (AD) está en funcionamiento en AD. Puede hacerlo instalando y configurando el rol [Active Directory Lightweight Directory Services (AD LDS)](/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831593(v=ws.11)) en la máquina de AD.
* Los volúmenes del protocolo dual no admiten actualmente Azure Active Directory Domain Services (AADDS). La opción LDAP a través de TLS no debe estar habilitada si está usando AADDS.
* La versión de NFS usada por un volumen de protocolo dual es NFSv3. Como tal, se aplican las siguientes consideraciones:
    * El protocolo dual no es compatible con los atributos extendidos de ACLS de Windows `set/get` desde clientes de NFS.
    * Los clientes de NFS no pueden cambiar los permisos para el estilo de seguridad de NTFS, y los clientes de Windows no pueden cambiar los permisos de los volúmenes del protocolo dual de estilo UNIX.   

    En la tabla siguiente se describen los estilos de seguridad y sus efectos:  
    
    | Estilo de seguridad    | Clientes que pueden modificar permisos   | Permisos que pueden usar los clientes  | Estilo de seguridad efectivo resultante    | Clientes que pueden tener acceso a archivos     |
    |-  |-  |-  |-  |-  |
    | `Unix`    | NFS   | Bits del modo NFSv3   | UNIX  | NFS y Windows   |
    | `Ntfs`    | Windows   | ACL de NTFS     | NTFS  |NFS y Windows|
* Los usuarios de UNIX que monten el volumen de estilo de seguridad NTFS mediante NFS se autenticarán como el usuario Windows `root` para `root` de UNIX y `pcuser` para todos los demás usuarios. Asegúrese de que estas cuentas de usuario existen en Active Directory antes de montar el volumen al usar NFS. 
* Si tiene topologías grandes y usa el estilo de seguridad `Unix` con un volumen de protocolo dual o LDAP con grupos extendidos, es posible que Azure NetApp Files no pueda acceder a todos los servidores de las topologías.  Si se produce esta situación, póngase en contacto con su equipo de cuentas para obtener ayuda.  <!-- NFSAAS-15123 --> 
* No es necesario un certificado de CA raíz del servidor para crear un volumen de protocolo dual. Solo es necesario si está habilitado LDAP sobre TLS.


## <a name="create-a-dual-protocol-volume"></a>Creación de un volumen de protocolo dual

1.  Haga clic en la hoja **Volúmenes** desde la hoja Grupos de capacidad. Haga clic en **+ Agregar volumen** para crear un volumen. 

    ![Vaya a Volúmenes](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png) 

2.  En la ventana Crear un volumen, haga clic en **Crear** y proporcione la información para los campos siguientes bajo la pestaña Aspectos básicos:   
    * **Nombre del volumen**      
        Especifique el nombre para el volumen que va a crear.   

        Un nombre de volumen debe ser único dentro de cada grupo de capacidad. Debe tener tres caracteres de longitud, como mínimo. Puede usar cualquier carácter alfanumérico.   

        No se puede usar `default` ni `bin` como nombre del volumen.

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
        
        Si no ha delegado una subred, haga clic en **Crear nuevo** en el volumen de creación de un volumen. A continuación, en la página de creación de la subred, especifique la información de la subred y seleccione **Microsoft.NetApp/volumes** para delegar la subred para Azure NetApp Files. En cada red virtual, solo puede delegarse una subred a Azure NetApp Files.   
 
        ![Crear un volumen](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Creación de una subred](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Si desea aplicar una directiva de instantáneas existente al volumen, haga clic en **Mostrar la sección avanzada** para expandirla, especifique si quiere ocultar la ruta de acceso de la instantánea y seleccione una directiva de instantáneas en el menú desplegable. 

        Para obtener información sobre cómo crear una directiva de instantáneas, consulte [Administración de directivas de instantánea](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Mostrar la sección avanzada](../media/azure-netapp-files/volume-create-advanced-selection.png)

3. Haga clic en **Protocolo** y realice las siguientes acciones:  
    * Seleccione el **protocolo dual (NFSv3 y SMB)** como tipo de protocolo para el volumen.   

    * Especifique la **ruta de acceso del volumen** para el volumen.   
    Esta ruta de acceso del volumen es el nombre del volumen compartido. El nombre debe comenzar con un carácter alfabético y debe ser único dentro de cada suscripción y región.  

    * Especifique el **estilo de seguridad** que se vaya a usar: NTFS (valor predeterminado) o UNIX.

    * Si lo desea, [configure la directiva de exportación para el volumen](azure-netapp-files-configure-export-policy.md).

    ![Especificación del protocolo dual](../media/azure-netapp-files/create-volume-protocol-dual.png)

4. Haga clic en **Revisar y crear** para revisar los detalles del volumen. Después, haga clic en **Crear** para crear el volumen.

    El volumen que creó aparece en la hoja Volúmenes. 
 
    Un volumen hereda los atributos de ubicación, la suscripción y el grupo de recursos de su grupo de capacidad. Para supervisar el estado de la implementación del volumen, puede usar la pestaña Notificaciones.

## <a name="allow-local-nfs-users-with-ldap-to-access-a-dual-protocol-volume"></a>Habilitación de los usuarios de NFS locales con LDAP para acceder a un volumen de dos protocolos 

Puede habilitar usuarios del cliente NFS locales que no estén presentes en el servidor LDAP de Windows para que accedan a un volumen de dos protocolos que tenga habilitado LDAP con grupos extendidos. Para ello, habilite la opción **Permitir usuarios locales de NFS con LDAP** de la siguiente manera:

1. Haga clic en **Conexiones de Active Directory**.  En una conexión de Active Directory existente, haga clic en el menú contextual (los tres puntos `…`) y seleccione **Editar**.  

2. En la ventana **Edit Active Directory settings** (Editar configuración de Active Directory) que aparece, seleccione la opción **Permitir usuarios locales de NFS con LDAP**.  

    ![Captura de pantalla que muestra la opción Permitir usuarios locales de NFS con LDAP](../media/azure-netapp-files/allow-local-nfs-users-with-ldap.png)  


## <a name="manage-ldap-posix-attributes"></a>Administración de los atributos POSIX de LDAP

Puede administrar los atributos POSIX como UID, el directorio particular y otros valores mediante el complemento de MMC Usuarios y equipos de Active Directory.  En el ejemplo siguiente se muestra el editor de atributos de Active Directory:  

![Editor de atributos de Active Directory](../media/azure-netapp-files/active-directory-attribute-editor.png) 

Debe establecer los siguientes atributos para los usuarios LDAP y los grupos LDAP: 
* Atributos necesarios para los usuarios LDAP:   
    `uid: Alice`, `uidNumber: 139`, `gidNumber: 555`, `objectClass: posixAccount`
* Atributos necesarios para los grupos LDAP:   
    `objectClass: posixGroup`, `gidNumber: 555`

## <a name="configure-the-nfs-client"></a>Configuración del cliente NFS 

Siga las instrucciones de [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md) para configurar el cliente NFS.  

## <a name="next-steps"></a>Pasos siguientes  

* [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md)
* [Solución de problemas de SMB o de volúmenes de dos protocolos](troubleshoot-dual-protocol-volumes.md)
* [Solución de problemas de volúmenes LDAP](troubleshoot-ldap-volumes.md)
