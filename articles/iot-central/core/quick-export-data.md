---
title: 'Inicio rápido: Exportación de datos desde Azure IoT Central'
description: 'Inicio rápido: Aprenda a usar la característica de exportación de datos de IoT Central para la integración con otros servicios en la nube.'
author: dominicbetts
ms.author: dobett
ms.date: 05/27/2021
ms.topic: quickstart
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: 06e387e4742d86f4176d262ae2e2e9db863386ca
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112300987"
---
# <a name="quickstart-export-data-from-an-iot-central-application"></a>Inicio rápido: Exportación de datos desde una aplicación de IoT Central

En este inicio rápido, se muestra cómo exportar continuamente datos desde la aplicación de Azure IoT Central a otro servicio en la nube. Para configurarlo rápidamente, este inicio rápido usa [Azure Data Explorer](/azure/data-explorer/data-explorer-overview) para almacenar, consultar y procesar los datos de telemetría procedentes de la aplicación de smartphone **IoT Plug and Play**.

En esta guía de inicio rápido:

- Use la característica de exportación de datos de IoT Central para exportar los datos de telemetría enviados por la aplicación de smartphone a una cola de Azure Event Hubs.
- Ingiera los datos de telemetría de la cola de Event Hubs en una base de datos de Azure Data Explorer.
- Use Azure Data Explorer para ejecutar consultas en los datos de telemetría.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar, debe completar el primer [Inicio rápido: Creación de una aplicación de Azure IoT Central y uso de un smartphone para enviar datos de telemetría](./quick-deploy-iot-central.md). El segundo inicio rápido, [Configuración de reglas y acciones para el dispositivo en Azure IoT Central](quick-configure-rules.md), es opcional.

## <a name="install-azure-services"></a>Instalación de los servicios de Azure

Para poder exportar datos desde la aplicación de IoT Central, debe crear una cola de Event Hubs y una base de datos de Azure Data Explorer.

Seleccione el botón siguiente para implementar los servicios:

[![Botón Implementar en Azure para la exportación continua de datos](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fiot-central-docs-samples%2Fmaster%2Fquickstart-cde%2Fdeploy-azure-services.json)

En el formulario **Implementación personalizada**:

- Cree un nuevo grupo de recursos llamado *central-quickstart*.
- Seleccione la región más cercana.
- Escriba un nombre único de Azure Data Explorer solo con letras minúsculas y números. Por ejemplo, *contosocentraladx*.
- Escriba un nombre de espacio de nombres de Azure Event Hubs único solo con letras minúsculas y números. Por ejemplo, *contosocentraleh*.

Seleccione **Revisar y crear** y, luego, **Crear**.

> [!TIP]
> Calcule que la implementación tardará unos 10 minutos.

Una vez completada la implementación, anote la cadena de conexión que devuelve para Azure Event Hubs:

1. Espere a que la implementación se complete.
1. Seleccione **Salidas**.
1. Anote el valor de **eventHubConnectionString**, lo usará más adelante:

    :::image type="content" source="media/quick-export-data/connection-string.png" alt-text="Cadena de conexión de Event Hubs.":::

## <a name="configure-data-export"></a>Configuración de la exportación de datos

En este escenario, se usa una cola de Azure Event Hubs para entregar los datos de telemetría recopilados por la aplicación de IoT Central a Azure Data Explorer.

Para configurar la exportación de datos desde IoT Central:

1. Vaya a la página **Exportación de datos** de la aplicación de IoT Central.
1. Seleccione la pestaña **Destinos** y, a continuación, **Agregar un destino**.
1. Escriba *Event Hub 1* (Centro de eventos 1) como nombre de destino. Elija **Azure Event Hubs** como tipo de destino.
1. Escriba la cadena de conexión de la cola de Event Hubs que guardó en la sección anterior. El campo **Centro de eventos** se rellena automáticamente con *centraltelemetry*.
1. Seleccione **Guardar**.
1. En la página **Exportación de datos**, seleccione la pestaña **Exportaciones** y, a continuación, **Agregar una exportación**.
1. Escriba *Telemetry Export* (Exportación de datos de telemetría) como nombre de la exportación.
1. Seleccione **Telemetría** como el tipo de datos que se van a exportar.
1. Agregue el centro de eventos **Event Hub 1** como destino.
1. Seleccione **Guardar**.

Cuando se ejecuta la exportación, el estado de la página **Exportación de datos** es **Correcto**:

:::image type="content" source="media/quick-export-data/healthy-export.png" alt-text="Captura de pantalla que muestra una exportación de datos en ejecución con el estado correcto.":::

## <a name="configure-data-ingestion"></a>Configuración de la ingesta de datos

Asegúrese de que la aplicación del teléfono esté conectada a la aplicación de IoT Central y enviando datos antes de continuar.

La aplicación de IoT Central exporta datos de telemetría continuamente a la cola de Event Hubs. En esta sección, configurará el clúster de Azure Data Explorer para ingerir continuamente los datos de telemetría en una tabla donde pueda consultarlos.

Para configurar la ingesta de datos:

1. En Azure Portal, vaya al clúster de Azure Data Explorer en el grupo de recursos **central-quickstart**:

    :::image type="content" source="media/quick-export-data/azure-data-explorer-portal.png" alt-text="Captura de pantalla de la página de información general de Azure Data Explorer.":::

1. Seleccione **Ingest new data** (Ingerir nuevos datos).
1. En la página **Ingest new data** (Ingerir nuevos datos), seleccione el clúster y la base de datos **iotcentraldata**.
1. Seleccione **Create new** (Crear nueva) para crear una tabla llamada *telemetry* (telemetría).
1. Seleccione **Event Hubs** como tipo de origen.
1. Escriba *IoT-Central-connection* como nombre de la conexión de datos.
1. Use la información de la tabla siguiente para rellenar el resto del formulario:

    | Campo                   | Value                            |
    |-------------------------|----------------------------------|
    | Subscription            | Selección de su suscripción a Azure   |
    | Espacio de nombres del Centro de eventos     | Seleccione el espacio de nombres de Event Hubs. |
    | Centro de eventos               | `centraltelemetry`               |
    | Grupo de consumidores          | `$Default`                       |
    | Formato de datos             | JSON                             |
    | Compresión             | None                             |
    | Propiedades del sistema de eventos | Déjelo en blanco                      |

1. Seleccione **Edit schema** (Editar esquema).
1. La página **Schema** (Esquema) muestra una vista previa de los datos de los mensajes de la cola de Event Hubs.
1. Cambie el valor de los niveles anidados a `3` para expandir el código JSON y mostrar cada valor de telemetría en su propia columna.
1. Seleccione **Start ingestion** (Iniciar ingesta). Espere hasta que se complete la ingesta de datos:

:::image type="content" source="media/quick-export-data/data-ingestion-complete.png" alt-text="Captura de pantalla que muestra la ingesta de datos completada en Azure Data Explorer.":::

Deje abierta esta página, la usará en la sección siguiente.

## <a name="query-exported-data"></a>Consulta de los datos exportados

El clúster de Azure Data Explorer ahora ingiere datos continuamente desde la aplicación de IoT Central. Para consultar los datos:

1. En la página de Azure Data Explorer de la sección anterior, seleccione la consulta rápida **Take 10**. Esta consulta selecciona diez registros de la tabla **telemetry** (telemetría).
1. Reemplace la consulta por la siguiente:

    ```kusto
    ['telemetry'] 
    | where isnotnull(telemetry_magnetometer_x)
    | project Time=todatetime(enqueuedTime), deviceId, telemetry_magnetometer_x, telemetry_magnetometer_y, telemetry_magnetometer_z
    | render timechart 
    ```

    Esta consulta traza los valores de telemetría del magnetómetro de la aplicación del teléfono en una escala de tiempo.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

Para quitar la instancia de Azure Data Explorer y el espacio de nombres de Event Hubs de la suscripción y evitar que se facturen innecesariamente, elimine el grupo de recursos **central-quickstarts** desde [Azure Portal](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceGroups).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a exportar datos continuamente desde IoT Central a otro servicio de Azure.

Ahora que sabe cómo exportar los datos, el siguiente paso que se recomienda es:

> [!div class="nextstepaction"]
> [Cree y administre una plantilla de dispositivo](howto-set-up-template.md).
