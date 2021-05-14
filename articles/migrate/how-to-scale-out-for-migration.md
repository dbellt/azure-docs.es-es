---
title: Configuración de un dispositivo de escalabilidad horizontal de Azure Migrate para la migración sin agente de VMware
description: Obtenga información sobre cómo configurar un dispositivo de escalabilidad horizontal de Azure Migrate para migrar VM de Hyper-V.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: how-to
ms.date: 03/02/2021
ms.openlocfilehash: b0f5bf01080d89e6dc6d6843312d96243b8526ba
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109484548"
---
# <a name="scale-agentless-migration-of-vmware-virtual-machines-to-azure"></a>Escalado de la migración sin agentes de máquinas virtuales de VMware a Azure

Este artículo le ayuda a entender cómo usar un dispositivo de escalabilidad horizontal para migrar un gran número de máquinas virtuales (VM) de VMware a Azure mediante el método sin agente de la herramienta de migración de servidores de Azure Migrate para la migración de VM de VMware.

Con el método de migración sin agente para máquinas virtuales de VMware, puede:

- Replicar hasta 300 VM desde un solo vCenter Server simultáneamente con un dispositivo de Azure Migrate.
- Replicar hasta 500 VM desde un único vCenter Server simultáneamente al implementar un segundo dispositivo de escalabilidad horizontal para la migración.

En este artículo, aprenderá a:

- Agregar un dispositivo de escalabilidad horizontal para la migración sin agente de máquinas virtuales de VMware.
- Migrar hasta 500 VM simultáneamente con el dispositivo de escalabilidad horizontal.

##  <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, debe llevar a cabo los siguientes pasos:

- Cree el proyecto de Azure Migrate.
- Implemente el dispositivo de Azure Migrate (dispositivo principal) y la detección completa de máquinas virtuales de VMware administradas por vCenter Server.
- Configure la replicación de una o varias máquinas virtuales que se migrarán.
> [!IMPORTANT]
> Deberá tener al menos una máquina virtual de replicación en el proyecto para que pueda agregar un dispositivo de escalabilidad horizontal para la migración.

Para obtener información sobre cómo realizar lo anterior, revise el tutorial sobre la [migración de máquinas virtuales de VMware a Azure con el método de migración sin agente](./tutorial-migrate-vmware.md).

## <a name="deploy-a-scale-out-appliance"></a>Implementación de un dispositivo de escalabilidad horizontal

Para agregar un dispositivo de escalabilidad horizontal, siga los pasos que se mencionan a continuación:

1. Haga clic en **Detectar** >  **¿Las máquinas están virtualizadas?** 
1. Seleccione **Sí, con VMware vSphere Hypervisor**.
1. Seleccione la replicación sin agente en el paso siguiente.
1. Seleccione **Scale-out an existing primary appliance** (Escalar horizontalmente un dispositivo principal existente) en el menú para seleccionar el tipo de dispositivo.
1. Seleccione el dispositivo principal (el dispositivo con el que se realizó la detección) que quiere escalar horizontalmente.

:::image type="content" source="./media/how-to-scale-out-for-migration/add-scale-out.png" alt-text="Página Detectar máquinas para la incorporación de escalabilidad horizontal":::

### <a name="1-generate-the-azure-migrate-project-key"></a>1. Generación de la clave del proyecto de Azure Migrate

1. En **Generación de la clave del proyecto de Azure Migrate**, especifique un nombre de sufijo para el dispositivo de escalabilidad horizontal. El sufijo solo puede contener caracteres alfanuméricos y tiene un límite de longitud de 14 caracteres.
2. Haga clic en **Generar clave** para iniciar la creación de los recursos de Azure necesarios. No cierre la página Detectar durante la creación de recursos.
3. Copie la clave generada. Necesitará la clave más tarde para completar el registro del dispositivo de escalabilidad horizontal.

### <a name="2-download-the-installer-for-the-scale-out-appliance"></a>2. Descarga del instalador para el dispositivo de escalabilidad horizontal

En **Descargar el dispositivo de Azure Migrate**, haga clic en **Descargar**. Debe descargar el script del instalador de PowerShell para implementar el dispositivo de escalabilidad horizontal en un servidor existente que ejecute Windows Server 2016 y tenga la configuración de hardware necesaria (32 GB de RAM, 8 vCPU, alrededor de 80 GB de almacenamiento en disco y acceso a Internet, ya sea directamente o a través de un proxy).
:::image type="content" source="./media/how-to-scale-out-for-migration/download-scale-out.png" alt-text="Descarga del script para el dispositivo de escalabilidad horizontal":::

> [!TIP]
> Puede validar la suma de comprobación del archivo ZIP descargado con los siguientes pasos:
>
> 1. Abra un símbolo del sistema como administrador.
> 2. Ejecute el siguiente comando para generar el código hash para el archivo ZIP:
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Ejemplo de uso para la nube pública: ```C:\>Get-FileHash -Path .\AzureMigrateInstaller-VMware-Public-Scaleout.zip -Algorithm SHA256 ```
> 3. Descargue la versión más reciente del instalador del dispositivo de escalabilidad horizontal desde el portal si el código hash calculado no coincide con esta cadena: 1E6B6E3EE8B2A800818B925F5DA67EF7874DAD87E32847120B32F3E21F5960F9.

### <a name="3-run-the-azure-migrate-installer-script"></a>3. Ejecución del script del instalador de Azure Migrate
El script del instalador hace lo siguiente:

- Instale el agente de puerta de enlace y el administrador de configuración de dispositivos para realizar más replicaciones de servidor simultáneas.
- Instala los roles de Windows, incluido el servicio de activación de Windows, IIS y PowerShell ISE.
- Descarga e instala un módulo de reescritura de IIS.
- Actualiza una clave del registro (HKLM) con detalles de configuración persistentes para Azure Migrate.
- Crea los siguientes archivos en la ruta de acceso:
    - **Archivos de configuración**:%Programdata%\Microsoft Azure\Config
    - **Archivos de registro**:%Programdata%\Microsoft Azure\Logs

Ejecute el script como se indica a continuación:

1. Extraiga el archivo ZIP en una carpeta del servidor que hospedará el dispositivo de escalabilidad horizontal.  No ejecute el script en un servidor con un dispositivo de Azure Migrate existente.
2. Inicie PowerShell en el servidor anterior con privilegios administrativos (elevados).
3. Cambie el directorio de PowerShell a la carpeta en la que se ha extraído el contenido del archivo ZIP descargado.
4. Ejecute el script denominado **AzureMigrateInstaller.ps1** con el comando siguiente:

    - Para la nube pública: 
    
        ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 ```

    El script iniciará el administrador de configuración del dispositivo cuando finalice la ejecución.

Si surge algún problema, puede acceder los registros de script en la ubicación: <br/> C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>marca de tiempo</em>.log para solucionar problemas.


### <a name="4-configure-the-appliance"></a>4. Configuración del dispositivo

Antes de empezar, asegúrese de que se pueda acceder a [estos puntos de conexión de Azure](migrate-appliance.md#public-cloud-urls) desde el dispositivo de escalabilidad horizontal.

- Abra un explorador en cualquier máquina que pueda conectarse al servidor del dispositivo de escalabilidad horizontal y abra la dirección URL del administrador de configuración del dispositivo: **https://*nombre o dirección IP del dispositivo de escalabilidad horizontal*: 44368**.

   También puede abrir el administrador de configuración desde el escritorio del servidor del dispositivo de escalabilidad horizontal a través del acceso directo al administrador de configuración.
- Acepte los **términos de licencia** y lea la información de terceros.
- En administrador de configuración > **Configurar los requisitos previos**, realice las siguientes operaciones:
   - **Conectividad**: el dispositivo comprueba que el servidor tenga acceso a Internet. Si el servidor usa un proxy:
     1. Haga clic en **Configurar el proxy** para especificar la dirección del proxy con los formatos http://ProxyIPAddress o http://ProxyFQDN) y el puerto de escucha.
     2. Especifique las credenciales si el proxy requiere autenticación.
     3. Solo se admite un proxy HTTP.
     4. Si ha agregado detalles del proxy o ha deshabilitado el proxy o la autenticación, haga clic en **Guardar** para desencadenar la comprobación de conectividad.
   - **Time sync** (Sincronización de hora): Para que la detección funcione correctamente, la hora del dispositivo debe estar sincronizada con la hora de Internet.
   - **Instalación de actualizaciones**: el dispositivo garantiza que se instalan las actualizaciones más recientes. Una vez finalizada la comprobación, puede hacer clic en **Ver servicios del dispositivo** para ver el estado y las versiones de los servicios que se ejecutan en el servidor del dispositivo.
   - **Install VDDK** (Instalar VDDK): El dispositivo comprueba si VMware vSphere Virtual Disk Development Kit (VDDK) está instalado. Si no está instalado, descargue VDDK 6.7 de VMware y extraiga el contenido del archivo ZIP descargado en la ubicación especificada del dispositivo, tal como se indica en las **instrucciones de instalación** de la pantalla del Administrador de configuración del dispositivo.


### <a name="register-the-appliance-with-azure-migrate"></a>Registro del dispositivo en Azure Migrate

1. Pegue la **clave del proyecto de Azure Migrate** copiada desde el portal. Si no tiene la clave, vaya a **Server Assessment > Detectar > Administrar los dispositivos existentes**, seleccione el nombre del dispositivo principal, busque el dispositivo de escalabilidad horizontal asociado a él y copie la clave correspondiente.
1. Necesitará un código de dispositivo para autenticarse con Azure. Al hacer clic en **Iniciar sesión** se abrirá un modal con el código del dispositivo, tal como se muestra a continuación.
:::image type="content" source="./media/tutorial-discover-vmware/device-code.png" alt-text="Modal que muestra el código del dispositivo":::

1. Haga clic en **Copiar código e Iniciar sesión** para copiar el código del dispositivo y abrir un símbolo del sistema de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.
1. En la nueva pestaña, pegue el código del dispositivo e inicie sesión con su nombre de usuario y contraseña de Azure.
   
   No se admite el inicio de sesión con un PIN.
3. En caso de que cierre accidentalmente la pestaña de inicio de sesión sin iniciar sesión, deberá actualizar la pestaña explorador del administrador de configuración del dispositivo para volver a habilitar el botón Iniciar sesión.
1. Una vez que haya iniciado sesión correctamente, vuelva a la pestaña anterior con el administrador de configuración del dispositivo.
1. Si la cuenta de usuario de Azure que se usa para el registro tiene los permisos adecuados en los recursos de Azure creados durante la generación de la clave, se iniciará el registro del dispositivo.
:::image type="content" source="./media/how-to-scale-out-for-migration/registration-scale-out.png" alt-text="Panel 2 del Administrador de configuración del dispositivo":::

#### <a name="import-appliance-configuration-from-primary-appliance"></a>Importación de la configuración del dispositivo desde el dispositivo principal

Para completar el registro del dispositivo de escalabilidad horizontal, haga clic en **importar** para obtener los archivos de configuración necesarios del dispositivo principal.

1. Al hacer clic en **Importar**, se abre una ventana emergente con instrucciones sobre cómo importar los archivos de configuración necesarios desde el dispositivo principal.
:::image type="content" source="./media/how-to-scale-out-for-migration/import-modal-scale-out.png" alt-text="Cuadro modal Importar configuración":::
1. Inicie sesión (escritorio remoto) en el dispositivo principal y ejecute los siguientes comandos de PowerShell:

    ``` PS cd 'C:\Program Files\Microsoft Azure Appliance Configuration Manager\Scripts\PowerShell' ```
    
    ``` PS .\ExportConfigFiles.ps1 ```

1. Copie el archivo ZIP creado mediante la ejecución de los comandos anteriores en el dispositivo de escalabilidad horizontal. El archivo ZIP contiene los archivos de configuración necesarios para registrar el dispositivo de escalabilidad horizontal.
1. En la ventana emergente que se abre en el paso anterior, seleccione la ubicación del archivo ZIP de configuración copiado y haga clic en **Guardar**.

Una vez que los archivos se hayan importado correctamente, se completará el registro del dispositivo de escalabilidad horizontal y se mostrará la marca de tiempo de la última importación correcta. También puede ver la información de registro si hace clic en **Ver detalles**.
:::image type="content" source="./media/how-to-scale-out-for-migration/import-success.png" alt-text="Captura de pantalla en la que se muestra el registro del dispositivo de escalabilidad horizontal con un proyecto de Azure Migrate.":::

En este momento, debe volver a validar que el dispositivo de escalabilidad horizontal pueda conectarse con vCenter Server. Haga clic en **Volver a validar** para validar la conectividad de vCenter Server desde el dispositivo de escalabilidad horizontal.
:::image type="content" source="./media/how-to-scale-out-for-migration/view-sources.png" alt-text="Captura de pantalla en la que se muestra la visualización de las credenciales y los orígenes de detección que se van a validar.":::

> [!IMPORTANT]
> Si modifica las credenciales de vCenter Server en el dispositivo principal, asegúrese de volver a importar los archivos de configuración en el dispositivo de escalabilidad horizontal para obtener la configuración más reciente y continuar con las replicaciones en curso.<br/> Si ya no necesita el dispositivo de escalabilidad horizontal, asegúrese de deshabilitarlo. [**Obtenga más información**](./common-questions-appliance.md) sobre cómo deshabilitar el dispositivo de escalado horizontal cuando no sea necesario.

## <a name="replicate"></a>Replicar

1. Una vez registrado el dispositivo de escalabilidad horizontal, en el icono Azure Migrate: Server Migration, haga clic en **Replicar**.

2.  Siga los pasos de la pantalla para empezar a replicar más máquinas virtuales. 

Con el dispositivo de escalabilidad horizontal, ahora puede replicar 500 VM simultáneamente. También puede migrar VM en lotes de 200 a través de Azure Portal.

La herramienta Azure Migrate: Server Migration se encargará de distribuir las máquinas virtuales entre el dispositivo principal y la de escalabilidad horizontal para la replicación. Una vez finalizada la replicación, puede migrar las máquinas virtuales.

> [!TIP]
> Se recomienda migrar las máquinas virtuales en lotes de 200 para lograr un rendimiento óptimo si quiere migrar un gran número de máquinas virtuales.
  
## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido lo siguiente:
- Cómo configurar un dispositivo de escalabilidad horizontal
- Cómo replicar VM mediante un dispositivo de escalabilidad horizontal


[Obtenga más información](./tutorial-migrate-vmware.md) sobre cómo migrar servidores a Azure mediante la herramienta Azure Migrate: Server Migration.
