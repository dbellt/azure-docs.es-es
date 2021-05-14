---
title: Dispositivo de replicación con Azure Migrate
description: Obtenga más información sobre el dispositivo de replicación de Azure Migrate para la migración de VMware basada en agente.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: conceptual
ms.date: 01/30/2020
ms.openlocfilehash: 5f63b033c3995932662fc9b68c1397bf57b0326e
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714981"
---
# <a name="replication-appliance"></a>Dispositivo de replicación

En este artículo se describe el dispositivo de replicación que usa la herramienta [Azure Migrate: Server Assessment](migrate-services-overview.md#azure-migrate-server-migration-tool) al migrar VM de VMware, máquinas físicas y VM de nube privada o pública a Azure mediante una migración basada en agente. 


## <a name="overview"></a>Información general

El dispositivo de replicación se implementa cuando se configura la migración basada en agente de VM de VMware o servidores físicos. Se implementa como una máquina local individual, ya sea como una VM de VMware o como un servidor físico. Ejecuta lo siguiente:

- **Dispositivo de replicación**: el dispositivo de replicación coordina las comunicaciones y administra la replicación de datos para las máquinas virtuales de VMware locales y los servidores físicos que se replican en Azure.
- **Servidor de proceso**: el servidor de proceso, que se instala de forma predeterminada en el dispositivo de replicación, hace lo siguiente:
    - **Puerta de enlace de replicación**: actúa como puerta de enlace de replicación. Recibe datos de replicación de las máquinas habilitadas para la replicación. Optimiza los datos de replicación con el almacenamiento en caché, la compresión y el cifrado, y los envía a Azure.
    - **Instalador de agente**: realiza una instalación de inserción de Mobility Service. Este servicio debe instalarse y ejecutarse en cada máquina local que quiera replicar para la migración.

## <a name="appliance-deployment"></a>Implementación del dispositivo

**Se usa para** | **Detalles**
--- |  ---
**Migración basada en agente de máquinas virtuales de VMware** | Descargue la plantilla OVA del centro de Azure Migrate e impórtela en vCenter Server para crear la máquina virtual del dispositivo.
**Migración basada en agente de máquinas físicas** | Si no tiene una infraestructura de VMware o si no puede crear una VM de VMware mediante una plantilla OVA, descargue un instalador de software desde el Centro de Azure Migrate y ejecútelo para configurar la máquina de la aplicación.

> [!NOTE]
> Si va a realizar la implementación en Azure Government, use el archivo de instalación para implementar el dispositivo de replicación.

## <a name="appliance-requirements"></a>Requisitos de los dispositivos

Al configurar el dispositivo de replicación con la plantilla OVA proporcionada en el concentrador de Azure Migrate, el dispositivo ejecuta Windows Server 2016 y cumple los requisitos de compatibilidad. Si configura el dispositivo de replicación manualmente en un servidor físico, asegúrese de que cumple los requisitos.

**Componente** | **Requisito**
--- | ---
 | **Dispositivo de VM de VMware**
PowerCLI | Si el dispositivo de replicación se ejecuta en una máquina virtual de VMware, se debe instalar [PowerCLI versión 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1).
Tipo de NIC | VMXNET3 (si el dispositivo es una máquina virtual de VMware)
 | **Configuración de hardware**
Núcleos de CPU | 8
RAM | 16 GB
Número de discos | Tres: el disco del sistema operativo, el disco de la caché del servidor de procesos y la unidad de retención.
Espacio libre en disco (caché) | 600 GB
Espacio libre en el disco (disco de retención) | 600 GB
**Configuración de software** |
Sistema operativo | Windows Server 2016 o Windows Server 2012 R2
Licencia | El dispositivo viene con una licencia de evaluación de Windows Server 2016 que es válida durante 180 días.<br/><br/> Si el período de evaluación está a punto de expirar, se recomienda descargar e implementar un nuevo dispositivo, o bien activar la licencia del sistema operativo de la máquina virtual del dispositivo.
Configuración regional del sistema operativo | Español (es-es)
TLS | TLS 1.2 debe estar habilitado.
.NET Framework | .NET Framework 4.6 o posterior debe estar instalado en la máquina (con criptografía segura habilitada).
MySQL | MySQL debe estar instalado en el dispositivo.<br/> Se debe instalar MySQL. Se puede instalar manualmente o lo puede instalar Site Recovery durante la implementación.
Otras aplicaciones | No ejecute otras aplicaciones en el dispositivo de replicación.
Roles de Windows Server | No habilite estos roles: <br> - Active Directory Domain Services <br>- Internet Information Services <br> - Hyper-V
Directivas de grupo | No habilite estas directivas de grupo: <br> - Impedir el acceso al símbolo del sistema. <br> - Impedir el acceso a herramientas de edición del Registro. <br> - Confiar en la lógica de datos adjuntos de archivos. <br> - Activar la ejecución de scripts. <br> [Más información](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))
IIS | - Ningún sitio web predeterminado debe existir previamente <br> - Ningún sitio web o aplicación que escuche en el puerto 443 deben existir previamente <br>- Habilitar la [autenticación anónima](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)) <br> - Habilitar la configuración de [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10))
**Configuración de red** |
Tipo de dirección IP | estática
Puertos | 443 (orquestación del canal de control)<br>9443 (Transporte de datos)
Tipo de NIC | VMXNET3

## <a name="mysql-installation"></a>Instalación de MySQL 

MySQL debe estar instalado en la máquina de la aplicación de replicación. Se puede instalar con uno de estos métodos.

**Método** | **Detalles**
--- | ---
Descargar e instalar manualmente | Descargue la aplicación MySQL y colóquela en la carpeta C:\Temp\ASRSetup; luego, instálela manualmente.<br/> Al configurar el dispositivo, MySQL se mostrará como ya instalado.
Sin descarga en línea | Coloque la aplicación del instalador de MySQL en la carpeta C:\Temp\ASRSetup. Al instalar el dispositivo y seleccionar para descargar e instalar MySQL, el programa de instalación usará el instalador que ha agregado.
Descargar e instalar en Azure Migrate | Cuando instale el dispositivo y se le solicite MySQL, seleccione **Descargar e instalar**.

## <a name="url-access"></a>acceso URL

El dispositivo de replicación necesita acceso a estas direcciones URL de la nube pública de Azure.

**URL** | **Detalles**
--- | ---
\*.backup.windowsazure.com | Se usa para la transferencia y coordinación de datos replicados
\*.store.core.windows.net | Se usa para la transferencia y coordinación de datos replicados
\*blob.core.windows.net | Se usa para tener acceso a la cuenta de almacenamiento que almacena los datos replicados
\*.hypervrecoverymanager.windowsazure.com | Se usa para las operaciones de administración de replicación y coordinación
https:\//management.azure.com | Se usa para las operaciones de administración de replicación y coordinación
*.services.visualstudio.com | Se usa con fines de registro (es opcional)
time.windows.com | Se usan para comprobar la sincronización de la hora entre el sistema y la hora global.
https:\//login.microsoftonline.com <br/> https:\//secure.aadcdn.microsoftonline-p.com <br/> https:\//login.live.com <br/> https:\//graph.windows.net <br/> https:\//login.windows.net <br/> https:\//www.live.com <br/> https:\//www.microsoft.com  | La configuración del dispositivo necesita acceso a estas direcciones URL. Azure Active Directory las usa para la administración de identidades y control de acceso
https:\//dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi | Para completar la descarga de MySQL. En algunas regiones, la descarga podría redirigirse a la dirección URL de la red CDN. Si fuera necesario, asegúrese de que la dirección URL de CDN también esté permitida.


## <a name="azure-government-url-access"></a>Acceso a la dirección URL de Azure Government

El dispositivo de replicación necesita acceso a estas direcciones URL de Azure Government.

**URL** | **Detalles**
--- | ---
\*.backup.windowsazure.us | Se usa para la transferencia y coordinación de datos replicados
\*.store.core.windows.net | Se usa para la transferencia y coordinación de datos replicados
\*blob.core.windows.net | Se usa para tener acceso a la cuenta de almacenamiento que almacena los datos replicados
\*.hypervrecoverymanager.windowsazure.us | Se usa para las operaciones de administración de replicación y coordinación
https:\//management.usgovcloudapi.net | Se usa para las operaciones de administración de replicación y coordinación
*.services.visualstudio.com | Se usa con fines de registro (es opcional)
time.nist.gov | Se usan para comprobar la sincronización de la hora entre el sistema y la hora global.
https:\//login.microsoftonline.com <br/> https:\//secure.aadcdn.microsoftonline-p.com <br/> https:\//login.live.com <br/> https:\//graph.windows.net <br/> https:\//login.windows.net <br/> https:\//www.live.com <br/> https:\//www.microsoft.com  | La configuración del dispositivo con OVA necesita acceso a estas direcciones URL. Azure Active Directory las usa para la administración de identidades y control de acceso.
https:\//dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi | Para completar la descarga de MySQL. En algunas regiones, la descarga podría redirigirse a la dirección URL de la red CDN. Si fuera necesario, asegúrese de que la dirección URL de CDN también esté permitida.  

>[!Note]
>
> Si el proyecto de Migrate tiene conectividad de punto de conexión privado, necesitará acceder a las siguientes direcciones URL con acceso de vínculo privado y más allá:   
> - *.blob.core.windows.com: Se usa para acceder a la cuenta de almacenamiento que almacena los datos replicados. Esto es opcional y no es obligatorio si la cuenta de almacenamiento tiene asociado un punto de conexión privado. 
> - https:\//management.azure.com para las operaciones de administración de replicación y la coordinación. 
>- https:\//login.microsoftonline.com <br/>https:\//login.windows.net <br/> https:\//www.live.com _y_ <br/> https:\//www.microsoft.com para el control de acceso y la administración de identidades por parte de Azure Active Directory

## <a name="port-access"></a>Acceso a puertos

**Dispositivo** | **Connection**
--- | ---
Máquinas virtuales | El servicio de movilidad que se ejecuta en las máquinas virtuales se comunica con el dispositivo de replicación local (servidor de configuración) en el puerto HTTPS 443 entrante para la administración de la replicación.<br/><br/> Las máquinas virtuales envían datos de replicación al servidor de procesos (que se ejecuta en la máquina del servidor de configuración) en el puerto HTTPS 9443 entrante. Este puerto se puede modificar.
Dispositivo de replicación | El dispositivo de replicación organiza la replicación con Azure a través del puerto HTTPS 443 saliente.
Servidor de proceso | El servidor de procesos recibe los datos de la replicación, los optimiza, los cifra y los envía a Azure Storage a través del puerto 443 de salida.<br/> De forma predeterminada, el servidor de procesos se ejecuta en el dispositivo de replicación.


## <a name="replication-process"></a>Proceso de replicación

1. Al habilitar la replicación para una VM, comienza la replicación inicial en el almacenamiento de Azure mediante la directiva de replicación especificada. 
2. El tráfico se replica en los puntos de conexión públicos de Azure Storage a través de Internet. No se admite la replicación del tráfico entre un sitio local y Azure a través de una red privada virtual (VPN) de sitio a sitio.
3. Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Las marcas de revisión de una máquina se registran.
4. Comunicación se realiza como se indica a continuación:
    - Las máquinas virtuales se comunican con el dispositivo de replicación en el puerto HTTPS 443 entrante para la administración de la replicación.
    - El dispositivo de replicación organiza la replicación con Azure a través del puerto HTTPS 443 saliente.
    - Las máquinas virtuales envían los datos de replicación al servidor de procesos (que se ejecuta en el dispositivo de replicación) en el puerto HTTPS 9443 entrante. Este puerto se puede modificar.
    - El servidor de procesos recibe los datos de la replicación, los optimiza, los cifra y los envía a Azure Storage a través del puerto 443 de salida.
5. Los datos de replicación registran el primer aterrizaje en una cuenta de almacenamiento en caché de Azure. Estos registros se procesan y los datos se almacenan en un disco administrado de Azure.

![En el diagrama se muestra la arquitectura del proceso de replicación.](./media/migrate-replication-appliance/architecture.png)

## <a name="appliance-upgrades"></a>Actualizaciones del dispositivo

El dispositivo se actualiza manualmente desde el centro de Azure Migrate. Se recomienda que ejecute siempre la versión más reciente.

1. En Azure Migrate > Servidores > Azure Migrate: Server Assessment, Servidores de infraestructura, seleccione **Servidores de configuración**.
2. En **Servidores de configuración**, aparece un vínculo en **Versión del agente** cuando hay disponible una nueva versión del dispositivo de replicación. 
3. Descargue el instalador en el equipo del dispositivo de replicación e instale la actualización. El instalador detecta la versión que se ejecuta actualmente en el dispositivo.
 
## <a name="next-steps"></a>Pasos siguientes

- [Obtenga información](tutorial-migrate-vmware-agent.md#set-up-the-replication-appliance) sobre cómo configurar el dispositivo de replicación para la migración de VM de VMware basada en agente.
- [Obtenga información](tutorial-migrate-physical-virtual-machines.md#set-up-the-replication-appliance) sobre cómo configurar el dispositivo de replicación para servidores físicos.
