---
title: Integración con Azure Time Series Insights
titleSuffix: Azure Digital Twins
description: Vea cómo configurar rutas de eventos de Azure Digital Twins a Azure Time Series Insights.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 4/7/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 8f87b373d640a330006b6b3675376ce46c1c02ae
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110078767"
---
# <a name="integrate-azure-digital-twins-with-azure-time-series-insights"></a>Integración de Azure Digital Twins con Azure Time Series Insights

En este artículo, aprenderá a integrar Azure Digital Twins con [Azure Time Series Insights (TSI)](../time-series-insights/overview-what-is-tsi.md).

La solución que se describe en este artículo le permitirá recopilar y analizar datos históricos sobre la solución de IoT. Azure Digital Twins es una buena opción para alimentar de datos a Time Series Insights, ya que permite poner en correlación varios flujos de datos y estandarizar la información antes de enviarla a Time Series Insights.

## <a name="prerequisites"></a>Requisitos previos

Para poder configurar una relación con Time Series Insights, deberá configurar los siguientes recursos:
* Un **centro de IoT**. Para instrucciones, consulte la sección [Creación de un IoT Hub](../iot-hub/quickstart-send-telemetry-cli.md#create-an-iot-hub) de la guía de inicio rápido *Enviar telemetría desde un dispositivo a IoT Hub y supervisarlo con la CLI de Azure*.
* Una **instancia de Azure Digital Twins**. Para instrucciones, consulte [Procedimiento: Configuración de una instancia de Azure Digital Twins y autenticación](./how-to-set-up-instance-portal.md).
* Un **modelo y un gemelo en la instancia de Azure Digital Twins**. Deberá actualizar la información del gemelo varias veces para ver que se realiza un seguimiento de los datos en Time Series Insights. Para obtener instrucciones, consulte la sección [Adición de un modelo y un gemelo](how-to-ingest-iot-hub-data.md#add-a-model-and-twin) del artículo *Ingesta de telemetría de IoT Hub en Azure Digital Twins*.

> [!TIP]
> En este artículo, los valores de los gemelos digitales variables que se ven en Time Series Insights se actualizan manualmente por motivos de simplicidad. Sin embargo, si quiere completar este artículo con datos simulados en directo, puede configurar una función de Azure que actualice los gemelos digitales en función de los eventos de telemetría de IoT desde un dispositivo simulado. Para obtener instrucciones, consulte [Ingesta de telemetría de IoT Hub en Azure Digital Twins](how-to-ingest-iot-hub-data.md), incluidos los pasos finales para ejecutar el simulador de dispositivos y validar que el flujo de datos funciona.
>
> Más adelante, busque otra SUGERENCIA para mostrar dónde empezar a ejecutar el simulador de dispositivos y hacer que las funciones de Azure actualicen los gemelos automáticamente, en lugar de enviar comandos de actualización manual de gemelos digitales.


## <a name="solution-architecture"></a>Arquitectura de la solución

Va a adjuntar Time Series Insights a Azure Digital Twins a través de la ruta de acceso siguiente.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-integrate-time-series-insights/diagram-simple.png" alt-text="Diagrama de los servicios de Azure en un escenario integral, con Time Series Insights resaltado" lightbox="media/how-to-integrate-time-series-insights/diagram-simple.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

## <a name="create-event-hub-namespace"></a>Creación de un espacio de nombres del centro de eventos

Antes de empezar a crear los centros de eventos, cree un espacio de nombres del centro de eventos que recibirá eventos de la instancia de Azure Digital Twins. Puede usar las instrucciones de la CLI de Azure que aparecen a continuación o usar Azure Portal: [Guía de inicio rápido: Creación de un centro de eventos mediante Azure Portal](../event-hubs/event-hubs-create.md). Para ver qué regiones admiten Event Hubs, visite [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=event-hubs).

```azurecli-interactive
az eventhubs namespace create --name <name-for-your-event-hubs-namespace> --resource-group <your-resource-group> --location <region>
```

> [!TIP]
> Si recibe un error que indica `BadRequest: The specified service namespace is invalid.`, asegúrese de que el nombre que ha elegido para el espacio de nombres cumple los requisitos de nomenclatura descritos en este documento de referencia: [Crear espacio de nombres](/rest/api/servicebus/create-namespace).

Usará este espacio de nombres de centros de eventos para hospedar los dos centros de eventos necesarios para este artículo:

  1. **Centro de gemelos**: centro de eventos para recibir eventos de cambio de gemelos
  2. **Centro de serie temporal**: centro de eventos para transmitir eventos a Time Series Insights

Las secciones siguientes le guiarán en la creación y configuración de estos centros en el espacio de nombres del centro de eventos.

## <a name="create-twins-hub"></a>Creación del centro de gemelos

El primer centro de eventos que creará en este artículo es el **centro de gemelos**. Este centro de eventos recibirá eventos de cambios de gemelos de Azure Digital Twins.
Para configurar el centro de gemelos, completará los pasos siguientes en esta sección:

1. Creación del centro de gemelos
2. Creación de una regla de autorización para controlar los permisos en el centro
3. Creación de un punto de conexión en Azure Digital Twins que use la regla de autorización para acceder al centro
4. Creación de una ruta en Azure Digital Twins que envía el evento de actualizaciones de gemelos al punto de conexión y al centro de gemelos conectado
5. Obtención de la cadena de conexión del centro de gemelos

Cree el **centro de gemelos** con el siguiente comando de la CLI. Especifique un nombre para el centro de gemelos.

```azurecli-interactive
az eventhubs eventhub create --name <name-for-your-twins-hub> --resource-group <your-resource-group> --namespace-name <your-event-hubs-namespace-from-above>
```

### <a name="create-twins-hub-authorization-rule"></a>Creación de una regla de autorización del centro de gemelos

Cree una [regla de autorización](/cli/azure/eventhubs/eventhub/authorization-rule?view=azure-cli-latest&preserve-view=true#az_eventhubs_eventhub_authorization_rule_create) con permisos de envío y recepción. Especifique un nombre para la regla.

```azurecli-interactive
az eventhubs eventhub authorization-rule create --rights Listen Send --name <name-for-your-twins-hub-auth-rule> --resource-group <your-resource-group> --namespace-name <your-event-hubs-namespace-from-earlier> --eventhub-name <your-twins-hub-from-above>
```

### <a name="create-twins-hub-endpoint"></a>Creación de un punto de conexión del centro de gemelos

Cree un [punto de conexión](concepts-route-events.md#create-an-endpoint) de Azure Digital Twins que vincule el centro de eventos con la instancia de Azure Digital Twins. Especifique un nombre para el punto de conexión del centro de gemelos.

```azurecli-interactive
az dt endpoint create eventhub --dt-name <your-Azure-Digital-Twins-instance-name> --eventhub-resource-group <your-resource-group> --eventhub-namespace <your-event-hubs-namespace-from-earlier> --eventhub <your-twins-hub-name-from-above> --eventhub-policy <your-twins-hub-auth-rule-from-earlier> --endpoint-name <name-for-your-twins-hub-endpoint>
```

### <a name="create-twins-hub-event-route"></a>Creación de la ruta de eventos del centro de gemelos

Las instancias de Azure Digital Twins pueden emitir [eventos de actualización de gemelos](./concepts-event-notifications.md) cada vez que se actualiza el estado de uno de estos elementos. En esta sección, creará una **ruta de eventos** de Azure Digital Twins que dirigirá estos eventos de actualización al centro de gemelos para su posterior procesamiento.

Cree una [ruta](concepts-route-events.md#create-an-event-route) en Azure Digital Twins para enviar eventos de actualización de gemelos al punto de conexión anterior. El filtro de esta ruta permitirá que solo los mensajes de actualización de gemelos pasen al punto de conexión. Especifique un nombre para la ruta de eventos del centro de gemelos.

```azurecli-interactive
az dt route create --dt-name <your-Azure-Digital-Twins-instance-name> --endpoint-name <your-twins-hub-endpoint-from-above> --route-name <name-for-your-twins-hub-event-route> --filter "type = 'Microsoft.DigitalTwins.Twin.Update'"
```

### <a name="get-twins-hub-connection-string"></a>Obtención de la cadena de conexión del centro de gemelos

Obtenga la [cadena de conexión del centro de eventos de gemelos](../event-hubs/event-hubs-get-connection-string.md) con las reglas de autorización que creó anteriormente para el centro de gemelos.

```azurecli-interactive
az eventhubs eventhub authorization-rule keys list --resource-group <your-resource-group> --namespace-name <your-event-hubs-namespace-from-earlier> --eventhub-name <your-twins-hub-from-above> --name <your-twins-hub-auth-rule-from-earlier>
```
Tome nota del valor **primaryConnectionString** del resultado para configurar la aplicación del centro de gemelos más adelante en este artículo.

## <a name="create-time-series-hub"></a>Creación de un centro de serie temporal

El segundo centro de eventos que creará en este artículo es el **centro de serie temporal**. Se trata de un centro de eventos que transmitirá los eventos de Azure Digital Twins a Time Series Insights.
Para configurar el centro de serie temporal, complete estos pasos:

1. Creación de un centro de serie temporal
2. Creación de una regla de autorización para controlar los permisos en el centro
3. Obtención de la cadena de conexión del centro de serie temporal

Más adelante, al crear la instancia de Time Series Insights, conectará este centro de serie temporal como origen de eventos para la instancia de Time Series Insights.

Cree el **centro de serie temporal** con el siguiente comando. Especifique un nombre para el centro de serie temporal.

```azurecli-interactive
 az eventhubs eventhub create --name <name-for-your-time-series-hub> --resource-group <your-resource-group> --namespace-name <your-event-hub-namespace-from-earlier>
```

### <a name="create-time-series-hub-authorization-rule"></a>Creación de una regla de autorización del centro de serie temporal

Cree una [regla de autorización](/cli/azure/eventhubs/eventhub/authorization-rule?view=azure-cli-latest&preserve-view=true#az_eventhubs_eventhub_authorization_rule_create) con permisos de envío y recepción. Especifique un nombre para la regla de autorización del centro de serie temporal.

```azurecli-interactive
az eventhubs eventhub authorization-rule create --rights Listen Send --name <name-for-your-time-series-hub-auth-rule> --resource-group <your-resource-group> --namespace-name <your-event-hub-namespace-from-earlier> --eventhub-name <your-time-series-hub-name-from-above>
```

### <a name="get-time-series-hub-connection-string"></a>Obtención de la cadena de conexión del centro de serie temporal

Obtenga la [cadena de conexión del centro de serie temporal](../event-hubs/event-hubs-get-connection-string.md) con las reglas de autorización que creó anteriormente para el centro de serie temporal:

```azurecli-interactive
az eventhubs eventhub authorization-rule keys list --resource-group <your-resource-group> --namespace-name <your-event-hub-namespace-from-earlier> --eventhub-name <your-time-series-hub-name-from-earlier> --name <your-time-series-hub-auth-rule-from-earlier>
```
Tome nota del valor **primaryConnectionString** del resultado para configurar la aplicación del centro de serie temporal más adelante en este artículo.

Además, tome nota de los siguientes valores para usarlos más adelante para crear una instancia de Time Series Insights.
* Espacio de nombres del centro de eventos
* Nombre del centro de serie temporal
* Regla de autorización del centro de serie temporal

## <a name="create-a-function"></a>Creación de una función

En esta sección, creará una función de Azure que convertirá los eventos de actualización de gemelos de su forma original como documentos de revisión JSON a objetos JSON, que contienen solo valores actualizados y agregados de sus gemelos.

### <a name="step-1-create-function-app"></a>Paso 1: Creación de una aplicación de funciones

En primer lugar, cree un proyecto de aplicación de funciones en Visual Studio. Para instrucciones sobre cómo hacerlo, consulte la sección [Creación de una aplicación de funciones en Visual Studio](how-to-create-azure-function.md#create-a-function-app-in-visual-studio) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

### <a name="step-2-add-a-new-function"></a>Paso 2: Adición de una nueva función

Cree una función de Azure denominada *ProcessDTUpdatetoTSI.cs* para actualizar los eventos de telemetría del dispositivo a Time Series Insights. El tipo de función será **Desencadenador del centro de eventos**.

:::image type="content" source="media/how-to-integrate-time-series-insights/create-event-hub-trigger-function.png" alt-text="Captura de pantalla de Visual Studio para crear una función de Azure de tipo desencadenador del centro de eventos":::

### <a name="step-3-fill-in-function-code"></a>Paso 3: Relleno del código de función

Agregue los siguientes paquetes al proyecto:
* [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs/)
* [Microsoft.Azure.WebJobs.Extensions.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs/)
* [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/)

Reemplace el código del archivo *ProcessDTUpdatetoTSI.cs* por el siguiente:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/updateTSI.cs":::

Guarde el código de función.

### <a name="step-4-publish-the-function-app-to-azure"></a>Paso 4: Publicación de la aplicación de funciones en Azure

Publique el proyecto con la función *ProcessDTUpdatetoTSI.cs* en una aplicación de funciones en Azure.

Para instrucciones sobre cómo hacerlo, consulte la sección [Publicación de la aplicación de funciones en Azure](how-to-create-azure-function.md#publish-the-function-app-to-azure) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

Guarde el nombre de la aplicación de funciones para usarlo más adelante para configurar los valores de la aplicación para los dos centros de eventos.

### <a name="step-5-security-access-for-the-function-app"></a>Paso 5: Acceso de seguridad para la aplicación de funciones

A continuación, **asigne un rol de acceso** para la función y **configure las opciones de la aplicación** para que pueda acceder a la instancia de Azure Digital Twins. Para instrucciones sobre cómo hacerlo, consulte la sección [Configuración del acceso de seguridad para la aplicación de funciones](how-to-create-azure-function.md#set-up-security-access-for-the-function-app) del artículo *Configuración de aplicaciones de funciones de Azure para procesar datos*.

### <a name="step-6-configure-app-settings-for-the-two-event-hubs"></a>Paso 6: Configuración de la aplicación para los dos centros de eventos

A continuación, agregará variables de entorno en la configuración de la aplicación de funciones que le permitan acceder al centro de gemelos y al centro de serie temporal.

Use el valor **primaryConnectionString** del centro de gemelos que guardó anteriormente para configurar la aplicación de funciones que contiene la cadena de conexión del centro de gemelos:

```azurecli-interactive
az functionapp config appsettings set --settings "EventHubAppSetting-Twins=<your-twins-hub-primaryConnectionString>" --resource-group <your-resource-group> --name <your-App-Service-(function-app)-name>
```

Use el valor **primaryConnectionString** del centro de serie temporal que guardó anteriormente para configurar la aplicación de funciones que contiene la cadena de conexión del centro de serie temporal:

```azurecli-interactive
az functionapp config appsettings set --settings "EventHubAppSetting-TSI=<your-time-series-hub-primaryConnectionString>" --resource-group <your-resource-group> --name <your-App-Service-(function-app)-name>
```

## <a name="create-and-connect-a-time-series-insights-instance"></a>Creación y conexión de una instancia de Time Series Insights

En esta sección, configurará la instancia de Time Series Insights para recibir datos del centro de serie temporal. Para más detalles sobre este proceso, consulte [Tutorial: Configuración de un entorno de pago por uso de Azure Time Series Insights Gen2](../time-series-insights/tutorial-set-up-environment.md). Siga estos pasos para crear un entorno de Time Series Insights.

1. En [Azure Portal](https://portal.azure.com), busque *Entornos de Time Series Insights* y seleccione el botón **Agregar**. Elija las siguientes opciones para crear el entorno de serie temporal.

    * **Suscripción**: elija su suscripción.
        - **Grupo de recursos**: elija su grupo de recursos.
    * **Nombre del entorno**: especifique un nombre para el entorno de serie temporal.
    * **Ubicación**: elija una ubicación.
    * **Nivel**: elija el plan de tarifa **Gen2(L1)** .
    * **Nombre de la propiedad**: escriba **$dtId** (obtenga más información sobre cómo seleccionar un valor de id. en [Procedimientos recomendados al elegir un id. de serie temporal](../time-series-insights/how-to-select-tsid.md)).
    * **Nombre de la cuenta de almacenamiento**: especifique un nombre para la cuenta de almacenamiento.
    * **Habilitar almacenamiento intermedio**: deje este campo establecido en *Sí*.

    Puede dejar los valores predeterminados de las otras propiedades de esta página. Seleccione el botón **Siguiente: Origen del evento >** .

    :::image type="content" source="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-1.png" alt-text="Captura de pantalla de Azure Portal para crear un entorno de Time Series Insights. Seleccione la suscripción, el grupo de recursos y la ubicación en las listas desplegables correspondientes y elija un nombre para el entorno" lightbox="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-1.png":::.
        
    :::image type="content" source="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-2.png" alt-text="Captura de pantalla de Azure Portal para crear un entorno de Time Series Insights. Está seleccionado el plan de tarifa Gen2(L1) y el nombre de la propiedad de identificador de serie temporal es $dtId" lightbox="media/how-to-integrate-time-series-insights/create-time-series-insights-environment-2.png":::.

2. En la pestaña *Origen del evento*, elija los campos siguientes:

   * **Create an event source?** ¿Crear un origen del evento? elija *Sí*.
   * **Tipo de origen**: elija *Centro de eventos*.
   * **Nombre**: especifique un nombre para el origen del evento.
   * **Suscripción**: elija la suscripción de Azure.
   * **Espacio de nombres del centro de eventos**: elija el espacio de nombres que creó anteriormente en este artículo.
   * **Nombre del centro de eventos**: elija el nombre del **centro de serie temporal** que creó anteriormente en este artículo.
   * **Nombre de la directiva de acceso del centro de eventos**: elija la *regla de autorización del centro de serie temporal* que creó anteriormente en este artículo.
   * **Grupo de consumidores del Centro de eventos**: seleccione *Nuevo* y especifique un nombre para el grupo de consumidores del centro de eventos. Después, seleccione *Agregar*.
   * **Nombre de la propiedad**: deje este campo en blanco.
    
    Elija el botón **Revisar y crear** para revisar todos los detalles. A continuación, vuelva a seleccionar el botón **Revisar y crear** para crear el entorno de serie temporal.

    :::image type="content" source="media/how-to-integrate-time-series-insights/create-tsi-environment-event-source.png" alt-text="Captura de pantalla de Azure Portal para crear un entorno de Time Series Insights. Va a crear un origen de eventos con la información del centro de eventos que se indicó anteriormente. También creará un grupo de consumidores" lightbox="media/how-to-integrate-time-series-insights/create-tsi-environment-event-source.png":::.

## <a name="send-iot-data-to-azure-digital-twins"></a>Envío de datos de IoT a Azure Digital Twins

Para empezar a enviar datos a Time Series Insights, deberá iniciar la actualización de las propiedades de gemelos digitales en Azure Digital Twins con valores de datos variables.

Use el siguiente comando de la CLI para actualizar la propiedad *Temperature* del gemelo thermostat67 que agregó a la instancia en la sección [Requisitos previos](#prerequisites).

```azurecli-interactive
az dt twin update --dt-name <your-azure-digital-twins-instance-name> --twin-id thermostat67 --json-patch '{"op":"replace", "path":"/Temperature", "value": 20.5}'
```

**Repita el comando al menos cuatro veces más con valores de temperatura diferentes** para crear varios puntos de datos que se puedan observar más adelante en el entorno de Time Series Insights.

> [!TIP]
> Si quiere completar este artículo con datos simulados en directo en lugar de actualizar manualmente los valores de los gemelos digitales, primero asegúrese de que ha aplicado la SUGERENCIA de la sección [Requisitos previos](#prerequisites) para configurar una función de Azure que actualice los gemelos desde un dispositivo simulado.
Después, puede ejecutar el dispositivo ahora para empezar a enviar datos simulados y actualizar el gemelo digital con ese flujo de datos.

## <a name="visualize-your-data-in-time-series-insights"></a>Visualización de los datos en Time Series Insights

Ahora, los datos deben fluir a la instancia de Time Series Insights, listos para su análisis. Siga los pasos que se indican a continuación para explorar los datos entrantes.

1. En [Azure Portal](https://portal.azure.com), busque el nombre del entorno de serie temporal que creó anteriormente. En las opciones de menú de la izquierda, seleccione *Información general* para ver la *URL del Explorador de Time Series Insights*. Seleccione la dirección URL para ver los cambios de temperatura reflejados en el entorno de Time Series Insights.

    :::image type="content" source="media/how-to-integrate-time-series-insights/view-environment.png" alt-text="Captura de pantalla de Azure Portal para seleccionar la dirección URL del Explorador de Time Series Insights en la pestaña de información general del entorno de Time Series Insights" lightbox="media/how-to-integrate-time-series-insights/view-environment.png":::

2. En el explorador, verá los gemelos de la instancia de Azure Digital Twins que se muestra a la izquierda. Seleccione el gemelo thermostat67, elija la propiedad *Temperature* (Temperatura) y seleccione **Add** (Agregar).

    :::image type="content" source="media/how-to-integrate-time-series-insights/add-data.png" alt-text="Captura de pantalla del Explorador de Time Series Insights para seleccionar thermostat67, seleccionar la propiedad de la temperatura y seleccionar Agregar." lightbox="media/how-to-integrate-time-series-insights/add-data.png":::

3. Ahora debería ver las lecturas de temperatura iniciales del termostato, como se muestra a continuación. 

    :::image type="content" source="media/how-to-integrate-time-series-insights/initial-data.png" alt-text="Captura de pantalla del Explorador de TSI para ver los datos de temperatura iniciales. Es una línea de valores aleatorios entre 68 y 85" lightbox="media/how-to-integrate-time-series-insights/initial-data.png":::.

Si permite que una simulación se ejecute durante mucho más tiempo, la visualización tendrá un aspecto similar al siguiente:

:::image type="content" source="media/how-to-integrate-time-series-insights/day-data.png" alt-text="Captura de pantalla del Explorador de TSI donde los datos de temperatura de cada gemelo se representan gráficamente en tres líneas paralelas de colores diferentes" lightbox="media/how-to-integrate-time-series-insights/day-data.png":::.

## <a name="next-steps"></a>Pasos siguientes

De forma predeterminada, los gemelos digitales se almacenan como una jerarquía plana en Time Series Insights, pero se pueden enriquecer con la información del modelo y una jerarquía de varios niveles para la organización. Para más información sobre este proceso, lea: 

* [Tutorial: Definir y aplicar un modelo](../time-series-insights/tutorial-set-up-environment.md#define-and-apply-a-model) 

Puede escribir la lógica personalizada para proporcionar automáticamente esta información con los datos del modelo y del gráfico ya almacenados en Azure Digital Twins. Para obtener más información sobre cómo administrar, actualizar y recuperar información del grafo de gemelos, vea las referencias siguientes:

* [Procedimiento: Administración de un gemelo digital](./how-to-manage-twin.md)
* [Procedimiento: Consulta del grafo de gemelos](./how-to-query-graph.md)