---
title: 'Configuración de un recurso compartido de archivos para conectar aplicaciones en formato MSIX en Windows Virtual Desktop: Azure'
description: Configuración de un recurso compartido de archivos para asociar aplicaciones en formato MSIX en el contexto de Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 04/13/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: a2d4ebee02d85d10d5db8ec2de0bb1be334770dc
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107717664"
---
# <a name="set-up-a-file-share-for-msix-app-attach"></a>Configuración de un recurso compartido de archivos para asociar aplicaciones en formato MSIX (versión preliminar)

Todas las imágenes MSIX deben almacenarse en un recurso compartido de red al que puedan acceder los usuarios de un grupo host con permisos de solo lectura.

La conexión de aplicaciones en formato MSIX no tiene dependencias en el tipo de tejido de almacenamiento que usa el recurso compartido de archivos. Las consideraciones para el recurso compartido de asociación en formato MSIX son las mismas que las de un recurso compartido de FSLogix. Para obtener más información sobre los requisitos de almacenamiento, consulte [Opciones de almacenamiento para los contenedores de perfiles de FSLogix de Windows Virtual Desktop](store-fslogix-profile.md).

## <a name="performance-requirements"></a>Requisitos de rendimiento

Los límites de tamaño de imágenes para la asociación de aplicaciones en formato MSIX correspondientes a su sistema dependerán del tipo de almacenamiento que se use para almacenar los archivos VHD o VHDx, así como las limitaciones de tamaño de los archivos VHD, VHSD o CIM y el sistema de archivos.

En la tabla siguiente se muestra un ejemplo de cuántos recursos necesita una sola imagen MSIX de 1 GB con una aplicación en formato MSIX dentro de ella para cada máquina virtual:

| Recurso             | Requisitos |
|----------------------|--------------|
| IOPS de estado estable    | 1 IOPS       |
| Inicio de sesión de arranque de la máquina | 10 IOPS      |
| Latencia              | 400 ms       |

Los requisitos pueden variar considerablemente en función del número de aplicaciones empaquetadas en formato MSIX que se almacenan en la imagen MSIX. En el caso de imágenes de MSIX de mayor tamaño, deberá asignar más ancho de banda.

### <a name="storage-recommendations"></a>Recomendaciones para almacenamiento

Azure ofrece varias opciones de almacenamiento que se pueden usar para asociar aplicaciones en formato MISX. Se recomienda usar Azure Files o Azure NetApp Files, ya que esas opciones ofrecen la mejor relación entre costo y sobrecarga de administración. En el artículo [Opciones de almacenamiento para los contenedores de perfiles de FSLogix de Windows Virtual Desktop](store-fslogix-profile.md) se comparan las diferentes soluciones de almacenamiento administrado que Azure ofrece en el contexto de Windows Virtual Desktop.

### <a name="optimize-msix-app-attach-performance"></a>Optimización del rendimiento de la asociación de aplicaciones en formato MSIX

Estas son algunas de las otras cosas que se recomiendan para optimizar el rendimiento de la asociación de aplicaciones en formato MSIX:

- La solución de almacenamiento que se use para asociar aplicaciones en formato MSIX debe estar en la misma ubicación del centro de datos que los hosts de sesión.
- Para evitar cuellos de botella de rendimiento, excluya los siguientes archivos VHD, VHDX y CIM de los exámenes antivirus:
   
    - <MSIXAppAttachFileShare\>\*.VHD
    - <MSIXAppAttachFileShare\>\*.VHDX
    - \\\\storageaccount.file.core.windows.net\\share\*\*.VHD
    - \\\\storageaccount.file.core.windows.net\\share\*\*.VHDX
    - <MSIXAppAttachFileShare>.CIM
    - \\\\storageaccount.file.core.windows.net\\share\*\*.CIM

- Separe el tejido de almacenamiento para asociar aplicaciones en formato MSIX desde contenedores de perfiles de FSLogix.
- Todas las cuentas de usuario y cuentas de sistema de máquinas virtuales deben tener permisos de solo lectura para acceder al recurso compartido de archivos.
- Los planes de recuperación ante desastres para Windows Virtual Desktop deben incluir la replicación del recurso compartido de archivos de asociación de aplicaciones en formato MSIX en la ubicación secundaria de conmutación por error. Para obtener más información sobre la recuperación ante desastres, consulte [Configuración de un plan de continuidad empresarial y recuperación ante desastres](disaster-recovery.md).

## <a name="how-to-set-up-the-file-share"></a>Configuración del recurso compartido de archivos

El proceso de instalación del recurso compartido de archivos de asociaciones en formato MSIX es, en gran medida, igual que el [de los recursos compartidos de archivos de perfiles de FSLogix](create-host-pools-user-profile.md). Sin embargo, deberá asignar distintos permisos a los usuarios. La asociación de aplicaciones en formato MSIX requiere permisos de solo lectura para acceder al recurso compartido de archivos.

Si va a almacenar aplicaciones en formato MSIX en Azure Files, para los hosts de sesión deberá asignar a todas las máquinas virtuales de host de sesión tanto permisos de control de acceso basado en rol (RBAC) en la cuenta de almacenamiento como permisos NTFS (New Technology File System) en el recurso compartido.

| Objeto de Azure                      | Rol necesario                                     | Función de rol                                  |
|-----------------------------------|--------------------------------------------------|-----------------------------------------------|
| Host de sesión (objetos de equipo de máquina virtual)| Colaborador de recursos compartidos de SMB de datos de archivos de almacenamiento          | Lectura y ejecución, Lectura, Mostrar el contenido de la carpeta  |
| Administradores en un recurso compartido de archivos              | Colaborador elevado de recursos compartidos de SMB de datos de archivos de Storage | Control total                                  |
| Usuarios en un recurso compartido de archivos               | Colaborador de recursos compartidos de SMB de datos de archivos de almacenamiento          | Lectura y ejecución, Lectura, Mostrar el contenido de la carpeta  |

Para asignar a las máquinas virtuales de host de sesión permisos en la cuenta de almacenamiento y el recurso compartido de archivos:

1. Cree un grupo de seguridad de Active Directory Domain Services (AD DS).

2. Agregue las cuentas de equipo de todas las máquinas virtuales de host de sesión como miembros del grupo.

3. Sincronice el grupo de AD DS con Azure Active Directory (Azure AD).

4. Cree una cuenta de almacenamiento.

5. Cree un recurso compartido de archivos en la cuenta de almacenamiento siguiendo las instrucciones de [Creación de un recurso compartido de archivos de Azure](../storage/files/storage-how-to-create-file-share.md#create-a-file-share).

6. Una la cuenta de almacenamiento a AD DS siguiendo las instrucciones de [Parte 1: Habilitación de la autenticación de AD DS para los recursos compartidos de archivos de Azure](../storage/files/storage-files-identity-ad-ds-enable.md#option-one-recommended-use-azfileshybrid-powershell-module).

7. Asigne el grupo de AD DS sincronizado a Azure AD, y asigne a la cuenta de almacenamiento el rol de colaborador de recursos compartidos de SMB de datos de archivos de Storage.

8. Monte el recurso compartido de archivos en cualquier host de sesión siguiendo las instrucciones de [Parte 2: Asignación de permisos de nivel de recurso compartido a una identidad](../storage/files/storage-files-identity-ad-ds-assign-permissions.md).

9. Conceda permisos NTFS en el recurso compartido de archivos al grupo de AD DS.

10. Configure los permisos NTFS para las cuentas de usuario. Necesitará una unidad operativa (UO) cuyo origen sea la instancia de AD DS a la que pertenecen las cuentas de la máquina virtual.

Una vez que haya asignado la identidad al almacenamiento, siga las instrucciones de los artículos de [Pasos siguientes](#next-steps) para conceder otros permisos necesarios a la identidad que ha asignado a las máquinas virtuales.

También debe asegurarse de que las máquinas virtuales de host de sesión tengan permisos NTFS. Debe tener un contenedor de unidad operativa cuyo origen sea Active Directory Domain Services (AD DS), y los usuarios deben ser miembros de esa unidad operativa para usar esos permisos.

## <a name="next-steps"></a>Pasos siguientes

Estos son los otros pasos que debe realizar después de configurar el recurso compartido de archivos:

- Obtenga información sobre cómo configurar Azure Active Directory Domain Services (AD DS) en [Creación de un contenedor de perfiles con Azure Files y AD DS](create-file-share.md).
- Aprenda a configurar Azure Files y Azure AD DS en [Creación de un contenedor de perfiles con Azure Files y Azure AD DS](create-profile-container-adds.md).
- Descubra cómo configurar Azure NetApp Files para asociar aplicaciones en formato MSIX en [Creación de un contenedor de perfiles con Azure NetApp Files y AD DS](create-fslogix-profile-container.md).
- Aprenda a usar un recurso compartido de archivos basado en máquina virtual en [Creación de un contenedor de perfiles para un grupo host mediante un recurso compartido de archivos](create-host-pools-user-profile.md).

Una vez que haya terminado, aquí se muestran otros recursos que pueden resultarle útiles:

- Pregunte a la comunidad sobre esta característica en el espacio de [TechCommunity sobre Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).
- También puede dejar comentarios sobre Windows Virtual Desktop en el [Concentrador de comentarios de Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).
- [Glosario sobre la conexión de aplicaciones en formato MSIX](app-attach-glossary.md)
- [Preguntas frecuentes sobre la asociación de aplicaciones en formato MSIX](app-attach-faq.md)
