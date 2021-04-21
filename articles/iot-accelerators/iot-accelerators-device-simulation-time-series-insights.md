---
title: 'Visualización de datos de telemetría simulados con Time Series Insights: Azure | Microsoft Docs'
description: Obtenga información sobre cómo configurar el entorno de Time Series Insights para explorar y analizar los datos de telemetría que genere el acelerador de soluciones de simulación de dispositivos.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.date: 08/20/2018
ms.topic: conceptual
ms.service: iot-accelerators
services: iot-accelerators
ms.openlocfilehash: 90a4b808daccc76e8cc9125973c69b13e8086fbf
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107713973"
---
# <a name="use-time-series-insights-to-visualize-telemetry-sent-from-the-device-simulation-solution-accelerator"></a>Use Time Series Insights para visualizar los datos de telemetría que se envíen desde el acelerador de soluciones de simulación de dispositivos

El acelerador de soluciones de simulación de dispositivos le permite generar datos de telemetría desde dispositivos simulados para probar sus soluciones de IoT. En esta guía se muestra cómo visualizar y analizar los datos de telemetría simulados mediante un entorno de Time Series Insights.

## <a name="prerequisites"></a>Prerequisites

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

En los pasos descritos en esta guía práctica se supone que ya ha implementado el acelerador de soluciones de simulación de dispositivos en su suscripción de Azure. Si aún no ha implementado Simulación de dispositivo, vea [Implementación de Simulación de dispositivo](https://github.com/Azure/azure-iot-pcs-device-simulation/blob/master/README.md) en GitHub.

En este artículo se da por supuesto que el nombre del acelerador de soluciones es **contoso-simulation**. Reemplace **contoso-simulation** con el nombre de su acelerador de soluciones a medida que completa los pasos siguientes.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-consumer-group"></a>Creación de un grupo de consumidores

Debe crear un grupo de consumidores dedicado en IoT Hub para usarlo en el streaming de datos de telemetría a Time Series Insights. Un origen del evento de Time Series Insights debe tener el uso exclusivo de un grupo de consumidores de IoT Hub.

Los pasos siguientes utilizan la CLI de Azure en Azure Cloud Shell para crear el grupo de consumidores:

1. IoT Hub es uno de varios recursos que creó cuando implementó el acelerador de soluciones de simulación de dispositivos. Ejecute el siguiente comando para buscar el nombre de su instancia de IoT Hub; recuerde que debe usar el nombre de su acelerador de soluciones:

    ```azurecli-interactive
    az resource list --resource-group contoso-simulation -o table
    ```

    IoT Hub es el recurso de tipo **Microsoft.Devices/IotHubs**.

1. Agregue un grupo de consumidores llamado **devicesimulationtsi** al centro. En el siguiente comando se usa el nombre de su acelerador de soluciones y el del centro:

    ```azurecli-interactive
    az iot hub consumer-group create --hub-name contoso-simulation7d894 --name devicesimulationtsi --resource-group contoso-simulation
    ```

    Ya puede cerrar Azure Cloud Shell.

## <a name="create-a-new-time-series-insights-environment"></a>Creación de un entorno de Time Series Insights

[Azure Time Series Insights](../../articles/time-series-insights/time-series-insights-overview.md) es un servicio de análisis, almacenamiento y visualización totalmente administrado que permite administrar los datos de serie temporal a escala de IoT en la nube. Para crear un entorno de Time Series Insights:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione **Crear un recurso** > **Internet de las cosas** > **Time Series Insights**.

    ![Nuevo entorno de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/new-time-series-insights.png)

1. Para crear un entorno de Time Series Insights en el mismo grupo de recursos que su acelerador de soluciones, use los valores de la tabla siguiente:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre del entorno | En la captura de pantalla siguiente se usa el nombre **Contoso-TSI**. Elija un nombre exclusivo de su elección al completar este paso. |
    | Subscription | Seleccione la suscripción de Azure en el menú desplegable. |
    | Resource group | **contoso-simulation**. Use el nombre del acelerador de la soluciones. |
    | Location | En este ejemplo se utiliza **Este de EE. UU.** . Cree el entorno en la misma región que el acelerador de simulación de dispositivos. |
    | SKU |**S1** |
    | Capacity | **1** |

    ![Creación del entorno de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/new-time-series-insights-create.png)

    > [!NOTE]
    > Agregar el entorno de Time Series Insights al mismo grupo de recursos que el acelerador de soluciones significa que se eliminará cuando elimine el acelerador de soluciones.

1. Haga clic en **Crear**. El entorno puede tardar unos minutos en crearse.

## <a name="create-event-source"></a>Creación de un origen de eventos

Cree un origen del evento para conectarlo a IoT Hub. Asegúrese de utilizar el grupo de consumidores creado en los pasos anteriores. Un origen del evento de Time Series Insights requiere que cada servicio tenga un grupo de consumidores dedicado que no esté en uso por otro servicio.

1. En Azure Portal, vaya al nuevo entorno de Time Series.

1. En el lado izquierdo, seleccione **Orígenes de eventos**:

    ![Visualización de orígenes de eventos](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-event-sources.png)

1. Haga clic en **Agregar**:

    ![Adición de orígenes de eventos](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-event-sources-add.png)

1. Para configurar el centro de IoT como un nuevo origen del evento, use los valores de la tabla siguiente:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre de origen de eventos | En la captura de pantalla siguiente se usa el nombre **contoso-iot-hub**. Use un nombre exclusivo de su elección al completar este paso. |
    | Source | **IoT Hub** |
    | Opción de importación | **Usar IoT Hub desde suscripciones disponibles** |
    | Id. de suscripción | Seleccione la suscripción de Azure en el menú desplegable. |
    | Nombre de IoT Hub | **contoso-simulation7d894**. Use el nombre de su IoT Hub en el acelerador de soluciones del simulador de dispositivos. |
    | Nombre de directiva de IoT Hub | **iothubowner** |
    | Clave de la directiva de IoT Hub | Este campo se rellena automáticamente. |
    | Grupo de consumidores de IoT Hub | **devicesimulationtsi** |
    | Formato de serialización de eventos | **JSON** |
    | Nombre de la propiedad de marca de tiempo | Déjelo en blanco |

    ![Creación de un origen del evento](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-event-source-create.png)

1. Haga clic en **Crear**.

> [!NOTE]
> También puede [conceder acceso a usuarios adicionales](../time-series-insights/concepts-access-policies.md#grant-data-access) al explorador de Time Series Insights.

## <a name="start-a-simulation"></a>Comenzar una simulación

Antes de usar el explorador de Time Series Insights, configure el acelerador de soluciones de simulación de dispositivos para generar los datos de telemetría. En la captura de pantalla siguiente se muestra la ejecución de una simulación con 10 dispositivos de refrigerador:

![Ejecutar la simulación de dispositivos](./media/iot-accelerators-device-simulation-time-series-insights/running-simulation.png)

## <a name="time-series-insights-explorer"></a>Explorador de Time Series Insights

El explorador de Time Series Insights es una aplicación web que puede usar para visualizar los datos de telemetría.

1. En Azure Portal, la pestaña **Información general** de Time Series Insights.

1. Haga clic en **Go To Environment** (Ir al entorno), que abrirá la aplicación web del explorador de Time Series Insights.

    ![Explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-environment.png)

1. En el panel de selección de tiempo, seleccione **Últimos 30 minutos** en el menú rápido de tiempo, y haga clic en **Buscar**.

    ![Búsqueda en el explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-search-time.png)

1. En el panel de términos de la izquierda, seleccione **temperatura** como **Medida** e **iothub-connection-device-id** como el valor **Dividir por**:

    ![Captura de pantalla que muestra el panel "términos" de Time Series Insights, con los valores "Medida" y "Dividir por" resaltados.](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-query1.png)

1. Haga clic con el botón derecho en el gráfico y seleccione **Explorar eventos**:

    ![Eventos del explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-explore-events.png)

1. Los datos del evento se muestran en una cuadrícula:

    ![Tabla del explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-table.png)

1. Haga clic en el botón de la vista de perspectiva.

    ![Perspectiva del explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-explorer-perspective.png)

1. Haga clic en **+** para agregar una nueva consulta a la perspectiva:

    ![Consulta para agregar en el explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-new-query.png)

1. Seleccione **últimos 30 minutos** como intervalo de tiempo, **Humedad** como **Medida**, y **iothub-connection-device-id** como el valor de **Dividir por**:

    ![Consulta en el explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-query2.png)

1. Haga clic en el botón de vista de perspectiva para ver el panel de datos de telemetría del dispositivo.

    ![Panel del explorador de Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-dashboard.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto explorar más a fondo, deje implementado el acelerador de soluciones.

Si ya no necesita el acelerador de soluciones, elimínelo de la página [Soluciones aprovisionadas](https://www.azureiotsolutions.com/Accelerators#dashboard). Para ello, selecciónelo y, a continuación, haga clic en **Eliminar solución**.

Si agregó el entorno de Time Series Insights al grupo de recursos del acelerador de soluciones, se eliminará automáticamente cuando elimine el acelerador de soluciones. En caso contrario, debe quitar manualmente el entorno de Time Series Insights desde Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo explorar y consultar datos en el explorador de Time Series Insights, consulte [Explorador de Azure Time Series Insights](../time-series-insights/time-series-insights-explorer.md).