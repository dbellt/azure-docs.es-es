---
title: Configuración de un entorno de ejecución de integración autohospedado como proxy para SSIS
description: Aprenda a configurar un entorno de ejecución de integración autohospedado como proxy para Azure-SSIS Integration Runtime.
ms.service: data-factory
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.custom: seo-lt-2019
ms.date: 04/25/2021
ms.openlocfilehash: 24138b1cbdf496f29a4975ebfd19acdc19cddd87
ms.sourcegitcommit: 12f15775e64e7a10a5daebcc52154370f3e6fa0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108001239"
---
# <a name="configure-a-self-hosted-ir-as-a-proxy-for-an-azure-ssis-ir-in-azure-data-factory"></a>Configuración de IR autohospedado como proxy para Azure-SSIS IR en Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este artículo se describe cómo ejecutar paquetes de SQL Server Integration Services (SSIS) en Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF) con un entorno de ejecución de integración autohospedado (IR autohospedado) configurado como proxy. 

Con esta característica, puede acceder a los datos y ejecutar tareas en el entorno local sin tener que [unir la instancia de Azure-SSIS IR a una red virtual](./join-azure-ssis-integration-runtime-virtual-network.md). La característica resulta útil cuando la configuración de la red corporativa es demasiado compleja o una directiva es demasiado restrictiva para que pueda insertar la instancia de Azure-SSIS IR en ella.

Esta característica solo se puede habilitar en las tareas Flujo de datos y Ejecutar SQL de SSIS por el momento. 

Cuando se habilita en la tarea Flujo de datos, esta característica la dividirá en dos tareas de almacenamiento provisional siempre que sea aplicable: 
* **Tarea de almacenamiento provisional en el entorno local**: esta tarea ejecuta el componente de flujo de datos que se conecta a un almacén de datos local en el IR autohospedado. Migra los datos del almacén de datos local a un área de almacenamiento provisional en su instancia de Azure Blob Storage o viceversa.
* **Tarea de almacenamiento provisional en la nube**: esta tarea ejecuta el componente de flujo de datos que no se conecta a un almacén de datos local en su instancia de Azure-SSIS IR. Migra los datos del área de almacenamiento provisional en su instancia de Azure Blob Storage a un almacén de datos en la nube o viceversa.

Si la tarea Flujo de datos migra datos del entorno local a la nube, la primera y la segunda tarea de almacenamiento provisional serán una tarea de almacenamiento provisional en el entorno local y una en la nube, respectivamente. Si la tarea Flujo de datos migra datos de la nube al entorno local, la primera y la segunda tarea de almacenamiento provisional serán una tarea de almacenamiento provisional en la nube y una en el entorno local, respectivamente. Si la tarea Flujo de datos migra datos de un entorno local a otro, la primera y la segunda tarea de almacenamiento provisional serán ambas en el entorno local. Si la tarea Flujo de datos migra datos de una instancia en la nube a otra, esta característica no aplica.

Si se habilita en la tarea Ejecutar SQL, esta característica la ejecutará en el entorno de ejecución de integración autohospedado. 

Otras ventajas y funcionalidades de esta característica permiten, por ejemplo, configurar IR autohospedado en regiones que Azure-SSIS IR todavía no admite y permitir la dirección IP estática pública de la instancia del entorno de ejecución de integración autohospedado en el firewall de los orígenes de datos.

## <a name="prepare-the-self-hosted-ir"></a>Preparación de IR autohospedado

Para usar esta característica, primero debe crear una factoría de datos y configurar una instancia de Azure-SSIS IR en ella. Si todavía no lo ha hecho, siga las instrucciones de [Configuración de un entorno de Azure-SSIS IR](./tutorial-deploy-ssis-packages-azure.md).

A continuación, configure la instancia de IR autohospedado en la misma factoría de datos en la que está configurada la instancia de Azure-SSIS IR. Para ello, consulte [Creación de un entorno de ejecución de integración autohospedado](./create-self-hosted-integration-runtime.md).

Por último, descargue e instale la versión más reciente de IR autohospedado, así como los controladores y el entorno de ejecución adicionales, en la máquina virtual de Azure o en el equipo local, tal como se indica a continuación:
- Descargue e instale la versión más reciente de [IR autohospedado](https://www.microsoft.com/download/details.aspx?id=39717).
- Si usa conectores de base de datos de vinculación e incrustación de objetos (OLEDB), de conectividad abierta de bases de datos (ODBC) o de ADO.NET en los paquetes, descargue e instale los controladores apropiados en el mismo equipo en el que está instalada la instancia de IR autohospedado, si aún no lo ha hecho.  

  Si usa la versión anterior del controlador OLEDB para SQL Server (SQL Server Native Client [SQLNCLI]), [descargue la versión de 64 bits](https://www.microsoft.com/download/details.aspx?id=50402).  

  Si usa la versión más reciente del controlador OLEDB para SQL Server (MSOLEDBSQL), [descargue la versión de 64 bits](https://www.microsoft.com/download/details.aspx?id=56730).  
  
  Si usa controladores OLEDB/ODBC/ADO.NET para otros sistemas de base de datos, como PostgreSQL, MySQL, Oracle, etc., puede descargar las versiones de 64 bits de sus sitios web.
- Si aún no lo ha hecho, [descargue e instale la versión de 64 bits del entorno de ejecución de Visual C++ (VC)](https://www.microsoft.com/download/details.aspx?id=40784) en la misma máquina en la que está instalada la instancia de IR autohospedado.

### <a name="enable-windows-authentication-for-on-premises-tasks"></a>Habilitación de la autenticación de Windows en tareas del entorno local

Si las tareas de almacenamiento provisional en el entorno local o las tareas Ejecutar SQL en el IR autohospedado requieren la autenticación de Windows, también debe [configurar la característica autenticación de Windows en la instancia de Azure-SSIS IR](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth.md). 

Las tareas de almacenamiento provisional en el entorno local o las tareas Ejecutar SQL se invocarán con la cuenta de servicio de IR autohospedado (*NT SERVICE\DIAHostService* de manera predeterminada) y se accederá a los almacenes de datos con la cuenta de autenticación de Windows. Ambas cuentas requieren que se les asignen determinadas directivas de seguridad. En el equipo de IR autohospedado, vaya a **Directiva de seguridad local** > **Directivas locales** > **Asignación de derechos de usuario** y haga lo siguiente:

1. Asigne las directivas *Ajustar las cuotas de la memoria para un proceso* y *Reemplazar un símbolo (token) de nivel de proceso* a la cuenta del servicio de IR autohospedado. Esto debe producirse automáticamente al instalar el IR autohospedado con la cuenta de servicio predeterminada. Si no es así, asigne esas directivas de forma manual. Si utiliza una cuenta de servicio diferente, asígnele las mismas directivas.

1. Asigne la directiva *Iniciar sesión como servicio* a la cuenta de autenticación de Windows.

## <a name="prepare-the-azure-blob-storage-linked-service-for-staging"></a>Preparación del servicio vinculado de Azure Blob Storage para almacenamiento provisional

Si aún no lo ha hecho, cree un servicio vinculado de Azure Blob Storage en la misma factoría de datos en la que está configurada la instancia de Azure-SSIS IR. Para ello, consulte [Creación de un servicio vinculado de Azure Data Factory](./quickstart-create-data-factory-portal.md#create-a-linked-service). Asegúrese de hacer lo siguiente:
- En **Almacén de datos**, seleccione **Azure Blob Storage**.  
- En **Conectar mediante el entorno de ejecución de integración**, seleccione **AutoResolveIntegrationRuntime** (no su IR autohospedado). De este modo, podremos ignorarlo y usar la instancia de Azure-SSIS IR en su lugar para capturar las credenciales de acceso de su instancia de Azure Blob Storage.
- En **Método de autenticación**, seleccione **Clave de cuenta**, **SAS URI** (URI de SAS), **Entidad de servicio** o **Identidad administrada**.  

>[!TIP]
>Si selecciona el método **Entidad de servicio**, conceda a la entidad de servicio al menos el rol *Colaborador de datos de Storage Blob*. Para más información, vea el [conector de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties). Si selecciona el método **Identidad administrada**, conceda el rol apropiado a la identidad administrada de ADF para acceder a Azure Blob Storage. Para más información, vea [Acceso a Azure Blob Storage mediante la autenticación de Azure Active Directory con la identidad administrada de ADF](/sql/integration-services/connection-manager/azure-storage-connection-manager.md#managed-identities-for-azure-resources-authentication).

![Preparación del servicio vinculado de Azure Blob Storage para almacenamiento provisional](media/self-hosted-integration-runtime-proxy-ssis/shir-azure-blob-storage-linked-service.png)

## <a name="configure-an-azure-ssis-ir-with-your-self-hosted-ir-as-a-proxy"></a>Configuración de una instancia de Azure-SSIS IR con IR autohospedado como proxy

Cuando haya preparado el servicio vinculado de Azure Blob Storage e IR autohospedado para el almacenamiento provisional, puede configurar la instancia nueva o existente de Azure-SSIS IR con el IR autohospedado como proxy en el portal o la aplicación de factoría de datos. Sin embargo, antes de hacerlo, si la instancia de Azure-SSIS IR existente ya está en ejecución, puede detenerla, editarla y reiniciarla después.

1. En el panel **Integration runtime setup** (Configuración del entorno de ejecución de integración), omita las secciones **Configuración general** y **Configuración de SQL** seleccionando **Siguiente**. 

1. En la sección **Configuración avanzada**, haga lo siguiente:

   1. Active la casilla **Set up Self-Hosted Integration Runtime as a proxy for your Azure-SSIS Integration Runtime** (Configurar el entorno de ejecución de integración autohospedado como proxy para su instancia de Azure-SSIS Integration Runtime). 

   1. En la lista desplegable **Self-Hosted Integration Runtime** (Integration Runtime autohospedado), seleccione la instancia existente de IR autohospedado como proxy para Azure-SSIS IR.

   1. En la lista desplegable **Staging Storage Linked Service** (Servicio vinculado de almacenamiento provisional), seleccione el servicio vinculado de Azure Blob Storage existente o cree uno para el almacenamiento provisional.

   1. En **Staging path** (Ruta de acceso provisional), especifique un contenedor de blobs de la cuenta seleccionada de Azure Storage o deje este campo vacío para usar uno predeterminado como almacenamiento provisional.

   1. Seleccione **Continuar**.

   ![Configuración avanzada con IR autohospedado](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-shir.png)

También puede configurar la instancia nueva o existente de Azure-SSIS IR con IR autohospedado como proxy mediante PowerShell.

```powershell
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
$AzureSSISName = "[your Azure-SSIS IR name]"
# Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data accesss
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force
```

## <a name="enable-ssis-packages-to-use-a-proxy"></a>Habilitación de paquetes SSIS para usar un proxy

Mediante el uso de SSDT más reciente, ya sea como extensión de proyectos SSIS para Visual Studio o un instalador independiente, puede encontrar la nueva propiedad `ConnectByProxy` en los administradores de conexiones para los componentes de flujo de datos compatibles y la propiedad `ExecuteOnProxy` en la tarea Ejecutar SQL.
* [Descargar la extensión de proyectos SSIS para Visual Studio](https://marketplace.visualstudio.com/items?itemName=SSIS.SqlServerIntegrationServicesProjects)
* [Descargar el instalador independiente](/sql/ssdt/download-sql-server-data-tools-ssdt#ssdt-for-vs-2017-standalone-installer)   

Al diseñar paquetes nuevos que contienen tareas Flujo de datos con componentes para acceder a datos en el entorno local, puede habilitar la propiedad `ConnectByProxy` si la establece en *True* en el panel **Propiedades** de los administradores de conexiones correspondientes.  

![Habilitación de la propiedad ConnectByProxy](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-manager-properties.png)

Al diseñar paquetes nuevos que contienen tareas Ejecutar SQL que se ejecutan en el entorno local, puede habilitar la propiedad `ExecuteOnProxy` si la establece en *True* en el panel **Propiedades** de las tareas correspondientes.

También puede habilitar las propiedades `ConnectByProxy`/`ExecuteOnProxy` al ejecutar paquetes existentes sin tener que cambiarlos manualmente uno a uno.  Hay dos opciones:
- **Opción A**: Abrir, recompilar y volver a implementar el proyecto que contiene los paquetes con la extensión SSDT más reciente para ejecutarla en Azure-SSIS IR. A continuación, puede habilitar la propiedad `ConnectByProxy` si la establece en *True* para los administradores de conexiones correspondientes que aparecen en la pestaña **Administradores de conexiones** de la ventana emergente **Ejecutar paquete** al ejecutar paquetes de SSMS.

  ![Habilitación de la propiedad ConnectByProxy 2](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-managers-tab-ssms.png)

  También puede habilitar la propiedad `ConnectByProxy` si la establece en *True* para los administradores de conexiones correspondientes que aparecen en la pestaña **Administradores de conexiones** de la actividad [Ejecutar paquete de SSIS](./how-to-invoke-ssis-package-ssis-activity.md) al ejecutar paquetes en las canalizaciones de Data Factory.
  
  ![Habilitación de la propiedad ConnectByProxy 3](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-managers-tab-ssis-activity.png)

- **Opción B:** Volver a implementar el proyecto que contiene los paquetes que se van a ejecutar en el entorno de ejecución de integración de SSIS. A continuación, puede habilitar las propiedades `ConnectByProxy`/`ExecuteOnProxy` si especifica sus rutas de acceso de propiedad `\Package.Connections[YourConnectionManagerName].Properties[ConnectByProxy]`/`\Package\YourExecuteSQLTaskName.Properties[ExecuteOnProxy]` y las establece en *True* como una invalidación de propiedad en la pestaña **Avanzadas** de la ventana emergente **Ejecutar paquete** cuando ejecute paquetes de SSMS.

  ![Habilitación de la propiedad ConnectByProxy 4](media/self-hosted-integration-runtime-proxy-ssis/shir-advanced-tab-ssms.png)

  También puede habilitar las propiedades `ConnectByProxy`/`ExecuteOnProxy` si especifica sus rutas de acceso de propiedad`\Package.Connections[YourConnectionManagerName].Properties[ConnectByProxy]`/`\Package\YourExecuteSQLTaskName.Properties[ExecuteOnProxy]` y las establece en *True* como una invalidación de propiedad en la pestaña **Invalidaciones de propiedad** de la [actividad Ejecutar paquete de SSIS](./how-to-invoke-ssis-package-ssis-activity.md) cuando ejecute paquetes en las canalizaciones de Data Factory.
  
  ![Habilitación de la propiedad ConnectByProxy 5](media/self-hosted-integration-runtime-proxy-ssis/shir-property-overrides-tab-ssis-activity.png)

## <a name="debug-the-on-premises-tasks-and-cloud-staging-tasks"></a>Depuración de tareas en el entorno local y tareas de almacenamiento provisional en la nube

En la instancia de IR autohospedado, puede encontrar los registros del entorno de ejecución en la carpeta *C:\ProgramData\SSISTelemetry* y los registros de ejecución de las tareas de almacenamiento provisional en el entorno local o de las tareas Ejecutar SQL en la carpeta *C:\ProgramData\SSISTelemetry\ExecutionLog*.  Puede encontrar los registros de ejecución de las tareas de almacenamiento provisional en la nube en la instancia de SSISDB, en las rutas de acceso de registro especificadas o en Azure Monitor, en función de si los paquetes se almacenan en SSISDB, si se habilita la [integración de Azure Monitor](./monitor-using-azure-monitor.md#monitor-ssis-operations-with-azure-monitor), etc. También puede encontrar los identificadores únicos de las tareas de almacenamiento provisional en el entorno local en los registros de ejecución de las tareas de almacenamiento provisional en la nube. 

![Identificador único de la primera tarea de almacenamiento provisional](media/self-hosted-integration-runtime-proxy-ssis/shir-first-staging-task-guid.png)

Si ha generado incidencias de soporte técnico, puede seleccionar el botón **Enviar registros** en la pestaña **Diagnósticos** de **Microsoft Integration Runtime Configuration Manager** que está instalado en el entorno de ejecución de integración autohospedado para enviar los registros de operaciones y ejecuciones recientes para que se investiguen.

## <a name="billing-for-the-on-premises-tasks-and-cloud-staging-tasks"></a>Facturación de las tareas en el entorno local y tareas de almacenamiento provisional en la nube

Las tareas de almacenamiento provisional en el entorno local o las tareas Ejecutar SQL que se ejecutan en la instancia de IR autohospedado se facturan por separado, de la misma forma que se facturan las actividades de movimiento de datos que se ejecutan en una instancia de IR autohospedado. Esto se especifica en el artículo [Azure Data Factory: Precios de las canalizaciones de datos](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/).

Las tareas de almacenamiento provisional en la nube que se ejecutan en la instancia de Azure-SSIS IR no se facturan por separado, pero la instancia de Azure-SSIS IR en ejecución se factura tal como se especifica en el artículo sobre [Precios de Azure-SSIS IR](https://azure.microsoft.com/pricing/details/data-factory/ssis/).

## <a name="enable-custom3rd-party-data-flow-components"></a>Habilitación de componentes de flujo de datos personalizados o de terceros 

Para habilitar componentes de flujo de datos personalizados o de terceros para acceder a los datos locales mediante IR autohospedado como un proxy para Azure-SSIS IR, siga estas instrucciones:

1. Instale los componentes de flujo de datos personalizados o de terceros que tengan como destino SQL Server 2017 en Azure-SSIS IR mediante las [instalaciones personalizadas estándar/rápidas](./how-to-configure-azure-ssis-ir-custom-setup.md).

1. Cree las siguientes claves del Registro DTSPath en IR autohospedado si aún no existen:
   1. `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\140\SSIS\Setup\DTSPath` se establece en `C:\Program Files\Microsoft SQL Server\140\DTS\`
   1. `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Microsoft SQL Server\140\SSIS\Setup\DTSPath` se establece en `C:\Program Files (x86)\Microsoft SQL Server\140\DTS\`
   
1. Instale los componentes de flujo de datos personalizados o de terceros que tengan como destino SQL Server 2017 en IR autohospedado en el registro DTSPath anterior y asegúrese de que el proceso de instalación realice lo siguiente:

   1. Crear las carpetas `<DTSPath>`, `<DTSPath>/Connections`, `<DTSPath>/PipelineComponents` y `<DTSPath>/UpgradeMappings` si aún no existen.
   
   1. Crear su propio archivo XML para las asignaciones de extensiones en la carpeta `<DTSPath>/UpgradeMappings`.
   
   1. Instalar todos los ensamblados a los que hacen referencia los ensamblados de componentes de flujo de datos personalizados o de terceros en la caché global de ensamblados (GAC).

Estos son algunos ejemplos de nuestros asociados, [Theobald Software](https://kb.theobald-software.com/xtract-is/XIS-for-Azure-SHIR) y [Aecorsoft](https://www.aecorsoft.com/blog/2020/11/8/using-azure-data-factory-to-bring-sap-data-to-azure-via-self-hosted-ir-and-ssis-ir), que han adaptado sus componentes de flujo de datos para usar nuestra instalación rápida personalizada e IR autohospedado como proxy para Azure-SSIS IR.

## <a name="enforce-tls-12"></a>Aplicación de TLS 1.2

Si necesita usar un protocolo de red de alta seguridad y cifrado seguro (TLS 1.2), así como deshabilitar las versiones anteriores de SSL/TLS al mismo tiempo en el entorno de ejecución de integración autohospedado, puede descargar y ejecutar el script *main.cmd* que se encuentra en la carpeta *CustomSetupScript/UserScenarios/TLS 1.2* del contenedor de blobs en versión preliminar pública.  Con el [Explorador de Azure Storage](https://storageexplorer.com/), puede conectarse al contenedor de blobs en versión preliminar pública; para ello, escriba el siguiente URI de SAS:

`https://ssisazurefileshare.blob.core.windows.net/publicpreview?sp=rl&st=2020-03-25T04:00:00Z&se=2025-03-25T04:00:00Z&sv=2019-02-02&sr=c&sig=WAD3DATezJjhBCO3ezrQ7TUZ8syEUxZZtGIhhP6Pt4I%3D`

## <a name="current-limitations"></a>Limitaciones actuales

- Actualmente, solo son compatibles los componentes de flujo de datos que están integrados o preinstalados en Azure-SSIS IR Standard Edition, excepto los componentes de Hadoop/HDFS/DQS; vea [todos los componentes integrados y preinstalados en Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md).
- Actualmente, solo son compatibles los componentes de flujo de datos personalizados o de terceros que se escriben en código administrado (.NET Framework). Sin embargo, no se admiten actualmente los que se escriben en código nativo (C++).
- Actualmente, no se admite el cambio de valores de variables en las tareas locales y de almacenamiento provisional en la nube.
- El cambio de valores de variables del objeto type en las tareas de almacenamiento provisional locales no se reflejará en otras tareas.
- Actualmente no se admite *ParameterMapping* en el origen de OLEDB. Como solución alternativa, use *Comando SQL de variable* como *AccessMode* y *Expresión* para insertar las variables o los parámetros en un comando SQL. Como ejemplo, vea el paquete *ParameterMappingSample.dtsx* que se puede encontrar en la carpeta *SelfHostedIRProxy/Limitations* del contenedor de blobs en versión preliminar pública. Con el Explorador de Azure Storage, puede conectarse al contenedor de blobs en versión preliminar pública si especifica el URI de SAS anterior.

## <a name="next-steps"></a>Pasos siguientes

Después de configurar la instancia de IR autohospedado como proxy para Azure-SSIS IR, puede implementar y ejecutar los paquetes para acceder a los datos locales como actividades de ejecución de paquetes SSIS en las canalizaciones de Data Factory. Para obtener información de cómo hacerlo, consulte [Ejecución de un paquete de SSIS mediante la actividad Ejecutar paquete SSIS de Data Factory](./how-to-invoke-ssis-package-ssis-activity.md).
