---
title: Ampliación de Azure IoT Central con análisis personalizados | Microsoft Docs
description: Como desarrollador de soluciones, configure una aplicación de IoT Central para realizar análisis y visualizaciones personalizados. Esta solución utiliza Azure Databricks.
author: philmea
ms.author: philmea
ms.date: 03/15/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: philmea
ms.openlocfilehash: 9ada9947b217944d9aec9f785f4716bfe43315b1
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107872774"
---
# <a name="extend-azure-iot-central-with-custom-analytics-using-azure-databricks"></a>Ampliación de Azure IoT Central con análisis personalizados mediante Azure Databricks

Esta guía paso a paso le muestra, como desarrollador de soluciones, cómo ampliar la aplicación IoT Central con análisis y visualizaciones personalizados. En el ejemplo se usa un área de trabajo de [Azure Databricks](/azure/azure-databricks/) para analizar el flujo de datos de telemetría de IoT Central y generar visualizaciones como [diagramas de cajas](https://wikipedia.org/wiki/Box_plot).  

Esta guía paso a paso le muestra cómo ampliar IoT Central todavía más con las [herramientas de análisis integradas](./howto-create-custom-analytics.md).

En esta guía paso a paso, aprenderá a:

* Hacer streaming de datos de telemetría desde una aplicación de IoT Central mediante la *exportación de datos continua*.
* Cree un entorno de Azure Databricks para analizar y representar los datos de telemetría del dispositivo.

## <a name="prerequisites"></a>Requisitos previos

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

### <a name="iot-central-application"></a>Aplicación de IoT Central

Cree una aplicación de IoT Central desde el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral) con la configuración siguiente:

| Configuración | Value |
| ------- | ----- |
| Plan de precios | Estándar |
| Plantilla de la aplicación | Análisis en la tienda: supervisión del estado |
| Nombre de la aplicación | Acepte el valor predeterminado o elija su propio nombre. |
| URL | Acepte el valor predeterminado o elija un prefijo de dirección URL único. |
| Directorio | El inquilino de Azure Active Directory |
| Suscripción de Azure | Su suscripción de Azure |
| Region | La región más cercana |

En los ejemplos y las capturas de pantalla de este artículo se usa la región **Estados Unidos**. Elija una ubicación cercana a usted y asegúrese de que crea todos los recursos en la misma región.

Esta plantilla de aplicación incluye dos dispositivos de termostato simulados que envían telemetría.

### <a name="resource-group"></a>Resource group

Use [Azure Portal para crear un grupo de recursos](https://portal.azure.com/#create/Microsoft.ResourceGroup) llamado **IoTCentralAnalysis** que contenga los otros recursos que cree. Cree los recursos de Azure en la misma ubicación que la aplicación de IoT Central.

### <a name="event-hubs-namespace"></a>Espacio de nombres de Event Hubs

Use [Azure Portal para crear un espacio de nombres de Event Hubs](https://portal.azure.com/#create/Microsoft.EventHub) con la siguiente configuración:

| Configuración | Valor |
| ------- | ----- |
| Nombre    | Elija el nombre del espacio de nombres |
| Plan de tarifa | Básica |
| Subscription | Su suscripción |
| Resource group | IoTCentralAnalysis |
| Location | Este de EE. UU. |
| Unidades de procesamiento | 1 |

### <a name="azure-databricks-workspace"></a>Área de trabajo de Azure Databricks

Use [Azure Portal para crear un servicio de Azure Databricks](https://portal.azure.com/#create/Microsoft.Databricks) con la siguiente configuración:

| Configuración | Value |
| ------- | ----- |
| Nombre del área de trabajo    | Elija el nombre del área de trabajo. |
| Subscription | Su suscripción |
| Resource group | IoTCentralAnalysis |
| Location | Este de EE. UU. |
| Nivel de precios | Estándar |

Cuando haya creado los recursos necesarios, el grupo de recursos **IoTCentralAnalysis** se parecerá al de la captura de pantalla siguiente:

:::image type="content" source="media/howto-create-custom-analytics/resource-group.png" alt-text="imagen del grupo de recursos de análisis de IoT Central.":::

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Puede configurar una aplicación de IoT Central para la exportación continua de datos de telemetría a un centro de eventos. En esta sección, creará un centro de eventos para recibir datos de telemetría de la aplicación de IoT Central. El centro de eventos entrega los datos de telemetría al trabajo de Stream Analytics para su procesamiento.

1. En Azure Portal, vaya al espacio de nombres de Event Hubs y seleccione **+ Centro de eventos**.
1. Ponga al centro de eventos el nombre **centralexport**.
1. En la lista de centros de eventos del espacio de nombres, seleccione **centralexport**. A continuación, elija **Directivas de acceso compartido**.
1. Seleccione **+Agregar**. Cree una directiva denominada **SendListen** con las notificaciones de **envío** y **escucha**.
1. Cuando la directiva esté preparada, selecciónela en la lista y, a continuación, copie el valor de **Cadena de conexión: clave principal**.
1. Tome nota de esta cadena de conexión, la usará más adelante cuando configure su cuaderno de Databricks para leer desde el centro de eventos.

El espacio de nombres de Event Hubs se parece a la captura de pantalla siguiente:

:::image type="content" source="media/howto-create-custom-analytics/event-hubs-namespace.png" alt-text="imagen del espacio de nombres de Event Hubs.":::

## <a name="configure-export-in-iot-central"></a>Configuración de la exportación en IoT Central

En esta sección va a configurar la aplicación para que emita los datos de telemetría desde los dispositivos simulados al centro de eventos.

En el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral), vaya a la aplicación de IoT Central que creó anteriormente. Para configurar la exportación, primero cree un destino:

1. Vaya a la página **Exportación de datos** y, después, seleccione **Destinos**.
1. Seleccione **+ Nuevo destino**.
1. Utilice los valores de la tabla siguiente para crear un destino:

    | Configuración | Value |
    | ----- | ----- |
    | Nombre del destino | Centro de eventos de telemetría |
    | Tipo de destino | Azure Event Hubs |
    | Cadena de conexión | La cadena de conexión del centro de eventos que anotó previamente. |

    El **centro de eventos** se muestra como **centralexport**.

    :::image type="content" source="media/howto-create-custom-analytics/data-export-1.png" alt-text="Captura de pantalla que muestra el destino de exportación de datos":::

1. Seleccione **Guardar**.

Para crear la definición de exportación:

1. Vaya a la página **Exportación de datos** y seleccione **+Nueva exportación**.

1. Use los valores de la tabla siguiente para configurar la exportación.

    | Configuración | Value |
    | ------- | ----- |
    | Nombre de exportación | Exportación del centro de eventos |
    | habilitado | Por |
    | Tipo de datos para exportar | Telemetría |
    | Destinations | Seleccione **+ Destino** y, después, seleccione **Telemetry event hub** (Centro de eventos de telemetría). |

1. Seleccione **Guardar**.

    :::image type="content" source="media/howto-create-custom-analytics/data-export-2.png" alt-text="Captura de pantalla que muestra la definición de exportación de datos":::

Espere hasta que el estado de la exportación sea **correcto** en la página de **Exportación de datos** antes de continuar.

## <a name="configure-databricks-workspace"></a>Configuración del área de trabajo de Databricks

En Azure Portal, vaya al servicio Azure Databricks y seleccione **Launch Workspace** (Iniciar área de trabajo). Se abre una nueva pestaña en el explorador y se inicia sesión en el área de trabajo.

### <a name="create-a-cluster"></a>Crear un clúster

En la página **Azure Databricks**, en la lista de tareas comunes, seleccione **Nuevo clúster**.

Use la información de la tabla siguiente para crear el clúster:

| Configuración | Value |
| ------- | ----- |
| Cluster Name | centralanalysis |
| Modo de clúster | Estándar |
| Versión de Databricks Runtime | 5.5 LTS (Scala 2.11, Spark 2.4.5) |
| Versión de Python | 3 |
| Habilitar escalado automático | No |
| Terminar después de estos minutos de inactividad | 30 |
| Tipo de trabajo | Standard_DS3_v2 |
| Trabajos | 1 |
| Tipo de controlador | Igual que el trabajo |

La creación de un clúster puede tardar varios minutos, espere a que se complete antes de continuar.

### <a name="install-libraries"></a>Instalar bibliotecas

En la página **Clústeres**, espere a que el estado del clúster sea **En ejecución**.

En los pasos siguientes se indica cómo importar la biblioteca que necesita el ejemplo en el clúster:

1. En la página **Clústeres**, espere hasta que el estado del clúster interactivo **centralanalysis** sea **En ejecución**.

1. Seleccione el clúster y, después, elija la pestaña **Bibliotecas**.

1. En la pestaña **Bibliotecas**, elija **Instalar nueva**.

1. En la página **Install Library** (Instalar biblioteca), elija **Maven** como origen de la biblioteca.

1. En el cuadro de texto **Coordenadas**, escriba el siguiente valor: `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.10`

1. Elija **Instalar** para instalar la biblioteca en el clúster.

1. El estado de la biblioteca es ahora **Instalada**:

:::image type="content" source="media/howto-create-custom-analytics/cluster-libraries.png" alt-text="Captura de pantalla de la biblioteca instalada.":::

### <a name="import-a-databricks-notebook"></a>Importación de un cuaderno de Databricks

Use los pasos siguientes para importar un cuaderno de Databricks que contiene el código de Python para analizar y visualizar los datos de telemetría de IoT Central:

1. Vaya al panel **Área de trabajo** del entorno de Databricks. Seleccione la lista desplegable situada junto a su nombre de cuenta y, a continuación, elija **Importar**.

1. Elija la opción para importar desde una dirección URL y escriba la siguiente dirección: [https://github.com/Azure-Samples/iot-central-docs-samples/blob/master/databricks/IoT%20Central%20Analysis.dbc?raw=true](https://github.com/Azure-Samples/iot-central-docs-samples/blob/master/databricks/IoT%20Central%20Analysis.dbc?raw=true)

1. Para importar el cuaderno, elija **Importar**.

1. Seleccione el **área de trabajo** para ver el cuaderno importado:

:::image type="content" source="media/howto-create-custom-analytics/import-notebook.png" alt-text="Captura de pantalla del cuaderno importado.":::

5. Edite el código en la primera celda de Python para agregar la cadena de conexión de Event Hubs que guardó anteriormente:

    ```python
    from pyspark.sql.functions import *
    from pyspark.sql.types import *

    ###### Event Hub Connection strings ######
    telementryEventHubConfig = {
      'eventhubs.connectionString' : '{your Event Hubs connection string}'
    }
    ```

## <a name="run-analysis"></a>Ejecución de análisis

Para ejecutar el análisis, debe asociar el cuaderno al clúster:

1. Seleccione **Desasociado** y, a continuación, seleccione el clúster **centralanalysis**.
1. Si el clúster no se está ejecutando, inícielo.
1. Para iniciar el cuaderno, seleccione el botón Ejecutar.

Puede ver un error en la última celda. Si es así, compruebe que las celdas anteriores se están ejecutando, espere un minuto para que algunos datos se escriban en el almacenamiento y, a continuación, ejecute de nuevo la última celda.

### <a name="view-smoothed-data"></a>Visualización de datos suavizados

En el cuaderno, desplácese hacia abajo hasta la celda 14 para ver un trazado con la humedad media acumulada por tipo de dispositivo. Este trazado se actualiza continuamente a medida que llegan los datos de telemetría por streaming:

:::image type="content" source="media/howto-create-custom-analytics/telemetry-plot.png" alt-text="Captura de pantalla del trazado de datos de telemetría suavizados.":::

Puede cambiar el tamaño del gráfico en el cuaderno.

### <a name="view-box-plots"></a>Visualización de diagramas de cajas

En el cuaderno, desplácese hacia abajo hasta la celda 20 para ver los [diagramas de cajas](https://en.wikipedia.org/wiki/Box_plot). Los diagramas de cajas se basan en datos estáticos. Por ello, para actualizarlos debe volver a ejecutar la celda:

:::image type="content" source="media/howto-create-custom-analytics/box-plots.png" alt-text="Captura de pantalla de los diagramas de cajas.":::

Puede cambiar el tamaño de los trazados en el cuaderno.

## <a name="tidy-up"></a>Limpieza

Para hacer limpieza después de este artículo de procedimientos y evitar costos innecesarios, elimine el grupo de recursos **IoTCentralAnalysis** de Azure Portal.

Puede eliminar la aplicación de IoT Central desde la página **Administración** de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

En esta guía paso a paso, ha aprendido lo siguiente:

* Hacer streaming de datos de telemetría desde una aplicación de IoT Central mediante la *exportación de datos continua*.
* Cree un entorno de Azure Databricks para analizar y representar datos de telemetría.

Ahora que sabe cómo crear un análisis personalizado, el siguiente paso sugerido es aprender a [Visualizar y analizar los datos de Azure IoT Central en un panel de Power BI](howto-connect-powerbi.md).
