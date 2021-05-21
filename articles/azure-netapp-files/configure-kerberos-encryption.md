---
title: Configuración del cifrado Kerberos de NFSv4.1 para Azure NetApp Files | Microsoft Docs
description: Se describe cómo configurar el cifrado Kerberos de NFSv4.1 para Azure NetApp Files y el impacto en el rendimiento.
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
ms.date: 05/06/2021
ms.author: b-juche
ms.openlocfilehash: 2b920b7a5794e1ee56b8ccc2c1d3aef45a1b5fe2
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483828"
---
# <a name="configure-nfsv41-kerberos-encryption-for-azure-netapp-files"></a>Configuración del cifrado Kerberos de NFSv4.1 para Azure NetApp Files

Azure NetApp Files admite el cifrado de cliente NFS en los modos de Kerberos (krb5, krb5i y krb5p) con el cifrado AES-256. En este artículo se describen las configuraciones necesarias para usar un volumen NFSv4.1 con el cifrado Kerberos.

## <a name="requirements"></a>Requisitos

Los siguientes requisitos se aplican al cifrado de cliente de NFSv4.1: 

* Conexión con Active Directory Domain Services (AD DS) o Azure Active Directory Domain Services (AADDS) para facilitar la creación de vales de Kerberos 
* Creación de registros A/PTR de DNS para las direcciones IP del servidor NFS de Azure NetApp Files y el cliente
* Un cliente Linux  
    En este artículo se proporcionan instrucciones para los clientes RHEL y Ubuntu.  Otros clientes funcionarán con pasos de configuración similares. 
* Acceso al servidor NTP  
    Puede usar uno de los controladores de dominio del Controlador de dominio de Active Directory (AD DC) de uso común.

## <a name="create-an-nfs-kerberos-volume"></a>Creación de un volumen Kerberos de NFS

1.  Siga los pasos de [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md) para crear el volumen de NFSv4.1.   

    En la página Crear un volumen, establezca la versión de NFS en **NFSv4.1** y establezca Kerberos en **Habilitado**.

    > [!IMPORTANT] 
    > No se puede modificar la selección de habilitación de Kerberos una vez creado el volumen.

    ![Creación del volumen NFSv4.1 de Kerberos](../media/azure-netapp-files/create-kerberos-volume.png)  

2. Seleccione **Directiva de exportación** para que coincida con el nivel deseado de la opción de acceso y seguridad (Kerberos 5, Kerberos 5i, o Kerberos 5p) del volumen.   

    Para ver el impacto en el rendimiento de Kerberos, vea [Impacto en el rendimiento de Kerberos en NFSv4.1](#kerberos_performance).  

    También puede modificar los métodos de seguridad de Kerberos para el volumen; para ello, haga clic en Directiva de exportación en el panel de navegación de Azure NetApp Files.

3.  Haga clic en **Revisar y crear** para crear el volumen de NFSv4.1.

## <a name="configure-the-azure-portal"></a>Configuración en Azure Portal 

1.  Siga las instrucciones de [Creación de una conexión de Active Directory](create-active-directory-connections.md).  

    Kerberos requiere la creación de al menos una cuenta de equipo en Active Directory. La información de la cuenta que proporcione se usa para crear las cuentas de los volúmenes de Kerberos SMB *y* NFSv4.1. Esta máquina se crea automáticamente durante la creación del volumen.

2.  En **Dominio Kerberos**, escriba el **Nombre del servidor de AD** y la dirección **IP de KDC**.

    El servidor de AD y la dirección IP del KDC pueden ser el mismo servidor. Esta información se usa para crear la cuenta de la máquina del SPN usada por Azure NetApp Files. Una vez creada la cuenta de equipo, Azure NetApp Files usará los registros del servidor DNS para buscar servidores KDC adicionales según sea necesario. 

    ![Dominio Kerberos](../media/azure-netapp-files/kerberos-realm.png)
 
3.  Haga clic en **Conectar** para guardar la configuración.

## <a name="configure-active-directory-connection"></a>Configuración de la conexión de Active Directory 

La configuración de NFSv4.1 de Kerberos crea dos cuentas de equipo en Active Directory:
* Una cuenta de equipo para los recursos compartidos SMB.
* Una cuenta de equipo para NFSv4.1: puede identificar esta cuenta mediante el prefijo `NFS-`. 

Una vez creado el primer volumen Kerberos NFSv4.1, establezca el tipo de cifrado de la cuenta de equipo mediante el siguiente comando de PowerShell:

`Set-ADComputer $NFSCOMPUTERACCOUNT -KerberosEncryptionType AES256`

## <a name="configure-the-nfs-client"></a>Configuración del cliente NFS 

Siga las instrucciones de [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md) para configurar el cliente NFS.  

## <a name="mount-the-nfs-kerberos-volume"></a><a name="kerberos_mount"></a>Montaje del volumen Kerberos de NFS

1. En la página **Volúmenes**, seleccione el volumen NFS que desea montar.

2. Seleccione **Instrucciones de montaje** en el volumen para mostrar las instrucciones.

    Por ejemplo: 

    ![Instrucciones de montaje para volúmenes de Kerberos](../media/azure-netapp-files/mount-instructions-kerberos-volume.png)  

3. Cree el directorio (punto de montaje) para el nuevo volumen.  

4. Establezca el tipo de cifrado predeterminado en AES 256 para la cuenta de equipo:  
    `Set-ADComputer $NFSCOMPUTERACCOUNT -KerberosEncryptionType AES256 -Credential $ANFSERVICEACCOUNT`

    * Solo tiene que ejecutar este comando una vez para cada cuenta de equipo.
    * Puede ejecutar este comando desde un controlador de dominio o desde un equipo con [RSAT](https://support.microsoft.com/help/2693643/remote-server-administration-tools-rsat-for-windows-operating-systems) instalado. 
    * La variable `$NFSCOMPUTERACCOUNT` es la cuenta de equipo creada en Active Directory al implementar el volumen de Kerberos. Se trata de la cuenta que tiene el prefijo `NFS-`. 
    * La variable `$ANFSERVICEACCOUNT` es una cuenta de usuario de Active Directory sin privilegios con controles delegados en la unidad organizativa en la que se ha creado la cuenta de equipo. 

5. Monte el volumen en el host: 

    `sudo mount -t nfs -o sec=krb5p,rw,hard,rsize=1048576,wsize=1048576,vers=4.1,tcp $ANFEXPORT $ANFMOUNTPOINT`

    * La variable `$ANFEXPORT` es la ruta de acceso `host:/export` que se encuentra en las instrucciones de montaje.
    * La variable `$ANFMOUNTPOINT` es la carpeta creada por el usuario en el host de Linux.

## <a name="performance-impact-of-kerberos-on-nfsv41"></a><a name="kerberos_performance"></a>Impacto en el rendimiento de Kerberos en NFSv4.1 

Debe comprender las opciones de seguridad disponibles para los volúmenes NFSv4.1, los vectores de rendimiento probados y el impacto esperado en el rendimiento de Kerberos. Para más información, consulte [Impacto en el rendimiento de Kerberos en los volúmenes NFSv4.1](performance-impact-kerberos.md).  

## <a name="next-steps"></a>Pasos siguientes  

* [Impacto en el rendimiento de Kerberos en los volúmenes NFSv4.1](performance-impact-kerberos.md)
* [Solución de problemas de los volúmenes NFSv4.1 de Kerberos](troubleshoot-nfsv41-kerberos-volumes.md)
* [Preguntas más frecuentes acerca de Azure NetApp Files](azure-netapp-files-faqs.md)
* [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Creación de una conexión de Active Directory](create-active-directory-connections.md)
* [Configuración de un cliente NFS para Azure NetApp Files](configure-nfs-clients.md) 
