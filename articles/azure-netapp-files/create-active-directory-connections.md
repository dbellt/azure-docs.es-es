---
title: Creación y administración de conexiones de Active Directory para Azure NetApp Files | Microsoft Docs
description: En este artículo se muestra cómo crear y administrar conexiones de Active Directory para Azure NetApp Files.
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
ms.date: 04/06/2021
ms.author: b-juche
ms.openlocfilehash: 27c2ab96106bbfcc05b8fa12daf9b6f7b816c5c7
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106579974"
---
# <a name="create-and-manage-active-directory-connections-for-azure-netapp-files"></a>Creación y administración de conexiones de Active Directory para Azure NetApp Files

Varias características de Azure NetApp Files requieren que tenga una conexión de Active Directory.  Por ejemplo, debe tener una conexión de Active Directory para poder crear un [volumen SMB](azure-netapp-files-create-volumes-smb.md) o un [volumen de protocolo dual](create-volumes-dual-protocol.md).  En este artículo se muestra cómo crear y administrar conexiones de Active Directory para Azure NetApp Files.

## <a name="before-you-begin"></a>Antes de empezar  

Debe haber configurado un grupo de capacidad.   
[Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)   
Debe haber una subred delegada en Azure NetApp Files.  
[Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md)

## <a name="requirements-for-active-directory-connections"></a>Requisitos para las conexiones de Active Directory

 Los requisitos para las conexiones de Active Directory son los siguientes: 

* La cuenta de administrador que use debe ser capaz de crear cuentas de máquina en la ruta de acceso de la unidad organizativa (OU) que se especifique.  

* Los puertos adecuados deben estar abiertos en el servidor de Windows Active Directory (AD) correspondiente.  
    Los puertos necesarios son los siguientes: 

    |     Servicio           |     Port     |     Protocolo     |
    |-----------------------|--------------|------------------|
    |    Servicios web de AD    |    9389      |    TCP           |
    |    DNS                |    53        |    TCP           |
    |    DNS                |    53        |    UDP           |
    |    ICMPv4             |    N/D       |    Echo Reply    |
    |    Kerberos           |    464       |    TCP           |
    |    Kerberos           |    464       |    UDP           |
    |    Kerberos           |    88        |    TCP           |
    |    Kerberos           |    88        |    UDP           |
    |    LDAP               |    389       |    TCP           |
    |    LDAP               |    389       |    UDP           |
    |    LDAP               |    3268      |    TCP           |
    |    Nombre de NetBIOS       |    138       |    UDP           |
    |    SAM/LSA            |    445       |    TCP           |
    |    SAM/LSA            |    445       |    UDP           |
    |    w32time            |    123       |    UDP           |

* La topología del sitio para la instancia de Active Directory Domain Services de destino debe cumplir las directrices, en particular, la red virtual de Azure donde se implementa Azure NetApp Files.  

    El espacio de direcciones de la red virtual donde se implementa Azure NetApp Files debe agregarse a un sitio de Active Directory nuevo o existente (donde se encuentre un controlador de dominio al que Azure NetApp Files pueda acceder). 

* Los servidores DNS especificados deben ser accesibles desde la [subred delegada](./azure-netapp-files-delegate-subnet.md) de Azure NetApp Files.  

    Consulte [Directrices para el planeamiento de red de Azure NetApp Files](./azure-netapp-files-network-topologies.md) para ver topologías de red admitidas.

    Los grupos de seguridad de red (NSG) y los firewalls deben tener reglas configuradas correctamente para permitir solicitudes de Active Directory y de tráfico DNS. 

* La subred delegada en Azure NetApp Files debe poder tener acceso a todos los controladores de dominio Active Directory Domain Services (ADDS) del dominio, incluidos todos los controladores de dominio locales y remotos. De lo contrario, puede producirse una interrupción del servicio.  

    Si tiene controladores de dominio a los que no se puede acceder mediante la subred delegada de Azure NetApp Files, puede especificar un sitio de Active Directory durante la creación de la conexión a Active Directory.  Azure NetApp Files solo debe comunicarse con los controladores de dominio del sitio en el que se encuentra el espacio de direcciones de la subred delegada de Azure NetApp Files.

    Consulte [Diseño de la topología de sitio](/windows-server/identity/ad-ds/plan/designing-the-site-topology) para obtener información acerca de los sitios y servicios de AD. 
    
* Para habilitar el cifrado AES para la autenticación de AD, active la casilla **cifrado AES** en la ventana [Unir Active Directory](#create-an-active-directory-connection). Azure NetApp Files admite los tipos de cifrado DES, Kerberos AES 128 y Kerberos AES 256 (de la menos segura a la más segura). Si habilita el cifrado AES, las credenciales de usuario utilizadas para unir Active Directory deben tener habilitada la opción de cuenta más alta correspondiente que coincida con las funcionalidades habilitadas para Active Directory.    

    Por ejemplo, si Active Directory solo tiene la funcionalidad AES-128, debe habilitar la opción de cuenta AES-128 para las credenciales de usuario. Si Active Directory tiene la funcionalidad AES-256, debe habilitar la opción de cuenta AES-256 (que también es compatible con AES-128). Si Active Directory no tiene ninguna funcionalidad de cifrado de Kerberos, Azure NetApp Files utiliza DES de forma predeterminada.  

    Puede habilitar las opciones de cuenta en las propiedades de Microsoft Management Console (MMC) de usuarios y equipos de Active Directory:   

    ![Complemento MMC de usuarios y equipos de Active Directory](../media/azure-netapp-files/ad-users-computers-mmc.png)

* Azure NetApp Files admite [Firma LDAP](/troubleshoot/windows-server/identity/enable-ldap-signing-in-windows-server), lo que permite una transmisión segura del tráfico LDAP entre el servicio Azure NetApp Files y los [controladores de dominio de Active Directory](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) de destino. Si sigue las instrucciones del aviso de Microsoft [ADV190023](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190023) para la firma LDAP, debe habilitar la característica de firma LDAP en Azure NetApp Files activando la casilla **Firma LDAP** en la ventana [Unir Active Directory](#create-an-active-directory-connection). 

    La configuración del [enlace de canal LDAP](https://support.microsoft.com/help/4034879/how-to-add-the-ldapenforcechannelbinding-registry-entry) por sí sola no tiene ningún efecto en el servicio Azure NetApp Files. Pero si usa el enlace de canal LDAP y LDAP seguro (por ejemplo, LDAPS o `start_tls`), se produce un error en la creación del volumen SMB.

* En el caso de DNS no integrado en AD, debe agregar un registro D o PTR de DNS para permitir que Azure NetApp Files funcione mediante un "nombre descriptivo". 

## <a name="decide-which-domain-services-to-use"></a>Decisión sobre los servicios de dominio que se van a usar 

Azure NetApp Files admite [Active Directory Domain Services](/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology) (ADDS) y Azure Active Directory Domain Services (AADDS) para las conexiones de AD.  Antes de crear una conexión de AD, debe decidir si va a usar ADDS o AADDS.  

Para obtener más información, consulte [Comparación de Active Directory Domain Services autoadministrado, Azure Active Directory y Azure Active Directory Domain Services administrado](../active-directory-domain-services/compare-identity-solutions.md). 

### <a name="active-directory-domain-services"></a>Active Directory Domain Services

Puede usar su ámbito favorito de [Sitios y servicios de Active Directory](/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology) para Azure NetApp Files. Esta opción habilita las lecturas y escrituras en los controladores de dominio de Active Directory Domain Services (ADDS) a los que [tiene acceso Azure NetApp Files](azure-netapp-files-network-topologies.md). También impide que el servicio se comunique con los controladores de dominio que no están en el sitio especificado en Sitios y servicios de Active Directory. 

Para encontrar el nombre del sitio al usar ADDS, puede ponerse en contacto con el grupo administrativo de su organización que se responsabiliza de Active Directory Domain Services. En el ejemplo siguiente se muestra el complemento de Sitios y servicios de Active Directory en el que se muestra el nombre del sitio: 

![Sitios y servicios de Active Directory](../media/azure-netapp-files/azure-netapp-files-active-directory-sites-services.png)

Al configurar una conexión de AD para Azure NetApp Files, especifique el nombre del sitio en el ámbito para el campo **Nombre de sitio de Active Directory**.

### <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services 

Para obtener instrucciones y la configuración de Azure Active Directory Domain Services (AADDS), consulte [Documentación de Azure AD Domain Services](../active-directory-domain-services/index.yml).

Se aplican consideraciones de AADDS adicionales para Azure NetApp Files: 

* Asegúrese de que la red virtual o la subred donde está implementado AADDS se encuentra en la misma región de Azure que la implementación de Azure NetApp Files.
* Si usa otra red virtual en la región donde está implementado Azure NetApp Files, debe crear un emparejamiento entre las dos redes virtuales.
* Azure NetApp Files admite los tipos `user` y `resource forest`.
* Como tipo de sincronización, puede seleccionar `All` o `Scoped`.   
    Si selecciona `Scoped`, asegúrese de que está seleccionado el grupo de Azure AD correcto para acceder a los recursos compartidos de SMB.  Si no está seguro, puede usar el tipo de sincronización `All`.
* Se requiere el uso de la SKU Enterprise o Premium. No se admite la SKU estándar.

Cuando cree una conexión de Active Directory, tenga en cuenta las siguientes características específicas de AADDS:

* Puede encontrar la información de **DNS principal**, **DNS secundario** y **Nombre de dominio DNS de AD** en el menú AADDS.  
En el caso de los servidores DNS, se usarán dos direcciones IP para configurar la conexión de Active Directory. 
* La **ruta de acceso de unidad organizativa** es `OU=AADDC Computers`.  
Esta opción está configurada en **Conexiones de Active Directory** debajo de **Cuenta de NetApp**:

  ![Ruta de acceso de unidad organizativa](../media/azure-netapp-files/azure-netapp-files-org-unit-path.png)

* Las credenciales de **Nombre de usuario** pueden ser cualquier usuario que sea miembro del grupo de **administradores de controladores de dominio de Azure AD**.


## <a name="create-an-active-directory-connection"></a>Creación de una conexión de Active Directory

1. En la cuenta de NetApp, haga clic en **Conexiones de Active Directory** y, a continuación, haga clic en **Unir**.  

    ![Conexiones de Active Directory](../media/azure-netapp-files/azure-netapp-files-active-directory-connections.png)

2. En la ventana Unir Active Directory, especifique la siguiente información según los servicios de dominio que desee utilizar:  

    Para obtener información específica sobre los servicios de dominio que utiliza, consulte [Decisión sobre los servicios de dominio que se van a usar](#decide-which-domain-services-to-use). 

    * **DNS principal**  
        Se trata del DNS que se requiere para las operaciones de autenticación de SMB y la unión a un dominio de Active Directory. 
    * **DNS secundario**   
        Este es el servidor DNS secundario para garantizar los servicios de nombre redundantes. 
    * **Nombre de dominio DNS de AD**  
        Se trata del nombre de dominio de Active Directory Domain Services al que desea unirse.
    * **Nombre de sitio de AD**  
        Este es el nombre del sitio al que se limitará la detección de controladores de dominio. Debe coincidir con el nombre del sitio en Sitios y servicios de Active Directory.
    * **Prefijo SMB (cuenta de equipo)**  
        Este es el prefijo de nomenclatura para la cuenta de máquina en Active Directory que Azure NetApp Files va a usar para la creación de nuevas cuentas.

        Por ejemplo, si el estándar de nomenclatura que su organización usa para los servidores de archivos es NAS-01, NAS-02...,NAS-045, escribiría "NAS" para el prefijo. 

        El servicio creará cuentas de máquina adicionales en Active Directory según sea necesario.

        > [!IMPORTANT] 
        > El cambio de nombre del prefijo del servidor de SMB después de crear la conexión de Active Directory es perjudicial. Tendrá que volver a montar los recursos compartidos de SMB existentes después de cambiar el nombre del prefijo del servidor de SMB.

    * **Ruta de acceso de unidad organizativa**  
        Se trata de la ruta de acceso LDAP para la unidad organizativa (UO) donde se crearán las cuentas de máquina del servidor SMB. Es decir, UO=segundo nivel, UO=primer nivel. 

        Si usa Azure NetApp Files con Azure Active Directory Domain Services, la ruta de acceso de la unidad organizativa es `OU=AADDC Computers` cuando configura Active Directory para su cuenta de NetApp.

        ![Unir Active Directory](../media/azure-netapp-files/azure-netapp-files-join-active-directory.png)

    * **Cifrado AES**   
        Seleccione esta casilla si desea habilitar el cifrado AES para un volumen SMB. Consulte [Requisitos para las conexiones de Active Directory](#requirements-for-active-directory-connections) para ver los requisitos. 

        ![Cifrado AES de Active Directory](../media/azure-netapp-files/active-directory-aes-encryption.png)

        La característica **Cifrado AES** está actualmente en su versión preliminar. Si es la primera vez que usa esta característica, regístrela antes de usarla: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFAesEncryption
        ```

        Compruebe el estado del registro de la característica: 

        > [!NOTE]
        > **RegistrationState** puede estar en el estado `Registering` hasta 60 minutos antes de cambiar a `Registered`. Espere hasta que el estado sea `Registered` antes de continuar.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFAesEncryption
        ```
        
        También puede usar los comandos de la [CLI de Azure](/cli/azure/feature) `az feature register` y `az feature show` para registrar la característica y mostrar el estado del registro. 

    * **Firma LDAP**   
        Seleccione esta casilla para habilitar la firma LDAP. Esta funcionalidad permite realizar búsquedas LDAP seguras entre el servicio de Azure NetApp Files y los controladores de dominio de [Active Directory Domain Services](/windows/win32/ad/active-directory-domain-services) especificados por el usuario. Para más información, consulte [ADV190023 | Guía de Microsoft para habilitar el enlace de canal LDAP y la firma LDAP](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190023).  

        ![Firma LDAP de Active Directory](../media/azure-netapp-files/active-directory-ldap-signing.png) 

        La característica **Firma LDAP** actualmente está en versión preliminar. Si es la primera vez que usa esta característica, regístrela antes de usarla: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapSigning
        ```

        Compruebe el estado del registro de la característica: 

        > [!NOTE]
        > **RegistrationState** puede estar en el estado `Registering` hasta 60 minutos antes de cambiar a `Registered`. Espere hasta que el estado sea `Registered` antes de continuar.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapSigning
        ```
        
        También puede usar los comandos de la [CLI de Azure](/cli/azure/feature) `az feature register` y `az feature show` para registrar la característica y mostrar el estado del registro. 

     * **Usuarios con privilegios de seguridad**   <!-- SMB CA share feature -->   
        Puede conceder privilegios de seguridad (`SeSecurityPrivilege`) a los usuarios que requieran privilegios elevados para acceder a los volúmenes de Azure NetApp files. Se permitirá que las cuentas de usuario especificadas realicen determinadas acciones en recursos compartidos de SMB de Azure NetApp Files que requieren privilegios de seguridad no asignados de forma predeterminada a los usuarios del dominio.   

        Por ejemplo, a las cuentas de usuario que se usan para instalar SQL Server en determinados escenarios se les debe conceder un privilegio de seguridad elevado. Si usa una cuenta que no es de administrador (dominio) para instalar SQL Server y la cuenta no tiene asignado el privilegio de seguridad, deberá agregar privilegios de seguridad a la cuenta.  

        > [!IMPORTANT]
        > El uso de la característica **Usuarios con privilegios de seguridad** requiere que envíe una solicitud de lista de espera desde la **[página de envío de la lista de espera de la versión preliminar pública de recursos compartidos con disponibilidad continua de SMB para Azure NetApp Files](https://aka.ms/anfsmbcasharespreviewsignup)** . Para poder usar esta característica, espere hasta recibir un correo electrónico de confirmación oficial del equipo de Azure NetApp Files.        
        > 
        > El uso de esta característica es opcional y solo se admite para SQL Server. La cuenta de dominio usada para instalar SQL Server debe existir antes de agregarla al campo **Security privilege users** (Usuarios con privilegios de seguridad). Al agregar la cuenta del instalador de SQL Server a **Security privilege users** (Usuarios con privilegios de seguridad), el servicio Azure NetApp Files podría validar la cuenta poniéndose en contacto con el controlador de dominio. El comando podría producir un error si no puede ponerse en contacto con el controlador de dominio.  

        Para más información sobre `SeSecurityPrivilege` y SQL Server, consulte [Error en la instalación de SQL Server si la cuenta de configuración no tiene determinados permisos del usuario](/troubleshoot/sql/install/installation-fails-if-remove-user-right).

        ![Captura de pantalla que muestra el cuadro de usuarios con privilegios de seguridad de la ventana de conexiones de Active Directory.](../media/azure-netapp-files/security-privilege-users.png) 

     * **Usuarios de la directiva de copia de seguridad**  
        Puede incluir cuentas adicionales que requieran privilegios elevados para la cuenta de equipo creada para su uso con Azure NetApp Files. Se permitirá a las cuentas especificadas cambiar los permisos de NTFS en el nivel de archivo o carpeta. Por ejemplo, puede especificar una cuenta de servicio sin privilegios que se usa para migrar los datos a un recurso compartido de archivos de SMB en Azure NetApp Files.  

        ![Usuarios de la directiva de copia de seguridad de Active Directory](../media/azure-netapp-files/active-directory-backup-policy-users.png)

        La característica **Usuarios de la directiva de copia de seguridad** se encuentra actualmente en la versión preliminar. Si es la primera vez que usa esta característica, regístrela antes de usarla: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFBackupOperator
        ```

        Compruebe el estado del registro de la característica: 

        > [!NOTE]
        > **RegistrationState** puede estar en el estado `Registering` hasta 60 minutos antes de cambiar a `Registered`. Espere hasta que el estado sea `Registered` antes de continuar.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFBackupOperator
        ```
        
        También puede usar los comandos de la [CLI de Azure](/cli/azure/feature) `az feature register` y `az feature show` para registrar la característica y mostrar el estado del registro. 

    * Las credenciales, incluidos el **nombre de usuario** y la **contraseña**

        ![Credenciales de Active Directory](../media/azure-netapp-files/active-directory-credentials.png)

3. Haga clic en **Unir**.  

    Aparece la conexión de Active Directory que creó.

    ![Conexiones de Active Directory creadas](../media/azure-netapp-files/azure-netapp-files-active-directory-connections-created.png)

## <a name="next-steps"></a>Pasos siguientes  

* [Creación de un volumen SMB](azure-netapp-files-create-volumes-smb.md)
* [Creación de un volumen de protocolo dual](create-volumes-dual-protocol.md)
* [Configuración del cifrado Kerberos de NFSv4.1](configure-kerberos-encryption.md)
* [Install a new Active Directory forest using Azure CLI](/windows-server/identity/ad-ds/deploy/virtual-dc/adds-on-azure-vm) (Instalación de un nuevo bosque de Active Directory en la CLI de Azure) 
