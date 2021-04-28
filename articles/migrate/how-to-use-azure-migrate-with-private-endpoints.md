---
title: Uso de Azure Migrate con puntos de conexión privados
description: Use el soporte de vínculo privado de Azure Migrate para detectar, evaluar y migrar mediante vínculo privado.
author: deseelam
ms.author: deseelam
ms.manager: bsiva
ms.topic: how-to
ms.date: 04/07/2020
ms.openlocfilehash: 97d4f0a387b75c9b23f64992a8ef39bc0bad17f0
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107715307"
---
# <a name="using-azure-migrate-with-private-endpoints"></a>Uso de Azure Migrate con puntos de conexión privados  

En este artículo se describe cómo usar Azure Migrate para detectar, evaluar y migrar servidores a través de una red privada mediante [Azure Private Link](https://docs.microsoft.com/azure/private-link/private-endpoint-overview). 

Puede usar las herramientas [Azure Migrate: Discovery and Assessment](https://docs.microsoft.com/azure/migrate/migrate-services-overview#azure-migrate-discovery-and-assessment-tool) y [Azure Migrate: Server Migration](https://docs.microsoft.com/azure/migrate/migrate-services-overview#azure-migrate-server-migration-tool) para conectarse de forma privada y segura al servicio Azure Migrate a través de un emparejamiento privado de ExpressRoute o una conexión VPN de sitio a sitio mediante Azure Private Link. 

Se recomienda el método de conectividad de punto de conexión privado cuando hay un requisito organizativo para acceder al servicio Azure Migrate y otros recursos de Azure sin atravesar redes públicas. También puede usar la compatibilidad con private link para usar los circuitos de emparejamiento privado de ExpressRoute existentes para mejorar los requisitos de ancho de banda o latencia. 

## <a name="support-requirements"></a>Requisitos de admisión 

### <a name="required-permissions"></a>Permisos necesarios

**Permisos de colaborador + administrador de acceso de usuario** o de **propietario** en la suscripción. 

### <a name="supported-scenarios-and-tools"></a>Herramientas y escenarios admitidos

**Implementación** | **Detalles** | **Herramientas** 
--- | --- | ---
**Detección y evaluación** | Realice una detección y una evaluación a escala sin agente de los servidores que se ejecutan en cualquier plataforma: plataformas de hipervisor como [VMware vSphere](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware) o [Microsoft Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-discover-hyper-v), nubes públicas como [AWS](https://docs.microsoft.com/azure/migrate/tutorial-discover-aws) o [GCP](https://docs.microsoft.com/azure/migrate/tutorial-discover-gcp), o incluso [servidores sin sistema operativo](https://docs.microsoft.com/azure/migrate/tutorial-discover-physical). | Azure Migrate: Discovery and Assessment  <br/> 
**Inventario de software** | Puede detectar aplicaciones, roles y características que se ejecutan en máquinas virtuales de VMware. | Azure Migrate: Discovery and Assessment  
**Visualización de dependencias** | Use la funcionalidad de análisis de dependencias para identificar y comprender las dependencias entre servidores. <br/> [La visualización de dependencias sin agente](https://docs.microsoft.com/azure/migrate/how-to-create-group-machine-dependencies-agentless) se admite de forma nativa con soporte de vínculo privado de Azure Migrate. <br/>[La visualización de dependencias basada en agente](https://docs.microsoft.com/azure/migrate/how-to-create-group-machine-dependencies) requiere conectividad a Internet. [Aprenda a usar](https://docs.microsoft.com/azure/azure-monitor/logs/private-link-security) puntos de conexión privados para la visualización de dependencias basada en agente. | Azure Migrate: Discovery and Assessment |
**Migración** | Realice [migraciones de Hyper-V sin agente](./tutorial-migrate-physical-virtual-machines.md) o use el enfoque basado en agente para migrar máquinas virtuales de [VMware](./tutorial-migrate-vmware-agent.md), máquinas virtuales de [Hyper-V](./tutorial-migrate-physical-virtual-machines.md), servidores [físicos](https://docs.microsoft.com/azure/migrate/tutorial-migrate-hyper-v), máquinas virtuales que se ejecutan en [AWS](./tutorial-migrate-aws-virtual-machines.md), máquinas virtuales que se ejecutan en [GCP](https://docs.microsoft.com/azure/migrate/tutorial-migrate-gcp-virtual-machines) o máquinas virtuales que se ejecutan en un proveedor de virtualización diferente. | Azure Migrate: Server Migration
 
>[!Note]
>
> [Las migraciones de VMware sin agente](https://docs.microsoft.com/azure/migrate/tutorial-migrate-vmware) requieren acceso a Internet o conectividad a través del emparejamiento de Microsoft ExpressRoute. <br/> [Aprenda a](https://docs.microsoft.com/azure/migrate/replicate-using-expressroute) usar puntos de conexión privados para realizar replicaciones a través del emparejamiento privado de ExpressRoute o una conexión VPN de sitio a sitio (S2S).  <br/><br/> 
   
#### <a name="other-integrated-tools"></a>Otras herramientas integradas

Es posible que algunas herramientas de migración no puedan cargar datos de uso en proyectos de Azure Migrate si el acceso a la red pública está deshabilitado. El proyecto de Azure Migrate debe configurarse para permitir que el tráfico de todas las redes reciba datos de otras ofertas de Microsoft o de [proveedores de software independientes (ISV)](https://docs.microsoft.com/azure/migrate/migrate-services-overview#isv-integration) externos. 


Para habilitar el acceso a la red pública para el proyecto de Azure Migrate, vaya a la **página de propiedades** de Azure Migrate en Azure Portal, seleccione **No** y seleccione **Guardar**.

![Diagrama que muestra cómo cambiar el modo de acceso a la red.](./media/how-to-use-azure-migrate-with-private-endpoints/migration-project-properties.png)

### <a name="other-considerations"></a>Otras consideraciones   

**Consideraciones** | **Detalles**
--- | --- 
**Precios** | Para obtener más información sobre los precios, consulte [Precios de Azure Blob](https://azure.microsoft.com/pricing/details/storage/page-blobs/) y [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).  
**Requisitos de red virtual** | El punto de conexión de ExpressRoute/VPN Gateway debe residir en la red virtual seleccionada o en una red virtual conectada a ella. Es posible que necesite ~15 direcciones IP en la red virtual.  

## <a name="create-a-project-with-private-endpoint-connectivity"></a>Creación de un proyecto con conectividad de punto de conexión privado

Use este [artículo](https://docs.microsoft.com/azure/migrate/create-manage-projects#create-a-project-for-the-first-time) para configurar un nuevo proyecto de Azure Migrate. 

> [!Note]
> No se puede cambiar el método de conectividad a conectividad de punto de conexión privado para proyectos de Azure Migrate existentes.

En la sección **Configuración avanzada**, proporcione los detalles siguientes para crear un punto de conexión privado para el proyecto de Azure Migrate.
- Elija **Punto de conexión privado** como **método de conectividad**. 
- En **Deshabilitar el acceso al punto de conexión público**, mantenga el valor predeterminado **No**. Es posible que algunas herramientas de migración no puedan cargar datos de uso en proyectos de Azure Migrate si el acceso a la red pública está deshabilitado. [Más información.](#other-integrated-tools)
- En **Suscripción de red virtual**, seleccione la suscripción de la red virtual del punto de conexión privado. 
- En **Red virtual**, seleccione la red virtual para el punto de conexión privado. El dispositivo de Azure Migrate y otros componentes de software que deben conectarse al proyecto de Azure Migrate deben estar en esta red virtual o en una conectada.
- En **Subred**, seleccione la subred para el punto de conexión privado. 

Seleccione **Crear**. Espere unos minutos para que se implemente el proyecto de Azure Migrate. No cierre esta página mientras se esté creando el proyecto.

![Crear proyecto](./media/how-to-use-azure-migrate-with-private-endpoints/create-project.png)

    
Esto crea un proyecto de migración y le asocia un punto de conexión privado. 

## <a name="discover-and-assess-servers-for-migration-using-azure-private-link"></a>Detección y valoración de servidores para la migración mediante Azure Private Link 

### <a name="set-up-the-azure-migrate-appliance"></a>Configuración del dispositivo de Azure Migrate 

1. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione el tipo de servidor.
2. En **Generación de la clave del proyecto de Azure Migrate**, especifique un nombre para el dispositivo de Azure Migrate. 
3. Seleccione **Generar clave** para crear los recursos de Azure necesarios. 

    > [!Important]
    > No cierre la página Detectar máquinas durante la creación de recursos.  
    - En este paso, Azure Migrate crea un almacén de claves, una cuenta de almacenamiento, un almacén de Recovery Services (solo para migraciones de VMware sin agente) y algunos recursos internos y asocia un punto de conexión privado a cada recurso. Los puntos de conexión privados se crean en la red virtual seleccionada durante la creación del proyecto.  
    - Una vez creados los puntos de conexión privados, los registros de recursos CNAME de DNS para los recursos de Azure Migrate se actualizan a un alias de un subdominio con el prefijo "privatelink". De forma predeterminada, Azure Migrate también crea una zona DNS privada correspondiente al subdominio "privatelink" para cada tipo de recurso e inserta registros DNS A para los puntos de conexión privados asociados. Esto permite que Azure Migrate dispositivo y otros componentes de software que residen en la red de origen lleguen a los puntos de conexión de recursos de Azure Migrate en direcciones IP privadas.  
    - Azure Migrate también habilita una [identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) para el proyecto de migración y concede permisos a la identidad administrada para acceder de forma segura a la cuenta de almacenamiento.  

4. Una vez generada correctamente la clave, copie los detalles de la clave para configurar y registrar el dispositivo.   

#### <a name="download-the-appliance-installer-file"></a>Descarga del archivo instalador del dispositivo

La herramienta Azure Migrate: Discovery and assessment usa un dispositivo de Azure Migrate ligero. El dispositivo de Azure Migrate detecta los servidores y envía los metadatos de configuración y rendimiento a Azure Migrate.

Para configurar el dispositivo, descargue el archivo comprimido que contiene el script del instalador desde el portal. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo.

Asegúrese de que el servidor cumple los [requisitos de hardware](https://docs.microsoft.com/azure/migrate/migrate-appliance) para el escenario elegido (VMware,Hyper-V/Físico u otro) y puede conectarse a las direcciones URL de Azure necesarias: nubes [públicas](./migrate-appliance.md#public-cloud-urls-for-private-link-connectivity) y [gubernamentales](./migrate-appliance.md#government-cloud-urls-for-private-link-connectivity).

Después de descargar el archivo comprimido, compruebe la seguridad de los archivos y ejecute el script del instalador para implementar el dispositivo.

#### <a name="verify-file-security"></a>Comprobación de la seguridad del archivo

Compruebe que el archivo comprimido es seguro antes de implementarlo.

1. Abra una ventana de comandos de administrador en el servidor en el que descargó el archivo. 
2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP.
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo: ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-VMware-PrivateEndPoint.zip SHA256```
3. Compruebe la versión y script más recientes del dispositivo para la nube pública de Azure:

    **Algoritmo** | **Descargar** | **SHA256**
    --- | --- | ---
    VMware (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2116601) | 85b74d93dfcee43412386141808d82147916330e6669df94c7969fe1b3d0fe72
    Hyper-V (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2116601) | 85b74d93dfcee43412386141808d82147916330e6669df94c7969fe1b3d0fe72
    Físico o de otro tipo (85,8 MB) | [La versión más reciente](https://go.microsoft.com/fwlink/?linkid=2116601) | 85b74d93dfcee43412386141808d82147916330e6669df94c7969fe1b3d0fe72

#### <a name="run-the-script"></a>Ejecute el script.

1. Extraiga el archivo comprimido en la carpeta del servidor que hospedará el dispositivo. 
2. Inicie PowerShell en el equipo con privilegios de administrador (elevados).
3. Cambie el directorio de PowerShell a la carpeta que contiene el contenido extraído del archivo comprimido descargado.
4. Ejecute el script **AzureMigrateInstaller.ps1** como se indica a continuación:

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1```
   
5. Una vez que el script se haya ejecutado correctamente, inicia el administrador de configuración del dispositivo para que pueda configurarlo. Si detecta algún problema, consulte los registros del script en C:\Archivos de programa\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="configure-the-appliance-and-start-continuous-discovery"></a>Configuración del dispositivo e inicio de la detección continua

Abra un explorador en cualquier equipo que pueda conectarse al servidor del dispositivo y abra la dirección URL del administrador de configuración del dispositivo: `https://appliance name or IP address: 44368`. También puede abrir el administrador de configuración desde el escritorio del servidor del dispositivo mediante la selección de su acceso directo.

#### <a name="set-up-prerequisites"></a>Configuración de requisitos previos

1. Lea la información de terceros y acepte los **términos de licencia**.    
 
2. En administrador de configuración > **Configurar los requisitos previos**, realice las siguientes operaciones:
   - **Conectividad:** el dispositivo comprueba el acceso a las direcciones URL necesarias. Si el servidor usa un proxy:
     - Haga clic en **Configurar el proxy** y especifique la dirección del proxy `http://ProxyIPAddress` o `http://ProxyFQDN`, y el puerto de escucha.
     - Especifique las credenciales si el proxy requiere autenticación. Solo se admite un proxy HTTP.
     - Si lo desea, puede agregar una lista de direcciones URL o direcciones IP que deben omitir el servidor proxy. Si usa el emparejamiento privado de ExpressRoute, asegúrese de omitir [estas direcciones URL](https://docs.microsoft.com/azure/migrate/replicate-using-expressroute#configure-proxy-bypass-rules-on-the-azure-migrate-appliance-for-vmware-agentless-migrations).
     - Debe seleccionar **Guardar** para registrar la configuración si ha actualizado los detalles del servidor proxy, o bien si ha agregado direcciones URL o IP para omitir el proxy.
     
        > [!Note]
        > Si recibe un error con el vínculo aka.ms/* durante la comprobación de conectividad y no desea que el dispositivo acceda a esta dirección URL a través de Internet, debe deshabilitar el servicio de actualización automática en el dispositivo siguiendo los pasos que se indican [**aquí**](https://docs.microsoft.com/azure/migrate/migrate-appliance#turn-off-auto-update). Una vez deshabilitada la actualización automática, se omitirá la comprobación de conectividad aka.ms/* de la dirección URL. 

   - **Time sync** (Sincronización de hora): Para que la detección funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
   - **Instalación de actualizaciones**: el dispositivo garantiza que se instalan las actualizaciones más recientes. Una vez finalizada la comprobación, puede seleccionar **Ver servicios del dispositivo** para ver el estado y las versiones de los servicios que se ejecutan en el servidor del dispositivo.
        > [!Note]
        > Si ha elegido deshabilitar el servicio de actualización automática en el dispositivo, puede actualizar los servicios del dispositivo manualmente para obtener las versiones más recientes de los servicios siguiendo los pasos que se indican [**aquí**](https://docs.microsoft.com/azure/migrate/migrate-appliance#manually-update-an-older-version).
   - **Install VDDK** (Instalar VDDK):(_solo necesario para dispositivos VMware)._ El dispositivo comprueba si VMware vSphere Virtual Disk Development Kit (VDDK) está instalado. Si no está instalado, descargue VDDK 6.7 de VMware y extraiga el contenido del archivo ZIP descargado en la ubicación especificada del dispositivo, tal como se indica en las **instrucciones de instalación**.

#### <a name="register-the-appliance-and-start-continuous-discovery"></a>Registro del dispositivo e inicio de la detección continua

Una vez completada la comprobación de requisitos previos, siga estos pasos para registrar el dispositivo e iniciar la detección continua para los escenarios respectivos: máquinas virtuales de [VMware](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#register-the-appliance-with-azure-migrate), máquinas virtuales de [Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-discover-hyper-v#register-the-appliance-with-azure-migrate), [servidores físicos](https://docs.microsoft.com/azure/migrate/tutorial-discover-physical#register-the-appliance-with-azure-migrate), [máquinas virtuales de AWS](https://docs.microsoft.com/azure/migrate/tutorial-discover-aws#register-the-appliance-with-azure-migrate), [máquinas virtuales de GCP](https://docs.microsoft.com/azure/migrate/tutorial-discover-gcp#register-the-appliance-with-azure-migrate).


>[!Note]
> Si recibe problemas de resolución DNS durante el registro del dispositivo o en el momento de iniciar la detección, asegúrese de que los recursos de Azure Migrate creados durante el paso **Generar clave** en el portal son accesibles desde el servidor local que hospeda el dispositivo Azure Migrate. [Obtenga más información sobre cómo comprobar la conectividad de red](#troubleshoot-network-connectivity).

### <a name="assess-your-servers-for-migration-to-azure"></a>Evaluación de servidores para migrar a máquinas virtuales de Azure
Una vez completada la detección, evalúe los servidores (máquinas virtuales de [VMware](https://docs.microsoft.com/azure/migrate/tutorial-assess-vmware-azure-vm), máquinas virtuales de [Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-assess-hyper-v), [servidores físicos](https://docs.microsoft.com/azure/migrate/tutorial-assess-vmware-azure-vm), [máquinas virtuales de AWS](https://docs.microsoft.com/azure/migrate/tutorial-assess-aws), [máquinas virtuales de GCP)](https://docs.microsoft.com/azure/migrate/tutorial-assess-gcp) para la migración a máquinas virtuales de Azure o Azure VMware Solution (AVS), mediante la herramienta Azure Migrate: Discovery and Assessment. 

También puede evaluar [las máquinas locales](https://docs.microsoft.com/azure/migrate/tutorial-discover-import#prepare-the-csv) con la herramienta Azure Migrate: Discovery and Assessment mediante un archivo de valores separados por comas (CSV) importado.   

## <a name="migrate-servers-to-azure-using-azure-private-link"></a>Migración de servidores a Azure mediante Azure Private Link

En las secciones siguientes se describen los pasos necesarios para usar Azure Migrate con [puntos de conexión privados](https://docs.microsoft.com/azure/private-link/private-endpoint-overview) para migraciones mediante el emparejamiento privado de ExpressRoute o conexiones VPN.  

En este artículo se muestra una ruta de implementación de prueba de concepto para que las replicaciones basadas en agente migren [las máquinas virtuales de VMware](./tutorial-migrate-vmware-agent.md), [las máquinas virtuales de Hyper-V](./tutorial-migrate-physical-virtual-machines.md), [los servidores físicos](./tutorial-migrate-physical-virtual-machines.md), [las máquinas virtuales que se ejecutan en AWS](./tutorial-migrate-aws-virtual-machines.md), [máquinas virtuales que se ejecutan en GCP](https://docs.microsoft.com/azure/migrate/tutorial-migrate-gcp-virtual-machines) o las máquinas virtuales que se ejecutan en un proveedor de virtualización diferente mediante puntos de conexión privados de Azure. Puede usar un enfoque similar para realizar [migraciones de Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-migrate-hyper-v) sin agente mediante Private Link.

>[!Note]
>[Las migraciones de VMware sin agente](https://docs.microsoft.com/azure/migrate/tutorial-assess-physical) requieren acceso a Internet o conectividad a través del emparejamiento de Microsoft ExpressRoute. 

### <a name="set-up-a-replication-appliance-for-migration"></a>Configuración de un dispositivo de replicación para la migración 

En el diagrama siguiente se muestra el flujo de trabajo de replicación basado en agente con puntos de conexión privados mediante la herramienta Azure Migrate: Server Migration.  

![Arquitectura de replicación](./media/how-to-use-azure-migrate-with-private-endpoints/replication-architecture.png)

La herramienta usa un dispositivo de replicación para replicar los servidores en Azure. Use este artículo para [preparar y configurar una máquina para el dispositivo de replicación](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#prepare-a-machine-for-the-replication-appliance).

Después de configurar el dispositivo de replicación, siga estas instrucciones para crear los recursos necesarios para la migración. 

1. En **Detectar máquinas** >  **¿Las máquinas están virtualizadas?** , seleccione **No virtualizado/Otro**.
2. En **Región de destino**, seleccione la región de Azure a la que desea migrar las máquinas.
3. Seleccione **Crear recursos** para crear los recursos de Azure necesarios. No cierre la página durante la creación de recursos.   
    - Esto crea un almacén de Recovery Services en segundo plano y habilita una identidad administrada para el almacén. Un almacén de Recovery Services es una entidad que contiene la información de replicación de las máquinas y se usa para desencadenar operaciones de Site Recovery.  
    - Si el proyecto Azure Migrate tiene conectividad de punto de conexión privado, se crea un punto de conexión privado para el almacén de Recovery Services. Esto agrega cinco nombres privados completos (FQDN) al punto de conexión privado, uno para cada microservicio vinculado al almacén de Recovery Services.   
    - Para dar formato a los cinco nombres de dominio se usa este patrón: <br/> _{Vault-ID}-asr-pod01-{type}-.{target-geo-code}_ .privatelink.siterecovery.windowsazure.com  
    - De forma predeterminada, Azure Migrate crea automáticamente una zona DNS privada y agrega registros DNS A para los microservicios del almacén de Recovery Services. A continuación, la zona DNS privada se vincula a la red virtual del punto de conexión privado. Esto permite que el dispositivo de replicación local resuelva los nombres de dominio completos en sus direcciones IP privadas.

4. Antes de registrar el dispositivo de replicación, asegúrese de que los FQDN del vínculo privado del almacén son accesibles desde la máquina que hospeda el dispositivo de replicación. [Obtenga más información sobre cómo comprobar la conectividad de red](#troubleshoot-network-connectivity). 

5. Una vez que compruebe la conectividad, descargue la configuración del dispositivo y el archivo de clave, ejecute el proceso de instalación y registre el dispositivo en Azure Migrate. Revise los [pasos detallados aquí](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#set-up-the-replication-appliance). Después de configurar el dispositivo de replicación, siga estas instrucciones para instalar el [servicio Mobility](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#install-the-mobility-service) en las máquinas que desea migrar. 

### <a name="replicate-servers-to-azure-using-azure-private-link"></a>Replicación de servidores en Azure mediante Azure Private Link 

Ahora, siga [estos pasos para](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#replicate-machines) seleccionar servidores para la replicación.  

En **Replicate** > **Target settings** > **Cache/Replication storage account** (Cuenta de almacenamiento de caché o replicación), use la lista desplegable para seleccionar una cuenta de almacenamiento para replicar a través de un vínculo privado.  

Si el proyecto de Azure Migrate tiene conectividad de punto de conexión privado, debe conceder permisos a la identidad administrada del almacén de  [Recovery Services](#grant-access-permissions-to-the-recovery-services-vault) para acceder a la cuenta de almacenamiento requerida por Azure Migrate.   

Además, para habilitar las replicaciones a través de un vínculo privado, [cree un punto de conexión privado para la cuenta de almacenamiento.](#create-a-private-endpoint-for-the-storage-account-optional)

#### <a name="grant-access-permissions-to-the-recovery-services-vault"></a>Concesión de permisos de acceso al almacén de Recovery Services

La identidad administrada del almacén de Recovery Services requiere permisos para el acceso autenticado a la cuenta de almacenamiento de caché o replicación. 

Use las instrucciones siguientes para identificar el almacén de Recovery Services creado por Azure Migrate y conceder los permisos necesarios. 

**_Identificación del almacén de Recovery Services y el identificador de objeto de identidad administrada_**

Puede encontrar los detalles del almacén de Recovery Services en la página de **propiedades** de Azure Migrate: Server Migration. 

1. Vaya al **centro de Azure Migrate**, seleccione **Información general** en el mosaico Azure Migrate: Server Migration.

    ![Página de información general en el centro de Azure Migrate](./media/how-to-use-azure-migrate-with-private-endpoints/hub-overview.png)

2. Seleccione **Propiedades** en el panel izquierdo. Anote el nombre del almacén de Recovery Services y el identificador de identidad administrada. El almacén tendrá un _punto de conexión privado_ como **tipo de conectividad** y _Otro_ como tipo **de replicación**. Necesitará esta información mientras proporciona acceso al almacén.
      
    ![Página de propiedades de Azure Migrate: Server Migration](./media/how-to-use-azure-migrate-with-private-endpoints/vault-info.png)

**_Concesión de los permisos necesarios para acceder a la cuenta de almacenamiento_**

 A la identidad administrada del almacén se le deben conceder los siguientes permisos de rol en la cuenta de almacenamiento necesaria para la replicación.  En este caso, debe crear de antemano la cuenta de almacenamiento.

>[!Note]
> Para migrar máquinas virtuales de Hyper-V a Azure mediante Private Link, debe conceder acceso tanto a la cuenta de almacenamiento de replicación como a la cuenta de almacenamiento de caché. 

Los permisos de rol varían en función del tipo de cuenta de almacenamiento.

- Cuentas de almacenamiento basadas en Resource Manager (tipo Estándar):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor) _y_
  - [Colaborador de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)
- Cuentas de almacenamiento basadas en Resource Manager (tipo Premium):
  - [Colaborador](../role-based-access-control/built-in-roles.md#contributor) _y_
  - [Propietario de datos de blobs de almacenamiento](../role-based-access-control/built-in-roles.md#storage-blob-data-owner)

1. Vaya a la cuenta de almacenamiento de replicación/caché seleccionada para la replicación. Seleccione **Control de acceso (IAM)** en el panel izquierdo. 

1. En la sección **Agregar una asignación de roles**, seleccione **Agregar**:

   ![Adición de una asignación de roles](./media/how-to-use-azure-migrate-with-private-endpoints/storage-role-assignment.png)


1. En la página **Agregar asignación de rol**, en el campo **Rol**, seleccione el rol adecuado en la lista de permisos mencionada anteriormente. Escriba el nombre del almacén y seleccione **Guardar**.

    ![Control de acceso basado en rol](./media/how-to-use-azure-migrate-with-private-endpoints/storage-role-assignment-select-role.png)

4. Además de estos permisos, también debe permitir el acceso a los servicios de confianza de Microsoft. Si el acceso a la red está restringido a redes seleccionadas, seleccione **Permitir que los usuarios de confianza** servicios Microsoft accedan a esta cuenta de almacenamiento en la sección **Excepciones** de la pestaña **Redes**. 

![Permitir que los servicios de Microsoft de confianza accedan a la cuenta de almacenamiento](./media/how-to-use-azure-migrate-with-private-endpoints/exceptions.png)


### <a name="create-a-private-endpoint-for-the-storage-account-optional"></a>Creación de un punto de conexión privado para la cuenta de almacenamiento

Para la replicación mediante ExpressRoute con emparejamiento privado, [cree un punto de conexión privado](https://docs.microsoft.com/azure/private-link/tutorial-private-endpoint-storage-portal#create-storage-account-with-a-private-endpoint) para las cuenta de almacenamiento de caché o replicación (subrecurso de destino **_: blob_**). 

>[!Note]
>
> - Solo se pueden crear puntos de conexión privados en una cuenta de almacenamiento De uso general v2 (GPv2). Para obtener más información sobre los precios, consulte [Precios de Azure Blob](https://azure.microsoft.com/pricing/details/storage/page-blobs/) y [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).

El punto de conexión privado de la cuenta de almacenamiento debe crearse en la misma red virtual que el punto de conexión privado del proyecto de Azure Migrate u otra red virtual conectada a esta red. 

Seleccione **Sí** y realice la integración con una zona DNS privada. La zona DNS privada ayuda a enrutar las conexiones de la red virtual a la cuenta de almacenamiento a través de un vínculo privado. Al seleccionar **Sí**, la zona se vincula automáticamente a la red virtual de origen y se agregan los registros DNS para la resolución tanto de las nuevas direcciones IP como de los nombres de dominio completos que se creen. Más información sobre las [zonas DNS privadas.](https://docs.microsoft.com/azure/dns/private-dns-overview)

Si el usuario que creará el punto de conexión privado también es el propietario de la cuenta de almacenamiento, el punto de conexión privado se aprobará automáticamente. De lo contrario, el propietario de la cuenta de almacenamiento deberá aprobar el punto de conexión privado para su uso. Para aprobar o rechazar una conexión de punto de conexión privado solicitada, vaya a **Conexiones de punto de conexión privado** en **Redes** en la página de la cuenta de almacenamiento.

Revise el estado del estado de conexión del punto de conexión privado antes de continuar. 

![Esrtado de aprobación del punto de conexión privado](./media/how-to-use-azure-migrate-with-private-endpoints/private-endpoint-connection-state.png)

Después de crear el punto de conexión privado, use la lista desplegable de **Replicate** > **Target settings** > **Cache storage account** (Cuenta de almacenamiento de caché o replicación) para seleccionar la cuenta de almacenamiento para replicar a través de un vínculo privado.  

Asegúrese de que el dispositivo de replicación local tiene conectividad de red a la cuenta de almacenamiento en su punto de conexión privado. [Obtenga más información sobre cómo comprobar la conectividad de red](#troubleshoot-network-connectivity).

>[!Note]
>
> - En el caso de las migraciones de máquinas virtuales de Hyper-V a Azure, si la cuenta de almacenamiento de replicación es de tipo _Premium_, debe seleccionar otra cuenta de almacenamiento de tipo _Estándar_ para la cuenta de almacenamiento en caché. En este caso, debe crear puntos de conexión privados para la replicación y la cuenta de almacenamiento en caché.  

A continuación, siga estas instrucciones para [revisar e iniciar la replicación](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#replicate-machines) y [realizar migraciones](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#run-a-test-migration).  

## <a name="troubleshoot-network-connectivity"></a>Solución de problemas de conectividad de red 

### <a name="validate-private-endpoints-configuration"></a>Validación de la configuración de puntos de conexión privados 

Asegúrese de que el punto de conexión privado es un estado aprobado.  

1. Vaya a la página de propiedades de Azure Migrate: Detección y evaluación y Migración de servidor.
2. La página de propiedades contiene la lista de puntos de conexión privados y FQDN de vínculo privado creados automáticamente por Azure Migrate.  

3. Filtre y seleccione el punto de conexión privado que quiera diagnosticar.  
    1. Compruebe que el estado de la conexión es Aprobado.
    2. Si la conexión está en estado Pendiente, debe aprobarla.
    3. También puede ir al recurso de punto de conexión privado y revisar si la red virtual coincide con la red virtual del punto de conexión privado del proyecto de migración. 

    ![Visualización de la conexión del punto de conexión privado](./media/how-to-use-azure-migrate-with-private-endpoints/private-endpoint-connection.png)

### <a name="verify-dns-resolution"></a>Verificación de resolución de DNS 

El dispositivo local (o proveedor de replicación) accederá a los recursos de Azure Migrate mediante sus nombres de dominio de vínculo privado (FQDN) completos. Puede requerir una configuración adicional de DNS para resolver la dirección IP privada de los puntos de conexión privados desde el entorno de origen. [Use este artículo](https://docs.microsoft.com/azure/private-link/private-endpoint-dns#on-premises-workloads-using-a-dns-forwarder) para comprender los escenarios de configuración de DNS que pueden ayudar a solucionar cualquier problema de conectividad de red.  

Para validar la conexión del enlace privado, realice una resolución DNS de los puntos de conexión de recursos de Azure Migrate (FQDN de los recursos de vínculo privado) desde el servidor local en el que se aloja el dispositivo, y asegúrese de que se resuelve en una dirección IP privada. Los detalles del punto de conexión privado y la información de los FQDN de recursos de vínculo privado están disponibles en las páginas de propiedades Detección y valoración, y Migración del servidor. Seleccione **Descargar configuración de DNS** para ver la lista.   

 ![Propiedades de Azure Migrate: Detección y evaluación](./media/how-to-use-azure-migrate-with-private-endpoints/server-assessment-properties.png)

 ![Página de propiedades de Azure Migrate: Server Migration](./media/how-to-use-azure-migrate-with-private-endpoints/azure-migrate-server-migration-properties.png)

Ejemplo ilustrativo de la resolución DNS del FQDN del vínculo privado de la cuenta de almacenamiento.  

- Escriba _nslookup< nombre_de_la_cuenta_de_almacenamiento>_ .blob.core.windows.net.  Reemplace <nombre_de_la_cuenta_de_almacenamiento> por el nombre de la cuenta de almacenamiento usada en Azure Migrate.  

    Recibirá un mensaje similar a este:  

   ![Ejemploo de resolución DNS](./media/how-to-use-azure-migrate-with-private-endpoints/dns-resolution-example.png)

- Se devuelve una dirección IP privada de 10.1.0.5 para la cuenta de almacenamiento. Esta dirección pertenece a la subred de la red virtual del punto de conexión privado.   

Puede comprobar la resolución DNS de otros artefactos de Azure Migrate mediante un enfoque similar.   

Si la resolución DNS es incorrecta, siga estos pasos:  

- Si usa un DNS personalizado, revise la configuración personalizada de DNS y compruebe que esta sea correcta. Para más información, consulte [Introducción al punto de conexión privado: Configuración de DNS](https://docs.microsoft.com/azure/private-link/private-endpoint-overview#dns-configuration).
- Si usa servidores DNS proporcionados por Azure, consulte la sección siguiente para solucionar más problemas.  

> [!Tip]
> Puede actualizar manualmente los registros DNS del entorno de origen editando el archivo de hosts DNS en el dispositivo local con los FQDN del recurso de vínculo privado y sus direcciones IP privadas asociadas. Esta opción solo se recomienda para pruebas. <br/>  


### <a name="validate-the-private-dns-zone"></a>Valide la zona DNS privada   
Si la resolución DNS no funciona tal como se describe en la sección anterior, es posible que haya un problema con la zona DNS privada.  

#### <a name="confirm-that-the-required-private-dns-zone-resource-exists"></a>Confirmación de que existe el recurso de zona DNS privada requerido  
De forma predeterminada, Azure Migrate también crea una zona DNS privada correspondiente al subdominio "privatelink" para cada tipo de recurso. La zona DNS privada se creará en el mismo grupo de recursos de Azure que el grupo de recursos del punto de conexión privado. El grupo de recursos de Azure debe contener recursos de zona DNS privados con el formato siguiente:
- privatelink.vaultcore.azure.net para el almacén de claves 
- privatelink.blob.core.windows.net para la cuenta de almacenamiento
- privatelink.siterecovery.windowsazure.com para el almacén de Recovery Services (para replicaciones basadas en agente y Hyper-V)
- privatelink.prod.migration.windowsazure.com: proyecto de migración, el proyecto de evaluación y el sitio de detección.   

La zona DNS privada se creará automáticamente mediante Azure Migrate (excepto para la cuenta de almacenamiento de caché o replicación seleccionada por el usuario). Para buscar la zona DNS privada vinculada, vaya a la página del punto de conexión privado y seleccione Configuraciones de DNS. Debería ver la zona DNS privada en la sección integración de DNS privado. 

![Captura de pantalla de Configuración de DNS](./media/how-to-use-azure-migrate-with-private-endpoints/dns-configuration.png)  

Si la zona DNS no está presente (como se muestra a continuación), cree un nuevo [recurso de zona DNS privada.](https://docs.microsoft.com/azure/dns/private-dns-getstarted-portal)  

![Creación de una zona DNS privada](./media/how-to-use-azure-migrate-with-private-endpoints/create-dns-zone.png) 

#### <a name="confirm-that-the-private-dns-zone-is-linked-to-the-virtual-network"></a>Confirme que la zona DNS privada está vinculada una red virtual.  
La zona DNS privada debe estar vinculada a la red virtual que contiene el punto de conexión privado de la consulta de DNS para resolver la dirección IP privada del punto de conexión del recurso. Si la zona DNS privada no está vinculada a la instancia de red virtual correcta, cualquier resolución DNS de esa instancia de red virtual omitirá la zona DNS privada.   

Vaya al recurso de zona DNS privada de Azure Portal y seleccione los vínculos de la red virtual en el menú de la izquierda. Debería ver las redes virtuales vinculadas.

![Visualización de vínculos de red virtual](./media/how-to-use-azure-migrate-with-private-endpoints/virtual-network-links.png) 

Esta opción mostrará una lista de vínculos, cada uno con el nombre de una instancia de la red virtual de la suscripción. La instancia de red virtual que contiene el recurso de punto de conexión privado debe aparecer aquí. De lo contrario, [siga las instrucciones de este artículo](https://docs.microsoft.com/azure/dns/private-dns-getstarted-portal#link-the-virtual-network) para vincular la zona DNS privada a una red virtual.    

Una vez que la zona DNS privada esté vinculada a la instancia de red virtual, las solicitudes DNS procedentes de esa instancia de red virtual buscarán nombres en la zona DNS privada. Esto es necesario para la correcta resolución de direcciones que se realiza en las máquinas virtuales y que está conectada a la instancia de red virtual donde se creó el punto de conexión privado.   

#### <a name="confirm-that-the-private-dns-zone-contains-the-right-a-records"></a>Confirmación de que la zona DNS privada contiene el registro A correcto 

Vaya a la zona DNS privada que desea solucionar. La página Información general muestra todos los registros DNS de esa zona DNS privada. Compruebe que existe un registro DNS A para el recurso. El valor del registro A (la dirección IP) debe ser la dirección IP privada del recurso. Si encuentra el registro A con la dirección IP incorrecta, debe quitar la dirección IP equivocada y agregar una nueva. Se recomienda quitar todo el registro A y agregar uno nuevo, y realizar un vaciado de DNS en el dispositivo de origen local.   

Un ejemplo ilustrativo del registro DNS A de la cuenta de almacenamiento en la zona DNS privada:

![Registros DNS](./media/how-to-use-azure-migrate-with-private-endpoints/dns-a-records.png)   

Un ejemplo ilustrativo de registros DNS A de microservicios de almacén de Recovery Services en la zona DNS privada: 

![Registros DNS para el almacén de Recovery Services](./media/how-to-use-azure-migrate-with-private-endpoints/rsv-a-records.png)   

>[!Note]
> Cuando se quita o se modifica un registro A, es posible que la máquina siga resolviendo la dirección IP anterior porque el valor de TTL (período de vida) no ha expirado todavía.  

#### <a name="other-things-that-may-affect-private-link-connectivity"></a>Otras cosas que pueden afectar a la conectividad de vínculo privado  

Esta es una lista no exhaustiva de elementos que se pueden encontrar en escenarios avanzados o complejos: 

- Configuración del firewall, ya sea Azure Firewall conectado a Virtual Network o una solución de firewall personalizada que se implementa en el equipo.  
- Emparejamiento de redes, que puede afectar a los servidores DNS que se usan y a cómo se enruta el tráfico.  
- Las soluciones de puerta de enlace personalizada (NAT) pueden afectar a la forma en que se enruta el tráfico, incluido el tráfico de las consultas de DNS.

Para obtener más información, consulte la [guía de solución de problemas de conectividad de puntos de conexión privados](https://docs.microsoft.com/azure/private-link/troubleshoot-private-endpoint-connectivity).  

## <a name="next-steps"></a>Pasos siguientes 
- [Complete el proceso de migración](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#complete-the-migration) y revise los procedimientos recomendados [posteriores a la migración](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines#post-migration-best-practices).