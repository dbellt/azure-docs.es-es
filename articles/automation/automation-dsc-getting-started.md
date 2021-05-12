---
title: Introducción a Azure Automation State Configuration
description: En este artículo se indica cómo realizar las tareas más comunes de Azure Automation State Configuration.
services: automation
ms.subservice: dsc
ms.date: 04/15/2019
ms.topic: conceptual
ms.openlocfilehash: 8ff365200828b786b4cf770508e5a811983abd58
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109732747"
---
# <a name="get-started-with-azure-automation-state-configuration"></a>Introducción a Azure Automation State Configuration

En este artículo se proporciona una guía paso a paso para realizar las tareas más comunes con Azure Automation State Configuration, como, por ejemplo, crear, importar y compilar configuraciones, habilitar máquinas para administrarlas y ver informes. Para una introducción a State Configuration, consulte [Introducción a State Configuration](automation-dsc-overview.md). Para obtener documentación de Desired State Configuration, consulte [Información general sobre la configuración de estado deseado de Windows PowerShell](/powershell/scripting/dsc/overview/overview).

Si busca un entorno de ejemplo que ya esté configurado sin seguir los pasos descritos en este artículo, puede usar [Azure Automation Managed Node](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.automation/automation-configuration). Esta plantilla configura un entorno completado de State Configuration (DSC), incluida una máquina virtual de Azure administrada por State Configuration (DSC).

## <a name="prerequisites"></a>Prerrequisitos

Para completar los ejemplos de este artículo, se requiere lo siguiente:

- Una cuenta de Azure Automation Para más información sobre las cuentas de Automation y sus requisitos, consulte [Introducción a la autenticación de cuentas de Automation](./automation-security-overview.md).
- Una máquina virtual de Azure Resource Manager (no clásico) ejecuta un [sistema operativo compatible](automation-dsc-overview.md#operating-system-requirements). Para obtener instrucciones sobre la creación de una máquina virtual, consulte [Creación de la primera máquina virtual de Windows en el Portal de Azure](../virtual-machines/windows/quick-create-portal.md)

## <a name="create-a-dsc-configuration"></a>Creación de una configuración de DSC

Crearemos una [configuración DSC](/powershell/scripting/dsc/configurations/configurations) simple que garantice la presencia o ausencia de la característica de Windows **Web-Server** (IIS), en función de cómo se asignen los nodos.

1. Inicie [VSCode](https://code.visualstudio.com/docs) (o cualquier editor de texto).
1. Escriba el siguiente texto:

    ```powershell
    configuration TestConfig
    {
        Node IsWebServer
        {
            WindowsFeature IIS
            {
                Ensure               = 'Present'
                Name                 = 'Web-Server'
                IncludeAllSubFeature = $true
            }
        }

        Node NotWebServer
        {
            WindowsFeature IIS
            {
                Ensure               = 'Absent'
                Name                 = 'Web-Server'
            }
        }
    }
    ```
1. Guarde el archivo como **TestConfig.ps1**.

Esta configuración llama a un recurso en cada bloque de nodo, el [recurso WindowsFeature](/powershell/scripting/dsc/reference/resources/windows/windowsfeatureresource). Este recurso garantiza la presencia o ausencia de la característica **Web-Server**.

## <a name="import-a-configuration-into-azure-automation"></a>Importar una configuración en Azure Automation

A continuación, debe importar la configuración en la cuenta de Automation.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, seleccione **State configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), haga clic en la pestaña **Configuraciones** y, después, haga clic en **Agregar**.
1. En la página Importar configuración, vaya al archivo `TestConfig.ps1` en el equipo.

   ![Captura de pantalla de la hoja **Importar configuración**](./media/automation-dsc-getting-started/AddConfig.png)

1. Haga clic en **OK**.

## <a name="view-a-configuration-in-azure-automation"></a>Vistan de una configuración en Azure Automation

Después de importar una configuración, puede verla en el Portal de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, seleccione **State configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), haga clic en la pestaña **Configuraciones** y, después, haga clic en **TestConfig**. Es el nombre de la configuración que importó en el procedimiento anterior.
1. En el panel Configuración de TestConfig, haga clic en **Ver el origen de configuración**.

   ![Captura de pantalla de la hoja TestConfig configuration (Configuración de TestConfig)](./media/automation-dsc-getting-started/ViewConfigSource.png)

   Se abre un panel Origen de configuración de TestConfig, que muestra el código de PowerShell de la configuración.

## <a name="compile-a-configuration-in-azure-automation"></a>Compilar una configuración en Azure Automation

Para poder aplicar un estado deseado a un nodo, se debe compilar una configuración de DSC que defina dicho estado en una o varias configuraciones de nodo (documento de MOF) y se debe colocar en el servidor de extracción de DSC de Automatización. Para una descripción más detallada de la compilación de configuraciones en State Configuration (DSC), consulte [Compilar configuraciones en DSC de Azure Automation](automation-dsc-compile.md).
Para más información sobre la compilación de configuraciones, consulte [Configuraciones DSC](/powershell/scripting/dsc/configurations/configurations).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), haga clic en la pestaña **Configuraciones** y, después, haga clic en **TestConfig**. Este es el nombre de la configuración importada previamente.
1. En el panel Configuración de TestConfig, haga clic en **Compilar** y luego en **Sí**. Así se inicia un trabajo de compilación.

   ![Captura de pantalla de la página de configuración de TestConfig en la que se resalta el botón Compilar](./media/automation-dsc-getting-started/CompileConfig.png)

> [!NOTE]
> Cuando se compila una configuración en Azure Automation, esta implementa automáticamente todos los archivos MOF de configuración de nodo creados en el servidor de extracción.

## <a name="view-a-compilation-job"></a>Vista de un trabajo de compilación

Después de iniciar una compilación, puede verla en el icono **Trabajos de compilación** de la página **Configuración**. El icono **Compilation Jobs** (Trabajos de compilación) muestra los trabajos con errores, completados y en ejecución. Cuando abre el panel de un trabajo de compilación, se muestra información acerca del trabajo, incluidos los errores o advertencias encontrados, los parámetros de entrada usados en la configuración y los registros de compilación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), haga clic en la pestaña **Configuraciones** y, después, haga clic en **TestConfig**. Este es el nombre de la configuración importada previamente.
1. En **Trabajos de compilación**, seleccione el trabajo de compilación que quiere ver. Se abre el panel Trabajo de compilación, etiquetado con la fecha en que se inició el trabajo de compilación.

   ![Captura de pantalla de la página Trabajo de compilación](./media/automation-dsc-getting-started/CompilationJob.png)

1. Haga clic en cualquiera de los iconos del panel Trabajo de compilación para ver detalles adicionales acerca del trabajo.

## <a name="view-node-configurations"></a>Vista de configuraciones de nodos

La finalización correcta de un trabajo de compilación crea una o varias configuraciones de nodo nuevas. Una configuración de nodo es un documento MOF que se implementa en el servidor de extracción y está listo ser extraído y que lo apliquen uno o varios nodos. Puede ver las configuraciones de nodos de su cuenta de Automation en la página State configuration (DSC). El formato del nombre de la configuración de un nodo es `ConfigurationName.NodeName`.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State configuration (DSC), haga clic en la pestaña **Configuraciones compiladas**.

   ![Captura de pantalla de la pestaña de configuraciones compiladas](./media/automation-dsc-getting-started/NodeConfigs.png)

## <a name="enable-an-azure-resource-manager-vm-for-management-with-state-configuration"></a>Habilitar una máquina virtual de Azure Resource Manager para su administración con State Configuration

Puede usar State Configuration para administrar máquinas virtuales de Azure (tanto clásicas como de Resource Manager), máquinas virtuales locales, máquinas Linux, máquinas virtuales de AWS y máquinas físicas locales. En este artículo, se explicará cómo habilitar solo máquinas virtuales de Azure Resource Manager. Para más información sobre cómo habilitar otros tipos de máquinas, consulte [Habilitar máquinas para administrarlas con Azure Automation State Configuration](automation-dsc-onboarding.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), seleccione la pestaña **Nodos** y haga clic en **Agregar**.

   ![Captura de pantalla de la página Nodos DSC en la que se resalta el botón Agregar máquina virtual de Azure](./media/automation-dsc-getting-started/OnboardVM.png)

1. En el panel Máquinas virtuales, seleccione la máquina virtual.
1. En la página de detalles de la máquina virtual, haga clic en **Conectar**.

   > [!IMPORTANT]
   > Debe ser una máquina virtual de Azure Resource Manager que ejecute un [sistema operativo compatible](automation-dsc-overview.md#operating-system-requirements).

2. En la página Registro, seleccione el nombre de la configuración del nodo que quiere aplicar a la máquina virtual en el campo **Nombre de la configuración del nodo**. Especificar un nombre en este momento es opcional. Puede cambiar la configuración de nodo asignada después de habilitar el nodo.

3. Marque **Reboot Node if Needed** (Reiniciar el nodo si es necesario) y haga clic en **Aceptar**.

   ![Captura de pantalla de la hoja Registro](./media/automation-dsc-getting-started/RegisterVM.png)

   La configuración de nodo que especificó se aplica a la máquina virtual a intervalos especificados por el valor proporcionado para **Frecuencia del modo de configuración**. La máquina virtual comprueba si hay actualizaciones de la configuración de nodo en los intervalos especificados por el valor **Frecuencia de actualización**. Para más información sobre cómo se utilizan estos valores, consulte [Configuración del administrador de configuración local](/powershell/scripting/dsc/managing-nodes/metaConfig).

Azure inicia el proceso de habilitar la máquina virtual. Cuando finalice, la máquina virtual se muestra en la pestaña **Nodos** de la página State configuration (DSC) de la cuenta de Automation.

## <a name="view-the-list-of-managed-nodes"></a>Vista de la lista de nodos administrados

Puede ver la lista de todas las máquinas que se han habilitado para administración en la cuenta de Automation, en la pestaña **Nodos**  de la página State configuration (DSC).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State configuration (DSC), haga clic en la pestaña **Nodos**.

## <a name="view-reports-for-managed-nodes"></a>Ver informes de nodos administrados

Cada vez que State Configuration realiza una comprobación de coherencia en un nodo administrado, el nodo envía un informe de estado al servidor de extracción. Estos informes se pueden ver en la página de dicho nodo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State configuration (DSC), haga clic en la pestaña **Nodos**. Aquí, puede ver la información general del estado de configuración y los detalles de cada nodo.

   ![Captura de pantalla de la página de nodos](./media/automation-dsc-getting-started/NodesTab.png)

1. En la pestaña **Nodos**, haga clic en el registro del nodo para abrir los informes. Haga clic en el informe del que quiere ver detalles adicionales.

   ![Captura de pantalla de la hoja Informe](./media/automation-dsc-getting-started/NodeReport.png)

En la hoja de un informe individual puede ver la siguiente información de estado de la comprobación de coherencia correspondiente:

- Estado del informe. Los valores posibles son:
    * Compatible: el nodo es conforme con la comprobación.
   * Error: la comprobación de la configuración fue incorrecta.
   * No conforme: el nodo está en modo `ApplyandMonitor` y el equipo no está en el estado deseado.
- La hora de inicio de la comprobación de coherencia.
- El tiempo de ejecución total de la comprobación de coherencia.
- El tipo de comprobación de coherencia.
- Todos los errores, incluidos el código de error y el mensaje de error.
- Los recursos de DSC utilizados en la configuración y el estado de cada recurso (si el nodo está en el estado deseado para dicho recurso). Puede hacer clic en cada recurso para obtener información más detallada sobre ese recurso.
- El nombre, la dirección IP y el modo de configuración del nodo.

También puede hacer clic en **Ver informe sin formato** para ver los datos reales que el nodo envía al servidor.
Para más información sobre el uso de esos datos, consulte [Uso de un servidor de informes de DSC](/powershell/scripting/dsc/pull-server/reportserver).

Después de que se habilita un nodo puede pasar un tiempo antes de que el primer informe esté disponible. Después de habilitar un nodo es posible que tenga que esperar hasta 30 minutos para que aparezca el primer informe.

## <a name="reassign-a-node-to-a-different-node-configuration"></a>Reasignación de un nodo a una configuración de nodo diferente

Puede asignar un nodo para que use una configuración de nodo diferente a la que le asignó inicialmente.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. A la izquierda, haga clic en **Todos los recursos** y, luego, en el nombre de la cuenta de Automation.
1. En la página Cuenta de Automation, haga clic en **State configuration (DSC)** en **Administración de configuración**.
1. En la página State configuration (DSC), haga clic en la pestaña **Nodos**.
1. En la página **Nodos**, haga clic en el nombre del nodo que quiere reasignar.
1. En la página de dicho nodo, haga clic en **Asignar configuración de nodo**.

    ![Captura de pantalla de la página de detalles del nodo donde se resalta el botón Asignar configuración de nodo](./media/automation-dsc-getting-started/AssignNode.png)

1. En la página Asignar configuración de nodo, seleccione la configuración de nodo a la que desea asignar el nodo y luego haga clic en **Aceptar**.

    ![Captura de pantalla de la página Asignar configuración de nodo](./media/automation-dsc-getting-started/AssignNodeConfig.png)

## <a name="unregister-a-node"></a>Anular el registro de un nodo

Si ya no desea que State Configuration administre un nodo, puede anular su registro. Consulte [Cómo quitar una configuración y un nodo de Automation State Configuration](./state-configuration/remove-node-and-configuration-package.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, consulte [Información general de Azure Automation State Configuration](automation-dsc-overview.md).
- Para habilitar la característica para las máquinas virtuales de su entorno, consulte [Habilitar Azure Automation State Configuration](automation-dsc-onboarding.md).
- Para comprender DSC de PowerShell, consulte [Información general sobre Desired State Configuration de PowerShell](/powershell/scripting/dsc/overview/overview).
- Para obtener información de precios, consulte [Precios de State Configuration de Azure Automation](https://azure.microsoft.com/pricing/details/automation/).
- Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation).
