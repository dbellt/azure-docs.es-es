---
title: 'Tutorial: Configuración del enrutamiento de mensajes para Azure IoT Hub mediante una plantilla de Azure Resource Manager'
description: 'Tutorial: Configuración del enrutamiento de mensajes para Azure IoT Hub mediante una plantilla de Azure Resource Manager'
author: robinsh
manager: philmeagit st
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 202ee16088e1f3b80751443d6ba5c39074e5589c
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689520"
---
# <a name="tutorial-use-an-azure-resource-manager-template-to-configure-iot-hub-message-routing"></a>Tutorial: Uso de una plantilla de Azure Resource Manager para configurar el enrutamiento de mensajes de IoT Hub

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="message-routing"></a>Enrutamiento de mensajes

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

## <a name="download-the-template-and-parameters-file"></a>Descarga de la plantilla y el archivo de parámetros

En la segunda parte de este tutorial, descargará y ejecutará una aplicación de Visual Studio para enviar mensajes a IoT Hub. En esa descarga hay una carpeta que contiene la plantilla de Azure Resource Manager y el archivo de parámetros, así como los scripts de la CLI de Azure y PowerShell.

Siga adelante y descargue ya los [ejemplos de C# de Azure IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Descomprima el archivo master.zip. La plantilla de Resource Manager y el archivo de parámetros están en /iot-hub/Tutorials/Routing/SimulatedDevice/resources/ as **template_iothub.json** y **template_iothub_parameters.json**.

## <a name="create-your-resources"></a>Creación de los recursos

Va a usar una plantilla de Azure Resource Manager (RM) para crear todos los recursos. En el caso de los scripts de la CLI de Azure y PowerShell, se pueden ejecutar unas cuantas líneas cada vez. Una plantilla de Resource Manager se implementa en un solo paso. En este artículo se muestran las secciones por separado para ayudarle a entender cada una de ellas. A continuación, se mostrará cómo implementar la plantilla y crear el dispositivo virtual para las pruebas. Después de implementar la plantilla, puede ver la configuración de enrutamiento de mensajes en el portal.

Hay varios nombres de recurso que deben ser únicos globalmente, como el nombre de IoT Hub y el nombre de la cuenta de almacenamiento. Para facilitar la asignación de nombres a los recursos, esos nombres de recursos se configuran para anexar un valor alfanumérico aleatorio generado a partir de la fecha y hora actuales. 

Si examina la plantilla, verá dónde están configuradas las variables para estos recursos que toman el parámetro pasado y concatenan *randomValue* con el parámetro. 

En la siguiente sección se explican los parámetros usados.

### <a name="parameters"></a>Parámetros

La mayoría de estos parámetros tiene valores predeterminados. Los que terminan con **_in** se concatenan con *randomValue* para que sean globalmente únicos. 

**randomValue**: este valor se genera a partir de la fecha y hora actuales al implementar la plantilla. Este campo no está en el archivo de parámetros, ya que se genera en la propia plantilla.

**subscriptionId**: este campo se establece en la suscripción en la que se va a implementar la plantilla. Este campo no está en el archivo de parámetros, ya que se establece automáticamente.

**IoTHubName_in**: este campo es el nombre de IoT Hub base, que se concatena con randomValue para que sea globalmente único.

**location**: este campo es la región de Azure en la que se va a realizar la implementación, por ejemplo, "westus".

**consumer_group**: este campo es el grupo de consumidores establecido para los mensajes que llegan a través del punto de conexión de enrutamiento. Sirve para filtrar los resultados de Azure Stream Analytics. Por ejemplo, está la secuencia entera donde obtiene todo, o si tiene datos que llegan con consumer_group establecido en **Contoso**, puede configurar una secuencia de Azure Stream Analytics (y el informe de Power BI) para mostrar solo esas entradas. Este campo se usa en la parte 2 de este tutorial.

**sku_name**: este campo es la escala de IoT Hub. Este valor debe ser S1 o superior; un nivel gratis no sirve para este tutorial porque no admite varios puntos de conexión.

**sku_units**: este campo se queda con el valor de **sku_name**, y es el número de unidades de IoT Hub que se pueden usar.

**d2c_partitions**: este campo es el número de particiones usadas para la secuencia de eventos.

**storageAccountName_in**: este campo es el nombre de la cuenta de almacenamiento que se va a crear. Los mensajes se enrutan a un contenedor de la cuenta de almacenamiento. Este campo se concatena con randomValue para que sea globalmente único.

**storageContainerName**: este campo es el nombre del contenedor en el que se almacenan los mensajes enrutados a la cuenta de almacenamiento.

**storage_endpoint**: este campo es el nombre del punto de conexión de la cuenta de almacenamiento que usa el enrutamiento de mensajes.

**service_bus_namespace_in**: este campo es el nombre del espacio de nombres de Service Bus que se va a crear. Este valor se concatena con randomValue para que sea globalmente único.

**service_bus_queue_in**: este campo es el nombre de la cola de Service Bus que se usa para el enrutamiento de mensajes. Este valor se concatena con randomValue para que sea globalmente único.

**AuthRules_sb_queue**: este campo son las reglas de autorización de la cola de Service Bus que se usan para recuperar la cadena de conexión de la cola.

### <a name="variables"></a>variables

Estos valores se usan en la plantilla y se derivan principalmente de los parámetros.

**queueAuthorizationRuleResourceId**: este campo es el valor de ResourceId de la regla de autorización de la cola de Service Bus. ResourceId se usa a su vez para recuperar la cadena de conexión de la cola.

**iotHubName**: este campo es el nombre de IoT Hub después de haber concatenado randomValue. 

**storageAccountName**: este campo es el nombre de la cuenta de almacenamiento después de haber concatenado randomValue. 

**service_bus_namespace**: este campo es el espacio de nombres después de haber concatenado randomValue.

**service_bus_queue**: este campo es el nombre de la cola de Service Bus después de haber concatenado randomValue.

**sbVersion**: la versión de la API de Service Bus que se usará. En este caso, es "2017-04-01".

### <a name="resources-storage-account-and-container"></a>Recursos: Cuenta de almacenamiento y contenedor

El primer recurso creado es la cuenta de almacenamiento, junto con el contenedor al que se enrutan los mensajes. El contenedor es un recurso en la cuenta de almacenamiento. Tiene una cláusula `dependsOn` para la cuenta de almacenamiento, que exige que esta se cree antes que el contenedor.

Este es el aspecto de esta sección:

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2018-07-01",
    "location": "[parameters('location')]",
    "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
    },
    "kind": "Storage",
    "properties": {},
    "resources": [
        {
        "type": "blobServices/containers",
        "apiVersion": "2018-07-01",
        "name": "[concat('default/', parameters('storageContainerName'))]",
        "properties": {
            "publicAccess": "None"
            } ,
        "dependsOn": [
            "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
            ]
        }
    ]
}
```

### <a name="resources-service-bus-namespace-and-queue"></a>Recursos: Espacio de nombres y cola de Service Bus

El segundo recurso creado es el espacio de nombres de Service Bus, junto con la cola de Service Bus a la que se enrutan los mensajes. La SKU se establece en estándar. La versión de API se recupera de las variables. También se establece para activar el espacio de nombres de Service Bus cuando se implementa esta sección (status:Active). 

```json
{
    "type": "Microsoft.ServiceBus/namespaces",
    "comments": "The Sku should be 'Standard' for this tutorial.",
    "sku": {
        "name": "Standard",
        "tier": "Standard"
    },
    "name": "[variables('service_bus_namespace')]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "properties": {
        "provisioningState": "Succeeded",
        "metricId": "[concat('a4295411-5eff-4f81-b77e-276ab1ccda12:', variables('service_bus_namespace'))]",
        "serviceBusEndpoint": "[concat('https://', variables('service_bus_namespace'),'.servicebus.windows.net:443/')]",
        "status": "Active"
    },
    "dependsOn": []
}
```

Esta sección crea la cola de Service Bus. Esta parte del script tiene una cláusula `dependsOn` que garantiza que el espacio de nombres se crea antes que la cola.

```json
{
    "type": "Microsoft.ServiceBus/namespaces/queues",
    "name": "[concat(variables('service_bus_namespace'), '/', variables('service_bus_queue'))]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "scale": null,
    "properties": {},
    "dependsOn": [
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]"
    ]
}
```

### <a name="resources-iot-hub-and-message-routing"></a>Recursos: IoT Hub y enrutamiento de mensajes

Ahora que se han creado la cuenta de almacenamiento y la cola de Service Bus, creará el centro de IoT que enruta los mensajes a ellas. La plantilla de Resource Manager usa cláusulas `dependsOn` para impedir el intento de crear el centro antes de que se hayan creado los recursos de Service Bus y la cuenta de almacenamiento. 

Esta es la primera parte de la sección de IoT Hub. Esta parte de la plantilla configura las dependencias y comienza con las propiedades.

```json
{
    "apiVersion": "2018-04-01",
    "type": "Microsoft.Devices/IotHubs",
    "name": "[variables('IoTHubName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces/queues', variables('service_bus_namespace'), variables('service_bus_queue'))]"
    ],
    "properties": {
        "eventHubEndpoints": {}
            "events": {
                "retentionTimeInDays": 1,
                "partitionCount": "[parameters('d2c_partitions')]"
                }
            },
```

La siguiente es la sección para la configuración del enrutamiento de mensajes de IoT Hub. Primero está la sección de los puntos de conexión. Esta parte de la plantilla configura los puntos de conexión de enrutamientos para la cola de Service Bus y la cuenta de almacenamiento, incluidas las cadenas de conexión.

Para crear la cadena de conexión para la cola, necesita el valor queueAuthorizationRulesResourcedId, que se recupera alineado. Para crear la cadena de conexión para la cuenta de almacenamiento, recuperará la clave de almacenamiento principal y, luego, la usará en el formato de la cadena de conexión.

En la configuración de los puntos de conexión es donde se establece también el formato de blob en `AVRO` o `JSON`.

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

 ```json
"routing": {
    "endpoints": {
        "serviceBusQueues": [
        {
            "connectionString": "[Concat('Endpoint=sb://',variables('service_bus_namespace'),'.servicebus.windows.net/;SharedAccessKeyName=',parameters('AuthRules_sb_queue'),';SharedAccessKey=',listkeys(variables('queueAuthorizationRuleResourceId'),variables('sbVersion')).primaryKey,';EntityPath=',variables('service_bus_queue'))]",
            "name": "[parameters('service_bus_queue_endpoint')]",
            "subscriptionId": "[parameters('subscriptionId')]", 
            "resourceGroup": "[resourceGroup().Name]"
        }
        ],
        "serviceBusTopics": [],
        "eventHubs": [],
        "storageContainers": [
            {
                "connectionString": 
                "[Concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'),';AccountKey=',listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value)]",
                "containerName": "[parameters('storageContainerName')]",
                "fileNameFormat": "{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}",
                "batchFrequencyInSeconds": 100,
                "maxChunkSizeInBytes": 104857600,
                "encoding": "avro",
                "name": "[parameters('storage_endpoint')]",
                "subscriptionId": "[parameters('subscriptionId')]",
                "resourceGroup": "[resourceGroup().Name]"
            }
        ]
    },
```

La siguiente sección es para las rutas de mensajes a los puntos de conexión. Hay una configurada para cada punto de conexión, así que hay una para la cola de Service Bus y otra para el contenedor de la cuenta de almacenamiento.

Recuerde que la condición de consulta de los mensajes enrutados al almacenamiento es `level="storage"`, y que la condición de consulta de los mensajes enrutados a la cola de Service Bus es `level="critical"`.

```json
"routes": [
    {
        "name": "contosoStorageRoute",
        "source": "DeviceMessages",
        "condition": "level=\"storage\"",
        "endpointNames": [
            "[parameters('storage_endpoint')]"
            ],
        "isEnabled": true
    },
    {
        "name": "contosoSBQueueRoute",
        "source": "DeviceMessages",
        "condition": "level=\"critical\"",
        "endpointNames": [
            "[parameters('service_bus_queue_endpoint')]"
            ],
        "isEnabled": true
    }
],
```

Este archivo JSON muestra el resto de la sección de IoT Hub, que contiene información predeterminada y la SKU del centro.

```json
            "fallbackRoute": {
                "name": "$fallback",
                "source": "DeviceMessages",
                "condition": "true",
                "endpointNames": [
                    "events"
                ],
                "isEnabled": true
            }
        },
        "storageEndpoints": {
            "$default": {
                "sasTtlAsIso8601": "PT1H",
                "connectionString": "",
                "containerName": ""
            }
        },
        "messagingEndpoints": {
            "fileNotifications": {
                "lockDurationAsIso8601": "PT1M",
                "ttlAsIso8601": "PT1H",
                "maxDeliveryCount": 10
            }
        },
        "enableFileUploadNotifications": false,
        "cloudToDevice": {
            "maxDeliveryCount": 10,
            "defaultTtlAsIso8601": "PT1H",
            "feedback": {
                "lockDurationAsIso8601": "PT1M",
                "ttlAsIso8601": "PT1H",
                "maxDeliveryCount": 10
            }
        }
    },
    "sku": {
        "name": "[parameters('sku_name')]",
        "capacity": "[parameters('sku_units')]"
    }
}
```

### <a name="resources-service-bus-queue-authorization-rules"></a>Recursos: Reglas de autorización de la cola de Service Bus

Las reglas de autorización de la cola de Service Bus se usan para recuperar la cadena de conexión de la cola de Service Bus. Usa una cláusula `dependsOn` para asegurarse de que no se crea antes que el espacio de nombres y la cola de Service Bus.

```json
{
    "type": "Microsoft.ServiceBus/namespaces/queues/authorizationRules",
    "name": "[concat(variables('service_bus_namespace'), '/', variables('service_bus_queue'), '/', parameters('AuthRules_sb_queue'))]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "scale": null,
    "properties": {
        "rights": [
            "Send"
        ]
    },
    "dependsOn": [
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces/queues', variables('service_bus_namespace'), variables('service_bus_queue'))]"
    ]
},
```

### <a name="resources-consumer-group"></a>Recursos: Grupo de consumidores

En esta sección, creará un grupo de consumidores para los datos de IoT Hub que se van a usar en Azure Stream Analytics en la segunda parte de este tutorial.

```json
{
    "type": "Microsoft.Devices/IotHubs/eventHubEndpoints/ConsumerGroups",
    "name": "[concat(variables('iotHubName'), '/events/',parameters('consumer_group'))]",
    "apiVersion": "2018-04-01",
    "dependsOn": [
        "[concat('Microsoft.Devices/IotHubs/', variables('iotHubName'))]"
    ]
}
```

### <a name="resources-outputs"></a>Recursos: Salidas

Si quiere reenviar un valor al script de implementación que se va a mostrar, usará una sección de salida. Esta parte de la plantilla devuelve la cadena de conexión de la cola de Service Bus. No es necesario devolver un valor, se incluye como ejemplo de cómo devolver resultados al script de llamada.

```json
"outputs": {
    "sbq_connectionString": {
      "type": "string",
      "value": "[Concat('Endpoint=sb://',variables('service_bus_namespace'),'.servicebus.windows.net/;SharedAccessKeyName=',parameters('AuthRules_sb_queue'),';SharedAccessKey=',listkeys(variables('queueAuthorizationRuleResourceId'),variables('sbVersion')).primaryKey,';EntityPath=',variables('service_bus_queue'))]"
    }
  }
```

## <a name="deploy-the-rm-template"></a>Implementación de la plantilla de Resource Manager

Para implementar la plantilla en Azure, cargue la plantilla y el archivo de parámetros en Azure Cloud Shell y, a continuación, ejecute un script para implementar la plantilla. Abra Azure Cloud Shell e inicie sesión. En este ejemplo se usa PowerShell.

Para cargar los archivos, seleccione el icono **Upload/Download files** (Cargar/Descargar archivos) en la barra de menús y, luego, elija Upload (Cargar).

![Captura de pantalla que resalta el icono Cargar/Descargar archivos.](media/tutorial-routing-config-message-routing-RM-template/CloudShell_upload_files.png)

Use el Explorador de archivos que aparece para buscar los archivos en el disco local, selecciónelos y elija **Open** (Abrir).

Una vez cargados los archivos, un cuadro de diálogo de resultados muestra algo similar a la siguiente imagen.

![Barra de menús de Cloud Shell con Upload/Download results (Cargar/Descargar resultados) resaltado](media/tutorial-routing-config-message-routing-RM-template/CloudShell_upload_results.png)

Los archivos se cargan en el recurso compartido usado mediante la instancia de Cloud Shell. 

Ejecute el script para realizar la implementación. La última línea de este script recupera la variable que se configuró para devolverse: la cadena de conexión de la cola de Service Bus.

El script establece y utiliza estas variables:

**$RGName** es el nombre del grupo de recursos en el que se va a implementar la plantilla. Este campo se crea antes de implementar la plantilla.

**$location** es la ubicación de Azure que se usará para la plantilla, por ejemplo, "westus".

**deploymentname** es un nombre que se asigna a la implementación para recuperar el valor de variable devuelto.

Este es el script de PowerShell. Copie este script de PowerShell, péguelo en la ventana de Cloud Shell y presione Entrar para ejecutarlo.

```powershell
$RGName="ContosoResources"
$location = "westus"
$deploymentname="contoso-routing"

# Remove the resource group if it already exists. 
#Remove-AzResourceGroup -name $RGName 
# Create the resource group.
New-AzResourceGroup -name $RGName -Location $location 

# Set a path to the parameter file. 
$parameterFile = "$HOME/template_iothub_parameters.json"
$templateFile = "$HOME/template_iothub.json"

# Deploy the template.
New-AzResourceGroupDeployment `
    -Name $deploymentname `
    -ResourceGroupName $RGName `
    -TemplateParameterFile $parameterFile `
    -TemplateFile $templateFile `
    -verbose

# Get the returning value of the connection string.
(Get-AzResourceGroupDeployment -ResourceGroupName $RGName -Name $deploymentname).Outputs.sbq_connectionString.value
```

Si se producen errores de script, puede editar el script localmente, volver a cargarlo en Cloud Shell y ejecutarlo de nuevo. Cuando el script termine de ejecutarse correctamente, continúe con el paso siguiente.

## <a name="create-simulated-device"></a>Creación de un dispositivo simulado

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

## <a name="view-message-routing-in-the-portal"></a>Visualización del enrutamiento de mensajes en el portal

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene todos los recursos configurados y las rutas de mensajes están establecidas, pase al siguiente tutorial para saber cómo procesar y mostrar la información sobre los mensajes enrutados.

> [!div class="nextstepaction"]
> [Parte 2: Visualización de los resultados del enrutamiento de mensajes](tutorial-routing-view-message-routing-results.md)
