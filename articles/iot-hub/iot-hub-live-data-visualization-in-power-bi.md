---
title: 'Visualización en tiempo real de datos de Azure IoT Hub: Power BI'
description: Use Power BI para visualizar datos de temperatura y humedad que se recopilan desde el sensor y se le envían a Azure IoT Hub.
author: robinsh
keywords: visualización de datos en tiempo real, visualización de datos en directo, visualización de datos de sensor
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 6/08/2020
ms.author: robinsh
ms.openlocfilehash: 6ff0a0bb8bc1ca9b26968d002bb44d206c36e158
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483918"
---
# <a name="visualize-real-time-sensor-data-from-azure-iot-hub-using-power-bi"></a>Visualización de datos del sensor en tiempo real desde Azure IoT Hub mediante Power BI

![Diagrama integral](./media/iot-hub-live-data-visualization-in-power-bi/end-to-end-diagram.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

En este artículo, aprenderá a visualizar los datos del sensor en tiempo real que la instancia de Azure IoT Hub recibe mediante Power BI. Si desea intentar visualizar los datos en su centro de IoT con una aplicación web, consulte el tema sobre el [uso de una aplicación web para visualizar datos del sensor en tiempo real desde Azure IoT Hub](iot-hub-live-data-visualization-in-web-apps.md).

## <a name="prerequisites"></a>Requisitos previos

* Realice el tutorial del [simulador en línea de Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md), o bien uno de los tutoriales del dispositivo. Por ejemplo, puede ir a [Raspberry Pi con node.js](iot-hub-raspberry-pi-kit-node-get-started.md) o a uno de los inicios rápidos de [envío de telemetría](quickstart-send-telemetry-dotnet.md). Estos artículos abarcan los requisitos siguientes:
  
  * Una suscripción de Azure activa.
  * Una instancia de Azure IoT Hub en la suscripción.
  * Una aplicación cliente que envía mensajes a su centro de Azure IoT.

* Una cuenta de Power BI ([pruebe Power BI de manera gratuita](https://powerbi.microsoft.com/)).

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## <a name="create-configure-and-run-a-stream-analytics-job"></a>Creación, configuración y ejecución de un trabajo de Stream Analytics

Comencemos creando un trabajo de Stream Analytics. Después de crear el trabajo, puede definir las entradas, las salidas y la consulta que se usa para recuperar los datos.

### <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Internet de las cosas** > **Stream Analytics Job**.

2. Escriba la siguiente información para el trabajo.

   **Nombre del trabajo**: Nombre del trabajo. El nombre debe ser único globalmente.

   **Grupo de recursos**: use el mismo grupo de recursos que usa el centro de IoT.

   **Ubicación**: use la misma ubicación que el grupo de recursos.

   ![Creación de un trabajo de Stream Analytics en Azure](./media/iot-hub-live-data-visualization-in-power-bi/create-stream-analytics-job.png)

3. Seleccione **Crear**.

### <a name="add-an-input-to-the-stream-analytics-job"></a>Adición de una entrada al trabajo de Stream Analytics

1. Abra el trabajo de Stream Analytics.

2. En **Topología de trabajo**, seleccione **Entradas**.

3. En el panel **Entradas**, seleccione **Agregar entrada de flujo** y, a continuación, seleccione **IoT Hub** en la lista desplegable. En el panel de la entrada nueva, escriba la siguiente información:

   **Alias de entrada**: escriba un alias único para la entrada.

   **Seleccionar centro de IoT de entre las suscripciones**: seleccione este botón de radio.

   **Suscripción**: seleccione la suscripción de Azure que utiliza para este tutorial.

   **IoT Hub**: seleccione la instancia de IoT Hub que utiliza para este tutorial.

   **Punto de conexión**: seleccione **Mensajería**.

   **Nombre de directiva de acceso compartido**: seleccione el nombre de la directiva de acceso compartido que quiera que utilice el trabajo de Stream Analytics para su centro de IoT. Para este tutorial, puede seleccionar *service*. La directiva *service* se crea de forma predeterminada en los centros de IoT nuevos y concede permiso de envío y recepción para los puntos de conexión de la nube que expone el centro de IoT. Para obtener más información, consulte el tema sobre [control de acceso y permisos](iot-hub-dev-guide-sas.md#access-control-and-permissions).

   **Clave de directiva de acceso compartido**: este campo se rellena automáticamente en función de su selección de nombre de directiva de acceso compartido.

   **Grupo de consumidores**: seleccione el grupo de consumidores que creó anteriormente.

   Deje todos los demás campos con sus valores predeterminados.

   ![Adición de una entrada a un trabajo de Stream Analytics en Azure](./media/iot-hub-live-data-visualization-in-power-bi/add-input-to-stream-analytics-job.png)

4. Seleccione **Guardar**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Adición de una salida al trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Salidas**.

2. En el panel **Salidas**, seleccione **Agregar** y **Power BI**.

3. En el panel **Power BI: Nueva salida** panel, seleccione **Autorizar** y siga las indicaciones para iniciar sesión en su cuenta de Power BI.

4. Una vez que haya iniciado sesión en Power BI, escriba la siguiente información:

   **Alias de salida**: alias único para la salida.

   **Área de trabajo de grupo**: seleccione el área de trabajo de grupo de destino.

   **Nombre del conjunto de datos**: escriba un nombre para el conjunto de datos.

   **Nombre de la tabla**: Escriba un nombre de tabla.

   **Modo de autenticación**: Deje la opción predeterminada.

   ![Adición de una salida a un trabajo de Stream Analytics en Azure](./media/iot-hub-live-data-visualization-in-power-bi/add-output-to-stream-analytics-job.png)

5. Seleccione **Guardar**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Configuración de la consulta del trabajo de Stream Analytics

1. En **Topología de trabajo**, seleccione **Consulta**.

2. Reemplace `[YourInputAlias]` por el alias de entrada del trabajo.

3. Reemplace `[YourOutputAlias]` por el alias de salida del trabajo.

   ![Adición de una consulta a un trabajo de Stream Analytics en Azure](./media/iot-hub-live-data-visualization-in-power-bi/add-query-to-stream-analytics-job.png)

4. Seleccione **Guardar consulta**.

### <a name="run-the-stream-analytics-job"></a>Ejecución del trabajo de Stream Analytics

En el trabajo de Stream Analytics, seleccione **Información general** y, a continuación, elija **Iniciar** > **Ahora** > **Iniciar**. Una vez que el trabajo se inicia correctamente, su estado cambia de **Detenido** a **En ejecución**.

![Ejecución de un trabajo de Stream Analytics en Azure](./media/iot-hub-live-data-visualization-in-power-bi/run-stream-analytics-job.png)

## <a name="create-and-publish-a-power-bi-report-to-visualize-the-data"></a>Creación y publicación de un informe de Power BI para visualizar los datos

En los pasos siguientes se muestra cómo crear y publicar un informe mediante el servicio Power BI. Puede seguir estos pasos, con algunas modificaciones, si desea utilizar el "nuevo aspecto" en Power BI. Para comprender las diferencias y cómo navegar dentro del "nuevo aspecto", consulte [Nuevo aspecto del servicio Power BI](/power-bi/consumer/service-new-look).

1. Asegúrese de que la aplicación de ejemplo se ejecuta en el dispositivo. Si no es así, puede hacer referencia a los tutoriales en [Configurar su dispositivo](./iot-hub-raspberry-pi-kit-node-get-started.md).

2. Inicie sesión en su cuenta de [Power BI](https://powerbi.microsoft.com/en-us/).

3. Seleccione el área de trabajo que usó, **Mi área de trabajo**.

4. Seleccione **Conjuntos de datos**.

   Debería ver el conjunto de datos que especificó en el momento de crear la salida para el trabajo de Stream Analytics.

5. Para el conjunto de datos que creó, seleccione **Agregar informe** (el primer icono a la derecha del nombre del conjunto de datos).

   ![Creación de informe de Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-create-report.png)

6. Cree un gráfico de líneas para mostrar la temperatura en tiempo real en un período determinado.

   1. En el panel **Visualizaciones** de la página de creación de informes, seleccione el icono de gráfico de líneas para agregar un gráfico de líneas.

   2. En el panel **Campos**, expanda la tabla que especificó en el momento de crear la salida para el trabajo de Stream Analytics.

   3. Arrastre **EventEnqueuedUtcTime** (Hora UTC de evento en cola) al **Eje** en el panel **Visualizaciones**.

   4. Arrastre **temperature** (temperatura) a **Valores**.

      Se ha creado un gráfico de líneas. El eje x muestra la fecha y hora en la zona horaria UTC. El eje y muestra la temperatura del sensor.

      ![Adición de un gráfico de líneas de temperatura a un informe de Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-add-temperature.png)

7. Cree otro gráfico de líneas para mostrar la humedad en tiempo real en un período determinado. Para ello, haga clic en una parte en blanco del lienzo y siga los mismos pasos anteriores y coloque **EventEnqueuedUtcTime** (Hora UTC de evento en cola) en el eje x y **humidity** (humedad) en el eje y.

   ![Adición de un gráfico de líneas de humedad a un informe de Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-add-humidity.png)

8. Seleccione **Guardar** para guardar el informe.

9. Seleccione **Informes** en el panel izquierdo y, a continuación, elija el informe recién creado.

10. Seleccione **Archivo** > **Publicar en Web**.

    ![Seleccione publicar en web para el informe de Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-select-publish-to-web.png)

    > [!NOTE]
    > Si recibe una notificación para ponerse en contacto con el administrador para habilitar la creación de código para insertar, puede que tenga que ponerse en contacto con él. La creación del código para insertar tiene que estar habilitada para poder completar este paso.
    >
    > ![Notificación de contacto con el administrador](./media/iot-hub-live-data-visualization-in-power-bi/contact-admin.png)

11. Seleccione **Crear código para insertar** y, a continuación, elija **Publicar**.

Se le ofrecerá el vínculo del informe, que puede compartir con cualquiera para concederle acceso a este, y un fragmento de código para que pueda integrarlo en su blog o sitio web.

![Publicación de informe de Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-web-output.png)

Microsoft también ofrece las [aplicaciones móviles de Power BI](https://powerbi.microsoft.com/en-us/documentation/powerbi-power-bi-apps-for-mobile-devices/) para ver e interactuar con los informes y paneles de Power BI desde su dispositivo móvil.

## <a name="next-steps"></a>Pasos siguientes

Ha utilizado correctamente Power BI para visualizar datos de sensor en tiempo real desde su centro de Azure IoT.

Para conocer otra forma de visualizar los datos desde Azure IoT Hub, consulte el tema sobre el [uso de una aplicación web para visualizar datos del sensor en tiempo real desde Azure IoT Hub](iot-hub-live-data-visualization-in-web-apps.md).

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
