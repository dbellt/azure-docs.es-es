---
title: Previsión meteorológica mediante Azure Machine Learning Studio (clásico) con datos de IoT Hub
description: Use Azure Machine Learning Studio (clásico) para predecir la posibilidad de lluvia en función de los datos de temperatura y humedad que IoT Hub recopila de un sensor.
author: robinsh
manager: philmea
keywords: Previsión meteorológica con Machine Learning
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 09/16/2020
ms.author: robinsh
ms.openlocfilehash: 455d78ed21403952046448dd4447b5ec54f77c00
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107566986"
---
# <a name="weather-forecast-using-the-sensor-data-from-your-iot-hub-in-azure-machine-learning-studio-classic"></a>Pronóstico meteorológico con los datos del sensor de IoT Hub en Azure Machine Learning Studio (clásico)

![Diagrama integral](media/iot-hub-get-started-e2e-diagram/6.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

El aprendizaje automático es una técnica de ciencia de datos que ayuda a los equipos a aprender de los datos existentes para prever tendencias, resultados y comportamientos futuros. Azure Machine Learning Studio (clásico) es un servicio de análisis predictivo en la nube que permite crear e implementar rápidamente modelos predictivos como soluciones de análisis. En este artículo, aprenderá a usar Azure Machine Learning Studio (clásico) para hacer previsiones meteorológicas (como la probabilidad de que llueva) mediante el uso de los datos de temperatura y humedad de Azure IoT Hub. La posibilidad de lluvia es el resultado de un modelo de pronóstico meteorológico preparado. El modelo se basa en datos históricos para predecir la posibilidad de lluvia en función de la temperatura y la humedad.

## <a name="prerequisites"></a>Requisitos previos

- Realice el tutorial del [simulador en línea de Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md), o bien uno de los tutoriales del dispositivo. Por ejemplo, puede acceder a [Raspberry Pi con node.js](iot-hub-raspberry-pi-kit-node-get-started.md) o a uno de los inicios rápidos de [envío de telemetría](quickstart-send-telemetry-dotnet.md). Estos artículos abarcan los requisitos siguientes:
  - Una suscripción de Azure activa.
  - Un centro de Azure IoT en su suscripción.
  - Una aplicación cliente que envía mensajes a su centro de Azure IoT.
- Una cuenta de [Azure Machine Learning Studio (clásico)](https://studio.azureml.net/).
- Una [cuenta de Azure Storage](../storage/common/storage-account-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-storage-accounts); es preferible tener **una cuenta de uso general V2**, pero también se puede usar cualquier cuenta de Azure Storage que admita Azure Blob Storage.

> [!Note]
> En este artículo se usa Azure Stream Analytics y otros servicios de pago. Recuerde que se aplican cargos adicionales en Azure Stream Analytics cuando los datos se deben transferir entre regiones de Azure. Por esta razón, es recomendable asegurarse de que el grupo de recursos, IoT Hub y la cuenta de Azure Storage, así como el área de trabajo de Machine Learning Studio (clásico) y el trabajo de Azure Stream Analytics que se agregarán más adelante en este tutorial, se encuentran en la misma región de Azure. Puede comprobar la compatibilidad regional de Azure Machine Learning Studio (clásico) y otros servicios de Azure en la página de [Disponibilidad de productos de Azure por región](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-studio&regions=all).

## <a name="deploy-the-weather-prediction-model-as-a-web-service"></a>Implementación del modelo de pronóstico meteorológico como un servicio web

En esta sección obtendrá el modelo de predicción meteorológica de la biblioteca de Azure AI. A continuación, agregará un módulo de script de R al modelo para limpiar los datos de temperatura y humedad. Por último, implementará el modelo como un servicio web predictivo.

### <a name="get-the-weather-prediction-model"></a>Obtención del modelo de predicción meteorológica

En esta sección obtendrá el modelo de predicción meteorológica de Azure AI Gallery y lo abrirá en Azure Machine Learning Studio (clásico).

1. Vaya a la [página del modelo de pronóstico meteorológico](https://gallery.cortanaintelligence.com/Experiment/Weather-prediction-model-1).

   ![Apertura de la página del modelo de predicción meteorológica en Azure AI Gallery](media/iot-hub-weather-forecast-machine-learning/weather-prediction-model-in-azure-ai-gallery.png)

1. Haga clic en **Open in Studio (classic)** (Abrir en Studio [clásico]) para abrir el modelo en Microsoft Azure Machine Learning Studio (clásico). Seleccione una región cerca de su centro de IoT y el área de trabajo correcta en la ventana emergente **Copy experiment from Gallery** (Copiar experimento de la galería).

   ![Apertura del modelo de predicción meteorológica en Azure Machine Learning Studio (clásico)](media/iot-hub-weather-forecast-machine-learning/open-ml-studio.png)

### <a name="add-an-r-script-module-to-clean-temperature-and-humidity-data"></a>Incorporación de un módulo de script de R para limpiar los datos de temperatura y humedad

Para que el modelo se comporte correctamente, los datos de temperatura y humedad deben poder convertirse en datos numéricos. En esta sección, agregará un módulo de script de R al modelo de predicción meteorológica que quita las filas que tienen valores de datos de temperatura o humedad que no se pueden convertir en valores numéricos.

1. En el lado izquierdo de la ventana de Azure Machine Learning Studio (clásico), haga clic en la flecha para expandir el panel de herramientas. Escriba "ejecutar" en el cuadro de búsqueda. Seleccione el módulo **Execute R Script** (Ejecutar script de R).

   ![Selección del módulo Execute R Script (Ejecutar script de R)](media/iot-hub-weather-forecast-machine-learning/select-r-script-module.png)

1. Arrastre el módulo **Execute R Script** (Ejecutar script de R) cerca del módulo **Clean Missing Data** (Limpiar datos que faltan) y el módulo **Execute R Script** (Ejecutar script de R) en el diagrama. Elimine la conexión entre los módulos **Clean Missing Data** (Limpiar datos que faltan) y **Execute R Script** (Ejecutar script de R) y, luego, conecte las entradas y salidas del nuevo módulo tal y como se muestra.

   ![Incorporación del módulo Execute R Script (Ejecutar script de R)](media/iot-hub-weather-forecast-machine-learning/add-r-script-module.png)

1. Seleccione el nuevo módulo **Execute R Script** (Ejecutar script de R) para abrir su ventana de propiedades. Copie y pegue el código siguiente en el cuadro **R Script** (Script de R).

   ```r
   # Map 1-based optional input ports to variables
   data <- maml.mapInputPort(1) # class: data.frame

   data$temperature <- as.numeric(as.character(data$temperature))
   data$humidity <- as.numeric(as.character(data$humidity))

   completedata <- data[complete.cases(data), ]

   maml.mapOutputPort('completedata')

   ```

   Cuando haya terminado, la ventana de propiedades debería tener un aspecto similar al siguiente:

   ![Incorporación de código al módulo Execute R Script (Ejecutar script de R)](media/iot-hub-weather-forecast-machine-learning/add-code-to-module.png)

### <a name="deploy-predictive-web-service"></a>Implementación de un servicio web predictivo

En esta sección, validará el modelo, configurará un servicio web predictivo basado en este y, luego, implementará el servicio web.

1. Haga clic en **Ejecutar** para validar los pasos del modelo. Este paso puede tardar unos minutos.

   ![Ejecución del experimento para validar los pasos](media/iot-hub-weather-forecast-machine-learning/run-experiment.png)

1. Haga clic en **CONFIGURAR SERVICIO WEB** > **Predictive Web Service** (Servicio web predictivo). Se abre el diagrama del experimento predictivo.

   ![Implementación del modelo de predicción meteorológica en Azure Machine Learning Studio (clásico)](media/iot-hub-weather-forecast-machine-learning/predictive-experiment.png)

1. En el diagrama del experimento predictivo, elimine la conexión entre el módulo **Web service input** (Entrada de servicio web) y **Select Columns in Dataset** (Seleccionar columnas en el conjunto de datos) que se encuentra en la parte superior. Luego, arrastre el módulo **Web service input** (Entrada de servicio web) cerca del módulo **Score Model** (Puntuar modelo) y conéctelo como se muestra:

   ![Conexión de dos módulos en Azure Machine Learning Studio (clásico)](media/iot-hub-weather-forecast-machine-learning/connect-modules-azure-machine-learning-studio.png)

1. Haga clic en **EJECUTAR** para validar los pasos del modelo.

1. Haga clic en **IMPLEMENTAR SERVICIO WEB** para implementar el modelo como un servicio web.

1. En el panel del modelo, descargue **Excel 2010 o el libro anterior** para **SOLICITUD/RESPUESTA**.

   > [!Note]
   > Asegúrese de descargar el **libro de Excel 2010 o anterior** aunque vaya a ejecutar una versión posterior de Excel en el equipo.

   ![Descargar Excel para el punto de conexión SOLICITUD/RESPUESTA](media/iot-hub-weather-forecast-machine-learning/download-workbook.png)

1. Abra el libro de Excel, tome nota de la **DIRECCIÓN URL DEL SERVICIO WEB** y de la **CLAVE DE ACCESO**.

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## <a name="create-configure-and-run-a-stream-analytics-job"></a>Creación, configuración y ejecución de un trabajo de Stream Analytics

### <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics

1. En [Azure Portal](https://portal.azure.com/), haga clic en **Crear un recurso**. Escriba "trabajo de Stream Analytics" en el cuadro de búsqueda y seleccione **trabajo de Stream Analytics** en la lista desplegable de resultados. Cuando se abra el panel del **trabajo de Stream Analytics**, seleccione **Crear**.
1. Escriba la siguiente información para el trabajo.

   **Nombre del trabajo**: Nombre del trabajo. El nombre debe ser único globalmente.

   **Suscripción**: seleccione su suscripción si es diferente de la predeterminada.

   **Grupo de recursos**: use el mismo grupo de recursos que usa el centro de IoT.

   **Ubicación**: use la misma ubicación que el grupo de recursos.

   Deje todos los demás campos con sus valores predeterminados.

   ![Creación de un trabajo de Stream Analytics en Azure](media/iot-hub-weather-forecast-machine-learning/create-stream-analytics-job.png)

1. Seleccione **Crear**.

### <a name="add-an-input-to-the-stream-analytics-job"></a>Adición de una entrada al trabajo de Stream Analytics

1. Abra el trabajo de Stream Analytics.
1. En **Topología de trabajo**, seleccione **Entradas**.
1. En el panel **Entradas**, seleccione **Agregar entrada de flujo** y, a continuación, seleccione **IoT Hub** en la lista desplegable. En el panel de **Nueva entrada**, elija **Seleccionar IoT Hub de las suscripciones** y escriba la siguiente información:

   **Alias de entrada**: el alias único para la entrada.

   **Suscripción**: seleccione su suscripción si es diferente de la predeterminada.

   **IoT Hub**: seleccionar el centro de IoT de la suscripción

   **Nombre de directiva de acceso compartido**: seleccione el **servicio**. (También puede usar **iothubowner**).

   **Grupo de consumidores**: seleccione el grupo de consumidores que ha creado.

   Deje todos los demás campos con sus valores predeterminados.

   ![Adición de una entrada al trabajo de Stream Analytics en Azure](media/iot-hub-weather-forecast-machine-learning/add-input-stream-analytics-job.png)

1. Seleccione **Guardar**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Adición de una salida al trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Salidas**.
1. En el panel de **salidas**, seleccione **Agregar** y, a continuación, seleccione **Blob storage/Data Lake Storage** en la lista desplegable. En el panel de **Nueva entrada**, elija **Seleccionar almacenamiento de las suscripciones** y escriba la siguiente información:

   **Alias de salida**: el alias único para la salida.

   **Suscripción**: seleccione su suscripción si es diferente de la predeterminada.

   **Cuenta de almacenamiento**: la cuenta de almacenamiento para Blob Storage. Puede crear una cuenta de almacenamiento o usar una existente.

   **Contenedor**: el contenedor donde se guarda el blob. Puede crear un contenedor o usar uno existente.

   **Formato de serialización de eventos**: seleccione **CSV**.

   ![Adición de una salida al trabajo de Stream Analytics en Azure](media/iot-hub-weather-forecast-machine-learning/add-output-stream-analytics-job.png)

1. Seleccione **Guardar**.

### <a name="add-a-function-to-the-stream-analytics-job-to-call-the-web-service-you-deployed"></a>Adición de una función al trabajo de Stream Analytics para llamar al servicio web implementado

1. En la **Topología de trabajo**, seleccione **Funciones**.
1. En el panel de **Funciones**, seleccione **Agregar** y, a continuación, seleccione **Azure ML Studio** en la lista desplegable. (Asegúrese de seleccionar **Azure ML Studio**, no el **Azure ML Service**). En el panel **Nueva función**, seleccione **Proporcionar la configuración de la función de Azure Machine Learning manualmente** y escriba la siguiente información:

   **Alias de función**: escriba `machinelearning`.

   **Dirección URL**: escriba la DIRECCIÓN URL DEL SERVICIO WEB que anotó del libro de Excel.

   **Clave**: escriba la CLAVE DE ACCESO que anotó del libro de Excel.

   ![Adición de una función al trabajo de Stream Analytics en Azure](media/iot-hub-weather-forecast-machine-learning/add-function-stream-analytics-job.png)

1. Seleccione **Guardar**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Configuración de la consulta del trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Consulta**.
1. Reemplace el código existente con el siguiente:

   ```sql
   WITH machinelearning AS (
      SELECT EventEnqueuedUtcTime, temperature, humidity, machinelearning(temperature, humidity) as result from [YourInputAlias]
   )
   Select System.Timestamp time, CAST (result.[temperature] AS FLOAT) AS temperature, CAST (result.[humidity] AS FLOAT) AS humidity, CAST (result.[scored probabilities] AS FLOAT ) AS 'probabalities of rain'
   Into [YourOutputAlias]
   From machinelearning
   ```

   Reemplace `[YourInputAlias]` por el alias de entrada del trabajo.

   Reemplace `[YourOutputAlias]` por el alias de salida del trabajo.

1. Seleccione **Guardar consulta**.

> [!Note]
> Si selecciona **Probar consulta**, aparecerá el siguiente mensaje: No se admiten las pruebas de consulta con las funciones de Machine Learning. Modifique la consulta y vuelva a intentarlo. Puede omitir este mensaje sin ningún riesgo y seleccionar **Aceptar** para cerrar el cuadro del mensaje. Asegúrese de guardar la consulta antes de continuar con la siguiente sección.

### <a name="run-the-stream-analytics-job"></a>Ejecución del trabajo de Stream Analytics

En el trabajo de Stream Analytics, seleccione **Información general** en el panel izquierdo. A continuación, seleccione **Inicio** > **Ahora** > **Iniciar**. Una vez que el trabajo se inicia correctamente, su estado cambia de **Detenido** a **En ejecución**.

![Ejecución del trabajo de Stream Analytics](media/iot-hub-weather-forecast-machine-learning/run-stream-analytics-job.png)

## <a name="use-microsoft-azure-storage-explorer-to-view-the-weather-forecast"></a>Uso del Explorador de Microsoft Azure Storage para consultar el pronóstico meteorológico

Ejecute la aplicación cliente para empezar a recopilar y enviar datos de temperatura y humedad a IoT Hub. Por cada mensaje que IoT Hub recibe, el trabajo de Stream Analytics llama al servicio web de pronóstico meteorológico para producir la posibilidad de lluvia. El resultado se guarda luego en Azure Blob Storage. El Explorador de Azure Storage es una herramienta que puede usar para consultar el resultado.

1. [Descargue e instale el Explorador de Microsoft Azure Storage](https://storageexplorer.com/).
1. Abra el Explorador de Azure Storage.
1. Inicie sesión en la cuenta de Azure.
1. Seleccione su suscripción.
1. Haga clic en la suscripción > **Cuentas de almacenamiento** > su cuenta de almacenamiento > **Contenedores de blob** > su contenedor.
1. Abra un archivo .csv para ver el resultado. La última columna registra la posibilidad de lluvia.

   ![Obtención del resultado del pronóstico meteorológico con Azure Machine Learning Studio (clásico)](media/iot-hub-weather-forecast-machine-learning/weather-forecast-result.png)

## <a name="summary"></a>Resumen

Ha usado Azure Machine Learning Studio (clásico) correctamente para predecir la posibilidad de lluvia en función de los datos de temperatura y humedad que IoT Hub recibe.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]