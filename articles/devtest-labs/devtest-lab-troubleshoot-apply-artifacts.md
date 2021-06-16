---
title: Solución de problemas con artefactos en Azure DevTest Labs | Microsoft Docs
description: Obtenga información acerca de cómo solucionar los problemas que se producen al aplicar artefactos en una máquina virtual de Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6d3f37a942010240835238648c48ad5671ec028d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110692437"
---
# <a name="troubleshoot-issues-when-applying-artifacts-in-an-azure-devtest-labs-virtual-machine"></a>Solución de problemas al aplicar artefactos en una máquina virtual de Azure DevTest Labs
Se puede producir un error en la aplicación de artefactos en una máquina virtual por varias razones. Este artículo le guía en algunos de los métodos para ayudar a identificar las posibles causas.

Si necesita más ayuda en cualquier momento con este artículo, puede ponerse en contacto con los expertos de Azure DevTest Labs (DTL) en los [foros de Azure MSDN y Stack Overflow](https://azure.microsoft.com/support/forums/). Como alternativa, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte técnico.   

> [!NOTE]
> Este artículo se aplica tanto a las máquinas virtuales Windows como a las que no son Windows. Aunque hay algunas diferencias, se describirán explícitamente en este artículo.

## <a name="quick-troubleshooting-steps"></a>Pasos rápidos para solucionar problemas
Compruebe que la máquina virtual está en ejecución. DevTest Labs requiere que la máquina virtual esté en ejecución y que el [agente de máquina virtual de Microsoft Azure (agente de VM)](../virtual-machines/extensions/agent-windows.md) esté instalado y listo.

> [!TIP]
> En **Azure Portal**, vaya a la página **Administrar artefactos** de la máquina virtual para ver si la máquina virtual está lista para aplicar artefactos. Verá un mensaje en la parte superior de la página. 
> 
> Con **Azure PowerShell**, inspeccione la marca **canApplyArtifacts**, que solo se devuelve cuando se expande en una operación GET. Consulte el siguiente comando de ejemplo:

```powershell
Select-AzSubscription -SubscriptionId $SubscriptionId | Out-Null
$vm = Get-AzResource `
        -Name "$LabName/$VmName" `
        -ResourceGroupName $LabRgName `
        -ResourceType 'microsoft.devtestlab/labs/virtualmachines' `
        -ApiVersion '2018-10-15-preview' `
        -ODataQuery '$expand=Properties($expand=ComputeVm)'
$vm.Properties.canApplyArtifacts
```

## <a name="ways-to-troubleshoot"></a>Formas de solucionar problemas 
Puede solucionar los problemas de las máquinas virtuales creadas mediante DevTest Labs y el modelo de implementación de Resource Manager con uno de los métodos siguientes:

- **Azure Portal**: excelente si necesita obtener rápidamente una sugerencia visual de lo que puede estar causando el problema.
- **Azure PowerShell**: si está familiarizado con el símbolo del sistema de PowerShell, consulte rápidamente los recursos de DevTest Labs mediante los cmdlets de Azure PowerShell.

> [!TIP]
> Para más información sobre cómo revisar la ejecución de artefactos dentro de una máquina virtual, consulte [Diagnóstico de errores de artefactos en el laboratorio](devtest-lab-troubleshoot-artifact-failure.md).

## <a name="symptoms-causes-and-potential-resolutions"></a>Síntomas, causas y posibles soluciones 

### <a name="artifact-appears-to-stop-responding"></a>Parece que el artefacto deja de responder

Parece que un artefacto deja de responder hasta que expira un período de tiempo de espera predefinido y el artefacto se marca como **Error**.

Cuando parece que un artefacto deja de responder, lo primero es determinar dónde está bloqueado. Un artefacto puede estar bloqueado en cualquiera de los siguientes pasos durante la ejecución:

- **Durante la solicitud inicial**. DevTest Labs crea una plantilla de Azure Resource Manager para solicitar el uso de la Extensión de script personalizado (CSE). Por lo tanto, en segundo plano, se desencadena una implementación de grupo de recursos. Cuando se produce un error en este nivel, se pueden obtener detalles en los **registros de actividad** del grupo de recursos para la máquina virtual en cuestión.  
    - Puede acceder al registro de actividad desde la barra de navegación de la página de la máquina virtual del laboratorio. Al seleccionarla, verá una entrada para **Aplicar artefactos a la máquina virtual** (si la operación Aplicar artefactos se desencadenó directamente) o **Agregar o modificar máquinas virtuales** (si la operación de aplicación de artefactos formaba parte del proceso de creación de la máquina virtual).
    - Busque errores en estas entradas. A veces, el error no estará etiquetado como corresponde y tendrá que investigar cada entrada.
    - Al investigar los detalles de cada entrada, asegúrese de revisar el contenido de la carga JSON. Es posible que vea un error en la parte inferior de ese documento.
- **Al intentar ejecutar el artefacto**. Podría deberse a problemas de redes o de almacenamiento. Consulte la sección correspondiente más adelante en este artículo para más información. También puede ocurrir debido al modo en el que se crea el script. Por ejemplo:
    - Un script de PowerShell tiene **parámetros obligatorios**, pero a uno no se le ha pasado un valor, ya sea porque se permite que el usuario lo deje en blanco o porque no tiene un valor predeterminado para la propiedad en el archivo de definición artifactfile.json. El script dejará de responder porque está esperando la entrada del usuario.
    - Un script de PowerShell **requiere de entrada del usuario** como parte de la ejecución. Los scripts se deben escribir para que funcionen de forma silenciosa sin necesidad de intervención del usuario.
- **El agente de máquina virtual tarda mucho en estar listo**. Cuando la máquina virtual se inicia por primera vez o cuando la extensión de script personalizado se instala por primera vez para atender la solicitud de aplicación de artefactos, es posible que la máquina virtual requiera actualizar el agente de máquina virtual o esperar a que se inicialice el agente de máquina virtual. Puede haber servicios de los que depende el agente de máquina virtual y que tardan mucho tiempo en inicializarse. En tales casos, consulte [Introducción al agente de máquina virtual de Azure](../virtual-machines/extensions/agent-windows.md) para más información de solución de problemas.

### <a name="to-verify-if-the-artifact-appears-to-stop-responding-because-of-the-script"></a>Para comprobar si parece que el artefacto deja de responder debido al script

1. Inicie sesión en la máquina virtual en cuestión.
2. Copie el script localmente en la máquina virtual o búsquelo en la máquina virtual en `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\<version>`. Es la ubicación donde se descargan los scripts del artefacto.
3. Con un símbolo del sistema con privilegios elevados, ejecute el script de forma local y proporcione los mismos valores de parámetro usados para provocar el problema.
4. Determine si el script se ve afectado por cualquier comportamiento no deseado. Si es así, solicite una actualización del artefacto (si procede del repositorio público); o bien, realice las correcciones personalmente (si procede de su repositorio privado).

> [!TIP]
> Puede corregir problemas con los artefactos hospedados en nuestro [repositorio público](https://github.com/Azure/azure-devtestlab) y enviar los cambios para nuestra revisión y aprobación. Consulte la sección **Contribuciones** en el documento [README.md](https://github.com/Azure/azure-devtestlab/blob/master/Artifacts/README.md).
> 
> Para obtener información sobre cómo escribir sus propios artefactos, consulte el documento [AUTHORING.md](https://github.com/Azure/azure-devtestlab/blob/master/Artifacts/AUTHORING.md).

### <a name="to-verify-if-the-artifact-appears-to-stop-responding-because-of-the-vm-agent"></a>Para comprobar si parece que el artefacto deja de responder debido al agente de máquina virtual:
1. Inicie sesión en la máquina virtual en cuestión.
2. Con el Explorador de archivos, vaya a **C:\WindowsAzure\logs**.
3. Busque y abra el archivo **WaAppAgent.log**.
4. Busque las entradas que se muestran cuando se inicia el agente de máquina virtual y cuando está finalizando la inicialización (es decir, cuando se envía el primer latido). Concéntrese en las entradas más recientes o especialmente en las que hay alrededor del período de tiempo en el que se experimenta el problema.

    ```
    [00000006] [11/14/2019 05:52:13.44] [INFO]  WindowsAzureGuestAgent starting. Version 2.7.41491.949
    ...
    [00000006] [11/14/2019 05:52:31.77] [WARN]  Waiting for OOBE to Complete ...
    ...
    [00000006] [11/14/2019 06:02:30.43] [WARN]  Waiting for OOBE to Complete ...
    [00000006] [11/14/2019 06:02:33.43] [INFO]  StateExecutor initialization completed.
    [00000020] [11/14/2019 06:02:33.43] [HEART] WindowsAzureGuestAgent Heartbeat.
    ```
    En este ejemplo, puede ver que la hora de inicio del agente de máquina virtual tardó 10 minutos y 20 segundos porque se envió un latido. La causa en este caso era que el servicio OOBE tardaba mucho tiempo en iniciarse.

> [!TIP]
> Para una información general sobre las extensiones de Azure, consulte [Características y extensiones de las máquinas virtuales de Azure](../virtual-machines/extensions/overview.md).

## <a name="storage-errors"></a>Errores de almacenamiento
DevTest Labs requiere acceso a la cuenta de almacenamiento del laboratorio que se crea para almacenar en memoria caché los artefactos. Cuando DevTest Labs aplica un artefacto, leerá la configuración del artefacto y sus archivos de los repositorios configurados. De forma predeterminada, DevTest Labs configura el acceso al **repositorio de artefactos públicos**.

En función de cómo se configure una máquina virtual, es posible que no tenga acceso directo a este repositorio. Por lo tanto, por diseño, DevTest Labs almacena en memoria caché los artefactos en una cuenta de almacenamiento que se crea cuando el laboratorio se inicializa por primera vez.

Si el acceso a esta cuenta de almacenamiento está bloqueado de cualquier manera y también cuando se bloquea el tráfico de la máquina virtual al servicio Azure Storage, es posible que vea un error similar al siguiente:

```shell
CSE Error: Failed to download all specified files. Exiting. Exception: Microsoft.WindowsAzure.Storage.StorageException: The remote server returned an error: (403) Forbidden. ---> System.Net.WebException: The remote server returned an error: (403) Forbidden.
```

El error anterior aparecería en la sección **Mensaje de implementación** de la página **Resultados del artefacto** en **Administrar artefactos**. También aparecerá en los **registros de actividad** en el grupo de recursos de la máquina virtual en cuestión.

### <a name="to-ensure-communication-to-the-azure-storage-service-isnt-being-blocked"></a>Para asegurarse de que no se bloquea la comunicación con el servicio Azure Storage:

- **Busque los grupos de seguridad de red (NSG) agregados**. Puede ser que se haya agregado una directiva de suscripción en la que los NSG se configuran automáticamente en todas las redes virtuales. También afectaría a la red virtual predeterminada del laboratorio, si se usa o a otra red virtual configurada en el laboratorio utilizada para la creación de máquinas virtuales.
- **Compruebe la cuenta de almacenamiento del laboratorio predeterminado** (es decir, la primera cuenta de almacenamiento creada cuando se creó el laboratorio, cuyo nombre normalmente empieza por la letra "a" y termina con un número de varios dígitos, es decir, a\<labname\>#).
    1. Vaya al grupo de recursos del laboratorio.
    2. Busque el recurso del tipo **cuenta de almacenamiento** cuyo nombre coincide con la convención.
    3. Vaya a la página de la cuenta de almacenamiento llamada **Firewalls y redes virtuales**.
    4. Asegúrese de que está establecida en **Todas las redes**. Si está seleccionada la opción **Redes seleccionadas**, asegúrese de que las redes virtuales del laboratorio usadas para crear las máquinas virtuales se han agregado a la lista.

Para solucionar problemas más detallados, consulte [Configuración de firewalls y redes virtuales de Azure Storage](../storage/common/storage-network-security.md).

> [!TIP]
> **Compruebe las reglas del grupo de seguridad de red**. Use la [Comprobación del flujo IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md#use-ip-flow-verify) para confirmar si una regla de un grupo de seguridad de red bloque el tráfico hacia o desde una máquina virtual. También puede revisar las reglas del grupo de seguridad vigentes para asegurarse de que existe una regla de entrada del grupo de seguridad de red de tipo **Permitir**. Para más información, consulte [Uso de las reglas de seguridad vigentes para solucionar problemas de flujo de tráfico de máquinas virtuales](../virtual-network/diagnose-network-traffic-filter-problem.md).

## <a name="other-sources-of-error"></a>Otros orígenes del error
Hay otros posibles orígenes del error menos frecuentes. Asegúrese de evaluar cada uno de ellos para ver si se aplica a su caso. Este es uno de ellos: 

- **Token de acceso personal expirado para el repositorio privado**. Cuando expira, el artefacto no aparecerá en la lista y cualquier script que haga referencia a artefactos de un repositorio con un token de acceso privado expirado producirá un error en consecuencia.

## <a name="next-steps"></a>Pasos siguientes
Si no se produjo ninguno de estos errores y sigue sin poder aplicar artefactos, puede abrir una incidencia de Soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.
