---
title: 'Tutorial: Uso de Azure Time Series Insights para almacenar y analizar la telemetría de dispositivos Azure IoT Plug and Play'
description: 'Tutorial: Configuración de un entorno de Time Series Insights y conexión con el centro de IoT para ver y analizar los datos de telemetría de los dispositivos IoT Plug and Play.'
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.date: 10/14/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 11679cde2bcd98cbc40b5f181eb0a460cd1889c7
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108325704"
---
# <a name="tutorial-create-and-configure-a-time-series-insights-gen2-environment"></a>Tutorial: Creación y configuración de un entorno de Time Series Insights Gen2

En este tutorial, aprenderá a crear y configurar un entorno de [Azure Time Series Insights Gen2](../time-series-insights/overview-what-is-tsi.md) para integrarlo con su solución IoT Plug and Play. Con Time Series Insights, puede recopilar, procesar, almacenar, consultar y visualizar datos de serie temporal en la escala de Internet de las cosas (IoT).

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Aprovisione un entorno de Time Series Insights y conecte el centro de IoT como origen de eventos de streaming.
> * Trabaje con la sincronización del modelo para crear el [modelo de serie temporal](../time-series-insights/concepts-model-overview.md).
> * Utilice los archivos de modelo de ejemplo de [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl) que ha usado para los dispositivos del controlador de temperatura y del termostato.

> [!NOTE]
> Esta integración entre Time Series Insights e IoT Plug and Play se encuentra en versión preliminar. La forma en que los modelos de dispositivos de DTDL se asignan al modelo de serie temporal de Time Series Insights podría cambiar.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-pnp-prerequisites](../../includes/iot-pnp-prerequisites.md)]

En este punto, debe tener:

* Un centro de Azure IoT.
* Una instancia de Device Provisioning Service (DPS) vinculada al centro de IoT. La instancia de DPS debe tener una inscripción de dispositivo individual para el dispositivo IoT Plug and Play.
* Una conexión al centro de IoT desde un dispositivo de un solo componente o de varios componentes, que transmite datos simulados.

## <a name="prepare-your-event-source"></a>Preparación del origen del evento

El centro de IoT que ha creado anteriormente será el [origen del evento](../time-series-insights/concepts-streaming-ingestion-event-sources.md) del entorno de Time Series Insights.

> [!IMPORTANT]
> Deshabilite las rutas de IoT Hub existentes. Hay un problema conocido cuando se utiliza un centro de IoT con el [enrutamiento](../iot-hub/iot-hub-devguide-messages-d2c.md#routing-endpoints) configurado. Deshabilite temporalmente los puntos de conexión de enrutamiento. Cuando el centro de IoT se conecta a Time Series Insights, puede volver a habilitar los puntos de conexión de enrutamiento.

En el centro de IoT, cree un grupo de consumidores exclusivo que Time Series Insights pueda consumir. En el ejemplo siguiente, reemplace `my-pnp-hub` por el nombre del centro de IoT que ha utilizado anteriormente.

```azurecli-interactive
az iot hub consumer-group create --hub-name my-pnp-hub --name tsi-consumer-group
```

## <a name="choose-a-time-series-id"></a>Elección de un identificador de Time Series

Al aprovisionar el entorno de Time Series Insights, debe seleccionar un *identificador de serie temporal*. Es importante seleccionar el identificador de serie temporal adecuado. Esta propiedad es inmutable y no se puede cambiar una vez establecida. Un identificador de serie temporal es como una clave de partición de base de datos. El identificador de serie temporal actúa como clave principal para el modelo de serie temporal. Para más información, consulte [Best practices for choosing a Time Series ID](../time-series-insights/how-to-select-tsid.md) (Procedimientos recomendados para elegir un identificador de Time Series).

Como usuario de IoT Plug and Play, para su identificador de serie temporal, especifique una _clave compuesta_ que conste de `iothub-connection-device-id` y `dt-subject`. El centro de IoT agrega estas propiedades del sistema, que contienen el identificador del dispositivo IoT Plug and Play y los nombres de componentes del dispositivo, respectivamente.

Aunque los modelos del dispositivo IoT Plug and Play no usan actualmente componentes, debe incluir `dt-subject` como parte de una clave compuesta para poder utilizar los componentes en el futuro. Dado que el identificador de serie temporal es inmutable, Microsoft recomienda habilitar esta opción como previsión en caso de que la necesite en el futuro.

> [!NOTE]
> Los ejemplos de este artículo son para el dispositivo `TemperatureController` de varios componentes. Sin embargo, los conceptos son los mismos para el dispositivo `Thermostat` sin componentes.

## <a name="provision-your-time-series-insights-environment"></a>Aprovisionamiento del entorno de Time Series Insights

En esta sección se describe cómo aprovisionar el entorno de Azure Time Series Insights Gen2.

Ejecute el siguiente comando para:

* Crear una cuenta de Azure Storage para el [almacenamiento en reposo](../time-series-insights/concepts-storage.md#cold-store) del entorno. Esta cuenta está diseñada para la retención y el análisis a largo plazo de datos históricos.
  * En el código, reemplace `mytsicoldstore` por un nombre único para la cuenta de almacenamiento en frío.
* Crear un entorno de Azure Time Series Insights Gen2. El entorno se creará con un almacenamiento intermedio que tiene un período de retención de siete días. Se adjuntará la cuenta de almacenamiento en frío para una retención infinita.
  * En el código, reemplace `my-tsi-env` por un nombre único para el entorno de Time Series Insights.
  * En el código, reemplace `my-pnp-resourcegroup` por el nombre del grupo de recursos que ha utilizado durante la configuración.
  * La propiedad del identificador de serie temporal es `iothub-connection-device-id, dt-subject`.

```azurecli-interactive
storage=mytsicoldstore
rg=my-pnp-resourcegroup
az storage account create -g $rg -n $storage --https-only
key=$(az storage account keys list -g $rg -n $storage --query [0].value --output tsv)
az tsi environment gen2 create --name "my-tsi-env" --location eastus2 --resource-group $rg --sku name="L1" capacity=1 --time-series-id-properties name=iothub-connection-device-id type=String --time-series-id-properties name=dt-subject type=String --warm-store-configuration data-retention=P7D --storage-configuration account-name=$storage management-key=$key
```

Conecte el origen del evento de IoT Hub. Reemplace `my-pnp-resourcegroup`, `my-pnp-hub` y `my-tsi-env` por los valores que eligió. El comando siguiente hace referencia al grupo de consumidores de Time Series Insights que ha creado anteriormente:

```azurecli-interactive
rg=my-pnp-resourcegroup
iothub=my-pnp-hub
env=my-tsi-env
es_resource_id=$(az iot hub create -g $rg -n $iothub --query id --output tsv)
shared_access_key=$(az iot hub policy list -g $rg --hub-name $iothub --query "[?keyName=='service'].primaryKey" --output tsv)
az tsi event-source iothub create --event-source-name iot-hub-event-source --environment-name $env --resource-group $rg --location eastus2 --consumer-group-name tsi-consumer-group --key-name iothubowner --shared-access-key $shared_access_key --event-source-resource-id $es_resource_id --iot-hub-name $iothub
```

En [Azure Portal](https://portal.azure.com), vaya al grupo de recursos y seleccione el nuevo entorno de Time Series Insights. Vaya a la **dirección URL del Explorador de Time Series Insights** que se muestra en la información general de la instancia:

![Captura de pantalla de información general del portal.](./media/tutorial-configure-tsi/view-environment.png)

En el Explorador, verá tres instancias:

* &lt;la identidad del dispositivo PnP&gt;, thermostat1
* &lt;la identidad del dispositivo PnP&gt;, thermostat2
* &lt;la identidad del dispositivo PnP&gt;, `null`

> [!NOTE]
> La tercera etiqueta representa la telemetría del propio dispositivo `TemperatureController`, como el espacio de trabajo de la memoria del dispositivo. Dado que se trata de una propiedad de nivel superior, el valor del nombre del componente es null. En un paso posterior, hará que este nombre sea más descriptivo.

![Captura de pantalla que muestra tres instancias en el Explorador.](./media/tutorial-configure-tsi/tsi-env-first-view.png)

## <a name="configure-model-translation"></a>Configuración de la conversión del modelo

A continuación, convertirá el modelo de dispositivo de DTDL en el modelo de recursos de Azure Time Series Insights. En Time Series Insights, el modelo de serie temporal es una herramienta de modelado semántico para la contextualización de datos. El modelo consta de tres componentes principales:

* Las [instancias del modelo de serie temporal](../time-series-insights/concepts-model-overview.md#time-series-model-instances) son representaciones virtuales de las propias series temporales. Las instancias se identifican de forma única mediante el identificador de serie temporal.
* Las [jerarquías del modelo de serie temporal](../time-series-insights/concepts-model-overview.md#time-series-model-hierarchies) organizan instancias al especificar los nombres de las propiedades y sus relaciones.
* Los [tipos del modelo de serie temporal](../time-series-insights/concepts-model-overview.md#time-series-model-types) le permiten definir [variables](../time-series-insights/concepts-variables.md) o fórmulas para realizar cálculos. Los tipos se asocian con una instancia concreta.

### <a name="define-your-types"></a>Definición de los tipos

Puede empezar a ingerir datos en Azure Time Series Insights Gen2 sin tener un modelo predefinido. Cuando llegan los datos de telemetría, Time Series Insights intenta resolver automáticamente las instancias de serie temporal en función de los valores de propiedad del identificador de serie temporal. Todas las instancias tienen asignado el *tipo predeterminado*. Debe crear manualmente un nuevo tipo para clasificarlas correctamente.

Los detalles siguientes describen el método más sencillo para sincronizar los modelos de DTDL de dispositivo con los tipos de modelo de serie temporal:

* El identificador de modelo del gemelo digital se convierte en el identificador de tipo.
* El nombre de tipo puede ser el nombre del modelo o el nombre para mostrar.
* La descripción del modelo se convierte en la descripción del tipo.
* Se crea al menos una variable de tipo para cada telemetría que tenga un esquema numérico.
  * Solo se pueden usar tipos de datos numéricos con las variables, pero si un valor se envía como otro tipo que se puede convertir (por ejemplo, `"0"`), se puede usar una función de [conversión](/rest/api/time-series-insights/reference-time-series-expression-syntax#conversion-functions), como `toDouble`.
* El nombre de la variable puede ser el nombre de la telemetría o el nombre para mostrar.
* Cuando defina la variable de la expresión de serie temporal, consulte el nombre de la telemetría en la conexión y el tipo de datos de la telemetría.

| JSON de DTDL | JSON de tipo de modelo de serie temporal | Valor de ejemplo |
|-----------|------------------|-------------|
| `@id` | `id` | `dtmi:com:example:TemperatureController;1` |
| `displayName`    | `name`   |   `Temperature Controller`  |
| `description`  |  `description`  |  `Device with two thermostats and remote reboot.` |
|`contents` (matriz)| `variables` (objeto)  | Consulte el ejemplo siguiente.

![Captura de pantalla que muestra el tipo de modelo de serie temporal de DTDL.](./media/tutorial-configure-tsi/DTDL-to-TSM-Type.png)

> [!NOTE]
> En este ejemplo se muestran tres variables, pero cada tipo puede tener hasta 100 variables. Distintas variables pueden hacer referencia al mismo valor de telemetría para realizar diferentes cálculos, según sea necesario. Para obtener la lista completa de filtros, agregados y funciones escalares, consulte [Sintaxis de expresiones de series temporales de Time Series Insights Gen2](/rest/api/time-series-insights/reference-time-series-expression-syntax).

Abra un editor de texto y guarde el siguiente código JSON en la unidad local.

```JSON
{
  "put": [
    {
      "id": "dtmi:com:example:TemperatureController;1",
      "name": "Temperature Controller",
      "description": "Device with two thermostats and remote reboot.",
      "variables": {
        "workingSet": {
          "kind": "numeric",
          "value": {
            "tsx": "coalesce($event.workingSet.Long, toLong($event.workingSet.Double))"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        },
        "temperature": {
          "kind": "numeric",
          "value": {
            "tsx": "coalesce($event.temperature.Long, toLong($event.temperature.Double))"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        },
        "eventCount": {
          "kind": "aggregate",
          "aggregation": {
            "tsx": "count()"
          }
        }
      }
    }
  ]
}
```

En el Explorador de Time Series Insights, seleccione el icono de modelo de la izquierda para abrir la pestaña **Modelo**. Seleccione **Tipos** y, a continuación, seleccione **Cargar JSON**:

![Captura de pantalla en que se muestra cómo cargar JSON.](./media/tutorial-configure-tsi/upload-type.png)

Seleccione **Elegir archivo**, después el archivo JSON que ha guardado anteriormente y, por último, **Cargar**.

Verá el tipo **Temperature Controller** que se acaba de definir.

### <a name="create-a-hierarchy"></a>Creación de una jerarquía

Cree una jerarquía para organizar las etiquetas bajo su dispositivo `TemperatureController` primario. Este sencillo ejemplo que se incluye a continuación tiene un solo nivel, pero las soluciones de IoT suelen tener muchos niveles de anidamiento, para contextualizar las etiquetas en su posición física y semántica dentro de una organización.

Seleccione **Jerarquías** y **Agregar jerarquía**. En nombre, escriba *Device Fleet* (Flota de dispositivos). Cree un nivel llamado *Device Name* (Nombre del dispositivo). Después, seleccione **Guardar**.

![Captura de pantalla en la que se muestra cómo agregar una jerarquía.](./media/tutorial-configure-tsi/add-hierarchy.png)

### <a name="assign-your-instances-to-the-correct-type"></a>Asignación del tipo correcto a las instancias

A continuación, debe cambiar el tipo de las instancias y colocarlas en la jerarquía.

Seleccione la pestaña **Instancias**. Busque la instancia que representa el espacio de trabajo del dispositivo y seleccione el icono **Editar** en el extremo derecho.

![Captura de pantalla en la que se muestra cómo editar una instancia.](./media/tutorial-configure-tsi/edit-instance.png)

Abra el menú desplegable **Tipo** y seleccione **Temperature Controller** (Controlador de temperatura). Escriba *defaultComponent, <your device name>* para actualizar el nombre de la instancia que representa todas las etiquetas de nivel superior asociadas con el dispositivo.

![Captura de pantalla que muestra cómo cambiar un tipo de instancia.](./media/tutorial-configure-tsi/change-type.png)

Antes de seleccionar **Guardar**, seleccione la pestaña **Campos de instancia** y active la casilla **Device Fleet** (Flota de dispositivos). Para agrupar la telemetría, escriba *\<your device name> - Temp Controller* (Controlador de temperatura). Después, seleccione **Guardar**.

![Captura de pantalla que muestra cómo asignar una instancia a una jerarquía](./media/tutorial-configure-tsi/assign-to-hierarchy.png)

Repita los pasos anteriores para asignar a las etiquetas del termostato el tipo y la jerarquía correctos.

## <a name="view-your-data"></a>Consulta de los datos

Vuelva al panel de gráficos y expanda **Device Fleet (Flota de dispositivos)** > su dispositivo. Seleccione **thermostat1**, seleccione la variable **Temperatura** y, a continuación, seleccione **Agregar** para representar el valor en el gráfico. Haga lo mismo para **thermostat2** y el valor **defaultComponent** **workingSet**.

![Captura de pantalla que muestra cómo cambiar el tipo de instancia para thermostat2.](./media/tutorial-configure-tsi/charting-values.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-pnp-clean-resources](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> Para más información sobre las distintas opciones de gráficos, incluidos el ajuste de tamaño del intervalo y los controles del eje Y, consulte [Explorador de Azure Time Series Insights](../time-series-insights/concepts-ux-panels.md).
