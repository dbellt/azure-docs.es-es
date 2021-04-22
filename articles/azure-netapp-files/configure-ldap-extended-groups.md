---
title: Configuración de ADDS LDAP con grupos extendidos para el acceso al volumen NFS de Azure NetApp Files | Microsoft Docs
description: Describe las consideraciones y pasos para habilitar LDAP con grupos extendidos al crear un volumen NFS mediante Azure NetApp Files.
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
ms.date: 04/09/2021
ms.author: b-juche
ms.openlocfilehash: 2546236399853f3ed6fad9e07e031edb568fbfe9
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311539"
---
# <a name="configure-adds-ldap-with-extended-groups-for-nfs-volume-access"></a>Configuración de ADDS LDAP con grupos extendidos para el acceso a volúmenes NFS

Al [crear un volumen NFS](azure-netapp-files-create-volumes.md), tiene la opción de habilitar la característica LDAP con grupos extendidos (la opción **LDAP**) para el volumen. Esta característica permite que los usuarios de Active Directory LDAP y grupos extendidos (hasta 1024 grupos) accedan al volumen. Puede usar la característica LDAP con grupos extendidos con volúmenes NFSv4.1 y NFSv3. 

En este artículo se explican las consideraciones y pasos para habilitar LDAP con grupos extendidos al crear un volumen NFS.  

## <a name="considerations"></a>Consideraciones

* LDAP con grupos extendidos solo se admite con Active Directory Domain Services (ADDS) o Azure Active Directory Domain Services (AADDS). No se admite OpenLDAP ni otros servicios de directorio de LDAP de terceros. 

* LDAP a través de TLS *no* se debe habilitar si usa Azure Active Directory Domain Services (AADDS).  

* Si habilita la característica LDAP con grupos extendidos, los [volúmenes Kerberos](configure-kerberos-encryption.md) habilitados para LDAP no mostrarán correctamente la propiedad de los archivos para los usuarios LDAP. Un archivo o directorio que haya sido creado por un usuario LDAP tendrá establecido `root` como propietario de manera predeterminada, en lugar del usuario LDAP real. Sin embargo, la cuenta `root` puede cambiar manualmente la propiedad del archivo mediante el comando `chown <username> <filename>`. 

* Una vez creado el volumen, no se puede modificar la opción LDAP (ya sea que esté habilitada o deshabilitada).  

* En la tabla siguiente se describe la configuración del período de vida (TTL) para la caché LDAP. Debe esperar hasta que se actualice la caché antes de intentar acceder a un archivo o directorio a través de un cliente. De lo contrario, aparece un mensaje de acceso denegado en el cliente. 

    |     Condición de error    |     Resolución    |
    |-|-|
    | Cache |  Tiempo de espera predeterminado |
    | Lista de pertenencia a grupos  | TTL de 24 horas  |
    | Grupos de UNIX  | TTL de 24 horas, TTL negativo de 1 minuto  |
    | Usuarios de UNIX  | TTL de 24 horas, TTL negativo de 1 minuto  |

    Las memorias caché tienen un período de tiempo de espera específico denominado *período de vida*. Una vez que se ha agotado ese tiempo de espera, las entradas expiran para que las entradas obsoletas no se conserven. El valor de *TTL negativo* es donde se encuentra una búsqueda en la que se ha producido un error. De este modo, se evitan problemas de rendimiento provocados por las consultas LDAP para objetos que podrían no existir.        

## <a name="steps"></a>Pasos

1. La característica LDAP con grupos extendidos se encuentra actualmente en versión preliminar. Antes de usar esta característica por primera vez, debe registrarla:  

    1. Registre la característica:   

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapExtendedGroups
        ```

    2. Compruebe el estado del registro de la característica: 

        > [!NOTE]
        > **RegistrationState** puede estar en el estado `Registering` hasta 60 minutos antes de cambiar a `Registered`. Espere hasta que el estado sea `Registered` antes de continuar.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapExtendedGroups
        ```
        
    También puede usar los comandos de la [CLI de Azure](/cli/azure/feature) `az feature register` y `az feature show` para registrar la característica y mostrar el estado del registro. 

2. Los volúmenes LDAP requieren una configuración de Active Directory para la configuración del servidor LDAP. Siga las instrucciones de [Requisitos para las conexiones de Active Directory](create-active-directory-connections.md#requirements-for-active-directory-connections) y [Creación de una conexión de Active Directory](create-active-directory-connections.md#create-an-active-directory-connection) para configurar las conexiones de Active Directory en Azure Portal.  

    > [!NOTE]
    > Asegúrese de que ha establecido la configuración de conexión de Active Directory. Se creará una cuenta de máquina en la unidad organizativa (UO) que se especifica en la configuración de la conexión de Active Directory. El cliente LDAP usa la configuración para autenticarse en Active Directory.

3. Asegúrese de que el servidor LDAP de Active Directory está en funcionamiento en Active Directory. 

4. Los usuarios LDAP de NFS deben tener determinados atributos POSIX en el servidor LDAP. Establezca los atributos para los usuarios LDAP y los grupos LDAP de la manera siguiente: 

    * Atributos necesarios para los usuarios LDAP:   
        `uid: Alice`, `uidNumber: 139`, `gidNumber: 555`, `objectClass: user`
    * Atributos necesarios para los grupos LDAP:   
        `objectClass: group`, `gidNumber: 555`

    Puede administrar los atributos POSIX mediante el complemento de MMC Usuarios y equipos de Active Directory. En el ejemplo siguiente se muestra el editor de atributos de Active Directory:  

    ![Editor de atributos de Active Directory](../media/azure-netapp-files/active-directory-attribute-editor.png) 

5. Si quiere configurar un cliente Linux NFSv4.1 integrado con LDAP, consulte [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md).

6.  Siga los pasos de [Creación de un volumen NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md) para crear el volumen de NFS. Durante el proceso de creación del volumen, en la pestaña **Protocolo**, habilite la opción **LDAP**.   

    ![Captura de pantalla que muestra la página Crear un volumen con la opción LDAP.](../media/azure-netapp-files/create-nfs-ldap.png)  

7. Opcional: puede habilitar usuarios del cliente NFS locales que no estén presentes en el servidor LDAP de Windows para que accedan a un volumen NFS que tenga habilitado LDAP con grupos extendidos. Para ello, habilite la opción **Permitir usuarios locales de NFS con LDAP** de la siguiente manera:
    1. Haga clic en **Conexiones de Active Directory**.  En una conexión de Active Directory existente, haga clic en el menú contextual (los tres puntos `…`) y seleccione **Editar**.  
    2. En la ventana **Edit Active Directory settings** (Editar configuración de Active Directory) que aparece, seleccione la opción **Permitir usuarios locales de NFS con LDAP**.  

    ![Captura de pantalla que muestra la opción Permitir usuarios locales de NFS con LDAP](../media/azure-netapp-files/allow-local-nfs-users-with-ldap.png)  

## <a name="next-steps"></a>Pasos siguientes  

* [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Creación y administración de conexiones de Active Directory](create-active-directory-connections.md)
* [Solución de problemas de volúmenes LDAP](troubleshoot-ldap-volumes.md)
