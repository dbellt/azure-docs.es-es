---
title: Migración de grupos de disponibilidad
description: Aprenda a migrar mediante lift and shift la solución de alta disponibilidad de grupos de disponibilidad AlwaysOn a SQL Server en máquinas virtuales de Azure mediante Azure Migrate.
ms.service: virtual-machines-sql
ms.subservice: migration-guide
author: rahugup
manager: bsiva
ms.topic: how-to
ms.date: 4/25/2021
ms.author: rahugup
ms.openlocfilehash: 96ee2422bad6e15e37e01c33d2724300a7a8f230
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777027"
---
# <a name="migrate-availability-group-to-sql-server-on-azure-vm"></a>Migración de grupos de disponibilidad a SQL Server en máquinas virtuales de Azure

En este artículo se indica cómo migrar el grupo de disponibilidad AlwaysOn de SQL Server a SQL Server en máquinas virtuales de Azure mediante la [herramienta Azure Migrate: Server Migration](../../../migrate/migrate-services-overview.md#azure-migrate-server-migration-tool). Con la herramienta de migración, podrá migrar cada réplica del grupo de disponibilidad a una máquina virtual de Azure que hospeda SQL Server, así como los metadatos del clúster, los metadatos del grupo de disponibilidad y otros componentes de alta disponibilidad necesarios. 

En este artículo aprenderá a: 

> [!div class="checklist"]
> * Preparar Azure y el entorno de origen para la migración.
> * Comenzar a replicar servidores.
> * Supervisar la replicación.
> * Ejecutar una migración completa del servidor.
> * Volver a configurar el grupo de disponibilidad AlwaysOn. 


En esta guía se usa el enfoque de migración basado en agente de Azure Migrate, que trata cualquier servidor o máquina virtual como un servidor físico. Al migrar máquinas físicas, Azure Migrate: Server Migration emplea la misma arquitectura de replicación que la recuperación ante desastres basada en agente del servicio Azure Site Recovery, y algunos componentes comparten el mismo código base. Puede que algún contenido se vincule a la documentación de Site Recovery.


## <a name="prerequisites"></a>Requisitos previos


Antes de comenzar este tutorial, debe cumplir los siguientes requisitos previos:

1. Suscripción a Azure. Crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/), en caso necesario. 
1. Instalar el [módulo `Az` de Azure PowerShell](/powershell/azure/install-az-ps). 
1. Descargar los [scripts de ejemplos de PowerShell](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/SQL%20Migration) del repositorio de GitHub.

## <a name="prepare-azure"></a>Preparación de Azure

Prepare Azure para la migración con la herramienta [Server Migration](../../../migrate/migrate-services-overview.md#azure-migrate-server-migration-tool).

|**Task** | **Detalles**|
|--- | ---
|**Crear un proyecto de Azure Migrate** | La cuenta de Azure necesita permisos de colaborador o propietario para [crear un proyecto](../../../migrate/create-manage-projects.md).|
|**Comprobación de los permisos de la cuenta de Azure** | La cuenta de Azure necesita permisos de colaborador o propietario en la suscripción de Azure, permisos para registrar aplicaciones de Azure Active Directory (AAD) y permisos de administrador de acceso de usuarios en la suscripción de Azure para crear un almacén de claves, crear una máquina virtual y escribir en un disco administrado de Azure. |
|**Configuración de una red virtual de Azure** | [Configure](/virtual-network/manage-virtual-network.md#create-a-virtual-network) una red virtual de Azure. Al realizar la replicación en Azure, se crean máquinas virtuales de Azure y se unen a la red virtual de Azure que se especifica al configurar la migración.|


Para comprobar que tiene los permisos adecuados, siga estos pasos: 

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y selecciónela para ver los permisos.
3. Debe tener permisos de **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.

Si necesita asignar permisos, siga los pasos descritos en [Preparación de una cuenta de usuario de Azure](../../../migrate/tutorial-discover-vmware.md#prepare-an-azure-user-account).


## <a name="prepare-for-migration"></a>Preparación para la migración

Para preparar la migración del servidor, compruebe la configuración del servidor físico y prepare la implementación de un dispositivo de replicación.

### <a name="check-machine-requirements"></a>Comprobación de los requisitos de la máquina 


Asegúrese de que las máquinas de origen cumplen los requisitos para migrar a Azure. Siga estos pasos: 

1. [Comprobación](../../../migrate/migrate-support-matrix-physical-migration.md#physical-server-requirements) de los requisitos del servidor.
1. Compruebe que las máquinas de origen que replique en Azure cumplan los [requisitos de máquina virtual de Azure](../../../migrate/migrate-support-matrix-physical-migration.md#azure-vm-requirements).
1. Algunos orígenes de [Windows](../../../migrate/prepare-for-migration.md#windows-machines) requieren algunos cambios adicionales. Migrar el origen antes de realizar estos cambios podría impedir que la máquina virtual arranque en Azure. En algunos sistemas operativos, Azure Migrate realiza estos cambios automáticamente. 


### <a name="prepare-for-replication"></a>Preparación para la replicación 

Azure Migrate: Server Migration usa un dispositivo de replicación para replicar máquinas en Azure. Este dispositivo de replicación ejecuta los siguientes componentes: 

- **Servidor de configuración**: El servidor de configuración coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos.
- **Servidor de proceso**: El servidor de procesos actúa como puerta de enlace de replicación. Recibe los datos de la replicación; los optimiza mediante el almacenamiento en la caché, la compresión y el cifrado, y los envía a una cuenta de almacenamiento en Azure. 

Para prepararse para la implementación del dispositivo, siga estos pasos:

- Cree una máquina con Windows Server 2016 para hospedar el dispositivo de replicación. Revise los [requisitos de la máquina](../../../migrate/migrate-replication-appliance.md#appliance-requirements).
- El dispositivo de replicación usa MySQL. Revise las [opciones](../../../migrate/migrate-replication-appliance.md#mysql-installation) para instalar MySQL en el dispositivo.
- Revise las direcciones URL de Azure necesarias para que el dispositivo de replicación acceda a las nubes [públicas](../../../migrate/migrate-replication-appliance.md#url-access) y [gubernamentales](../../../migrate/migrate-replication-appliance.md#azure-government-url-access).
- Revise los requisitos de acceso a [puertos](../../../migrate/migrate-replication-appliance.md#port-access) para el dispositivo de replicación.

> [!NOTE]
> El dispositivo de replicación debe instalarse en una máquina que no sea la máquina de origen que va a replicar o migrar, y no en una que haya tenido instalado antes el dispositivo de detección y evaluación de Azure Migrate.

### <a name="download-replication-appliance-installer"></a>Descarga del instalador del dispositivo de replicación

Para descargar el instalador del dispositivo de replicación, siga estos pasos: 

1. En el proyecto de Azure Migrate, en **Servidores**, en **Azure Migrate: Server Migration**, seleccione **Detectar**.

    ![Detectar máquinas virtuales](../../../migrate/media/tutorial-migrate-physical-virtual-machines/migrate-discover.png)

1. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **Físico o de otro tipo (AWS, GCP, Xen, etc.)** .
1. En **Región de destino**, seleccione la región de Azure a la que desea migrar las máquinas.
1. Seleccione **Confirme que la región de destino de la migración es nombreDeRegión**.
1. Seleccione **Crear recursos**. Esto crea un almacén de Azure Site Recovery en segundo plano.
    - Si ya ha configurado la migración con Azure Migrate: Server Migration, no se puede configurar la opción de destino, ya que los recursos se configuraron anteriormente.    
    - Después de seleccionar este botón, ya no se puede cambiar la región de destino de este proyecto.
    - Todas las migraciones posteriores se realizan a esta región.

1. En **¿Quiere instalar un nuevo dispositivo de replicación?** , seleccione **Instalar un dispositivo de replicación**.
1. En **Descargue e instale el software del dispositivo de replicación**, descargue el instalador del dispositivo y la clave de registro. Necesitará la clave para registrar el dispositivo. La clave será válida durante cinco días a partir del momento en que se descarga.

    ![Descarga del proveedor](../../../migrate/media/tutorial-migrate-physical-virtual-machines/download-provider.png)

1. Copie el archivo de instalación y el archivo de clave del dispositivo en el equipo con Windows Server 2016 que creó para el dispositivo.
1. Una vez finalizada la instalación, el asistente para la configuración de dispositivos se iniciará automáticamente (también puede iniciarlo manualmente mediante el método abreviado cspsconfigtool que se crea en el escritorio de la máquina de dispositivo). Use la pestaña **Administrar cuentas** del asistente para crear una cuenta ficticia con los detalles siguientes:

   -  "guest" como nombre descriptivo.
   -  "username" como nombre de usuario.
   -  "password" como contraseña de la cuenta. 
   
   Esta cuenta ficticia la usará en la fase de habilitación de la replicación. 

1. Después de finalizar la configuración, y una vez que se ha reiniciado el dispositivo, en **Detectar máquinas**, seleccione el nuevo dispositivo en **Seleccionar servidor de configuración** y haga clic en **Finalizar el registro**. El paso de finalización del registro realiza un par de tareas finales para preparar el dispositivo de replicación.

    ![Finalizar el registro](../../../migrate/media/tutorial-migrate-physical-virtual-machines/finalize-registration.png)


## <a name="install-mobility-service"></a>Instalación del servicio Mobility

Instale el agente del servicio Mobility en los servidores que quiera migrar. Los instaladores del agente están disponibles en el dispositivo de replicación. Encuentre el instalador correcto e instale el agente en cada máquina que quiera migrar. 


Para instalar el servicio Mobility, siga estos pasos: 

1. Inicie sesión en el dispositivo de replicación.
1. Vaya a **%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository**.
1. Busque el instalador correspondiente al sistema operativo y la versión del equipo. Revise los [sistemas operativos compatibles](/site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines). 
1. Copie el archivo de instalación en el equipo que desea migrar.
1. Asegúrese de que tiene la frase de contraseña que se generó al implementar el dispositivo.
    - Almacene el archivo en un archivo de texto temporal de la máquina.
    - Puede obtener la frase de contraseña en el dispositivo de replicación. Desde la línea de comandos, ejecute **C:\ProgramData\ASR\home\svsystems\bin\genpassphrase.exe -v** para ver la frase de contraseña actual.
    - No regenere la frase de contraseña. Esto interrumpirá la conectividad y tendrá que volver a registrar el dispositivo de replicación.
    - En el parámetro */Platform*, especifique *VMware* para máquinas de VMware y máquinas físicas. 

1. Conéctese a la máquina y extraiga el contenido del archivo del instalador en una carpeta local (por ejemplo, c:\temp). Ejecútelo en un símbolo del sistema de administrador: 

    ```
    ren Microsoft-ASR_UA*Windows*release.exe MobilityServiceInstaller.exe
    MobilityServiceInstaller.exe /q /x:C:\Temp\Extracted
    cd C:\Temp\Extracted
    ```

2. Ejecute el instalador de Mobility Service:

    ```
   UnifiedAgent.exe /Role "MS" /Platform "VmWare" /Silent
    ```

3. Registre el agente en el dispositivo de replicación:

    ```
    cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
    UnifiedAgentConfigurator.exe  /CSEndPoint <replication appliance IP address> /PassphraseFilePath <Passphrase File Path>
    ```

Tras la instalación, pueden pasar unos minutos hasta que las máquinas detectadas aparezcan en Azure Migrate: Server Migration. A medida que se detectan las máquinas virtuales, aumenta el número de **Servidores detectados**.

![Servidores detectados](../../../migrate/media/tutorial-migrate-physical-virtual-machines/discovered-servers.png)

## <a name="prepare-source-machines"></a>Preparación de las máquinas de origen

Para preparar las máquinas de origen, ejecute el script `Get-ClusterInfo.ps1` en un nodo de clúster para recuperar información sobre los recursos del clúster. El script mostrará el nombre del rol, el nombre del recurso, la dirección IP y el puerto de sondeo en el archivo `Cluster-Config.csv`. 

```powershell
./Get-ClusterInfo.ps1
```

## <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Para que el clúster y los roles de clúster respondan correctamente a las solicitudes, se requiere un equilibrador de carga de Azure. Sin un equilibrador de carga, las demás máquinas virtuales no pueden acceder a la dirección IP del clúster, ya que no se reconocen como que pertenezcan a la red o al clúster. 

Para crear el equilibrador de carga, siga estos pasos: 

1. Rellene las columnas del archivo `Cluster-Config.csv`:

**Encabezado de columna** | **Descripción**
--- | ---
NewIP | Especifique la dirección IP en la red virtual (o subred) de Azure para cada recurso del archivo CSV.
ServicePort | Especifique el puerto de servicio que usará cada recurso en el archivo CSV. En el recurso en clúster de SQL, use el mismo valor para el puerto de servicio que para el puerto de sondeo en el archivo CSV. Con otros roles de clúster, los valores predeterminados usados son 1433, pero puede seguir usando los números de puerto configurados en la configuración actual. 


2. Ejecute el script `Create-ClusterLoadBalancer.ps1` para crear el equilibrador de carga con los parámetros siguientes: 

**Parámetro** | **Tipo** | **Descripción**
--- | --- | ---
ConfigFilePath | Mandatory| Especifique la ruta de acceso del archivo `Cluster-Config.csv` que ha rellenado en el paso anterior.
ResourceGroupName | Mandatory|Especifique el nombre del grupo de recursos en el que se va a crear el equilibrador de carga. 
VNetName | Mandatory|Especifique el nombre de la red virtual de Azure a la que se asociará el equilibrador de carga. 
SubnetName | Mandatory|Especifique el nombre de la subred en la red virtual de Azure a la que se asociará el equilibrador de carga. 
VNetResourceGroupName | Mandatory|Especifique el nombre del grupo de recursos de la red virtual de Azure a la que se asociará el equilibrador de carga. 
Location | Mandatory|Especifique la ubicación en la que se debe crear el equilibrador de carga. 
LoadBalancerName | Mandatory|Especifique el nombre del equilibrador de carga que se va a crear. 


```powershell
./Create-ClusterLoadBalancer.ps1 -ConfigFilePath ./clsuterinfo.csv -ResourceGroupName $resoucegroupname -VNetName $vnetname -subnetName $subnetname -VnetResourceGroupName $vnetresourcegroupname -Location "eastus" -LoadBalancerName $loadbalancername
```

## <a name="replicate-machines"></a>Replicación de máquinas

Ahora, seleccione las máquinas para la migración. Puede replicar hasta 10 máquinas juntas. Si necesita replicar más, replíquelas simultáneamente en lotes de 10.

Para replicar máquinas, siga estos pasos: 

1. En el proyecto de Azure Migrate > **Servidores**, **Azure Migrate: Migración del servidor**, seleccione **Replicar**.

    ![Captura de la pantalla Servidores de Azure Migrate que muestra el botón Replicar seleccionado en Azure Migrate: Server Migration en Herramientas de migración.](../../../migrate/media/tutorial-migrate-physical-virtual-machines/select-replicate.png)

1. En **Replicar**, **Configuración de origen** > **Are your machines virtualized?** (¿Están las máquinas virtualizadas?), seleccione **Physical or other (AWS, GCP, Xen, etc.)** (Físicos u otros [AWS, GCP, Xen, etc.]).
1. En **Dispositivo local**, seleccione el nombre del dispositivo de Azure Migrate que configuró.
1. En **Servidor de procesos**, seleccione el nombre del dispositivo de replicación.
1. En **Guest credentials** (Credenciales de invitado), seleccione la cuenta ficticia creada durante la [instalación del instalador de replicación](#download-replication-appliance-installer) anteriormente en este artículo. A continuación, seleccione **Siguiente: Máquinas virtuales**.   

    ![Captura de pantalla de la pestaña Configuración de origen en la pantalla Replicar con el campo Credenciales de invitado resaltado.](../../../migrate/media/tutorial-migrate-physical-virtual-machines/source-settings.png)

1. En **Máquinas virtuales**, en **¿Quiere importar la configuración de migración de una evaluación?** , deje la configuración predeterminada **No, especificaré la configuración de migración manualmente**.
1. Compruebe todas las máquinas virtuales que desea migrar. Después, seleccione **Next: Configuración de destino**.

    ![Selección de las máquinas virtuales](../../../migrate/media/tutorial-migrate-physical-virtual-machines/select-vms.png)


1. En **Configuración de destino**, seleccione la suscripción y la región de destino a la que va a migrar, y especifique el grupo de recursos en el que residirán las máquinas virtuales de Azure después de la migración.
1. En **Red virtual**, seleccione la red virtual o la subred de Azure a la que se unirán las máquinas virtuales de Azure después de la migración.
1. En **Opciones de disponibilidad**, seleccione:
    -  La zona de disponibilidad para anclar la máquina migrada a una zona de disponibilidad específica de la región. Use esta opción para distribuir los servidores que forman una capa de aplicación de varios nodos en Availability Zones. Si selecciona esta opción, deberá especificar la zona de disponibilidad que se va a usar en cada una de las máquinas seleccionadas en la pestaña Proceso. Esta opción solo está disponible si la región de destino seleccionada para la migración admite Availability Zones.
    -  El conjunto de disponibilidad para colocar la máquina migrada en un conjunto de disponibilidad. Para usar esta opción, el grupo de recursos de destino seleccionado debe tener uno o varios conjuntos de disponibilidad.
    - No se requiere ninguna opción de redundancia de infraestructura si no necesita ninguna de estas configuraciones de disponibilidad para las máquinas migradas.
    
1. En **Disk encryption type** (Tipo de cifrado de disco), seleccione:
    - Cifrado en reposo con clave administrada por la plataforma
    - Cifrado en reposo con clave administrada por el cliente
    - Cifrado doble con claves administradas por el cliente y por la plataforma

    > [!NOTE]
    > Para replicar máquinas virtuales con CMK, será necesario [crear un conjunto de cifrado de disco](https://go.microsoft.com/fwlink/?linkid=2151800) en el grupo de recursos de destino. Un objeto de conjunto de cifrado de disco asigna instancias de Managed Disks a una instancia de Key Vault que contiene las claves CMK que se van a usar para SSE.
  
1. En **Ventaja híbrida de Azure**:

    - Seleccione **No** si no desea aplicar la Ventaja híbrida de Azure. Luego, seleccione **Siguiente**.
    - Seleccione **Sí** si tiene equipos con Windows Server que están incluidos en suscripciones activas de Software Assurance o Windows Server y desea aplicar el beneficio a las máquinas que va a migrar. Luego, seleccione **Siguiente**.

    :::image type="content" source="../../../migrate/media/tutorial-migrate-vmware/target-settings.png" alt-text="Configuración de destino":::

1. En **Proceso**, revise el nombre, el tamaño, el tipo de disco del sistema operativo y la configuración de disponibilidad (si se ha seleccionado en el paso anterior) de la máquina virtual. Las máquinas virtuales deben cumplir los [requisitos de Azure](../../../migrate/migrate-support-matrix-physical-migration.md#azure-vm-requirements).

    - **Tamaño de VM**: si usa las recomendaciones de la evaluación, el menú desplegable de tamaño de máquina virtual muestra el tamaño recomendado. De lo contrario, Azure Migrate elige un tamaño en función de la coincidencia más cercana en la suscripción de Azure. También puede elegir un tamaño de manera manual en **Tamaño de la máquina virtual de Azure**.
    - **Disco del sistema operativo**: especifique el disco del sistema operativo (arranque) de la máquina virtual. Este es el disco que tiene el cargador de arranque y el instalador del sistema operativo.
    - **Zona de disponibilidad**: especifique la zona de disponibilidad que se va a usar.
    - **Conjunto de disponibilidad**: especifique el conjunto de disponibilidad que se va a usar.

   ![Configuración de Proceso](../../../migrate/media/tutorial-migrate-physical-virtual-machines/compute-settings.png)

1. En **Discos**, especifique si los discos de máquina virtual se deben replicar en Azure y seleccione el tipo de disco (discos SSD o HDD estándar o bien discos administrados premium) en Azure. Luego, seleccione **Siguiente**.  

    ![Configuración de discos](../../../migrate/media/tutorial-migrate-physical-virtual-machines/disks.png)

1. En **Revisar e iniciar la replicación**, revise la configuración y seleccione **Replicar** para iniciar la replicación inicial de los servidores.

> [!NOTE]
> Puede actualizar la configuración de replicación en cualquier momento antes de que esta comience; para ello, vaya a **Administrar** > **Replicación de máquinas**. Una vez iniciada la replicación, su configuración no se puede cambiar.

## <a name="track-and-monitor"></a>Seguimiento y supervisión

La replicación continúa en la siguiente secuencia: 

- Al seleccionar **Replicar**, comienza el trabajo _Iniciar replicación_. 
- Cuando el trabajo _Iniciar replicación_ finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en los discos de réplica de Azure.


Puede realizar un seguimiento del estado del trabajo en las notificaciones del portal.

Para supervisar el estado de la replicación, seleccione **Replicando servidores** en **Azure Migrate: Server Migration**.
![Supervisión de la replicación](../../../migrate/media/tutorial-migrate-physical-virtual-machines/replicating-servers.png)


## <a name="migrate-vms"></a>Migración de máquinas virtuales

Una vez replicadas las máquinas, están listas para la migración. Para migrar los servidores, siga estos pasos: 


1. En el proyecto de Azure Migrate > **Servidores** > **Azure Migrate: Server Migration**, seleccione **Replicando servidores**.

    ![Replicando servidores](../../../migrate/media/tutorial-migrate-physical-virtual-machines/replicate-servers.png)

2. Para asegurarse de que el servidor migrado está sincronizado con el servidor de origen, detenga el servicio SQL Server en todas las réplicas del grupo de disponibilidad, empezando por las réplicas secundarias (en **Administrador de configuración de SQL Server** > Servicios) y asegúrese de que los discos que hospedan datos SQL están en línea.   
3. En **Replicación de máquinas** > seleccione el nombre del servidor > **Información general**, compruebe que la última marca de tiempo sincronizada sea posterior a la acción de haber detenido el servicio SQL Server en los servidores que se van a migrar antes de continuar con el paso siguiente. Esto solo tardará unos minutos. 
2. En **Replicación de máquinas**, haga clic con el botón derecho en la máquina virtual > **Migrar**.
3. En **Migrar** > **Shut down virtual machines and perform a planned migration with no data loss** (¿Apagar las máquinas virtuales y realizar una migración planeada sin pérdida de datos?), seleccione **No** > **Aceptar**.
   
    > [!NOTE]
    > En el caso de la migración de servidores físicos, no se admite automáticamente el apagado de la máquina de origen. La recomendación es desactivar la aplicación como parte de la ventana de migración (no permitir que las aplicaciones acepten ninguna conexión) y, luego, iniciar la migración (el servidor debe mantenerse en ejecución, por lo que los cambios restantes puedan sincronizarse) antes de que se complete la migración.

4. Se inicia un trabajo de migración de la máquina virtual. Realice un seguimiento del trabajo en las notificaciones de Azure.
5. Una vez finalizado el trabajo, la máquina virtual puede ver y administrar desde la página **Máquinas virtuales**.

## <a name="reconfigure-cluster"></a>Reconfiguración del clúster 

Una vez que las máquinas virtuales se hayan migrado, vuelva a configurar el clúster. Siga estos pasos: 

1. Apague los servidores migrados en Azure.
1.  Agregue las máquinas migradas al grupo de back-end del equilibrador de carga. Vaya a **Load Balancer** > **Grupos de back-end** > seleccione el grupo de back-end > **Add migrated machines** (Agregar máquinas migradas). 3. Inicie los servidores migrados en Azure e inicie sesión en cualquier nodo. 
1. Copie el archivo `ClusterConfig.csv` y ejecute el script `Update-ClusterConfig.ps1` pasando el archivo CSV como parámetro. Así se garantiza que los recursos del clúster se actualizan con la nueva configuración para que el clúster funcione en Azure. 

   ```powershell
   ./Update-ClusterConfig.ps1 -ConfigFilePath $filepath
   ```

El grupo de disponibilidad AlwaysOn está listo. 

## <a name="complete-the-migration"></a>Completar la migración

1. Una vez finalizada la migración, haga clic con el botón derecho en la máquina virtual > **Detener migración**. Esto hace lo siguiente:
    - Detiene la replicación en la máquina local.
    - Quita la máquina del recuento de **Servidores en replicación** en Azure Migrate: Server Migration.
    - Limpia la información del estado de replicación de la máquina.
2. Instale el agente de [Windows](/virtual-machines/extensions/agent-windows.md) de la máquina virtual de Azure en las máquinas migradas.
3. Realice los ajustes de la aplicación posteriores a la migración, como actualizar las cadenas de conexión de la base de datos y las configuraciones del servidor web.
4. Realice las pruebas finales de la aplicación y la aceptación de la migración en la aplicación migrada que ahora se ejecuta en Azure.
5. Pase el tráfico a la instancia de máquina virtual de Azure migrada.
6. Quite las máquinas virtuales locales del inventario de máquinas virtuales local.
7. Quite las máquinas virtuales locales de las copias de seguridad locales.
8. Actualice la documentación interna para mostrar la nueva ubicación y la dirección IP las máquinas virtuales de Azure. 

## <a name="post-migration-best-practices"></a>Procedimientos recomendados después de la migración

- En SQL Server:
    -  Instale la [extensión del agente de IaaS de SQL Server](../../virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md) para automatizar las tareas de administración. 
    - [Optimice](../../virtual-machines/windows/performance-guidelines-best-practices-checklist.md) el rendimiento de SQL Server en las máquinas virtuales de Azure. 
    - Conozca los [precios](../../virtual-machines/windows/pricing-guidance.md#free-licensed-sql-server-editions) de SQL Server en Azure. 
- Para aumentar la resistencia:
    - Proteja los datos mediante la copia de seguridad de máquinas virtuales de Azure con el [servicio Azure Backup](../../../backup/quick-backup-vm-portal.md). 
    - Mantenga las cargas de trabajo en ejecución y disponibles continuamente mediante la replicación de máquinas virtuales de Azure en una región secundaria con [Site Recovery](../../../site-recovery/azure-to-azure-tutorial-enable-replication.md).
- Para aumentar la seguridad:
    - Bloquee y limite el acceso de tráfico de entrada con la [administración Just-In-Time de Azure Security Center](../../../security-center/security-center-just-in-time.md).
    - Restrinja el tráfico de red a los puntos de conexión de administración con [grupos de seguridad de red](../../../virtual-network/network-security-groups-overview.md).
    - Implemente [Azure Disk Encryption](../../../security/fundamentals/azure-disk-encryption-vms-vmss.md) para ayudar a proteger discos y datos frente al robo y acceso no autorizado.
    - Obtenga más información sobre la [protección de recursos IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) y visite [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Para supervisión y administración:
    - Considere la posibilidad de implementar [Azure Cost Management](../../../cost-management-billing/cloudyn/overview.md) para supervisar el gasto y el uso de recursos.


## <a name="next-steps"></a>Pasos siguientes

Investigue el [proceso de la migración en la nube](/azure/architecture/cloud-adoption/getting-started/migrate) en el marco de Cloud Adoption Framework para Azure.