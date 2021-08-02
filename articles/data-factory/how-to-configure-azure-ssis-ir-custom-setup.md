---
title: Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime
description: En este artículo se describe cómo usar la interfaz de instalación personalizada en una instancia de Azure-SSIS Integration Runtime para instalar componentes adicionales o cambiar los valores de configuración.
ms.service: data-factory
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.custom: seo-lt-2019
ms.date: 04/30/2021
ms.openlocfilehash: ddd837387de955e04cdf39ec73104dfda98e3cf4
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110470925"
---
# <a name="customize-the-setup-for-an-azure-ssis-integration-runtime"></a>Personalización de la instalación en una instancia de Azure-SSIS Integration Runtime

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Puede personalizar su instancia de Azure-SQL Server Integration Services (SSIS) Integration Runtime (IR) en Azure Data Factory (ADF) mediante instalaciones personalizadas. Estas le permiten agregar sus propios pasos durante el aprovisionamiento o la reconfiguración de la instancia de Azure-SSIS IR. 

Mediante las instalaciones personalizadas, puede modificar la configuración de funcionamiento predeterminada o el entorno de la instancia de Azure-SSIS IR. Por ejemplo, iniciar servicios adicionales de Windows, conservar las credenciales de acceso de los recursos compartidos de archivos o usar únicamente un protocolo de red de criptografía fuerte o más seguro (TLS 1.2). También, puede instalar componentes adicionales, como ensamblados, controladores o extensiones, en cada nodo de Azure-SSIS IR. Pueden ser componentes personalizados, de código abierto o de terceros. Para obtener más información sobre los componentes integrados o preinstalados, vea el artículo [Componentes integrados y preinstalados en Azure-SSIS Integration Runtime](./built-in-preinstalled-components-ssis-integration-runtime.md).

Puede realizar instalaciones personalizadas en la instancia de Azure-SSIS IR de dos maneras: 
* **Instalación personalizada estándar con un script**: prepare un script y sus archivos asociados y cárguelos todos juntos en un contenedor de blobs de la cuenta de Azure Storage. Luego proporcione un identificador uniforme de recursos (URI) de Firma de acceso compartido (SAS) para el contenedor de blobs cuando configure o vuelva a configurar la instancia de Azure-SSIS IR. Entonces, cada nodo de la instancia de Azure-SSIS IR descarga el script y sus archivos asociados del contenedor de blobs y ejecuta la instalación personalizada con privilegios elevados. Una vez finalizada la instalación personalizada, cada nodo carga la salida estándar de la ejecución y otros registros en el contenedor de blobs.
* **Instalación personalizada rápida sin un script**: ejecute algunas configuraciones del sistema y comandos de Windows comunes o instale algunos componentes conocidos o recomendados sin usar ningún script.

Puede instalar tanto componentes gratuitos (sin licencia) como componentes de pago (con licencia) con instalaciones personalizadas rápidas y estándar. Si es fabricante de software independiente (ISV), consulte [Desarrollo de componentes con licencia o de pago para Azure-SSIS IR](how-to-develop-azure-ssis-ir-licensed-components.md).

> [!IMPORTANT]
> Para beneficiarse de las mejoras futuras, se recomienda usar la versión v3 o posterior de la serie de nodos para Azure-SSIS IR con la configuración personalizada.

## <a name="current-limitations"></a>Limitaciones actuales

Las siguientes limitaciones solo se aplican a instalaciones personalizadas estándar:

- Si quiere usar *gacutil.exe* en el script para instalar ensamblados en la caché global de ensamblados (GAC), deberá proporcionar *gacutil.exe* como parte de su instalación personalizada. O puede usar la copia proporcionada en la carpeta *Sample* del contenedor de blobs Public Preview; vea la sección **Ejemplos de instalación personalizada estándar** a continuación.

- Si quiere hacer referencia a una subcarpeta del script, *msiexec.exe* no admite la notación `.\` para hacer referencia a la carpeta raíz. Use un comando como `msiexec /i "MySubfolder\MyInstallerx64.msi" ...` en lugar de `msiexec /i ".\MySubfolder\MyInstallerx64.msi" ...`.

- Los recursos compartidos administrativos o los recursos compartidos de red ocultos creados automáticamente por Windows no se admiten actualmente en Azure-SSIS IR.

- El controlador ODBC de IBM iSeries Access no se admite en Azure-SSIS IR. Puede que vea errores de instalación durante la instalación personalizada. Si este es el caso, póngase en contacto con el servicio de soporte técnico de IBM para obtener ayuda.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para personalizar Azure-SSIS IR, necesita los siguientes elementos:

- [Una suscripción de Azure](https://azure.microsoft.com/)

- [Aprovisionamiento del entorno de ejecución de integración de Azure-SSIS](./tutorial-deploy-ssis-packages-azure.md)

- [Una cuenta de Azure Storage](https://azure.microsoft.com/services/storage/). No se necesita para las instalaciones rápidas personalizadas. Para instalaciones estándar personalizadas, cargue y almacene el script de la instalación personalizada y sus archivos asociados en un contenedor de blobs. El proceso de instalación personalizada también carga sus registros de ejecución en el mismo contenedor de blobs.

## <a name="instructions"></a>Instructions

Puede aprovisionar o volver a configurar la instancia de Azure-SSIS IR con instalaciones personalizadas en la interfaz de usuario de ADF. Si quiere hacer lo mismo con PowerShell, descargue e instale [Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="standard-custom-setup"></a>Instalación personalizada estándar

Para aprovisionar o volver a configurar la instancia de Azure-SSIS IR con instalaciones personalizadas en la interfaz de usuario de ADF, haga lo siguiente.

1. Prepare el script de instalación personalizada y sus archivos asociados (por ejemplo, archivos .bat, .cmd, .exe, .dll, .msi o. ps1).

   * Debe tener un archivo de script denominado *main.cmd*, que es el punto de entrada de la instalación personalizada.  
   * Para asegurarse de que el script se puede ejecutar en modo silencioso, pruébelo primero en la máquina local.  
   * Si quiere que se carguen en el contenedor de blobs registros adicionales generados mediante otras herramientas (por ejemplo, *msiexec.exe*), especifique la variable de entorno predefinida, `CUSTOM_SETUP_SCRIPT_LOG_DIR`, como la carpeta de registros de los scripts (por ejemplo, *msiexec /i xxx.msi /quiet /lv %CUSTOM_SETUP_SCRIPT_LOG_DIR%\install.log*).

1. Descargue, instale y abra el [Explorador de Azure Storage](https://storageexplorer.com/).

   a. En **Local y asociado**, haga clic con el botón derecho en **Cuentas de almacenamiento** y seleccione **Conectar a Azure Storage**.

      ![Conectar a Azure Storage](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image1.png)

   b. Seleccione **Cuenta de almacenamiento o servicio**, **Nombre y clave de la cuenta** y **Siguiente**.

   c. Escriba el nombre de la cuenta de Azure Storage y la clave, seleccione **Siguiente** y, a continuación, seleccione **Conectar**.

      ![Proporcionar el nombre y la clave de la cuenta de almacenamiento](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image3.png)

   d. En la cuenta conectada de Azure Storage, haga clic con el botón derecho en **Contenedores de blobs**, seleccione **Crear contenedor de blobs** y ponga un nombre al nuevo contenedor de blobs.

      ![Creación de un contenedor de blobs](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image4.png)

   e. Seleccione el nuevo contenedor de blobs y cargue el script de la instalación personalizada y sus archivos asociados. Asegúrese de cargar *main.cmd* en el nivel superior del contenedor de blobs, no en una carpeta. El contenedor de blobs debe incluir solo los archivos necesarios de la instalación personalizada para que la descarga posterior en Azure-SSIS IR no tarde demasiado. La duración máxima de una instalación personalizada está establecida actualmente en 45 minutos antes de que se agote el tiempo de espera. Esto incluye el tiempo para descargar todos los archivos del contenedor de blobs e instalarlos en Azure-SSIS IR. Si la instalación requiere más tiempo, genere una incidencia de soporte técnico.

      ![Carga de archivos en el contenedor de blobs](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image5.png)

   f. Haga clic con el botón derecho en el contenedor de blobs y seleccione **Obtener firma de acceso compartido**.

      ![Obtención de la firma de acceso compartido del contenedor de blobs](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image6.png)

   g. Cree el URI de SAS del contenedor de blobs con un tiempo de expiración suficientemente largo y con permisos de lectura, escritura y enumeración. Necesita el URI de SAS para descargar y ejecutar el script de instalación personalizada y sus archivos asociados. Esto es así siempre que se reinicia o se restablece la imagen inicial de cualquier nodo de Azure-SSIS IR. También se necesita permiso de escritura para cargar los registros de ejecución del programa de instalación.

      > [!IMPORTANT]
      > Asegúrese de que el URI de SAS no expira y que los recursos de instalación personalizada están siempre disponibles durante todo el ciclo de vida de Azure-SSIS IR, desde la creación a la eliminación, especialmente si lo detiene e inicia periódicamente durante este tiempo.

      ![Generación de la firma de acceso compartido del contenedor de blobs](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image7.png)

   h. Copie y guarde el URI de SAS del contenedor de blobs.

      ![Copia y guardado de la firma de acceso compartido](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image8.png)

1. Active la casilla **Customize your Azure-SSIS Integration Runtime with additional system configurations/component installations** (Personalizar su Azure-SSIS IR con configuraciones del sistema e instalaciones de componentes adicionales) en la página **Configuración avanzada** del panel **Integration runtime setup** (Configuración de Integration Runtime). A continuación, escriba el URI de SAS del contenedor de blobs en el cuadro de texto **Custom setup container SAS URI** (URI de SAS del contenedor de instalación personalizada).

   ![Configuración avanzada con instalaciones personalizadas](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-custom.png)

Después de que finalice la instalación personalizada estándar y se inicie Azure-SSIS IR, puede encontrar todos los registros de la instalación personalizada en la carpeta *main.cmd.log* del contenedor de blobs. Estos registros incluyen la salida estándar de *main.cmd* y otros registros de ejecución.

### <a name="express-custom-setup"></a>Instalación personalizada rápida

Para aprovisionar o volver a configurar la instancia de Azure-SSIS IR con instalaciones personalizadas rápidas, realice los pasos siguientes.

1. Active la casilla **Customize your Azure-SSIS Integration Runtime with additional system configurations/component installations** (Personalizar su Azure-SSIS IR con configuraciones del sistema e instalaciones de componentes adicionales) en la página **Configuración avanzada** del panel **Integration runtime setup** (Configuración de Integration Runtime). 

1. Seleccione **Nueva** para abrir el panel **Add express custom setup** (Agregar instalación rápida personalizada) y, luego, seleccione un tipo en la lista desplegable **Express custom setup type** (Tipo de instalación rápida personalizada). Actualmente ofrecemos instalaciones personalizadas rápidas para ejecutar el comando cmdkey, agregar variables de entorno, instalar Azure PowerShell e instalar componentes con licencia.

#### <a name="running-cmdkey-command"></a>Ejecución del comando cmdkey

Si selecciona el tipo **Run cmdkey command** (Ejecutar comando cmdkey) para la instalación personalizada rápida, puede ejecutar el comando cmdkey de Windows en la instancia de Azure-SSIS IR. Para ello, escriba el nombre del equipo de destino o el nombre de dominio, el nombre de usuario o el nombre de la cuenta, y la contraseña o la clave de cuenta en los cuadros de texto **/Add** (/Agregar), **/User** ((Usuario) y **/Pass** (/Contraseña), respectivamente. De esta forma, podrá conservar las credenciales de acceso de los servidores SQL Server, los recursos compartidos de archivos o Azure Files en Azure-SSIS IR. Por ejemplo, para acceder a Azure Files, puede especificar `YourAzureStorageAccountName.file.core.windows.net`, `azure\YourAzureStorageAccountName` y `YourAzureStorageAccountKey` en **/Add** (/Agregar), **/User** (/Usuario) y **/Pass** (/Contraseña), respectivamente. Esta acción es similar a ejecutar el comando [cmdkey](/windows-server/administration/windows-commands/cmdkey) de Windows en la máquina local. Por ahora solo se admite una instalación rápida personalizada para ejecutar el comando cmdkey. Para ejecutar varios comandos de cmdkey, utilice una configuración personalizada estándar en su lugar.

#### <a name="adding-environment-variables"></a>Adición de variables de entorno

Si selecciona el tipo **Agregar variable de entorno** para la instalación rápida personalizada, puede agregar una variable de entorno de Windows en Azure-SSIS IR. Para ello, escriba el nombre y el valor de la variable de entorno en los cuadros de texto **Nombre de la variable de entorno** y **Valor de variable**, respectivamente. De esta forma, podrá usar la variable de entorno en los paquetes que se ejecutan en Azure-SSIS IR, por ejemplo, en componentes o tareas de scripts. Esta acción es similar a ejecutar el comando [set](/windows-server/administration/windows-commands/set_1) de Windows en la máquina local.

#### <a name="installing-azure-powershell"></a>Instalación de Azure PowerShell

Si selecciona el tipo **Instalar Azure PowerShell** para la instalación personalizada rápida, puede instalar el módulo Az de PowerShell en Azure-SSIS IR. Para ello, escriba el número de versión del módulo Az (x.y.z) que prefiera de una [lista de los admitidos](https://www.powershellgallery.com/packages/az). Así, podrá ejecutar los cmdlets o scripts de Azure PowerShell de los paquetes para administrar los recursos de Azure, por ejemplo, [Azure Analysis Services (AAS)](../analysis-services/analysis-services-powershell.md).

#### <a name="installing-licensed-components"></a>Instalación de componentes con licencia

Si selecciona el tipo **Install licensed component** (Instalar componente con licencia) para la instalación personalizada rápida, puede seleccionar los componentes integrados de nuestros asociados de ISV en la lista desplegable **Nombre del componente**:

* Si selecciona el componente **SentryOne's Task Factory** (Generador de tareas de SentryOne), puede instalar el conjunto de componentes de [Task Factory](https://www.sentryone.com/products/task-factory/high-performance-ssis-components) (Generador de tareas) desde SentryOne en su instancia de Azure-SSIS IR mediante la especificación de la clave de licencia del producto que compró en el campo **Clave de licencia**. La versión integrada actual es la **2020.21.2**.

* Si selecciona el componente **HEDDA.IO de oh22**, puede instalar el componente de calidad y limpieza de datos [HEDDA.IO](https://github.com/oh22is/HEDDA.IO/tree/master/SSIS-IR) desde oh22 en su instancia de Azure-SSIS IR. Para ello, debe comprar el servicio de antemano. La versión integrada actual es la **1.0.14**.

* Si selecciona el componente **SQLPhonetics.NET de oh22**, puede instalar el componente de calidad y coincidencia de datos [SQLPhonetics.NET](https://appsource.microsoft.com/product/web-apps/oh22.sqlphonetics-ssis) desde oh22 en su instancia de Azure-SSIS IR. Para ello, escriba la clave de licencia del producto que compró de antemano en el cuadro de texto **Clave de licencia**. La versión integrada actual es la **1.0.45**.
   
* Si selecciona el componente **SSIS Integration Toolkit de KingswaySoft**, puede instalar el conjunto de conectores de [SSIS Integration Toolkit](https://www.kingswaysoft.com/products/ssis-integration-toolkit-for-microsoft-dynamics-365) para aplicaciones de CRM/ERP/marketing/colaboración, como Microsoft Dynamics/SharePoint/Project Server, Oracle/Salesforce Marketing Cloud, etc., desde KingswaySoft en su instancia de Azure-SSIS IR, escribiendo la clave de licencia del producto que compró en el cuadro **Clave de licencia**. La versión integrada actual es la **20.2**.

* Si selecciona el componente **SSIS Productivity Pack de KingswaySoft**, puede instalar el conjunto de componentes de [SSIS Productivity Pack](https://www.kingswaysoft.com/products/ssis-productivity-pack) desde KingswaySoft en su instancia de Azure-SSIS IR, escribiendo la clave de licencia del producto que compró en el campo **Clave de licencia**. La versión integrada actual es la **20.2**.

* Si selecciona el componente **Xtract IS de Theobald Software**, puede instalar el conjunto [Xtract IS](https://theobald-software.com/en/xtract-is/) de conectores para el sistema SAP (ERP, S/4HANA, BW) desde Theobald Software en Azure-SSIS IR; para ello, arrastre y suelte o cargue el archivo de licencia del producto que compró en el cuadro del **archivo de licencia**. La versión integrada actual es la **6.5.13.18**.

* Si selecciona el componente **Integration Service de AecorSoft**, puede instalar el conjunto de conectores de [Integration Service](https://www.aecorsoft.com/en/products/integrationservice) para sistemas SAP y Salesforce desde AecorSoft en su instancia de Azure-SSIS IR. Para ello, escriba la clave de licencia del producto que compró de antemano en el cuadro de texto **Clave de licencia**. La versión integrada actual es la **3.0.00**.

* Si selecciona el componente **Paquete SSIS estándar de CData**, puede instalar el conjunto de [paquetes SSIS estándar](https://www.cdata.com/kb/entries/ssis-adf-packages.rst#standard) de los componentes más populares de CData, como los conectores de Microsoft SharePoint, en su instancia de Azure-SSIS IR. Para ello, escriba la clave de licencia del producto que compró de antemano en el cuadro de texto **Clave de licencia**. La versión integrada actual es la **19.7354**.

* Si selecciona el componente **Paquete SSIS extendido de CData**, puede instalar el conjunto de [paquetes SSIS extendidos](https://www.cdata.com/kb/entries/ssis-adf-packages.rst#extended) de todos los componentes de CData, como los conectores de Microsoft Dynamics 365 Business Central y otros componentes en el **paquete SSIS estándar**, en su instancia de Azure-SSIS IR. Para ello, escriba la clave de licencia del producto que compró de antemano en el cuadro de texto **Clave de licencia**. La versión integrada actual es la **19.7354**. Debido a su gran tamaño, para evitar que se agote el tiempo de espera de la instalación, asegúrese de que la instancia de Azure-SSIS IR tenga al menos 4 núcleos de CPU por nodo.

Las instalaciones rápidas personalizadas que agregue aparecerán en la página **Advanced settings** (Configuración avanzada). Para quitarlas, active sus casillas y, luego, seleccione **Eliminar**.

### <a name="azure-powershell"></a>Azure PowerShell

Para aprovisionar o volver a configurar la instancia de Azure-SSIS IR con instalaciones personalizadas mediante Azure PowerShell, haga lo siguiente:

1. Si Azure-SSIS IR ya se ha iniciado o se está ejecutando, deténgalo primero.

1. Luego, puede agregar o quitar instalaciones personalizadas mediante la ejecución del cmdlet `Set-AzDataFactoryV2IntegrationRuntime` antes de iniciar la instancia de Azure-SSIS IR.

   ```powershell
   $ResourceGroupName = "[your Azure resource group name]"
   $DataFactoryName = "[your data factory name]"
   $AzureSSISName = "[your Azure-SSIS IR name]"
   # Custom setup info: Standard/express custom setups
   $SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
   $ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService|CData.Standard|CData.Extended or leave it empty]" # OPTIONAL to configure an express custom setup without script

   # Add custom setup parameters if you use standard/express custom setups
   if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
   {
       Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
           -DataFactoryName $DataFactoryName `
           -Name $AzureSSISName `
           -SetupScriptContainerSasUri $SetupScriptContainerSasUri
   }
   if(![string]::IsNullOrEmpty($ExpressCustomSetup))
   {
       if($ExpressCustomSetup -eq "RunCmdkey")
       {
           $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
           $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
           $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
       }
       if($ExpressCustomSetup -eq "SetEnvironmentVariable")
       {
           $variableName = "YourVariableName"
           $variableValue = "YourVariableValue"
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
       }
       if($ExpressCustomSetup -eq "InstallAzurePowerShell")
       {
           $moduleVersion = "YourAzModuleVersion"
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
       }
       if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
       {
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
       }
       if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }    
       if($ExpressCustomSetup -eq "Theobald.XtractIS")
       {
           $jsonData = Get-Content -Raw -Path YourLicenseFile.json
           $jsonData = $jsonData -replace '\s',''
           $jsonData = $jsonData.replace('"','\"')
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "CData.Standard")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "CData.Extended")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }    
       # Create an array of one or more express custom setups
       $setups = New-Object System.Collections.ArrayList
       $setups.Add($setup)

       Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
           -DataFactoryName $DataFactoryName `
           -Name $AzureSSISName `
           -ExpressCustomSetup $setups
   }
   Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
       -DataFactoryName $DataFactoryName `
       -Name $AzureSSISName `
       -Force
   ```

### <a name="standard-custom-setup-samples"></a>Ejemplos de instalación personalizada estándar

Para ver y reutilizar algunos ejemplos de instalaciones personalizadas estándar, haga lo siguiente:

1. Conéctese al contenedor de blobs Public Preview mediante el Explorador de Azure Storage.

   a. En **Local y asociado**, haga clic con el botón derecho en **Cuentas de almacenamiento** y seleccione **Conectar a Azure Storage**.

      ![Conectar a Azure Storage](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image1.png)

   b. Seleccione **Contenedor de blobs**, **URL de firma de acceso compartido (SAS)** y **Siguiente**.

   c. En el cuadro de texto **Dirección URL de SAS del contenedor de blobs**, escriba el URI de SAS del contenedor de blobs Public Preview de debajo, seleccione **Siguiente** y luego **Conectar**.

      `https://ssisazurefileshare.blob.core.windows.net/publicpreview?sp=rl&st=2020-03-25T04:00:00Z&se=2025-03-25T04:00:00Z&sv=2019-02-02&sr=c&sig=WAD3DATezJjhBCO3ezrQ7TUZ8syEUxZZtGIhhP6Pt4I%3D`

   d. En el panel izquierdo, seleccione el contenedor de blobs **publicpreview** conectado y haga doble clic en la carpeta *CustomSetupScript*. En esta carpeta se encuentran los siguientes elementos:

      * Una carpeta *Sample*, que contiene una instalación personalizada para instalar una tarea básica en cada nodo de la instancia de Azure-SSIS IR. La tarea no hace nada, pero se queda en suspensión durante unos segundos. La carpeta también contiene una carpeta *gacutil*, cuyo contenido completo (*gacutil.exe*, *gacutil.exe.config* y *1033\gacutlrc.dll*) se puede copiar tal cual en el contenedor de blobs.

      * Una carpeta *UserScenarios*, que contiene varios ejemplos de instalaciones personalizadas de escenarios de usuario reales. Si quiere instalar varios ejemplos en la instancia de Azure-SSIS IR, puede combinar los archivos del script de instalación personalizada (*main.cmd*) en un único archivo y cargarlo con todos sus archivos asociados en el contenedor de blobs.

        ![Contenido del contenedor de blobs public preview](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image11.png)

   e. Haga doble clic en la carpeta *UserScenarios* para buscar los elementos siguientes:

      * Una carpeta *.NET FRAMEWORK 3.5*, que contiene un script de instalación personalizada (*main.cmd*) para instalar una versión anterior de .NET Framework en cada nodo de su instancia de Azure-SSIS IR. Algunos componentes personalizados podrían necesitar esta versión.

      * Una carpeta *BCP*, que contiene un script de instalación personalizada (*main.cmd*) para instalar utilidades de línea de comandos de SQL Server (*MsSqlCmdLnUtils.msi*) en cada nodo de su instancia de Azure-SSIS IR. Una de esas utilidades es el programa de copia masiva (*BCP*).

      * Una carpeta de *sufijo de DNS*, la cual contiene un script de instalación personalizado (*main.cmd*) para anexar su propio sufijo DNS (por ejemplo, *test.com*) a cualquier nombre de dominio de una sola etiqueta no calificado y convertirlo en un nombre de dominio completo (FQDN) antes de utilizarlo en consultas de DNS de su Azure-SSIS IR.

      * Una carpeta *EXCEL*, que contiene un script de instalación personalizada (*main.cmd*) para instalar algunos ensamblados y bibliotecas de C# en cada nodo de su instancia de Azure-SSIS IR. Puede usarlos en Script Tasks para leer y escribir archivos de Excel de forma dinámica. 
      
        Primero, descargue [*ExcelDataReader.dll*](https://www.nuget.org/packages/ExcelDataReader/) y [*DocumentFormat.OpenXml.dll*](https://www.nuget.org/packages/DocumentFormat.OpenXml/) y cárguelos todos junto con *main.cmd* en el contenedor de blobs. Como alternativa, si solo quiere usar los conectores estándar de Excel (Administrador de conexiones, Origen y Destino), el acceso redistribuible necesario ya está preinstalado en la instancia de Azure-SSIS IR, por lo que no necesita ninguna instalación personalizada.
      
      * Una carpeta *MYSQL ODBC*, que contiene un script de instalación personalizada (*main.cmd*) para instalar los controladores ODBC de MySQL en cada nodo de Azure-SSIS IR. Esta instalación permite usar los conectores de ODBC (Administrador de conexiones, Origen y Destino) para conectarse al servidor de MySQL. 
     
        Primero, [descargue las versiones más recientes de 64 y 32 bits de los instaladores del controlador ODBC de MySQL](https://dev.mysql.com/downloads/connector/odbc/) (por ejemplo, *mysql-connector-odbc-8.0.13-winx64.msi* y *mysql-connector-odbc-8.0.13-win32.msi*) y, luego, cárguelas todas junto con *main.cmd* en el contenedor de blobs.
        
        Si se usa el nombre del origen de datos (DSN) en la conexión, se necesita la configuración de DSN en el script de instalación. Por ejemplo: C:\Windows\SysWOW64\odbcconf.exe /A {CONFIGSYSDSN "MySQL ODBC 8.0 Unicode Driver" "DSN=\<dsnname\>|PORT=3306|SERVER=\<servername\>"}

      * Una carpeta *ORACLE ENTERPRISE*, que contiene un script de instalación personalizada (*main.cmd*) y el archivo de configuración de instalación silenciosa (*client.rsp*) para instalar los conectores de Oracle y el controlador de OCI en cada nodo de Enterprise Edition de Azure-SSIS IR. Esta instalación permite usar el Administrador de conexiones, el origen y el destino de Oracle para conectarse al servidor de Oracle. 
      
        Primero, descargue Microsoft Connectors v5.0 para Oracle (*AttunitySSISOraAdaptersSetup.msi* y *AttunitySSISOraAdaptersSetup64.msi*) del [Centro de descarga de Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=55179) y el cliente de Oracle más reciente (por ejemplo, *winx64_12102_client.zip*) desde [Oracle](https://www.oracle.com/database/technologies/oracle19c-windows-downloads.html). A continuación, cárguelos todos junto con *main.cmd* y *client.rsp* en el contenedor de blobs. Si usa TNS para conectarse a Oracle, también debe descargar *tnsnames.ora*, editarlo y cargarlo en el contenedor de blobs. De esta manera, se puede copiar en la carpeta de instalación de Oracle durante la instalación.

      * Una carpeta *ORACLE STANDARD ADO.NET*, que contiene un script de instalación personalizada (*main.cmd*) para instalar el controlador ODP.NET de Oracle en cada nodo de Azure-SSIS-IR. Esta instalación permite usar el Administrador de conexiones, el origen y el destino de ADO.NET para conectarse al servidor de Oracle. 
      
        Primero, [descargue el controlador ODP.NET más reciente de Oracle](https://www.oracle.com/technetwork/database/windows/downloads/index-090165.html) (por ejemplo, *ODP.NET_Managed_ODAC122cR1.zip*) y, luego, cárguelo junto con *main.cmd* en el contenedor de blobs.
       
      * Una carpeta *ORACLE STANDARD OLEDB*, que contiene un script de instalación personalizada (*main.cmd*) para instalar el controlador OLEDB de Oracle en cada nodo de su instancia de Azure-SSIS IR. El script también configura el nombre del origen de datos (DSN). Esta instalación le permite usar el Administrador de conexiones, el origen y el destino de ODBC, o el Administrador de conexiones y el origen de Power Query con el tipo de origen de datos ODBC para conectarse al servidor de Oracle. 
      
        Primero, descargue el cliente Oracle Instant más reciente (Basic Package o Basic Lite Package) y ODBC Package y cárguelos todos junto con *main.cmd* en el contenedor de blobs:
        * [Descargar paquetes de 64 bits](https://www.oracle.com/technetwork/topics/winx64soft-089540.html) (Basic Package: *instantclient-basic-windows.x64-18.3.0.0.0dbru.zip*; Basic Lite Package: *instantclient-basiclite-windows.x64-18.3.0.0.0dbru.zip*; ODBC Package: *instantclient-odbc-windows.x64-18.3.0.0.0dbru.zip*) 
        * [Descargar paquetes de 32 bits](https://www.oracle.com/technetwork/topics/winsoft-085727.html) (Basic Package: *instantclient-basic-nt-18.3.0.0.0dbru.zip*; Basic Lite Package: *instantclient-basiclite-nt-18.3.0.0.0dbru.zip*; ODBC Package: *instantclient-odbc-nt-18.3.0.0.0dbru.zip*)

      * Una carpeta *ORACLE STANDARD OLEDB*, que contiene un script de instalación personalizada (*main.cmd*) para instalar el controlador OLEDB de Oracle en cada nodo de la instancia de Azure-SSIS IR. Esta instalación permite usar el Administrador de conexiones, el origen y el destino de OLEDB para conectarse al servidor de Oracle. 
     
        Primero, [descargue el controlador OLEDB de Oracle más reciente](https://www.oracle.com/partners/campaign/index-090165.html) (por ejemplo, *ODAC122010Xcopy_x64.zip*) y, luego, cárguelo junto con *main.cmd* en el contenedor de blobs.

      * Una carpeta *POSTGRESQL ODBC*, que contiene un script de instalación personalizada (*main.cmd*) para instalar los controladores ODBC de PostgreSQL en cada nodo de Azure-SSIS IR. Esta instalación permite usar el Administrador de conexiones, el origen y el destino de ODBC para conectarse al servidor de PostgreSQL. 
     
        Primero, [descargue las versiones más recientes de 64 y 32 bits de los instaladores del controlador ODBC de PostgreSQL](https://www.postgresql.org/ftp/odbc/versions/msi/) (por ejemplo, *psqlodbc_x64.msi* y *psqlodbc_x86.msi*) y, luego, cárguelas todas junto con *main.cmd* en el contenedor de blobs.

      * Una carpeta *SAP BW*, que contiene un script de instalación personalizada (*main.cmd*) para instalar el ensamblado de conector .NET de SAP (*librfc32.dll*) en cada nodo de Enterprise Edition de Azure-SSIS IR. Esta configuración permite usar el Administrador de conexiones, el origen o el destino de SAP BW para conectarse al servidor de SAP BW. 
      
        Primero, cargue la versión de 64 o 32 bits de *librfc32.dll* desde la carpeta de instalación de SAP junto con *main.cmd* en el contenedor de blobs. El script copia entonces el ensamblado de SAP en la carpeta *%windir%\SysWow64* o *%windir%\System32* durante la instalación.

      * Una carpeta *STORAGE*, que contiene un script de instalación personalizada (*main.cmd*) para instalar Azure PowerShell en cada nodo de Azure-SSIS IR. Esta instalación le permite implementar y ejecutar paquetes de SSIS que ejecutan [scripts o cmdlets de PowerShell para administrar la cuenta de Azure Storage](../storage/blobs/storage-quickstart-blobs-powershell.md). 
      
        Copie *main.cmd*, un archivo *AzurePowerShell.msi* de ejemplo (o use la versión más reciente) y *storage.ps1* en el contenedor de blobs. Use *PowerShell.dtsx* como plantilla para los paquetes. La plantilla de paquetes combina una [tarea de descarga de blobs de Azure](/sql/integration-services/control-flow/azure-blob-download-task), que descarga un script de PowerShell modificable (*storage.ps1*), y una [tarea de ejecución de proceso](https://blogs.msdn.microsoft.com/ssis/2017/01/26/run-powershell-scripts-in-ssis/), que ejecuta el script en cada nodo.

      * Una carpeta *TERADATA*, que contiene un script de instalación personalizada (*main.cmd*), su archivo asociado (*install.cmd*) y los paquetes del instalador ( *.msi*). Estos archivos instalan los conectores de Teradata, la API Teradata Parallel Transporter (TPT) y el controlador ODBC en cada nodo de Enterprise Edition de Azure-SSIS Integration Runtime. Esta instalación permite usar el Administrador de conexiones, el origen y el destino de Teradata para conectarse al servidor de Teradata. 
      
        Primero, [descargue el archivo ZIP de herramientas y utilidades de Teradata 15.x](http://partnerintelligence.teradata.com) (por ejemplo, *TeradataToolsAndUtilitiesBase__windows_indep.15.10.22.00.zip*) y, luego, cárguelo junto con los archivos *.cmd* y *.msi* mencionados anteriormente en el contenedor de blobs.

      * Una carpeta *TLS 1.2*, que contiene un script de instalación personalizada (*main.cmd*) para usar solo un protocolo de red de criptografía fuerte o más seguro (TLS 1.2) en cada nodo de la instancia de Azure-SSIS IR. El script también deshabilita las versiones anteriores de SSL/TLS (SSL 3.0, TLS 1.0, TLS 1.1) al mismo tiempo.

      * Una carpeta *ZULU OPENJDK*, que contiene un script de instalación personalizada (*main.cmd*) y un archivo de PowerShell (*install_openjdk.ps1*) para instalar Zulu OpenJDK en cada nodo de Azure-SSIS IR. Esta instalación permite usar conectores de Azure Data Lake Store o Flexible File para procesar archivos ORC o Parquet. Para más información, consulte [Azure Feature Pack para Integration Services](/sql/integration-services/azure-feature-pack-for-integration-services-ssis#dependency-on-java). 
      
        Primero, [descargue el paquete Zulu OpenJDK más reciente](https://www.azul.com/downloads/zulu/zulu-windows/) (por ejemplo, *zulu8.33.0.1-jdk8.0.192-win_x64.zip*) y, luego, cárguelo junto con *main.cmd* e *install_openjdk.ps1* en el contenedor de blobs.

        ![Carpetas en la carpeta de escenarios de usuario](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image12.png)

   f. Para reutilizar estos ejemplos de instalación personalizada estándar, copie el contenido de la carpeta seleccionada en el contenedor de blobs.

1. Al aprovisionar o volver a configurar la instancia de Azure-SSIS IR en la interfaz de usuario de ADF, active la casilla **Customize your Azure-SSIS Integration Runtime with additional system configurations/component installations** (Personalizar Azure-SSIS Integration Runtime con configuraciones de sistema/instalación de componentes adicionales) en la sección **Configuración avanzada** del panel **Integration runtime setup** (Configuración de Integration Runtime). A continuación, escriba el URI de SAS del contenedor de blobs en el cuadro de texto **Custom setup container SAS URI** (URI de SAS del contenedor de instalación personalizada).
   
1. Al aprovisionar o volver a configurar la instancia de Azure-SSIS IR mediante Azure PowerShell, deténgala si ya se ha iniciado o está en ejecución, ejecute el cmdlet `Set-AzDataFactoryV2IntegrationRuntime` con el URI de SAS del contenedor de blobs como valor del parámetro `SetupScriptContainerSasUri` y, luego, inicie Azure-SSIS IR.

1. Después de que finalice la instalación personalizada estándar y se inicie Azure-SSIS IR, puede encontrar todos los registros de la instalación personalizada en la carpeta *main.cmd.log* del contenedor de blobs. Estos registros incluyen la salida estándar de *main.cmd* y otros registros de ejecución.

## <a name="next-steps"></a>Pasos siguientes

- [Instalación de Enterprise Edition de Azure-SSIS IR](how-to-configure-azure-ssis-ir-enterprise-edition.md)
- [Desarrollo de componentes con licencia o de pago en Azure-SSIS IR](how-to-develop-azure-ssis-ir-licensed-components.md)