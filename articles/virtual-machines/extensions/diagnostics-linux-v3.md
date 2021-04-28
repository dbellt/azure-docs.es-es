---
title: 'Azure Compute: extensión Diagnostic de Linux 3.0'
description: Instrucciones de configuración de la extensión Diagnostic de Linux (LAD) 3.0 de Azure para recopilar métricas y registrar eventos de VM de Linux que se ejecuten en Azure.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 12/13/2018
ms.openlocfilehash: 121ea47322303ab06a9240cd4a80e61ad882954d
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108138308"
---
# <a name="use-linux-diagnostic-extension-30-to-monitor-metrics-and-logs"></a>Uso de la extensión Diagnostics de Linux 3.0 para supervisar métricas y registros

En este documento se describen las versiones 3.0 y posteriores de la extensión Diagnostics de Linux (LAD).

> [!IMPORTANT]
> Para obtener información acerca de la versión 2.3 y anteriores, consulte [Supervisión de los datos de rendimiento y diagnóstico de una VM de Linux](/previous-versions/azure/virtual-machines/linux/classic/diagnostic-extension-v2).

## <a name="introduction"></a>Introducción

La extensión Diagnostics de Linux ayuda a los usuarios a supervisar el mantenimiento de VM de Linux que se ejecuten en Microsoft Azure. Ofrece la siguiente funcionalidad:

* Recopila métricas de rendimiento del sistema de la máquina virtual y las almacena en una tabla específica de una cuenta de almacenamiento designada.
* Recupera eventos de registro de syslog y los almacena en una tabla específica de la cuenta de almacenamiento designada.
* Permite a los usuarios personalizar las métricas de datos que se recopilan y se cargan.
* Permite a los usuarios personalizar las utilidades de syslog y los niveles de gravedad de los eventos que se recopilan y se cargan.
* Permite a los usuarios cargar los archivos de registro especificados en la tabla de almacenamiento designada.
* Admite el envío de métricas y eventos de registro a puntos de conexión arbitrarios de Azure Event Hubs y blobs con formato JSON en la cuenta de almacenamiento designada.

Esta extensión funciona con los dos modelos de implementación de Azure.

## <a name="install-the-extension-on-a-vm"></a>Instalación de la extensión en una VM

Puede habilitar esta extensión mediante cmdlets de Azure PowerShell, scripts de la CLI de Azure, plantillas de Azure Resource Monitor (plantillas de ARM) o Azure Portal. Para obtener más información, consulte [Características de extensiones](features-linux.md).

>[!NOTE]
>Algunos componentes de la extensión de VM de LAD también se incluyen en la [extensión de VM de Log Analytics](./oms-linux.md). Gracias a esta arquitectura, se pueden producir conflictos si se crean instancias de las dos extensiones en la misma plantilla de ARM. 
>
>Para evitar estos conflictos en tiempo de instalación, use la [`dependsOn`directiva](../../azure-resource-manager/templates/define-resource-dependency.md#dependson) para asegurarse de que las extensiones se instalan de forma secuencial. Las extensiones se pueden instalar en cualquier orden.

Mediante estas instrucciones de instalación y una [configuración de ejemplo descargable](https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json) se configura LAD 3.0 para:

* Capturar y almacenar las mismas métricas que las proporcionadas por LAD 2.3.
* Capturar un conjunto útil de métricas del sistema de archivos. Esta funcionalidad es nueva en LAD 3.0.
* Capturar la recopilación predeterminada de syslog habilitada por LAD 2.3.
* Habilitar la experiencia de Azure Portal para la creación de gráficos y el desencadenamiento de alertas en métricas de VM.

La configuración descargable es solo un ejemplo. Modifíquela para satisfacer sus necesidades.

### <a name="supported-linux-distributions"></a>Distribuciones de Linux compatibles

LAD es compatible con las siguientes distribuciones y versiones. La lista de distribuciones y versiones es válida únicamente con las imágenes de proveedor de Linux aprobadas por Azure. Por lo general, la extensión no admite imágenes BYOL ni BYOS de terceros, como los dispositivos.

En todas las versiones secundarias también se admite una distribución que enumera solo versiones principales, como Debian 7. Si se especifica una versión secundaria concreta, solo se admite esa versión. Si se anexa un signo más (+), se admitirán las versiones secundarias que sean iguales o posteriores a la versión especificada.

Distribuciones y versiones admitidas:

- Ubuntu 18.04, 16.04, 14.04
- CentOS 7, 6.5+
- Oracle Linux 7, 6.4+
- OpenSUSE 13.1+
- SUSE Linux Enterprise Server 12
- Debian 9, 8, 7
- Red Hat Enterprise Linux (RHEL) 7, 6.7+

### <a name="prerequisites"></a>Requisitos previos

* **Versión 2.2.0 o posterior del agente Linux de Azure**. La mayoría de las imágenes de la galería de máquina virtual Linux de Azure incluyen la versión 2.2.7 o posterior. Ejecute `/usr/sbin/waagent -version` para confirmar la versión instalada en la máquina virtual. Si la VM está ejecutando una versión anterior, [actualice el agente invitado](./update-linux-agent.md).
* La **CLI de Azure**. Si fuera necesario, [instale el entorno de la CLI de Azure](/cli/azure/install-azure-cli) en la máquina.
* El **comando wget**. Si aún no lo tiene, ejecute `sudo apt-get install wget`.
* Una **suscripción de Azure** existente. 
* Una **cuenta de almacenamiento de uso general** para almacenar los datos. Las cuentas de almacenamiento de uso general deben admitir el almacenamiento de tablas. No funcionará una cuenta de Blob Storage.
* **Python 2**.

### <a name="python-requirement"></a>Requisito de Python

La extensión Diagnostics de Linux requiere Python 2. Si la máquina virtual usa una distribución que no incluye Python 2 de manera predeterminada, debe instalarlo. Los siguientes comandos de ejemplo instalan Python 2 en varias distribuciones:   

- Red Hat, CentOS, Oracle: `yum install -y python2`
- Ubuntu, Debian: `apt-get install -y python2`
- SUSE: `zypper install -y python2`

El archivo ejecutable `python2` debe tener el valor *python* como alias. Este es un método para establecer este alias:

1. Ejecute el siguiente comando para quitar los alias existentes.
 
    ```
    sudo update-alternatives --remove-all python
    ```

2. Para crear el alias, ejecute el siguiente comando.

    ```
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    ```

### <a name="sample-installation"></a>Instalación de ejemplo

La configuración de ejemplo descargada en los siguientes ejemplos recopila un conjunto de datos estándar y los envía a un almacenamiento de tabla. La dirección URL de la configuración de ejemplo y su contenido pueden modificarse. 

En la mayoría de los casos, debe descargar una copia del archivo JSON de configuración del portal y personalizarlo en función de sus necesidades. A continuación, emplee plantillas o la automatización para usar una versión personalizada del archivo de configuración, en lugar de descargarlo desde la URL cada vez.

> [!NOTE]
> Para los siguientes ejemplos, rellene los valores correctos para las variables de la primera sección antes de ejecutar el código. 
#### <a name="azure-cli-sample"></a>Ejemplo de la CLI de Azure

```azurecli
# Set your Azure VM diagnostic variables.
my_resource_group=<your_azure_resource_group_name_containing_your_azure_linux_vm>
my_linux_vm=<your_azure_linux_vm_name>
my_diagnostic_storage_account=<your_azure_storage_account_for_storing_vm_diagnostic_data>

# Login to Azure before you do anything else.
az login

# Select the subscription that contains the storage account.
az account set --subscription <your_azure_subscription_id>

# Download the sample public settings. (You could also use curl or any web browser.)
wget https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json -O portal_public_settings.json

# Build the VM resource ID. Replace the storage account name and resource ID in the public settings.
my_vm_resource_id=$(az vm show -g $my_resource_group -n $my_linux_vm --query "id" -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" portal_public_settings.json
sed -i "s#__VM_RESOURCE_ID__#$my_vm_resource_id#g" portal_public_settings.json

# Build the protected settings (storage account SAS token).
my_diagnostic_storage_account_sastoken=$(az storage account generate-sas --account-name $my_diagnostic_storage_account --expiry 2037-12-31T23:59:00Z --permissions wlacu --resource-types co --services bt -o tsv)
my_lad_protected_settings="{'storageAccountName': '$my_diagnostic_storage_account', 'storageAccountSasToken': '$my_diagnostic_storage_account_sastoken'}"

# Finally, tell Azure to install and enable the extension.
az vm extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 3.0 --resource-group $my_resource_group --vm-name $my_linux_vm --protected-settings "${my_lad_protected_settings}" --settings portal_public_settings.json
```
#### <a name="azure-cli-sample-to-install-lad-30-on-the-virtual-machine-scale-set-instance"></a>Ejemplo de la CLI de Azure para la instalación de LAD 3.0 en la instancia del conjunto de escalado de máquinas virtuales

```azurecli
#Set your Azure Virtual Machine Scale Sets diagnostic variables.
$my_resource_group=<your_azure_resource_group_name_containing_your_azure_linux_vm>
$my_linux_vmss=<your_azure_linux_vmss_name>
$my_diagnostic_storage_account=<your_azure_storage_account_for_storing_vm_diagnostic_data>

# Login to Azure before you do anything else.
az login

# Select the subscription that contains the storage account.
az account set --subscription <your_azure_subscription_id>

# Download the sample public settings. (You could also use curl or any web browser.)
wget https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json -O portal_public_settings.json

# Build the virtual machine scale set resource ID. Replace the storage account name and resource ID in the public settings.
$my_vmss_resource_id=$(az vmss show -g $my_resource_group -n $my_linux_vmss --query "id" -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" portal_public_settings.json
sed -i "s#__VM_RESOURCE_ID__#$my_vmss_resource_id#g" portal_public_settings.json

# Build the protected settings (storage account SAS token).
$my_diagnostic_storage_account_sastoken=$(az storage account generate-sas --account-name $my_diagnostic_storage_account --expiry 2037-12-31T23:59:00Z --permissions wlacu --resource-types co --services bt -o tsv)
$my_lad_protected_settings="{'storageAccountName': '$my_diagnostic_storage_account', 'storageAccountSasToken': '$my_diagnostic_storage_account_sastoken'}"

# Finally, tell Azure to install and enable the extension.
az vmss extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 3.0 --resource-group $my_resource_group --vmss-name $my_linux_vmss --protected-settings "${my_lad_protected_settings}" --settings portal_public_settings.json
```

#### <a name="powershell-sample"></a>Ejemplo de PowerShell

```powershell
$storageAccountName = "yourStorageAccountName"
$storageAccountResourceGroup = "yourStorageAccountResourceGroupName"
$vmName = "yourVMName"
$VMresourceGroup = "yourVMResourceGroupName"

# Get the VM object
$vm = Get-AzVM -Name $vmName -ResourceGroupName $VMresourceGroup

# Get the public settings template from GitHub and update the templated values for storage account and resource ID
$publicSettings = (Invoke-WebRequest -Uri https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json).Content
$publicSettings = $publicSettings.Replace('__DIAGNOSTIC_STORAGE_ACCOUNT__', $storageAccountName)
$publicSettings = $publicSettings.Replace('__VM_RESOURCE_ID__', $vm.Id)

# If you have customized public settings, you can inline those rather than using the preceding template: $publicSettings = '{"ladCfg":  { ... },}'

# Generate a SAS token for the agent to use to authenticate with the storage account
$sasToken = New-AzStorageAccountSASToken -Service Blob,Table -ResourceType Service,Container,Object -Permission "racwdlup" -Context (Get-AzStorageAccount -ResourceGroupName $storageAccountResourceGroup -AccountName $storageAccountName).Context -ExpiryTime $([System.DateTime]::Now.AddYears(10))

# Build the protected settings (storage account SAS token)
$protectedSettings="{'storageAccountName': '$storageAccountName', 'storageAccountSasToken': '$sasToken'}"

# Finally, install the extension with the settings you built
Set-AzVMExtension -ResourceGroupName $VMresourceGroup -VMName $vmName -Location $vm.Location -ExtensionType LinuxDiagnostic -Publisher Microsoft.Azure.Diagnostics -Name LinuxDiagnostic -SettingString $publicSettings -ProtectedSettingString $protectedSettings -TypeHandlerVersion 3.0 
```

### <a name="update-the-extension-settings"></a>Actualización de la configuración de la extensión

Después de cambiar la configuración pública o protegida, ejecute el mismo comando para implementarla en la VM. Si se cambia alguna configuración, las actualizaciones se envían a la extensión. LAD recarga la configuración y se reinicia automáticamente.

### <a name="migrate-from-previous-versions-of-the-extension"></a>Migración desde versiones anteriores de la extensión

La versión más reciente de la extensión es la *4.0*. 

> [!IMPORTANT]
> En esta extensión se introducen importantes cambios de configuración. Uno de estos cambios mejora la seguridad de la extensión, por lo que no se pudo mantener la compatibilidad con versiones anteriores con la versión 2.x. Además, el publicador de esta extensión es diferente del de las versiones 2.x.
>
> Para migrar de la version 2.x a la nueva versión, desinstale primero la extensión anterior (con el nombre del publicador anterior). A continuación, instale la versión 3.

Recomendaciones:

* Instale la extensión con la actualización de versión secundaria automática habilitada.
  * En las VM con el modelo de implementación clásica, especifique la versión `3.*` si va a instalar la extensión mediante la CLI XPLAT de Azure o PowerShell.
  * En las VM con el modelo de implementación de Azure Resource Manager, incluya `"autoUpgradeMinorVersion": true` en la plantilla de implementación de VM.
* Use una cuenta de almacenamiento nueva o diferente para LAD 3.0. Hay varias pequeñas incompatibilidades entre LAD 2.3 y LAD 3.0 que hacen que compartir una cuenta sea problemático:
  * LAD 3.0 almacena los eventos de syslog en una tabla con un nombre diferente.
  * Las cadenas de `counterSpecifier` para las métricas `builtin` son diferentes en LAD 3.0.

## <a name="protected-settings"></a>Configuración protegida

Este conjunto de información de configuración contiene información confidencial que debería protegerse del acceso público. Contiene, por ejemplo, credenciales de almacenamiento. Esta configuración se transmite la extensión y es almacenada por esta de forma cifrada.

```json
{
    "storageAccountName" : "the storage account to receive data",
    "storageAccountEndPoint": "the hostname suffix for the cloud for this account",
    "storageAccountSasToken": "SAS access token",
    "mdsdHttpProxy": "HTTP proxy settings",
    "sinksConfig": { ... }
}
```

Nombre | Value
---- | -----
storageAccountName | Es el nombre de la cuenta de almacenamiento en la que la extensión escribe los datos.
storageAccountEndPoint | (Opcional) Punto de conexión que identifica la nube en la que existe la cuenta de almacenamiento. Si no hubiera configuración, LAD selecciona la nube pública de Azure (`https://core.windows.net`) de forma predeterminada. Para usar una cuenta de almacenamiento en Azure Alemania, Azure Government o Azure China 21Vianet, establezca este valor según sea necesario.
storageAccountSasToken | [Token de SAS de una cuenta](https://azure.microsoft.com/blog/sas-update-account-sas-now-supports-all-storage-services/) para los servicios de blobs y tablas (`ss='bt'`). Este token se aplica a contenedores y objetos (`srt='co'`). Asimismo, concede permisos de incorporación, creación, enumeración, actualización y escritura (`sp='acluw'`). *No* incluya el signo de interrogación principal (?).
mdsdHttpProxy | (Opcional) Información del proxy HTTP que necesita la extensión para conectarse a la cuenta de almacenamiento y el punto de conexión especificados.
sinksConfig | (Opcional) Es información sobre destinos alternativos a los que pueden enviarse métricas y eventos. En las secciones siguientes se abordan los detalles de cada receptor de datos compatible con la extensión.

Para obtener un token de SAS en una plantilla de ARM, use la función `listAccountSas`. Para ver una plantilla de ejemplo, consulte un [ejemplo de función de lista](../../azure-resource-manager/templates/template-functions-resource.md#list-example).

Es posible construir el token de SAS necesario mediante Azure Portal:

1. Seleccione la cuenta de almacenamiento de uso general en la que quiere que se escriba la extensión.
1. En el menú de la izquierda, en **Configuración**, seleccione **Firma de acceso compartido**.
1. Realice las selecciones tal como se describió anteriormente.
1. Seleccione **Generar SAS**.

:::image type="content" source="./media/diagnostics-linux/make_sas.png" alt-text="Captura de pantalla que muestra la página Firma de acceso compartido con el botón Generar SAS.":::

Copie el valor de SAS generado en el campo `storageAccountSasToken`. Quite el signo de interrogación inicial (?).

### <a name="sinksconfig"></a>sinksConfig

```json
"sinksConfig": {
    "sink": [
        {
            "name": "sinkname",
            "type": "sinktype",
            ...
        },
        ...
    ]
},
```

En esta sección opcional `sinksConfig` se definen más destinos a los que la extensión envía la información que recopila. La matriz `sink` contiene un objeto para cada receptor de datos adicional. El atributo `type` determina los demás atributos del objeto.

Elemento | Value
------- | -----
name | Una cadena que hace referencia a este receptor en cualquier otra parte de la configuración de la extensión.
type | Es el tipo de receptor que se va a definir. Determina los demás valores (si los hubiera) en instancias de este tipo.

La versión 3.0 de LAD admite dos tipos de receptor: `EventHub` y `JsonBlob`.

#### <a name="eventhub-sink"></a>Receptor de EventHub

```json
"sink": [
    {
        "name": "sinkname",
        "type": "EventHub",
        "sasURL": "https SAS URL"
    },
    ...
]
```

La entrada `"sasURL"` contiene la URL completa, incluido el token de SAS, para el centro de eventos en el que deberían publicarse los datos. LAD necesita una SAS para definir una directiva que habilite la notificación Send. 

Por ejemplo:

* Creación de un espacio de nombres de Azure Event Hubs denominado `contosohub`.
* Cree un centro de eventos en el espacio de nombres llamado `syslogmsgs`.
* Cree una directiva de acceso compartido en el centro de eventos que habilite la notificación Send. Nombre de la directiva `writer`.

Si la SAS está activa hasta la medianoche (UTC) del 1 de enero de 2018, el valor sasURL podría ser como este ejemplo:

```https
https://contosohub.servicebus.windows.net/syslogmsgs?sr=contosohub.servicebus.windows.net%2fsyslogmsgs&sig=xxxxxxxxxxxxxxxxxxxxxxxxx&se=1514764800&skn=writer
```

Para obtener más información acerca de la generación y recuperación de información sobre tokens de SAS para Event Hubs, consulte [Generación de un token de SAS](/rest/api/eventhub/generate-sas-token#powershell).

#### <a name="jsonblob-sink"></a>Receptor de JsonBlob

```json
"sink": [
    {
        "name": "sinkname",
        "type": "JsonBlob"
    },
    ...
]
```

Los datos dirigidos a un receptor de `JsonBlob` se almacenan en blobs en Azure Storage. Cada instancia de LAD crea un blob cada hora para cada nombre de receptor. Todos los blobs siempre contienen una matriz de objetos JSON sintácticamente válida. Se agregan nuevas entradas a la matriz automáticamente. 

Los blobs se almacenan en un contenedor que tiene el mismo nombre que el receptor. Las reglas de Azure Storage para los nombres de contenedores de blobs se aplican a los nombres de los receptores de `JsonBlob`. El nombre deben tener entre 3 y 63 caracteres ASCII alfanuméricos en minúsculas o con guiones.

## <a name="public-settings"></a>Configuración pública

La estructura de la configuración pública contiene varios bloques de valores de configuración que controlan la información que la extensión recopila. Cada valor de configuración es opcional. Si especifica `ladCfg`, también debe especificar `StorageAccount`.

```json
{
    "ladCfg":  { ... },
    "perfCfg": { ... },
    "fileLogs": { ... },
    "StorageAccount": "the storage account to receive data",
    "mdsdHttpProxy" : ""
}
```

Elemento | Value
------- | -----
StorageAccount | Es el nombre de la cuenta de almacenamiento en la que la extensión escribe los datos. Debe ser el nombre que se especificó en la [configuración protegida](#protected-settings).
mdsdHttpProxy | (Opcional) Es el mismo que en la [Configuración protegida](#protected-settings). El valor público reemplaza al privado, en caso de estar establecido. Coloque elementos de configuración de proxy que contengan un secreto, como una contraseña, en la [Configuración protegida](#protected-settings).

En las secciones siguientes se proporcionan detalles acerca de los elementos restantes.

### <a name="ladcfg"></a>ladCfg

```json
"ladCfg": {
    "diagnosticMonitorConfiguration": {
        "eventVolume": "Medium",
        "metrics": { ... },
        "performanceCounters": { ... },
        "syslogEvents": { ... }
    },
    "sampleRateInSeconds": 15
}
```

La estructura `ladCfg` es opcional. Esta controla la recopilación de métricas y registros para enviarlos al servicio de Métricas de Azure Monitor y otros receptores de datos. Debe especificar:

* `performanceCounters`, `syslogEvents` o ambos. 
* La estructura `metrics`.

Elemento | Value
------- | -----
eventVolume | (Opcional) Controla el número de particiones creadas en la tabla de almacenamiento. Debe ser `"Large"`, `"Medium"` o `"Small"`. Si no se especifica, el valor predeterminado es `"Medium"`.
sampleRateInSeconds | (Opcional) Intervalo predeterminado entre la recopilación de métricas sin procesar (sin agregar). La frecuencia de muestreo más pequeña admitida es de 15 segundos. Si no se especifica alguno, el valor predeterminado es `15`.

#### <a name="metrics"></a>Métricas

```json
"metrics": {
    "resourceId": "/subscriptions/...",
    "metricAggregation" : [
        { "scheduledTransferPeriod" : "PT1H" },
        { "scheduledTransferPeriod" : "PT5M" }
    ]
}
```

Elemento | Value
------- | -----
resourceId | El id. de recurso de Azure Resource Manager de la VM o del conjunto de escalado al que pertenezca la VM. Esta configuración también debe especificarse en caso de usar cualquier receptor `JsonBlob` en la configuración.
scheduledTransferPeriod | Frecuencia con la que se computan las métricas agregadas y se transfieren a las Métricas de Azure Monitor. La frecuencia se expresa como un intervalo de tiempo ISO 8601. El período de transferencia mínimo es de 60 segundos, es decir, PT1M. Especifique al menos un `scheduledTransferPeriod`.

Se recopilan muestras de las métricas especificadas en la sección `performanceCounters` cada 15 segundos o a la frecuencia de muestreo definida para el contador de manera explícita. Si aparecen varias frecuencias de `scheduledTransferPeriod`, como en el ejemplo, cada agregación se procesa de manera independiente.

#### <a name="performancecounters"></a>performanceCounters

```json
"performanceCounters": {
    "sinks": "",
    "performanceCounterConfiguration": [
        {
            "type": "builtin",
            "class": "Processor",
            "counter": "PercentIdleTime",
            "counterSpecifier": "/builtin/Processor/PercentIdleTime",
            "condition": "IsAggregate=TRUE",
            "sampleRate": "PT15S",
            "unit": "Percent",
            "annotation": [
                {
                    "displayName" : "Aggregate CPU %idle time",
                    "locale" : "en-us"
                }
            ]
        }
    ]
}
```

La sección `performanceCounters` opcional controla la recopilación de métricas. Se agregan muestras sin procesar para cada elemento [`scheduledTransferPeriod`](#metrics) a fin de generar estos valores:

* Media
* Mínima
* Máxima
* Último valor recopilado
* Número de muestras sin procesar que se usan para procesar el agregado

Elemento | Value
------- | -----
sinks | (Opcional) Lista separada por comas de nombres de receptores a los que LAD envía los resultados de las métricas agregadas. Todas las métricas agregadas se publican en cada receptor indicado. Ejemplo: `"EHsink1, myjsonsink"`. Para más información, consulte [`sinksConfig`](#sinksconfig).
type | Identifica el proveedor real de la métrica.
class | Junto con `"counter"`, identifica la métrica específica en el espacio de nombres del proveedor.
counter | Junto con `"class"`, identifica la métrica específica en el espacio de nombres del proveedor.
counterSpecifier | Identifica la métrica específica en el espacio de nombres de Métricas de Azure Monitor.
condición | (Opcional) Selecciona una instancia específica del objeto al que se aplica la métrica. O bien, selecciona la agregación en todas las instancias de ese objeto. 
sampleRate | Intervalo de ISO 8601 que establece la frecuencia de recopilación de muestras sin procesar de esta métrica. Si el valor no está establecido, el intervalo de recopilación se establece a partir del valor de [`sampleRateInSeconds`](#ladcfg). La frecuencia de muestreo más corta admitida es de 15 segundos (PT15S).
unit | Define la unidad para la métrica. Debería ser una de estas cadenas: `"Count"`, `"Bytes"`, `"Seconds"`, `"Percent"`, `"CountPerSecond"`, `"BytesPerSecond"` o `"Millisecond"`. Los consumidores de los datos recopilados esperan que los valores de los datos recopilados coincidan con esta unidad. LAD omite este campo.
DisplayName | Etiqueta que se agregará a los datos en Métricas de Azure Monitor. Esta etiqueta se establece en el idioma que se ha especificado mediante la configuración regional asociada. LAD omite este campo.

El elemento `counterSpecifier` es un identificador arbitrario. Los consumidores de métricas, como la característica de creación de gráficos y desencadenamiento de alertas de Azure Portal, usan `counterSpecifier` como "clave" para identificar una métrica o instancia de una métrica. 

Para las métricas `builtin`, se recomienda usar valores `counterSpecifier` que empiecen por `/builtin/`. Si va a recopilar una instancia específica de una métrica, es recomendable adjuntar el identificador de la instancia al valor de `counterSpecifier`. 

A continuación se muestran algunos ejemplos:

* `/builtin/Processor/PercentIdleTime`: tiempo de inactividad promediado en todas las vCPU
* `/builtin/Disk/FreeSpace(/mnt)`: espacio disponible para el sistema de archivos `/mnt`
* `/builtin/Disk/FreeSpace`: espacio libre del cual se ha realizado el promedio entre todos los sistemas de archivos montados

Ni LAD ni Azure Portal esperan que el valor de `counterSpecifier` coincida con ningún patrón. Por lo tato, debe ser coherente con la forma en que se construyen los valores `counterSpecifier`.

Si especifica `performanceCounters`, LAD siempre escribe datos en una tabla en Azure Storage. Los mismos datos se pueden escribir en blobs de JSON o instancias de Event Hubs, o ambos. Sin embargo, no puede deshabilitar el almacenamiento de datos en una tabla. 

Todas las instancias de LAD que usan el mismo nombre de cuenta de almacenamiento y el mismo punto de conexión agregan sus métricas y registros a la misma tabla. Si demasiadas VM escriben en la misma partición de tabla, Azure puede limitar las escrituras a dicha partición. 

El valor `eventVolume` hace que las entradas se repartan entre 1 (small), 10 (medium) o 100 (large) particiones. Normalmente, las particiones medianas son suficientes para evitar los límites de tráfico. 

La característica Métricas de Azure Monitor de Azure Portal usa los datos de esta tabla para crear gráficos o desencadenar alertas. El nombre de tabla es la concatenación de las siguientes cadenas:

* `WADMetrics`
* `"scheduledTransferPeriod"` para los valores agregados almacenados en la tabla
* `P10DV2S`
* Una fecha en formato "AAAAMMDD", que cambia cada 10 días

Algunos ejemplos son `WADMetricsPT1HP10DV2S20170410` y `WADMetricsPT1MP10DV2S20170609`.

#### <a name="syslogevents"></a>syslogEvents

```json
"syslogEvents": {
    "sinks": "",
    "syslogEventConfiguration": {
        "facilityName1": "minSeverity",
        "facilityName2": "minSeverity",
        ...
    }
}
```

La sección `syslogEvents` opcional controla la recopilación de eventos de registro de syslog. Si se omite la sección, no se captura ningún evento de syslog.

La recopilación de `syslogEventConfiguration` tiene una entrada para cada recurso de syslog de interés. Si el valor de `minSeverity` es `"NONE"` para un recurso en particular, o si el recurso no aparece en el elemento, no se captura ningún evento de dicho recurso.

Elemento | Value
------- | -----
sinks | Es una lista separada por comas de nombres de receptores en los que se publican los eventos de registros individuales. Todos los eventos de registro que coincidan con las restricciones de `syslogEventConfiguration` se publican en cada receptor indicado. Ejemplo: `"EHforsyslog"`
facilityName | Nombre de instalación de syslog, como `"LOG_USER"` o `"LOG\LOCAL0"`. Para obtener más información, consulte la sección "Facility" de la [página man de syslog](http://man7.org/linux/man-pages/man3/syslog.3.html).
minSeverity | Nivel de gravedad de syslog (como `"LOG_ERR"` o `"LOG_INFO"`). Para obtener más información, consulte la sección "Level" de la [página man de syslog](http://man7.org/linux/man-pages/man3/syslog.3.html). La extensión captura eventos enviados al recurso al nivel especificado o uno superior.

Si especifica `syslogEvents`, LAD siempre escribe datos en una tabla en Azure Storage. Los mismos datos se pueden escribir en blobs de JSON o instancias de Event Hubs, o ambos. Sin embargo, no puede deshabilitar el almacenamiento de datos en una tabla. 

El comportamiento de la creación de particiones para esta tabla es el mismo descrito para `performanceCounters`. El nombre de tabla es la concatenación de las siguientes cadenas:

* `LinuxSyslog`
* Una fecha en formato "AAAAMMDD", que cambia cada 10 días

Algunos ejemplos son `LinuxSyslog20170410` y `LinuxSyslog20170609`.

### <a name="perfcfg"></a>perfCfg

La sección `perfCfg` es opcional. Controla la ejecución de consultas arbitrarias de [Open Management Infrastructure (OMI).](https://github.com/Microsoft/omi)

```json
"perfCfg": [
    {
        "namespace": "root/scx",
        "query": "SELECT PercentAvailableMemory, PercentUsedSwap FROM SCX_MemoryStatisticalInformation",
        "table": "LinuxOldMemory",
        "frequency": 300,
        "sinks": ""
    }
]
```

Elemento | Value
------- | -----
espacio de nombres | (Opcional) Es el espacio de nombres OMI en el que la consulta debería ejecutarse. Si no se especifica, se usa el valor predeterminado `"root/scx"`. Lo implementan los [proveedores multiplataforma de System Center](https://github.com/Microsoft/SCXcore).
Query | Consulta de OMI que se va a ejecutar.
table | (Opcional) Es la tabla de Azure Storage, en la cuenta de almacenamiento designada. Para obtener más información, consulte la [Configuración protegida](#protected-settings).
frequency | (Opcional) Número de segundos entre las ejecuciones de consulta. El valor predeterminado es 300 (5 minutos). El valor mínimo es de 15 segundos.
sinks | (Opcional) Lista de nombres separada por comas de más receptores en los que deberían publicarse los resultados de las métricas de muestra sin procesar. Ni la extensión ni las Métricas de Azure Monitor computan ninguna agregación de estos ejemplos sin procesar.

Debe especificarse `"table"` o `"sinks"`, o ambos.

### <a name="filelogs"></a>fileLogs

La sección `fileLogs` controla la captura de los archivos de registro. LAD captura las líneas de texto nuevas a medida que se escriben en el archivo. A continuación, las escribe en filas de la tabla o en cualquier receptor especificado (como `JsonBlob` y `EventHub`).

> [!NOTE]
> Los elementos `fileLogs` se capturan mediante un subcomponente de LAD denominado `omsagent`. Para recopilar `fileLogs`, asegúrese de que el usuario `omsagent` tiene permisos de lectura en los archivos que haya especificado. El usuario también debe tener permisos de ejecución en todos los directorios de la ruta de acceso a ese archivo. Después de instalar LAD, puede comprobar los permisos mediante la ejecución de `sudo su omsagent -c 'cat /path/to/file'`.

```json
"fileLogs": [
    {
        "file": "/var/log/mydaemonlog",
        "table": "MyDaemonEvents",
        "sinks": ""
    }
]
```

Elemento | Value
------- | -----
archivo | Ruta de acceso completa del archivo de registro que va a inspeccionarse y capturarse. El nombre de la ruta de acceso debe dar nombre a un único archivo. No puede nombrar un directorio ni puede contener caracteres comodín. La cuenta de usuario `omsagent` debe tener acceso de lectura a la ruta de acceso del archivo.
table | (Opcional) Tabla de Azure Storage en la que se escriben nuevas líneas de la "cola" del archivo. La tabla debe estar en la cuenta de almacenamiento designada, tal como se especificó en la configuración protegida. 
sinks | (Opcional) Lista de nombres separada por comas de receptores adicionales a los que se envían líneas de registro.

Debe especificarse `"table"` o `"sinks"`, o ambos.

## <a name="metrics-supported-by-the-builtin-provider"></a>Métricas admitidas por el proveedor de builtin

El proveedor de métricas `builtin` es una fuente de las métricas más interesantes para una gran cantidad de usuarios. Estas métricas se dividen en cinco clases amplias:

* Procesador
* Memoria
* Red
* Sistema de archivos
* Disco

### <a name="builtin-metrics-for-the-processor-class"></a>Métricas builtin para la clase Processor

La clase de métricas Processor proporciona información sobre el uso del procesador en la máquina virtual. Cuando se agregan porcentajes, el resultado es el promedio de todas las CPU. 

En una VM con dos vCPU, si una está ocupada al 100 % y la otra está inactiva al 100 %, el valor de `PercentIdleTime` notificado es 50. Si la actividad de cada vCPU es del 50 % durante el mismo período, el resultado indicado también es de 50. En una VM de cuatro vCPU, cuando una está ocupada al 100 % y las otras están inactivas, el valor de `PercentIdleTime` notificado es 75.

Contador | Significado
------- | -------
PercentIdleTime | Porcentaje de tiempo del período de agregación durante el que los procesadores ejecutaron el bucle de inactividad del kernel.
PercentProcessorTime | Porcentaje de tiempo durante el que se ejecutó un subproceso no inactivo.
PercentIOWaitTime | Porcentaje de tiempo durante el que se esperó la finalización de operaciones de E/S.
PercentInterruptTime | Porcentaje de tiempo durante el que se ejecutaron interrupciones de hardware o software y DPC (llamadas a procedimiento aplazadas).
PercentUserTime | Del tiempo no inactivo durante el período de agregación, el porcentaje que se empleó en el modo de usuario en prioridad normal.
PercentNiceTime | Del tiempo no inactivo, el porcentaje empleado en prioridad disminuida (nice)
PercentPrivilegedTime | Del tiempo no inactivo, el porcentaje empleado en modo privilegiado (kernel)

Los cuatro primeros contadores deberían sumar un 100 %. Los tres últimos contadores también suman un 100 %. Estos tres contadores subdividen la suma de `PercentProcessorTime`, `PercentIOWaitTime` y `PercentInterruptTime`.

Para agregar una única métrica de todos los procesadores, establezca `"condition": "IsAggregate=TRUE"`. Para obtener una métrica para un procesador específico, como el segundo procesador lógico de una máquina virtual de cuatro vCPU, establezca `"condition": "Name=\\"1\\""`. Los números de procesador lógico se encuentran en el rango `[0..n-1]`.

### <a name="builtin-metrics-for-the-memory-class"></a>Métricas builtin para la clase Memory

La clase de métricas Memory proporciona información sobre el uso, paginación e intercambio de memoria.

Contador | Significado
------- | -------
AvailableMemory | Memoria física disponible en MiB
PercentAvailableMemory | Memoria física disponible en forma de porcentaje del total de memoria.
UsedMemory | Memoria física en uso (MiB)
PercentUsedMemory | Memoria física en uso en forma de porcentaje del total de memoria.
PagesPerSec | Paginación total (lectura y escritura)
PagesReadPerSec | Páginas leídas del almacén de respaldo, como el archivo de intercambio, el archivo de programa y el archivo asignado.
PagesWrittenPerSec | Páginas escritas en el almacén de respaldo, como el archivo de intercambio y el archivo asignado.
AvailableSwap | Espacio de intercambio sin usar (MiB)
PercentAvailableSwap | Espacio de intercambio sin usar en forma de porcentaje del total de intercambio.
UsedSwap | Espacio de intercambio (MiB) en uso
PercentUsedSwap | Espacio de intercambio en uso en forma de porcentaje del total de intercambio.

Esta clase de métrica tiene solo una instancia. El atributo `"condition"` no tiene valores de configuración útiles y debería omitirse.

### <a name="builtin-metrics-for-the-network-class"></a>Métricas builtin para la clase Network

La clase de métricas Network proporciona información sobre la actividad de red de una interfaz de red individual desde el inicio. 

LAD no expone métricas de ancho de banda. Estas se pueden obtener a partir de las métricas de host.

Contador | Significado
------- | -------
BytesTransmitted | Total de bytes enviados desde el inicio.
BytesReceived | Total de bytes recibidos desde el inicio.
BytesTotal | Total de bytes enviados o recibidos desde el inicio.estrella.
PacketsTransmitted | Total de paquetes enviados desde el inicio.
PacketsReceived | Total de paquetes recibidos desde el inicio.
TotalRxErrors | Número de errores de recepción desde el inicio.
TotalTxErrors | Número de errores de transmisión desde el inicio.
TotalCollisions | Número de colisiones notificadas por los puertos de red desde el inicio.

Aunque la clase Network está instanciada, LAD no admite la captura de agregados de métricas Network de todos los dispositivos de red. Para obtener las métricas de una interfaz instanciada, como eth0, establezca `"condition": "InstanceID=\\"eth0\\""`.

### <a name="builtin-metrics-for-the-file-system-class"></a>Métricas "builtin" para la clase del sistema de archivos.

La clase de métricas del sistema de archivos proporciona información acerca del uso del sistema de archivos. Los valores absolutos y porcentuales se notifican tal como se mostrarían a un usuario normal (no raíz).

Contador | Significado
------- | -------
FreeSpace | Espacio disponible en disco en bytes
UsedSpace | Espacio usado en disco en bytes
PercentFreeSpace | Porcentaje de espacio libre
PercentUsedSpace | Porcentaje de espacio usado
PercentFreeInodes | Porcentaje de nodos de índice sin usar (inodes).
PercentUsedInodes | Porcentaje de inodes asignados (en uso) sumado de todos los sistemas de archivos.
BytesReadPerSecond | Bytes leídos por segundo
BytesWrittenPerSecond | Bytes escritos por segundo
BytesPerSecond | Bytes leídos o escritos por segundo
ReadsPerSecond | Operaciones de lectura por segundo
WritesPerSecond | Operaciones de escritura por segundo
TransfersPerSecond | Operaciones de lectura o escritura por segundo

Puede obtener valores agregados de todos los archivos del sistema si establece `"condition": "IsAggregate=True"`. Obtenga los valores para un sistema de archivos montado específico, como `"/mnt"`; para ello, establezca `"condition": 'Name="/mnt"'`. 

> [!NOTE]
> Si usa Azure Portal en lugar de JSON, el formato correcto para el campo de condición es `Name='/mnt'`.

### <a name="builtin-metrics-for-the-disk-class"></a>Métricas builtin para la clase Disk

La clase de métricas Disk proporciona información sobre el uso de dispositivos de disco. Estas estadísticas se aplican a toda la unidad. 

Cuando un dispositivo tiene varios sistemas de archivos, los contadores para dicho dispositivo se agregan, de hecho, en todos los sistemas de archivos.

Contador | Significado
------- | -------
ReadsPerSecond | Operaciones de lectura por segundo
WritesPerSecond | Operaciones de escritura por segundo
TransfersPerSecond | Total de operaciones por segundo
AverageReadTime | Promedio de segundos por operación de lectura
AverageWriteTime | Promedio de segundos por operación de escritura
AverageTransferTime | Promedio de segundos por operación
AverageDiskQueueLength | Promedio de operaciones de disco en cola
ReadBytesPerSecond | Número de bytes leídos por segundo
WriteBytesPerSecond | Número de bytes escritos por segundo
BytesPerSecond | Número de bytes leídos o escritos por segundo

Puede obtener valores agregados de todos los disco si establece `"condition": "IsAggregate=True"`. Para obtener información para un dispositivo en concreto (por ejemplo, `/dev/sdf1`), establezca `"condition": "Name=\\"/dev/sdf1\\""`.

## <a name="install-and-configure-lad-30"></a>Instalación y configuración de LAD 3.0

### <a name="azure-cli"></a>Azure CLI

Si la configuración protegida se encuentra en el archivo *ProtectedSettings.json* y la información de configuración pública está en *PublicSettings.json*, ejecute este comando.

```azurecli
az vm extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 3.0 --resource-group <resource_group_name> --vm-name <vm_name> --protected-settings ProtectedSettings.json --settings PublicSettings.json
```

El comando supone que se está usando el modo de Azure Resource Manager de la CLI de Azure. Si quiere configurar LAD para las VM con el modelo de implementación clásica, cambie al modo "asm" (`azure config mode asm`) y omita el nombre del grupo de recursos en el comando. 

Para obtener más información, consulte la [documentación de la CLI multiplataforma](/cli/azure/authenticate-azure-cli).

### <a name="powershell"></a>PowerShell

Si la configuración protegida se encuentra en la variable `$protectedSettings` y la información de configuración pública está en la variable `$publicSettings`, ejecute este comando:

```powershell
Set-AzVMExtension -ResourceGroupName <resource_group_name> -VMName <vm_name> -Location <vm_location> -ExtensionType LinuxDiagnostic -Publisher Microsoft.Azure.Diagnostics -Name LinuxDiagnostic -SettingString $publicSettings -ProtectedSettingString $protectedSettings -TypeHandlerVersion 3.0
```

## <a name="example-lad-30-configuration"></a>Configuración de LAD 3.0 de ejemplo

Conforme a las definiciones anteriores, esta sección incluye un ejemplo de configuración de la extensión LAD 3.0 y una explicación. Para aplicar este ejemplo a su caso, use su propio nombre de cuenta de almacenamiento, token de SAS de cuenta y tokens de SAS de Event Hubs.

> [!NOTE]
> Dependiendo de si usa la CLI de Azure o PowerShell para instalar LAD, el método para proporcionar la configuración pública y protegida es diferente: 
>
> * Si está usando la CLI de Azure, guarde la siguiente configuración en *ProtectedSettings.json* y *PublicSettings.json* para usarla con el comando de ejemplo anterior. 
> * Si está usando PowerShell, guarde la siguiente configuración en `$protectedSettings` y `$publicSettings` mediante la ejecución de `$protectedSettings = '{ ... }'`.

### <a name="protected-settings"></a>Configuración protegida

La configuración protegida define:

* Una cuenta de almacenamiento.
* Un token de SAS de cuenta coincidente.
* Varios receptores (`JsonBlob` o `EventHub` con tokens de SAS).

```json
{
  "storageAccountName": "yourdiagstgacct",
  "storageAccountSasToken": "sv=xxxx-xx-xx&ss=bt&srt=co&sp=wlacu&st=yyyy-yy-yyT21%3A22%3A00Z&se=zzzz-zz-zzT21%3A22%3A00Z&sig=fake_signature",
  "sinksConfig": {
    "sink": [
      {
        "name": "SyslogJsonBlob",
        "type": "JsonBlob"
      },
      {
        "name": "FilelogJsonBlob",
        "type": "JsonBlob"
      },
      {
        "name": "LinuxCpuJsonBlob",
        "type": "JsonBlob"
      },
      {
        "name": "MyJsonMetricsBlob",
        "type": "JsonBlob"
      },
      {
        "name": "LinuxCpuEventHub",
        "type": "EventHub",
        "sasURL": "https://youreventhubnamespace.servicebus.windows.net/youreventhubpublisher?sr=https%3a%2f%2fyoureventhubnamespace.servicebus.windows.net%2fyoureventhubpublisher%2f&sig=fake_signature&se=1808096361&skn=yourehpolicy"
      },
      {
        "name": "MyMetricEventHub",
        "type": "EventHub",
        "sasURL": "https://youreventhubnamespace.servicebus.windows.net/youreventhubpublisher?sr=https%3a%2f%2fyoureventhubnamespace.servicebus.windows.net%2fyoureventhubpublisher%2f&sig=yourehpolicy&skn=yourehpolicy"
      },
      {
        "name": "LoggingEventHub",
        "type": "EventHub",
        "sasURL": "https://youreventhubnamespace.servicebus.windows.net/youreventhubpublisher?sr=https%3a%2f%2fyoureventhubnamespace.servicebus.windows.net%2fyoureventhubpublisher%2f&sig=yourehpolicy&se=1808096361&skn=yourehpolicy"
      }
    ]
  }
}
```

### <a name="public-settings"></a>Configuración pública

Los valores públicos hacen que LAD:

* Cargue métricas de porcentaje de tiempo de procesador y espacio usado en disco en la tabla `WADMetrics*`.
* Cargue mensajes de la instalación de syslog `"user"` y la gravedad `"info"` en la tabla `LinuxSyslog*`.
* Cargue los resultados de la consulta OMI sin procesar (`PercentProcessorTime` y `PercentIdleTime`) en la tabla denominada `LinuxCPU`.
* Cargue líneas anexadas en el archivo `/var/log/myladtestlog` en la tabla `MyLadTestLog`.

En cualquier caso, también se cargan datos en:

* Azure Blob Storage. El nombre del contenedor es el que se definió en el receptor `JsonBlob`.
* El punto de conexión de Event Hubs, tal y como se especificó en el receptor `EventHub`.

```json
{
  "StorageAccount": "yourdiagstgacct",
  "ladCfg": {
    "sampleRateInSeconds": 15,
    "diagnosticMonitorConfiguration": {
      "performanceCounters": {
        "sinks": "MyMetricEventHub,MyJsonMetricsBlob",
        "performanceCounterConfiguration": [
          {
            "unit": "Percent",
            "type": "builtin",
            "counter": "PercentProcessorTime",
            "counterSpecifier": "/builtin/Processor/PercentProcessorTime",
            "annotation": [
              {
                "locale": "en-us",
                "displayName": "Aggregate CPU %utilization"
              }
            ],
            "condition": "IsAggregate=TRUE",
            "class": "Processor"
          },
          {
            "unit": "Bytes",
            "type": "builtin",
            "counter": "UsedSpace",
            "counterSpecifier": "/builtin/FileSystem/UsedSpace",
            "annotation": [
              {
                "locale": "en-us",
                "displayName": "Used disk space on /"
              }
            ],
            "condition": "Name=\"/\"",
            "class": "Filesystem"
          }
        ]
      },
      "metrics": {
        "metricAggregation": [
          {
            "scheduledTransferPeriod": "PT1H"
          },
          {
            "scheduledTransferPeriod": "PT1M"
          }
        ],
        "resourceId": "/subscriptions/your_azure_subscription_id/resourceGroups/your_resource_group_name/providers/Microsoft.Compute/virtualMachines/your_vm_name"
      },
      "eventVolume": "Large",
      "syslogEvents": {
        "sinks": "SyslogJsonBlob,LoggingEventHub",
        "syslogEventConfiguration": {
          "LOG_USER": "LOG_INFO"
        }
      }
    }
  },
  "perfCfg": [
    {
      "query": "SELECT PercentProcessorTime, PercentIdleTime FROM SCX_ProcessorStatisticalInformation WHERE Name='_TOTAL'",
      "table": "LinuxCpu",
      "frequency": 60,
      "sinks": "LinuxCpuJsonBlob,LinuxCpuEventHub"
    }
  ],
  "fileLogs": [
    {
      "file": "/var/log/myladtestlog",
      "table": "MyLadTestLog",
      "sinks": "FilelogJsonBlob,LoggingEventHub"
    }
  ]
}
```

El elemento `resourceId` de la configuración debe coincidir con el de la VM o el del conjunto de escalado de máquinas virtuales.

* La característica de creación de gráficos y desencadenamiento de alertas de métricas de la plataforma de Azure conoce el elemento `resourceId` de la VM en la que se esté trabajando. Espera encontrar los datos de la VM mediante el elemento `resourceId` de la clave de búsqueda.
* Si usa la escalabilidad automática de Azure, el elemento `resourceId` de la configuración de escalabilidad automática debe coincidir con el `resourceId` que usa LAD.
* El elemento `resourceId` está integrado en los nombres de los blobs JSON que ha escrito LAD.

## <a name="view-your-data"></a>Ver los datos

Use Azure Portal para consultar los datos de rendimiento o establecer alertas:

:::image type="content" source="./media/diagnostics-linux/graph_metrics.png" alt-text="Captura de pantalla que muestra Azure Portal. La métrica &quot;Espacio en disco usada en&quot; está seleccionada. Se muestra el gráfico resultante.":::

Los datos de `performanceCounters` siempre se almacenan en una tabla de Azure Storage. Las API de Azure Storage están disponibles para múltiples lenguajes y plataformas.

Los datos enviados a los receptores de `JsonBlob` se almacenan en blobs de la cuenta de almacenamiento a los que se dio nombre en la [configuración protegida](#protected-settings). Puede consumir los datos de blob mediante cualquier API de Azure Blob Storage.

También puede usar las siguientes herramientas de la interfaz de usuario para acceder a los datos de Azure Storage:

* Explorador de servidores de Visual Studio
* [Explorador de Azure Storage](https://azurestorageexplorer.codeplex.com/)

La siguiente captura de pantalla de una sesión del Explorador de Azure Storage muestra las tablas y los contenedores de Azure Storage generados a partir de una extensión de LAD 3.0 configurada correctamente en una VM de prueba. La imagen no coincide exactamente con la [configuración de ejemplo de LAD 3.0](#example-lad-30-configuration).

:::image type="content" source="./media/diagnostics-linux/stg_explorer.png" alt-text="Captura de pantalla que muestra el Explorador de Azure Storage.":::


Para obtener más información acerca de cómo consumir mensajes publicados en un punto de conexión de Event Hubs, consulte la [documentación de Event Hubs](../../event-hubs/event-hubs-about.md) pertinente.

## <a name="next-steps"></a>Pasos siguientes

* En [Azure Monitor](../../azure-monitor/alerts/alerts-classic-portal.md), cree alertas para las métricas que recopile.
* Cree [gráficos de supervisión](../../azure-monitor/data-platform.md) para las métricas.
* [Cree un conjunto de escalado de máquinas virtuales](../linux/tutorial-create-vmss.md) con sus propias métricas para controlar la escalabilidad automática.