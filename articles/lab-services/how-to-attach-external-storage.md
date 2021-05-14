---
title: Uso de almacenamiento de archivos externo en servicios de laboratorio | Microsoft Docs
description: Obtenga información sobre cómo configurar un laboratorio que use almacenamiento de archivos externo en servicios de laboratorio.
author: emaher
ms.topic: article
ms.date: 03/30/2021
ms.author: enewman
ms.openlocfilehash: 70be69cad59cd00ef9feaa78ad2294c64626d07a
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125686"
---
# <a name="using-external-file-storage-in-lab-services"></a>Uso de almacenamiento de archivos externo en servicios de laboratorio

En este artículo se tratarán algunas de las opciones de almacenamiento de archivos externos al utilizar Azure Lab Services.  [Azure Files](https://azure.microsoft.com/services/storage/files/) ofrece recursos compartidos de archivos en la nube totalmente administrados a los que [se puede acceder mediante SMB 2.1 and SMB 3.0.](../storage/files/storage-how-to-use-files-windows.md)  Un recurso compartido de Azure Files puede conectarse de forma pública o privada dentro de una red virtual.  Además, se puede configurar para utilizar las credenciales de AD de alumno para conectarse al recurso compartido.  El uso de Azure NetApp Files con volúmenes NFS para máquinas Linux es otra opción para el almacenamiento de archivos externos con Azure Lab Services.  

## <a name="deciding-which-solution-to-use"></a>Decidir qué solución usar

Cada solución tiene requisitos y capacidades diferentes.  En la tabla siguiente se enumeran los puntos importantes que se deben tener en cuenta para cada solución.  

|     Solución    |    Consideraciones importantes    |
| -------------- | ------------------------ |
| [Recurso compartido de Azure Files con el punto de conexión público](#azure-files-share) | <ul><li>Todos tienen acceso de lectura y escritura.</li><li>No se requiere emparejamiento de Vnet.</li><li>Accesible para todas las máquinas virtuales, no solo máquinas virtuales de laboratorio.</li><li>Si usa Linux, los alumnos tendrán acceso a la clave de la cuenta de almacenamiento.</li></ul> |
| [Recurso compartido de Azure Files con punto de conexión privado](#azure-files-share) | <ul><li>Todos tienen acceso de lectura y escritura.</li><li>Se requiere emparejamiento de Vnet.</li><li>Accesible solo a las máquinas virtuales de la misma red (o red emparejada) que la cuenta de almacenamiento.</li><li>Si usa Linux, los alumnos tendrán acceso a la clave de la cuenta de almacenamiento.</li></ul> |
| [Azure Files con autorización de base de identidad](#azure-files-with-identity-base-authorization) | <ul><li>Se pueden establecer permisos de acceso de lectura o de lectura y escritura para la carpeta o el archivo.</li><li>Se requiere emparejamiento de Vnet.</li><li>La cuenta de almacenamiento debe estar conectada a Active Directory.</li><li>Los dispositivos deben estar unidos a un dominio.</li><li>La clave de la cuenta de almacenamiento no se utiliza para que los alumnos se conecten al recurso compartido de archivos.</li></ul> |
| [Archivos de NetApp con volúmenes NFS](#netapp-files-with-nfs-volumes) | <ul><li>Se puede establecer el acceso de lectura o de lectura y escritura para los volúmenes.</li><li>Los permisos se establecen mediante la dirección IP de la máquina virtual de alumno.</li><li>Se requiere emparejamiento de Vnet.</li><li>Puede que tenga que registrarse para usar el servicio de archivos de NetApp.</li><li>Solo Linux.</li></ul>

El costo de usar el almacenamiento externo no se incluye en el costo de usar Azure Lab Services.  Para obtener más información sobre los precios, consulte [Precios de Azure Files](https://azure.microsoft.com/pricing/details/storage/files/) y [Precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/netapp/).

## <a name="azure-files-share"></a>Recurso compartido de Azure Files

Se tiene acceso a los recursos compartidos de Azure mediante un punto de conexión público o privado.  Los recursos compartidos de Azure se montan con la clave de cuenta de almacenamiento como contraseña.  Con este enfoque, todos tienen acceso de lectura y escritura al recurso compartido.

Si usa un punto de conexión público al recurso compartido de Azure Files, es importante recordar lo siguiente:

- La red virtual de la cuenta de almacenamiento no tiene que estar emparejada con la cuenta de laboratorio.  El recurso compartido se puede crear en cualquier momento antes de publicar la máquina virtual de la plantilla.
- Se puede tener acceso al recurso compartido desde cualquier equipo si un usuario tiene la clave de la cuenta de almacenamiento.  
- Los alumnos de Linux pueden ver la clave de la cuenta de almacenamiento.  Las credenciales para montar un recurso compartido de Azure Files se almacenan en máquinas virtuales Linux `{file-share-name}.cred` y son legibles por sudo.  Dado que a los alumnos se les concede acceso sudo de forma predeterminada en las máquinas virtuales del servicio de laboratorio de Azure, pueden leer la clave de la cuenta de almacenamiento.  Si el punto de conexión de la cuenta de almacenamiento es público, los alumnos pueden obtener acceso al recurso compartido fuera de la máquina virtual de los alumnos.  Considere la posibilidad de rotar la clave de cuenta de almacenamiento una vez finalizada la clase y usando recursos compartidos privados.

Si usa un punto de conexión privado en el recurso compartido de Azure Files, es importante recordar lo siguiente:

- El acceso está restringido al tráfico que se origina desde la red privada y no se puede acceder a él a través de la red pública de Internet.  Solo las máquinas virtuales de la red virtual privada, las máquinas virtuales de una red emparejadas con la red virtual privada o las máquinas conectadas a una VPN para la red privada pueden acceder al recurso compartido.  
- Los alumnos de Linux pueden ver la clave de la cuenta de almacenamiento.  Las credenciales para montar un recurso compartido de Azure Files se almacenan en máquinas virtuales Linux `{file-share-name}.cred` y son legibles por sudo.  Dado que a los alumnos se les concede acceso sudo de forma predeterminada en las máquinas virtuales del servicio de laboratorio de Azure, pueden leer la clave de la cuenta de almacenamiento.  Considere la posibilidad de rotar la clave de la cuenta de almacenamiento una vez finalizada la clase.
- Este enfoque requiere que la red virtual de recurso compartido de archivos esté emparejada con la cuenta de laboratorio.  La red virtual de la cuenta de Azure Storage se debe emparejar con la red virtual de la cuenta de laboratorio **antes** de que se cree el laboratorio.

> [!NOTE]
> Los recursos compartidos de archivos de más de 5 TB solo están disponibles para [[cuentas de almacenamiento con redundancia local (LRS)]](../storage/files/storage-files-how-to-create-large-file-share.md#restrictions).

Siga estos pasos para crear una máquina virtual conectada a un recurso compartido de archivos de Azure.

1. Creación de una [cuenta de Azure Storage](../storage/files/storage-how-to-create-file-share.md). En la página "método de conectividad", elija extremo público o punto de conexión privado.
2. Si usa, cree un [punto de conexión privado](../private-link/tutorial-private-endpoint-storage-portal.md) para que los recursos compartidos de archivos sean accesibles desde la red virtual.  Cree una [zona DNS privada](../dns/private-dns-privatednszone.md) o use una existente. Las zonas de Azure DNS privadas proporcionan la resolución de nombres dentro de una red virtual.
3. Cree un [recurso compartido de archivos de Azure](../storage/files/storage-how-to-create-file-share.md). El recurso compartido de archivos es accesible mediante el nombre de host público de la cuenta de almacenamiento.
4. Montaje del recurso compartido de archivos de Azure en la plantilla de máquina virtual:
    - [[Windows]](../storage/files/storage-how-to-use-files-windows.md)
    - [[Linux]](../storage/files/storage-how-to-use-files-linux.md).  Consulte [uso de Azure Files con Linux](#using-azure-files-with-linux) para evitar el montaje de problemas en máquinas virtuales de alumnos.
5. [Publicación](how-to-create-manage-template.md#publish-the-template-vm) de la plantilla de máquina virtual.

> [!IMPORTANT]
> Asegúrese de que el Firewall no está bloqueando la conexión SMB saliente a través del puerto 445. De forma predeterminada, SMB se permite para las máquinas virtuales de Azure.

### <a name="using-azure-files-with-linux"></a>Uso de Azure Files con Linux

Si usa las _instrucciones predeterminadas_ para montar un recurso compartido de archivos de Azure, parecerá que el recurso compartido de archivos desaparece en las máquinas virtuales de los alumnos una vez publicada la plantilla.  A continuación se muestra un script modificado que soluciona este problema.  

```bash
#!/bin/bash

# Assign variables values for your storage account and file share
storage_account_name=""
storage_account_key=""
fileshare_name=""

# Do not use 'mnt' for mount directory.
# Using ‘mnt’ will cause issues on student VMs.
mount_directory="prm-mnt" 

sudo mkdir /$mount_directory/$fileshare_name
if [ ! -d "/etc/smbcredentials" ]; then
    sudo mkdir /etc/smbcredentials
fi
if [ ! -f "/etc/smbcredentials/$storage_account_name.cred" ]; then
    sudo bash -c "echo ""username=$storage_account_name"" >> /etc/smbcredentials/$storage_account_name.cred"
    sudo bash -c "echo ""password=$storage_account_key"" >> /etc/smbcredentials/$storage_account_name.cred"
fi
sudo chmod 600 /etc/smbcredentials/$storage_account_name.cred

sudo bash -c "echo ""//$storage_account_name.file.core.windows.net/$fileshare_name /$mount_directory/$fileshare_name cifs nofail,vers=3.0,credentials=/etc/smbcredentials/$storage_account_name.cred,dir_mode=0777,file_mode=0777,serverino"" >> /etc/fstab"
sudo mount -t cifs //$storage_account_name.file.core.windows.net/$fileshare_name /$mount_directory/$fileshare_name -o vers=3.0,credentials=/etc/smbcredentials/$storage_account_name.cred,dir_mode=0777,file_mode=0777,serverino
```

Si la máquina virtual de plantilla ya está publicada que monta el recurso compartido de archivos de Azure en el directorio `/mnt`, el alumno puede realizar una de las siguientes acciones.

- Mueva la instrucción para montar `/mnt` en la parte superior del archivo `/etc/fstab`.  
- Modifique la instrucción para montar `/mnt/{file-share-name}` en otro directorio como `/prm-mnt/{file-share-name}`.

Los alumnos deben ejecutar `mount -a` para volver a montar los directorios.

Para obtener más información general sobre el uso de recursos compartidos de archivos con Linux, consulte [uso de Azure Files con Linux](../storage/files/storage-how-to-use-files-linux.md).

## <a name="azure-files-with-identity-base-authorization"></a>Azure Files con autorización de base de identidad

También se puede tener acceso a los recursos compartidos de archivos Azure Files mediante la autenticación de AD si

1. La máquina virtual de alumno está unida a un dominio.
2. La autenticación de AD está [habilitada en la cuenta Azure Storage](../storage/files/storage-files-active-directory-overview.md) que hospeda el recurso compartido de archivos.  

La unidad de red está montada en la máquina virtual mediante la identidad del usuario, no la clave de la cuenta de almacenamiento.  El acceso a la cuenta de almacenamiento puede usar puntos de conexión públicos o privados.

Vamos a repasar algunos puntos importantes de este enfoque.

- Los permisos se pueden establecer en un directorio o en un nivel de archivo.
- Las credenciales del usuario actual se usan para autenticarse en el recurso compartido de archivos.

Si usa un punto de conexión público al recurso compartido de Azure Files, es importante recordar lo siguiente:

- La red virtual de la cuenta de almacenamiento no tiene que estar emparejada con la cuenta de laboratorio.  El recurso compartido se puede crear en cualquier momento antes de publicar la máquina virtual de la plantilla.

Si usa un punto de conexión privado en el recurso compartido de Azure Files, es importante recordar lo siguiente:

- El acceso está restringido al tráfico que se origina desde la red privada y no se puede acceder a él a través de la red pública de Internet.  Solo las máquinas virtuales de la red virtual privada, las máquinas virtuales de una red emparejadas con la red virtual privada o las máquinas conectadas a una VPN para la red privada pueden acceder al recurso compartido.  
- Este enfoque requiere que la red virtual de recurso compartido de archivos esté emparejada con la cuenta de laboratorio.  La red virtual de la cuenta de Azure Storage se debe emparejar con la red virtual de la cuenta de laboratorio **antes** de que se cree el laboratorio.

Siga los pasos que se indican a continuación para crear un recurso compartido de Azure Files con autenticación AD y unirse al dominio de las máquinas virtuales del laboratorio.

1. Creación de una [cuenta de Azure Storage](../storage/files/storage-how-to-create-file-share.md).
2. Si usa, cree un [punto de conexión privado](../private-link/tutorial-private-endpoint-storage-portal.md) para que los recursos compartidos de archivos sean accesibles desde la red virtual.  Cree una [zona DNS privada](../dns/private-dns-privatednszone.md) o use una existente. Las zonas de Azure DNS privadas proporcionan la resolución de nombres dentro de una red virtual.
3. Cree un [recurso compartido de archivos de Azure](../storage/files/storage-how-to-create-file-share.md).
4. Siga los pasos para habilitar la autorización basada en identidad.  Si usa AD local que se sincroniza con Azure AD, siga los pasos para [la autenticación de Active Directory Domain Services local a través de SMB para recursos compartidos de archivos de Azure](../storage/files/storage-files-identity-auth-active-directory-enable.md).  Si solo usa Azure AD, siga los pasos para [Habilitar la autenticación de Azure Active Directory Domain Services en Azure Files](../storage/files/storage-files-identity-auth-active-directory-domain-service-enable.md).
    >[!IMPORTANT]
    >Hable con el equipo que administra su AD para comprobar que se cumplen todos los requisitos previos enumerados en las instrucciones.
5. Asignación de roles de permisos de recurso compartido SMB en Azure.  Para obtener más información sobre los permisos que se conceden a cada rol, consulte [permisos de nivel de recurso compartido](../storage/files/storage-files-identity-ad-ds-assign-permissions.md).
    1. El rol 'Almacenamiento de datos de recurso compartido de SMB con privilegios elevados' se debe asignar a la persona o grupo que configurará los permisos para el contenido del recurso compartido de archivos.
    2. El rol 'Almacenamiento de datos de recurso compartido SMB' debe asignarse a los alumnos que necesiten agregar o editar archivos en el recurso compartido de archivos.
    3. El rol "Almacenamiento del recurso compartido de SMB de datos de archivos de almacenamiento" se debe asignar a los alumnos que solo necesiten leer los archivos del recurso compartido de archivos.
6. Configurar los permisos a nivel de directorio/archivo para el recurso compartido.  Los permisos deben configurarse desde un equipo unido a un dominio que tenga acceso de red al recurso compartido de archivos.  **Para modificar los permisos de nivel de archivo o directorio, monte el recurso compartido de archivos con la clave de almacenamiento, no con las credenciales de AAD.**  El comando [Set-ACL](/powershell/module/microsoft.powershell.security/set-acl) de PowerShell o [icacls](/windows-server/administration/windows-commands/icacls) son las herramientas recomendadas que se deben usar al asignar permisos.
7. [Empareje la red virtual](how-to-connect-peer-virtual-network.md) de la cuenta de almacenamiento con la cuenta de laboratorio.
8. [Cree el laboratorio de clase](how-to-manage-classroom-labs.md).
9. Guarde un script en la plantilla de la máquina virtual que los alumnos puedan ejecutar para conectarse a la unidad de red.  Para obtener un script de ejemplo:
    1. Abra la cuenta de almacenamiento en Azure Portal.
    1. Seleccione **Recursos compartidos de archivos** en **Servicio de archivo** en el menú.
    1. Busque el recurso compartido al que desea conectarse, seleccione el botón de puntos suspensivos en el extremo derecho y elija **Conectar**.
    1. La **conexión** hacia afuera se abrirá con instrucciones para Windows, Linux y macOS.  Si usa Windows, establezca el **método de autenticación** en **Active Directory**.
    1. Copie el código del ejemplo y guárdelo en el equipo de la plantilla en un archivo `.ps1` para Windows o `.sh` para Linux.
10. En el equipo de la plantilla, descargue y ejecute el script para [unir las máquinas de los alumnos al dominio](https://github.com/Azure/azure-devtestlab/blob/master/samples/ClassroomLabs/Scripts/ActiveDirectoryJoin/README.md#usage).  El script `Join-AzLabADTemplate`[publicará la máquina virtual de la plantilla](how-to-create-manage-template.md#publish-the-template-vm) automáticamente.  
    > [!NOTE]
    > La plantilla de máquina virtual no estará unido al dominio. Los instructores deben asignar una máquina virtual de alumno publicada para ver los archivos del recurso compartido.
11. Los alumnos que usan Windows pueden conectarse al recurso compartido de Azure Files mediante el [Explorador de archivos](../storage/files/storage-how-to-use-files-windows.md) con sus credenciales una vez dada la ruta de acceso al recurso compartido de archivos.  Como alternativa, los alumnos pueden ejecutar el script creado anteriormente para conectarse a la unidad de red.  Para los alumnos que usan Linux, ejecute el script que creó anteriormente.

## <a name="netapp-files-with-nfs-volumes"></a>NetApp Files con volúmenes NFS

[Azure NetApp Files](https://azure.microsoft.com/services/netapp/) es un servicio de almacenamiento de archivos de alto rendimiento, medido y de clase empresarial.

- Las directivas de acceso se pueden establecer por volumen.
- Las directivas de permisos se basan en IP para cada volumen.
- Si los estudiantes necesitan su propio volumen al que otros alumnos no tienen acceso, deben asignarse directivas de permisos después de publicar el laboratorio.
- En el contexto de Azure Lab Services, solo se admiten máquinas Linux.
- La red virtual del grupo de capacidad de Azure NetApp Files se debe emparejar con la red virtual de la cuenta de laboratorio **antes** de que se cree el laboratorio.

Siga los pasos siguientes para usar un recurso compartido de Azure NetApp Files en Azure Lab Services.

1. Incorporar a [Azure NetApp Files](https://aka.ms/azurenetappfiles), si es necesario.
2. Para crear un grupo de capacidad de NetApp Files y volúmenes NFS, consulte [configuración del volumen de Azure NetApp Files y NFS](../azure-netapp-files/azure-netapp-files-quickstart-set-up-account-create-volumes.md).  Para obtener información sobre los niveles de servicio, consulte [niveles de servicio para Azure NetApp Files](../azure-netapp-files/azure-netapp-files-service-levels.md).
3. [Emparejar la red virtual](how-to-connect-peer-virtual-network.md) del grupo de capacidad de NetApp Files con la cuenta de laboratorio.
4. [Cree el laboratorio de clase](how-to-manage-classroom-labs.md).
5. En la plantilla de la máquina virtual, instale los componentes necesarios para usar recursos compartidos de archivos NFS.
    - Ubuntu:

        ```bash
        sudo apt update
        sudo apt install nfs-common
        ```

    - CentOS

        ```bash
        sudo yum install nfs-utils
        ```

6. En la plantilla de la máquina virtual, guarde el script siguiente como `mount_fileshare.sh` para [montar el recurso compartido de NetApp Files](../azure-netapp-files/azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md).  Asigne `capacity_pool_ipaddress` a la variable la dirección IP de destino de montaje para el grupo de capacidad.  Obtenga las instrucciones de montaje del volumen para encontrar el valor adecuado.  El script espera la ruta de acceso o el nombre del volumen de NetApp Files.  No olvide ejecutar `chmod u+x mount_fileshare.sh` para asegurarse de que los usuarios pueden ejecutar el script.

    ```bash
    #!/bin/bash
    
    if [ $# -eq 0 ];  then
        echo "Must provide volume name."
        exit 1
    fi
    
    volume_name=$1
    capacity_pool_ipaddress=0.0.0.0 # IP address of capacity pool
    
    # Do not use 'mnt' for mount directory.
    # Using ‘mnt’ may cause issues on student VMs.
    mount_directory="prm-mnt" 
    
    sudo mkdir -p /$mount_directory
    sudo mkdir /$mount_directory/$folder_name
    
    sudo mount -t nfs -o rw,hard,rsize=65536,wsize=65536,vers=3,tcp $capacity_pool_ipaddress:/$volume_name /$mount_directory/$volume_name
    sudo bash -c "echo ""$capacity_pool_ipaddress:/$volume_name /$mount_directory/$volume_name nfs bg,rw,hard,noatime,nolock,rsize=65536,wsize=65536,vers=3,tcp,_netdev 0 0"" >> /etc/fstab"
    ```

7. Si todos los alumnos están compartiendo el acceso al mismo volumen de NetApp Files, el script `mount_fileshare.sh` se puede ejecutar en la máquina de la plantilla antes de la publicación.  Si cada alumno obtiene su propio volumen, guarde el script para que el alumno lo ejecute más tarde.
8. [Publicación](how-to-create-manage-template.md#publish-the-template-vm) de la plantilla de máquina virtual.
9. [Configure la directiva](../azure-netapp-files/azure-netapp-files-configure-export-policy.md) para el recurso compartido.  La directiva de exportación puede permitir que una sola máquina virtual o varias máquinas virtuales tengan acceso a un volumen.  Se puede conceder acceso de solo lectura o de lectura y escritura.
10. Los alumnos deben iniciar su máquina virtual y ejecutar el script para montar el recurso compartido de archivos.  Solo tendrán que ejecutar el script una vez.  El comando tendrá un aspecto similar a `./mount_fileshare.sh myvolumename`.

## <a name="next-steps"></a>Pasos siguientes

Los pasos siguientes son comunes a la configuración de cualquier laboratorio.

- [Creación y administración de plantillas](how-to-create-manage-template.md)
- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de la cuota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users)