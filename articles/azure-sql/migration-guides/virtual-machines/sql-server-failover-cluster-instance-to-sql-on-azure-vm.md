---
title: Migración de una instancia de clúster de conmutación por error
description: Aprenda a elevar y cambiar la solución de alta disponibilidad de la instancia de clúster de conmutación por error de Always On a SQL Server en máquinas virtuales de Azure mediante Azure Migrate.
ms.service: virtual-machines-sql
ms.subservice: migration-guide
author: rahugup
manager: bsiva
ms.topic: how-to
ms.date: 4/25/2021
ms.author: rahugup
ms.openlocfilehash: 88cade73c03f1dd3562ac196cbad5091981c2058
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777024"
---
# <a name="migrate-failover-cluster-instance-to-sql-server-on-azure-vms"></a>Migración de una instancia de clúster de conmutación por error a SQL Server en máquinas virtuales de Azure

En este artículo se enseña a migrar la instancia de clúster de conmutación por error (FCI) de Always On a SQL Server en máquinas virtuales de Azure mediante [Azure Migrate: Herramienta Server Migration](../../../migrate/migrate-services-overview.md#azure-migrate-server-migration-tool). Con la herramienta de migración, podrá migrar cada nodo de la instancia de clúster de conmutación por error a una máquina virtual de Azure que hospeda SQL Server, así como los metadatos de clúster y FCI.  

En este artículo aprenderá a: 

> [!div class="checklist"]
> * Preparar Azure y el entorno de origen para la migración.
> * Iniciar la replicación de las máquinas virtuales.
> * Supervisar la replicación.
> * Ejecutar una migración completa de la máquina virtual.
> * Volver a configurar un clúster de conmutación por error de SQL con discos compartidos de Azure.


En esta guía se usa el enfoque de migración basado en agente de Azure Migrate, que trata cualquier servidor o máquina virtual como un servidor físico. Al migrar máquinas físicas, Azure Migrate:Server Migration emplea la misma arquitectura de replicación que la recuperación ante desastres basada en agente del servicio Azure Site Recovery, y algunos componentes comparten el mismo código base. Puede que algún contenido se vincule a la documentación de Site Recovery.


## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, debe:

1. Suscripción a Azure. Cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/), si es necesario. 
1. Instale el [módulo `Az` de Azure PowerShell](/powershell/azure/install-az-ps). 
1. Descargue los [scripts de ejemplos de PowerShell](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/SQL%20Migration) desde el repositorio de GitHub.

## <a name="prepare-azure"></a>Preparación de Azure

Prepare Azure para la migración con Server Migration.

**Task** | **Detalles**
--- | ---
**Crear un proyecto de Azure Migrate** | La cuenta de Azure necesita permisos de colaborador o propietario para [crear un proyecto](https://docs.microsoft.com/azure/migrate/create-manage-projects).
**Comprobación de los permisos de la cuenta de Azure** | La cuenta de Azure necesita permisos de colaborador o propietario en la suscripción de Azure, permisos para registrar aplicaciones de Azure Active Directory (AAD) y permisos de administrador de acceso de usuarios en la suscripción de Azure para crear una instancia de Key Vault, crear una máquina virtual y escribir en un disco administrado de Azure.
**Configuración de una red virtual de Azure** | [Configure](../../../virtual-network/manage-virtual-network.md#create-a-virtual-network) una red virtual de Azure (VNet). Al realizar la replicación en Azure, se crean máquinas virtuales de Azure y se unen a la red virtual de Azure que se especifica al configurar la migración.


Para comprobar que tiene los permisos adecuados, siga estos pasos: 

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y, después, selecciónela para ver los permisos.
3. Debe tener permisos de **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.

Si necesita asignar permisos, siga los pasos descritos en [Preparación de una cuenta de usuario de Azure](../../../migrate/tutorial-discover-vmware.md#prepare-an-azure-user-account).


## <a name="prepare-for-migration"></a>Preparación para la migración

Para preparar la migración del servidor, debe comprobar la configuración del servidor y preparar la implementación de un dispositivo de replicación.

### <a name="check-machine-requirements"></a>Comprobación de los requisitos de la máquina

Asegúrese de que las máquinas cumplen los requisitos para la migración a Azure. 

1. [Comprobación](../../../migrate/migrate-support-matrix-physical-migration.md#physical-server-requirements) de los requisitos del servidor.
2. Compruebe que las máquinas de origen que replique en Azure cumplan los [requisitos de máquina virtual de Azure](../../../migrate/migrate-support-matrix-physical-migration.md#azure-vm-requirements).
1. Algunos orígenes de [Windows](../../../migrate/prepare-for-migration.md#windows-machines) requieren algunos cambios adicionales. Migrar el origen antes de realizar estos cambios podría impedir que la máquina virtual arranque en Azure. En algunos sistemas operativos, Azure Migrate realiza estos cambios automáticamente. 

### <a name="prepare-for-replication"></a>Preparación para la replicación 

Azure Migrate: Server Migration usa un dispositivo de replicación para replicar máquinas en Azure. Este dispositivo ejecuta los siguientes componentes: 

- **Servidor de configuración**: El servidor de configuración coordina la comunicación entre el entorno local y Azure, además de administrar la replicación de datos.
- **Servidor de proceso**: El servidor de procesos actúa como puerta de enlace de replicación. Recibe los datos de la replicación; los optimiza mediante el almacenamiento en la caché, la compresión y el cifrado, y los envía a una cuenta de almacenamiento en Azure. 

Para prepararse para la implementación del dispositivo, siga estos pasos:

- Cree una máquina con Windows Server 2016 para hospedar el dispositivo de replicación.  Revise los [requisitos de la máquina](../../../migrate/migrate-replication-appliance.md#appliance-requirements).
- El dispositivo de replicación usa MySQL. Revise las [opciones](../../../migrate/migrate-replication-appliance.md#mysql-installation) para instalar MySQL en el dispositivo.
- Revise las direcciones URL de Azure necesarias para que el dispositivo de replicación acceda a las nubes [públicas](../../../migrate/migrate-replication-appliance.md#url-access) y [gubernamentales](../../../migrate/migrate-replication-appliance.md#azure-government-url-access).
- Revise los requisitos de acceso a [puertos](../../../migrate/migrate-replication-appliance.md#port-access) para el dispositivo de replicación.

> [!NOTE]
> El dispositivo de replicación debe instalarse en una máquina que no sea la máquina de origen que va a replicar o migrar, y no en ninguna máquina en la que se haya instalado el dispositivo de detección y evaluación de Azure Migrate antes.

### <a name="download-replication-appliance-installer"></a>Descarga del instalador del dispositivo de replicación

Para descargar el instalador del dispositivo de replicación, siga estos pasos: 

1. En el proyecto de Azure Migrate > **Servidores**, en **Azure Migrate: Server Migration**, seleccione **Detectar**.

    ![Detectar máquinas virtuales](../../../migrate/media/tutorial-migrate-physical-virtual-machines/migrate-discover.png)

1. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **Físico o de otro tipo (AWS, GCP, Xen, etc.)** .
1. En **Región de destino**, seleccione la región de Azure a la que desea migrar las máquinas.
1. Seleccione **Confirme que la región de destino de la migración es nombreDeRegión**.
1. Seleccione **Crear recursos**. Esto crea un almacén de Azure Site Recovery en segundo plano.
    - Si ya ha configurado la migración con Azure Migrate Server Migration, no se puede configurar la opción de destino, ya que los recursos se configuraron anteriormente.    
    - Después de seleccionar este botón ya no se puede cambiar la región de destino de este proyecto.
    - Todas las migraciones posteriores se realizan a esta región.

1. En **¿Quiere instalar un nuevo dispositivo de replicación?** , seleccione **Instalar un dispositivo de replicación**.
1. En **Descargue e instale el software del dispositivo de replicación**, descargue el instalador del dispositivo y la clave de registro. Necesitará la clave para registrar el dispositivo. La clave será válida durante cinco días a partir del momento en que se descarga.

    ![Descarga del proveedor](../../../migrate/media/tutorial-migrate-physical-virtual-machines/download-provider.png)

1. Copie el archivo de instalación y el archivo de clave del dispositivo en el equipo con Windows Server 2016 que creó para el dispositivo.
1. Una vez finalizada la instalación, el asistente para la configuración de dispositivos se iniciará automáticamente (también puede iniciarlo manualmente mediante el acceso directo cspsconfigtool que se crea en el escritorio de la máquina del dispositivo). Use la pestaña **Administrar cuentas** del asistente para crear una cuenta ficticia con los detalles siguientes:

   -  "guest" como nombre descriptivo
   -  "username" como nombre de usuario
   -  "password" como contraseña de la cuenta 
   
   Esta cuenta ficticia la usará en la fase de habilitación de la replicación. 

1. Una vez que se haya completado la configuración y reiniciado el dispositivo, en **Detectar máquinas**, seleccione el nuevo dispositivo en **Seleccionar servidor de configuración** y seleccione **Finalizar registro**. El paso de finalización del registro realiza un par de tareas finales para preparar el dispositivo de replicación.

    ![Finalizar el registro](../../../migrate/media/tutorial-migrate-physical-virtual-machines/finalize-registration.png)


## <a name="install-the-mobility-service"></a>Instalación de Mobility Service 

Instale el servicio Mobility en los servidores que desea migrar. Los instaladores del agente están disponibles en el dispositivo de replicación. Debe encontrar el instalador correcto e instalar el agente en cada máquina que desee migrar. 

Para instalar el servicio Mobility, siga estos pasos: 

1. Inicie sesión en el dispositivo de replicación.
2. Vaya a **%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository**.
3. Busque el instalador correspondiente al sistema operativo y la versión del equipo. Revise los [sistemas operativos compatibles](/site-recovery/vmware-physical-azure-support-matrix.md#replicated-machines). 
4. Copie el archivo de instalación en el equipo que desea migrar.
5. Asegúrese de que tiene la frase de contraseña que se generó al implementar el dispositivo.
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

Para preparar las máquinas de origen, necesitará información del clúster. 

> [!CAUTION]
> - Mantenga la propiedad del disco a lo largo del proceso de replicación hasta la transición final. Si hay un cambio en la propiedad del disco, existe la posibilidad de que los volúmenes se puedan dañar y que la replicación tenga que volver a desencadenarse. Establezca el propietario preferido de cada disco para evitar la transferencia de propiedad durante el proceso de replicación. 
> - Evite las actividades de aplicación de revisiones y los reinicios del sistema durante el proceso de replicación para evitar la transferencia de la propiedad del disco.

Para preparar las máquinas de origen, realice lo siguiente: 

1. **Identifique la propiedad del disco:** inicie sesión en uno de los nodos de clúster y abra Administrador de clústeres de conmutación por error. Identifique el nodo propietario de los discos para determinar los discos que deben migrarse con cada servidor.  
2. **Recupere la información del clúster:** ejecute el script `Get-ClusterInfo.ps1`en un nodo de clúster para recuperar información sobre los recursos de clúster. El script generará el nombre del rol, el nombre del recurso, la dirección IP y el puerto de sondeo en el archivo `Cluster-Config.csv`. Use este archivo CSV para crear y asignar recursos en Azure más adelante en este artículo.  
    
   ```powershell
   ./Get-ClusterInfo.ps1
   ```

## <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Para que el clúster y los roles de clúster respondan correctamente a las solicitudes, se requiere un equilibrador de carga de Azure. Sin un equilibrador de carga, las demás máquinas virtuales no pueden acceder a la dirección IP del clúster, ya que no se reconocen como que pertenecen a la red o al clúster. 

1. Rellene las columnas del archivo `Cluster-Config.csv`: 

    **Encabezado de columna** | **Descripción**
    --- | ---
    NewIP | Especifique la dirección IP en la red virtual de Azure (o subred) para cada recurso del archivo CSV.
    ServicePort | Especifique el puerto de servicio que usará cada recurso en el archivo CSV. Para el recurso de clúster de SQL, use el mismo valor para el puerto de servicio que el puerto de sondeo en el CSV. Para otros roles de clúster, los valores predeterminados usados son 1433, pero puede seguir usando los números de puerto configurados en la configuración actual. 


1. Ejecute el script `Create-ClusterLoadBalancer.ps1` para crear el equilibrador de carga con los siguientes parámetros obligatorios: 

    **Parámetro** | **Tipo** | **Descripción**
    --- | --- | ---
    ConfigFilePath | Mandatory |  Especifique la ruta de acceso del archivo `Cluster-Config.csv` que ha rellenado en el paso anterior.
    ResourceGroupName | Mandatory| Especifique el nombre del grupo de recursos en el que se va a crear el equilibrador de carga. 
    VNetName | Mandatory| Especifique el nombre de la red virtual de Azure a la que se asociará el equilibrador de carga. 
    SubnetName | Mandatory| Especifique el nombre de la subred de la red virtual de Azure a la que se asociará el equilibrador de carga. 
    VNetResourceGroupName | Mandatory| Especifique el nombre del grupo de recursos de la red virtual de Azure al que se asociará el equilibrador de carga. 
    Location | Mandatory| Especifique la ubicación en la que se debe crear el equilibrador de carga. 
    LoadBalancerName | Mandatory| Especifique el nombre del equilibrador de carga que se va a crear. 


    ```powershell
    ./Create-ClusterLoadBalancer.ps1 -ConfigFilePath ./cluster-config.csv -ResourceGroupName $resoucegroupname -VNetName $vnetname -subnetName $subnetname -VnetResourceGroupName $vnetresourcegroupname -Location “eastus” -LoadBalancerName $loadbalancername
    ```

## <a name="replicate-machines"></a>Replicación de máquinas

Ahora, seleccione las máquinas para la migración. Puede replicar hasta 10 máquinas juntas. Si necesita replicar más, replíquelas simultáneamente en lotes de 10.

1. En el proyecto de Azure Migrate > **Servidores**, **Azure Migrate: Migración del servidor**, seleccione **Replicar**.

    ![Captura de pantalla de Azure Migrate > Servidores que muestra el botón Replicar seleccionado en Azure Migrate: Migración del servidor en Herramientas de migración.](../../../migrate/media/tutorial-migrate-physical-virtual-machines/select-replicate.png)

1. En **Replicar**, > **Configuración de origen** >  **¿Las máquinas están virtualizadas?** , seleccione **Físico o de otro tipo (AWS, GCP, Xen, etc.)** .
1. En **Dispositivo local**, seleccione el nombre del dispositivo de Azure Migrate que configuró.
1. En **Servidor de procesos**, seleccione el nombre del dispositivo de replicación.
1. En **Credenciales de invitado**, seleccione la cuenta ficticia creada anteriormente en la [configuración del instalador de la replicación](#download-replication-appliance-installer). A continuación, seleccione **Siguiente: Máquinas virtuales**.

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
    - Use la lista que creó anteriormente para seleccionar los discos que se replicarán con cada servidor. Excluya los demás discos de la replicación.
   

    ![Configuración de discos](../../../migrate/media/tutorial-migrate-physical-virtual-machines/disks.png)

1. En **Revisar e iniciar la replicación**, revise la configuración y seleccione **Replicar** para iniciar la replicación inicial de los servidores.

> [!NOTE]
> Puede actualizar la configuración de replicación en cualquier momento antes de que esta comience; para ello, vaya a **Administrar** > **Replicación de máquinas**. Una vez iniciada la replicación, su configuración no se puede cambiar.

## <a name="track-and-monitor"></a>Seguimiento y supervisión

La replicación continúa en la siguiente secuencia: 

- Al seleccionar **Replicar**, comienza el trabajo de _inicio de replicación_. 
- Cuando el trabajo de _inicio de replicación_ finaliza correctamente, las máquinas comienzan su replicación inicial en Azure.
- Cuando finaliza esta replicación inicial, comienza la replicación diferencial. Los cambios incrementales de los discos locales se replican periódicamente en los discos de réplica de Azure.
- Una vez completada la replicación inicial, configure los elementos Proceso y Red para cada máquina virtual. Normalmente, los clústeres tienen varias NIC, pero solo se requiere una para la migración (establezca las demás como No crear).

Puede realizar un seguimiento del estado del trabajo en las notificaciones del portal.

Para supervisar el estado de la replicación, seleccione **Replicando servidores** en **Azure Migrate: Server Migration**.
![Supervisión de la replicación](../../../migrate/media/tutorial-migrate-physical-virtual-machines/replicating-servers.png)


## <a name="migrate-vms"></a>Migración de máquinas virtuales

Una vez replicadas las máquinas, están listas para la migración. Para migrar los servidores, siga estos pasos: 

1. En el proyecto de Azure Migrate > **Servidores** > **Azure Migrate: Server Migration**, seleccione **Replicando servidores**.

    ![Replicando servidores](../../../migrate/media/tutorial-migrate-physical-virtual-machines/replicate-servers.png)

1. Para asegurarse de que el servidor migrado está sincronizado con el servidor de origen, detenga el recurso de SQL Server (en **Administrador de clústeres de conmutación por error** > **Roles** > **Otros recursos**) y asegúrese de que los discos del clúster están en línea.   
1. En **Replicación de máquinas** > seleccione en nombre del servidor > **Información general** y asegúrese de que la última marca de tiempo sincronizada es después de haber detenido un recurso de SQL Server en los servidores que se van a migrar antes de pasar al paso siguiente. Esta acción solo tardará unos minutos. 
1. En **Replicación de máquinas**, haga clic con el botón derecho en la máquina virtual > **Migrar**.
1. En **Migrar** >  **¿Quiere apagar las máquinas virtuales y realizar una migración planificada sin perder datos?** , seleccione **No** > **Aceptar**.
   
   > [!NOTE]
   > Para la migración de servidor físico, no se admite automáticamente el apagado de la máquina de origen. Se recomienda que la aplicación deje de estar disponible como parte de la ventana de migración (no permita que las aplicaciones acepten ninguna conexión) y, a continuación, inicie la migración (el servidor debe mantenerse en ejecución, por lo que los cambios restantes pueden sincronizarse) antes de que se complete la migración.

1. Se inicia un trabajo de migración de la máquina virtual. Realice un seguimiento del trabajo en las notificaciones de Azure.
1. Una vez finalizado el trabajo, la máquina virtual puede ver y administrar desde la página **Máquinas virtuales**.

## <a name="reconfigure-cluster"></a>Reconfiguración del clúster 

Una vez que las máquinas virtuales se hayan migrado, vuelva a configurar el clúster. Siga estos pasos: 

1. Apague los servidores migrados en Azure.
1. Agregue las máquinas migradas al grupo de back-end del equilibrador de carga. Vaya a **Load Balancer** > **Grupos de back-end** > seleccione el grupo de back-end > **Agregar máquinas migradas**. 

1. Vuelva a configurar los discos migrados de los servidores como discos compartidos mediante la ejecución del script `Create-SharedDisks.ps1`. El script es interactivo y solicitará una lista de máquinas y, a continuación, mostrará los discos disponibles que se van a extraer (solo discos de datos). Se le pedirá una vez que seleccione qué máquinas contienen las unidades que se van a convertir en discos compartidos. Una vez seleccionado, se le pedirá de nuevo, una vez por máquina, que elija los discos específicos. 

   **Parámetro** | **Tipo** | **Descripción**
   --- | --- | ---
   ResourceGroupName | Mandatory |  Especifique el nombre del grupo de recursos que contiene los servidores migrados.
   NumberofNodes | Opcional | Especifique el número de nodos de la instancia de clúster de conmutación por error. Este parámetro se usa para identificar la SKU adecuada para los discos compartidos que se crearán. De manera predeterminada, el script supone que el número de nodos del clúster es 2.   
   DiskNamePrefix | Opcional | Especifique el prefijo que desea agregar a los nombres de los discos compartidos. 
   
   ```powershell 
   ./Create-SharedDisks.ps1 -ResourceGroupName $resoucegroupname -NumberofNodes $nodesincluster -DiskNamePrefix $disknameprefix 
   ```

1. Asocie los discos compartidos a los servidores migrados mediante la ejecución del script `Attach-SharedDisks.ps1`. 

   **Parámetro** | **Tipo** |**Descripción**
   --- | ---  | ---
   ResourceGroupName | Mandatory | Especifique el nombre del grupo de recursos que contiene los servidores migrados.
   StartingLunNumber | Opcional |Especifique el número de LUN inicial que está disponible para los discos compartidos a los que se va a asociar. De manera predeterminada, el script intenta asociar discos compartidos a LUN a partir de 0.  
   
   ```powershell 
   ./Attach-ShareDisks.ps1 -ResourceGroupName $resoucegroupname 
   ```

1. Inicie los servidores migrados en Azure e inicie sesión en cualquier nodo. 

1. Copie el archivo `ClusterConfig.csv` y ejecute el script pasando el archivo CSV como parámetro `Update-ClusterConfig.ps1`. Esto garantizará que los recursos del clúster se actualizan con la nueva configuración para que el clúster funcione en Azure. 

   ```powershell
   ./Update-ClusterConfig.ps1 -ConfigFilePath $filepath
   ```

La instancia de clúster de conmutación por error de SQL Server está lista. 

## <a name="complete-the-migration"></a>Completar la migración

1. Una vez finalizada la migración, haga clic con el botón derecho en la máquina virtual > **Detener migración**. Esto hace lo siguiente:
    - Detiene la replicación en la máquina local.
    - Quita la máquina del recuento de **Servidores en replicación** en Azure Migrate: Server Migration.
    - Limpia la información del estado de replicación de la máquina.
1. Instale el agente de [Windows](/virtual-machines/extensions/agent-windows.md) de la máquina virtual de Azure en las máquinas migradas.
1. Realice los ajustes de la aplicación posteriores a la migración, como actualizar las cadenas de conexión de la base de datos y las configuraciones del servidor web.
1. Realice las pruebas finales de la aplicación y la aceptación de la migración en la aplicación migrada que ahora se ejecuta en Azure.
1. Pase el tráfico a la instancia de máquina virtual de Azure migrada.
1. Quite las máquinas virtuales locales del inventario de máquinas virtuales local.
1. Quite las máquinas virtuales locales de las copias de seguridad locales.
1. Actualice la documentación interna para mostrar la nueva ubicación y la dirección IP las máquinas virtuales de Azure. 

## <a name="post-migration-best-practices"></a>Procedimientos recomendados después de la migración

- En SQL Server:
    -  Instale la [extensión del agente de IaaS de SQL Server](../../virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md) para automatizar las tareas de administración. 
    - [Optimice](../../virtual-machines/windows/performance-guidelines-best-practices-checklist.md) el rendimiento de SQL Server en máquinas virtuales de Azure. 
    - Entienda los [precios](../../virtual-machines/windows/pricing-guidance.md#free-licensed-sql-server-editions) de SQL Server en Azure. 
- Para aumentar la resistencia:
    - Proteja los datos mediante la copia de seguridad de máquinas virtuales de Azure mediante el [servicio Azure Backup](../../../backup/quick-backup-vm-portal.md). 
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
