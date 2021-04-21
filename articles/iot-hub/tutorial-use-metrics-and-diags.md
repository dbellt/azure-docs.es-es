---
title: Configuración y uso de métricas y registros con Azure IoT Hub
description: 'Tutorial: Aprenda a configurar y usar las métricas y los registros con un centro de Azure IoT. Esto le proporcionará datos para su análisis que le ayudarán a diagnosticar los problemas que pueda tener el centro de conectividad.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 10/29/2020
ms.author: robinsh
ms.custom:
- mvc
- mqtt
- devx-track-azurecli
- devx-track-csharp
ms.openlocfilehash: 099b7f4e812e92503c7ed8e3eb733f2e49ccd8b9
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768078"
---
# <a name="tutorial-set-up-and-use-metrics-and-logs-with-an-iot-hub"></a>Tutorial: Configuración y uso de métricas y registros con un centro de IoT

Puede usar Azure Monitor para recopilar métricas y registros del centro de IoT que pueden ayudarle a supervisar el funcionamiento de la solución y a solucionar problemas cuando se produzcan. En este artículo, verá cómo crear gráficos basados en métricas, cómo crear alertas que se desencadenen en función de las métricas, cómo enviar operaciones y errores de IoT Hub a registros de Azure Monitor y cómo comprobar los errores de los registros.

En este tutorial se usa el ejemplo de Azure del [inicio rápido sobre el envío de telemetría de .NET](quickstart-send-telemetry-dotnet.md) para enviar mensajes a IoT Hub. Siempre puede usar un dispositivo u otro ejemplo para enviar mensajes, pero es posible que tenga que modificar algunos pasos en consecuencia.

Antes de comenzar este tutorial, puede ser de ayuda estar familiarizado con algunos conceptos de Azure Monitor. Para más información, consulte [Supervisión de IoT Hub](monitor-iot-hub.md). Para más información sobre las métricas y los registros de recursos que emite IoT Hub, consulte [Supervisión de la referencia a datos](monitor-iot-hub-reference.md).

En este tutorial se realizan las siguientes tareas:

> [!div class="checklist"]
>
> * Usar la CLI de Azure para crear un centro de IoT, registrar un dispositivo simulado y crear un área de trabajo de Log Analytics.  
> * Enviar conexiones de IoT Hub y registros de recursos de telemetría de dispositivos a los registros de Azure Monitor en el área de trabajo de Log Analytics.
> * Usar el explorador de métricas para crear un gráfico basado en métricas seleccionadas y anclarlo al panel.
> * Crear alertas de métricas para recibir notificaciones por correo electrónico cuando se produzcan condiciones importantes.
> * Descargar y ejecutar una aplicación que simule un dispositivo IoT enviando mensajes al centro de IoT.
> * Ver las alertas cuando se produzcan las condiciones.
> * Ver el gráfico de métricas en el panel.
> * Ver los errores y las operaciones de IoT Hub en los registros de Azure Monitor.

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

- Necesita el SDK de .NET Core 2.1, o una versión posterior, en la máquina de desarrollo. Puede descargar el SDK de .NET Core para varias plataformas desde [.NET](https://www.microsoft.com/net/download/all).

  Puede verificar la versión actual de C# en el equipo de desarrollo con el comando siguiente:

  ```cmd/sh
  dotnet --version
  ```

- Una cuenta de correo electrónico capaz de recibir correo electrónico.

- Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivos de ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="set-up-resources"></a>Configuración de recursos

En este tutorial, necesitará un centro de IoT, un área de trabajo de Log Analytics y un dispositivo IoT simulado. Todos estos recursos se pueden crear con la CLI de Azure o Azure PowerShell. Use el mismo grupo de recursos y la misma ubicación para todos los recursos. Después, cuando haya finalizado el tutorial, puede quitar todo de una vez eliminando el grupo de recursos.

Estos son los pasos necesarios.

1. Cree un [grupo de recursos](../azure-resource-manager/management/overview.md).

2. Cree un Centro de IoT.

3. Cree un área de trabajo de Log Analytics.

4. Registre una identidad de dispositivo para el dispositivo simulado que envía mensajes al centro de IoT. Guarde la cadena de conexión del dispositivo que se va a usar para configurar el dispositivo simulado.

### <a name="set-up-resources-using-azure-cli"></a>Configuración de los recursos mediante la CLI de Azure

Copie y pegue este script en Cloud Shell. Suponiendo que ya haya iniciado sesión, el script se ejecutará línea a línea. Algunos comandos pueden tardar un tiempo en ejecutarse. Los nuevos recursos se crean en el grupo de recursos *ContosoResources*.

El nombre de algunos recursos debe ser único en todo Azure. El script genera un valor aleatorio con la función `$RANDOM` y lo almacena en una variable. Con estos recursos, el script anexa este valor aleatorio a un nombre base del recurso, lo que hace que el nombre del recurso sea único.

Solo se permite un centro de IoT gratuito por suscripción. Si ya tiene un centro de IoT gratuito en su suscripción, elimínelo antes de ejecutar el script, o modifique el script para que use su centro de IoT gratuito o un centro de IoT que utilice el nivel estándar o básico.

El script imprime el nombre del centro de IoT, el nombre del área de trabajo de Log Analytics y la cadena de conexión del dispositivo que se registra. Asegúrese de anotarlos, ya que los necesitará más adelante en este artículo.

```azurecli-interactive

# This is the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity.
az extension add --name azure-iot

# Set the values for the resource names that don't have to be globally unique.
# The resources that have to have unique names are named in the script below
#   with a random number concatenated to the name so you can probably just
#   run this script, and it will work with no conflicts.
location=westus
resourceGroup=ContosoResources
iotDeviceName=Contoso-Test-Device
randomValue=$RANDOM

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, so add a random number to the end.
iotHubName=ContosoTestHub$randomValue
echo "IoT hub name = " $iotHubName

# Create the IoT hub in the Free tier. Partition count must be 2.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --partition-count 2 \
    --sku F1 --location $location

# The Log Analytics workspace name must be globally unique, so add a random number to the end.
workspaceName=contoso-la-workspace$randomValue
echo "Log Analytics workspace name = " $workspaceName


# Create the Log Analytics workspace
az monitor log-analytics workspace create --resource-group $resourceGroup \
    --workspace-name $workspaceName --location $location

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the primary connection string for the device identity, then copy it to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show-connection-string --device-id $iotDeviceName \
    --hub-name $iotHubName

```

>[!NOTE]
>Al crear la identidad del dispositivo, puede recibir el siguiente error: *No keys found for policy iothubowner of IoT Hub ContosoTestHub* (No se encontraron claves para la directiva iothubowner del centro de IoT ContosoTestHub). Para corregir este error, actualice la extensión IoT de la CLI de Azure y, luego, vuelva a ejecutar los dos últimos comandos en el script. 
>
>Este es el comando para actualizar la extensión. Ejecútelo en su instancia de Cloud Shell.
>
>```cli
>az extension update --name azure-iot
>```

## <a name="collect-logs-for-connections-and-device-telemetry"></a>Recopilación de registros de conexiones y telemetría de dispositivos

IoT Hub emite registros de recursos de varias categorías de operación; sin embargo, para ver estos registros, debe crear una configuración de diagnóstico que los envíe a un destino. Uno de estos destinos son los registros de Azure Monitor, que se recopilan en un área de trabajo de Log Analytics. Los registros de recursos de IoT Hub se agrupan en distintas categorías. Puede seleccionar las categorías que quiere que se envíen a los registros de Azure Monitor en la configuración de diagnóstico. En este artículo, se recopilarán registros de operaciones y errores que se producen que tienen que ver con las conexiones y la telemetría de los dispositivos. Para ver una lista completa de las categorías admitidas en IoT Hub, consulte [Registros de recursos de IoT Hub](monitor-iot-hub-reference.md#resource-logs).

Para crear una configuración de diagnóstico para enviar los registros de recursos de IoT Hub a los registros de Azure Monitor, siga estos pasos:

1. En primer lugar, si no está aún en su centro en el portal, haga clic en **Grupos de recursos** y seleccione el grupo de recursos ContosoResources. Seleccione el centro de IoT de la lista de recursos que se muestran.

1. Busque la sección **Supervisión** en la hoja IoT Hub. Seleccione **Configuración de diagnóstico**. Después, seleccione **Agregar configuración de diagnóstico**.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/open-diagnostic-settings.png" alt-text="Captura de pantalla que resalta la opción Configuración de diagnóstico en la sección Supervisión.":::

1. En el panel **Configuración de diagnóstico**, asigne a la configuración un nombre descriptivo, por ejemplo, "Enviar conexiones y telemetría a los registros".

1. En **Detalles de la categoría**, seleccione **Conexiones** y **Device Telemetry** (Telemetría de dispositivo).

1. En **Detalles del destino**, seleccione **Enviar a Log Analytics**, y, luego, use el selector de áreas de trabajo de Log Analytics para seleccionar el área de trabajo que anotó anteriormente. Cuando haya terminado, la configuración de diagnóstico debe ser similar a la captura de pantalla siguiente:

   :::image type="content" source="media/tutorial-use-metrics-and-diags/add-diagnostic-setting.png" alt-text="Captura de pantalla que muestra la configuración de registro de diagnóstico final.":::

1. Haga clic en **Save** (Guardar) para guardar la configuración. Cierre el panel **Configuración de diagnóstico**. Puede ver la nueva configuración en la lista de configuraciones de diagnóstico.

## <a name="set-up-metrics"></a>Configuración de las métricas

Ahora usaremos el explorador de métricas para crear un gráfico que muestre las métricas cuyo seguimiento desea realizar. Este gráfico se anclará al panel predeterminado en Azure Portal.

1. En el panel izquierdo del centro de IoT, en la sección **Supervisión**, seleccione **Métricas**.

1. En la parte superior de la pantalla, seleccione **Last 24 hours (Automatic)** (Últimas 24 horas [Automático]). En la lista desplegable que aparece, seleccione **Últimas 4 horas** en **Intervalo de tiempo**, establezca **Granularidad de tiempo** en **1 minuto** y seleccione **Local** en **Mostrar hora como**. Seleccione **Aplicar** para guardar la configuración. La configuración debería indicar ahora **Hora local: Últimas 4 horas (1 minuto)** .

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-select-time-range.png" alt-text="Captura de pantalla que muestra la configuración de tiempo de las métricas.":::

1. En el gráfico, hay una configuración de métrica parcial que se muestra en el ámbito del centro de IoT. Deje los valores predeterminados de **Ámbito** y **Espacio de nombres de métricas**. Seleccione la opción **Métrica** y escriba "Telemetría"; luego, seleccione **Telemetry messages sent** (Mensajes de telemetría enviados) en la lista desplegable. **Agregación** se establecerá automáticamente en **Suma**. Observe que el título del gráfico también cambia.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-telemetry-messages-sent.png" alt-text="Captura de pantalla que muestra la adición al gráfico de la métrica de mensajes de telemetría enviados.":::

1. Ahora, seleccione **Agregar métrica** para agregar otra métrica al gráfico. En **Métrica**, seleccione **Total number of messages used** (Número total de mensajes usados). **Agregación** se establecerá automáticamente en **Media**. Observe de nuevo que el título del gráfico ha cambiado para incluir esta métrica.

   Ahora la pantalla muestra la métrica minimizada de *Telemetry messages sent* (Mensajes de telemetría enviados), más la nueva métrica de *Total number of messages used* (Número total de mensajes usados).

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-total-number-of-messages-used.png" alt-text="Captura de pantalla que muestra la adición al gráfico de la métrica de número total de mensajes utilizados.":::

1. En la esquina superior derecha del gráfico, seleccione **Anclar al panel**.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-total-number-of-messages-used-pin.png" alt-text="Captura de pantalla que resalta el botón Anclar al panel.":::

1. En **Anclar al panel**, seleccione la pestaña **Existente**. Seleccione **Privado** y, luego, elija **Panel** en la lista desplegable de paneles. Por último, seleccione **Anclar** para anclar el gráfico al panel predeterminado de Azure Portal. Si no ancla el gráfico a un panel, la configuración no se conserva al salir del explorador de métricas.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/pin-to-dashboard.png" alt-text="Captura de pantalla que muestra la configuración de Anclar al panel.":::

## <a name="set-up-metric-alerts"></a>Configuración de alertas de métricas

Ahora, vamos a configurar alertas para que se desencadenen con dos métricas: *Telemetry messages sent* (Mensajes de telemetría enviados) y *Total number of messages used* (Número total de mensajes utilizados).

La métrica *Telemetry messages sent* (Mensajes de telemetría enviados) es adecuada para supervisar el rendimiento de los mensajes y evitar que se limiten. En el caso de un centro de IoT de nivel gratis, el límite es de 100 mensajes por segundo. Con un solo dispositivo, no se puede lograr ese tipo de rendimiento, así que, en su lugar, configuraremos la alerta para que se desencadene si el número de mensajes supera los 1000 en un período de 5 minutos. En producción, puede establecer la señal en un valor más significativo según el nivel, la edición y el número de unidades del centro de IoT.

La métrica *Total number of messages used* (Número total de mensajes utilizados) realiza un seguimiento del número diario de mensajes usados. Esta métrica se restablece todos los días a las 00:00 UTC. Si supera la cuota diaria por encima de un umbral determinado, el centro de IoT dejará de aceptar mensajes. En el caso de un centro de IoT de nivel gratis, la cuota de mensajes diaria es de 8000. Configuraremos la alerta para que se desencadene si el número total de mensajes supera los 4000, el 50 % de la cuota. En la práctica, probablemente establecería este porcentaje en un valor superior. El valor de la cuota diaria depende del nivel, la edición y el número de unidades del centro de IoT.

Para más información sobre los límites de cuota y las restricciones de limitación con IoT Hub, consulte [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md).

Para configurar alertas de métricas:

1. Vaya a su centro de IoT en Azure Portal.

1. En **Supervisión**, seleccione **Alertas**. Luego, seleccione **Nueva regla de alertas**.  Se abre el panel **Crear regla de alertas**.

    :::image type="content" source="media/tutorial-use-metrics-and-diags/create-alert-rule-pane.png" alt-text="Captura de pantalla que muestra el panel Crear regla de alertas.":::

    En el panel **Crear regla de alertas**, hay cuatro secciones:

    * **Ámbito**: ya está establecido en el centro de IoT, por lo que no haremos nada.
    * **Condición**: establece la señal y las condiciones que desencadenarán la alerta.
    * **Acciones**: configura lo que ocurre cuando se desencadena la alerta.
    * **Alert rule details** (Detalles de la regla de alerta): le permite establecer un nombre y una descripción para la alerta.

1. En primer lugar, configure la condición que desencadenará la alerta.

    1. En **Condición**, seleccione **Agregar condición**. En el panel **Configurar lógica señal**, escriba "telemetría" en el cuadro de búsqueda y seleccione **Telemetry messages sent** (Mensajes de telemetría enviados).

       :::image type="content" source="media/tutorial-use-metrics-and-diags/configure-signal-logic-telemetry-messages-sent.png" alt-text="Captura de pantalla que muestra la selección de la métrica.":::

    1. En el panel **Configurar lógica de señal**, establezca o confirme los campos siguientes en **Lógica de alerta** (puede omitir el gráfico):

       **Umbral**:  *Estática*.

       **Operador**: *Mayor que*.

       **Tipo de agregación**: *Total*.

       **Valor del umbral**: 1000.

       **Granularidad de agregación (período)** : *5 minutos*.

       **Frecuencia de evaluación**: *Cada minuto*

        :::image type="content" source="media/tutorial-use-metrics-and-diags/configure-signal-logic-set-conditions.png" alt-text="Captura de pantalla que muestra la configuración de las condiciones de alerta.":::

       Esta configuración establece la señal en el número total de mensajes durante un período de 5 minutos. Este total se evaluará cada minuto y, si transcurridos los 5 minutos anteriores el número de mensajes es superior a 1000, se desencadenará la alerta.

       Seleccione **Listo** para guardar la lógica de señal.

1. Ahora, configure la acción de la alerta.

    1. De nuevo en el panel **Crear regla de alertas**, en **Acciones**, seleccione **Add action groups** (Agregar grupos de acción). En **Seleccionar un grupo de acciones para asociar a esta regla de alerta**, elija **Crear grupo de acciones**.

    1. En la pestaña **Aspectos básicos** del panel **Crear grupo de acciones**, asigne un nombre al grupo de acciones y un nombre para mostrar.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-action-group-basics.png" alt-text="Captura de pantalla que muestra la pestaña Aspectos básicos del panel Crear grupo de acciones.":::

    1. Seleccione la pestaña **Notificaciones**. En **Tipo de notificación**, seleccione **Email/SMS message/Push/Voice** (Correo electrónico/mensaje SMS/notificación push/mensaje de voz) en la lista desplegable. Se abre el panel **Email/SMS message/Push/Voice** (Correo electrónico/mensaje SMS/notificación push/mensaje de voz).

    1. En el panel **Email/SMS message/Push/Voice** (Correo electrónico/mensaje SMS/notificación push/mensaje de voz), seleccione correo electrónico y escriba su dirección de correo electrónico y, luego, elija **Aceptar**.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/set-email-address.png" alt-text="Captura de pantalla que muestra la configuración de dirección de correo electrónico.":::

    1. De nuevo en el panel **Notificaciones**, escriba un nombre para la notificación.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-action-group-notification-complete.png" alt-text="Captura de pantalla que muestra el panel de notificaciones completadas.":::

    1. (Opcional) Si selecciona la pestaña **Acciones** y, luego, elige la lista desplegable **Tipo de acción**, puede ver los tipos de acciones que puede desencadenar con una alerta. En este artículo, solo usaremos notificaciones, por lo que puede omitir la configuración de esta pestaña.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/action-types.png" alt-text="Captura de pantalla que muestra los tipos de acciones disponibles en el panel Acciones.":::

    1. Seleccione la pestaña **Revisar y crear**, compruebe la configuración y elija **Crear**.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-action-group-review-and-create.png" alt-text="Captura de pantalla que muestra el panel Revisar y crear.":::

    1. De nuevo en el panel **Crear regla de alertas**, observe que el nuevo grupo de acciones se ha agregado a las acciones de la alerta.  

1. Por último, configure los detalles de la regla de alertas y guárdela.

    1. En el panel **Crear regla de alertas**, en Detalles de la regla de alertas, escriba un nombre y una descripción para la alerta; por ejemplo, "Enviar alerta si hay más de 1000 mensajes en 5 minutos". Asegúrese de que **Enable alert rule upon creation** (Habilitar regla de alertas tras la creación) esté activada. La regla de alertas completada tendrá un aspecto similar al de esta captura de pantalla.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-alert-rule-final.png" alt-text="Captura de pantalla que muestra el panel Crear regla de alertas completado.":::

    1. Seleccione **Crear regla de alertas** para guardar la nueva regla.

1. Ahora, configure otra alerta para *Total number of messages used* (Número total de mensajes usados). Esta métrica es útil si quiere enviar una alerta cuando el número de mensajes utilizados se aproxime a la cuota diaria del centro de IoT; en ese momento, el centro de IoT comenzará a rechazar mensajes. Siga los pasos que hizo antes, con las siguientes diferencias.

    * En el panel **Configurar lógica de señal**, seleccione **Total number of messages used** (Número total de mensajes utilizados).

    * En el panel **Configurar lógica de señal**, establezca o confirme los campos siguientes (puede omitir el gráfico):

       **Umbral**:  *Estática*.

       **Operador**: *Mayor que*.

       **Tipo de agregación**: *Máxima*.

       **Valor del umbral**: 4000.

       **Granularidad de agregación (período)** : *1 minuto*.

       **Frecuencia de evaluación**: *Cada minuto*

       Esta configuración establece la señal que se activará cuando el número de mensajes alcance los 4000. La métrica se evalúa cada minuto.

    * Cuando especifique la acción de la regla de alertas, seleccione el grupo de acciones que creó anteriormente.

    * En los detalles de la alerta, elija un nombre y una descripción diferentes a los que usó anteriormente.

1. En **Supervisión**, en el panel izquierdo del centro de IoT, seleccione **Alertas**. Ahora, en el menú de la parte superior del panel **Alertas**, seleccione **Administrar reglas de alerta**. Se abre el panel **Reglas**. Ahora debería ver las dos alertas:

   :::image type="content" source="media/tutorial-use-metrics-and-diags/rules-management.png" alt-text="Captura de pantalla que muestra el panel Reglas con las nuevas reglas de alertas.":::

1. Cierre el panel **Reglas**.

Con esta configuración, se desencadenará una alerta y recibirá una notificación por correo electrónico cuando se envíen más de 1000 mensajes dentro de un intervalo de tiempo de 5 minutos y también cuando el número total de mensajes utilizados supere los 4000 (el 50 % de la cuota diaria de un centro de IoT en el nivel gratis).

## <a name="run-the-simulated-device-app"></a>Ejecución de una aplicación de dispositivo simulada

En la sección [Configurar recursos](#set-up-resources), ha registrado una identidad de dispositivo que se usará para simular el uso de un dispositivo IoT. En esta sección, descargará una aplicación de consola .NET que simula un dispositivo que envía mensajes del dispositivo a la nube a un centro de IoT, la configurará para enviar estos mensajes al centro de IoT y, luego, la ejecutará.

> [!IMPORTANT]
>
> Las alertas pueden tardar hasta 10 minutos en configurarse y habilitarse completamente mediante IoT Hub. Espere al menos 10 minutos entre que configura la última alerta y ejecuta la aplicación de dispositivo simulado.

Descargue la solución para la [simulación de un dispositivo IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Este vínculo descarga un repositorio que contiene varias aplicaciones; la que busca está en iot-hub/Quickstarts/simulated-device/.

1. En una ventana del terminal local, vaya a la carpeta raíz de la solución. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device**.

1. Abra el archivo **SimulatedDevice.cs** en el editor de texto de su elección.

    1. Reemplace el valor de la variable `s_connectionString` por la cadena de conexión del dispositivo que anotó cuando ejecutó el script para configurar los recursos.

    1. En el método `SendDeviceToCloudMessagesAsync`, cambie `Task.Delay` de 1000 a 1, lo que reduce la cantidad de tiempo entre envíos de mensajes de 1 a 0,001 segundos. Al acortar este retraso aumenta el número de mensajes enviados. (Es probable que no obtenga una tasa de 100 mensajes por segundo).

        ```csharp
        await Task.Delay(1);
        ```

    1. Guarde los cambios en **SimulatedDevice.cs**.

1. En la ventana del terminal local, ejecute el comando siguiente para instalar los paquetes necesarios para la aplicación de dispositivo simulado:

    ```cmd/sh
    dotnet restore
    ```

1. En la ventana de terminal local, ejecute el comando siguiente para compilar la aplicación de dispositivo simulado y ejecutarla:

    ```cmd/sh
    dotnet run
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    :::image type="content" source="media/tutorial-use-metrics-and-diags/simulated-device-output.png" alt-text="Captura de pantalla que muestra la salida del dispositivo simulado.":::

Deje que la aplicación se ejecute durante al menos 10 o 15 minutos. Lo mejor es dejar que lo haga hasta que deje de enviar mensajes (aproximadamente 20 o 30 minutos). Este hecho se producirá cuando haya superado la cuota diaria de mensajes de su centro de IoT y haya dejado de aceptar más mensajes.

> [!NOTE]
> Si deja que la aplicación de dispositivo se ejecute durante un período prolongado después de que deje de enviar mensajes, puede que reciba una excepción. Esta excepción se puede omitir sin riesgo alguno. Luego, cierre la ventana de la aplicación.

## <a name="view-metrics-chart-on-your-dashboard"></a>Visualización del gráfico de métricas en el panel

1. En la esquina superior izquierda de Azure Portal, abra el menú del portal y, luego, seleccione **Panel**.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/select-dashboard.png" alt-text="Captura de pantalla sobre cómo seleccionar el panel.":::

1. Busque el gráfico que ancló anteriormente y haga clic en cualquier lugar del icono fuera de los datos del gráfico para expandirlo. Aparecen en el gráfico los mensajes de telemetría enviados y el número total de mensajes utilizados. Los números más recientes aparecen en la parte inferior del gráfico. Puede mover el cursor en el gráfico para ver los valores de las métricas en determinados momentos. También puede cambiar el valor de tiempo y la granularidad en la parte superior del gráfico para restringir o expandir los datos a un período de tiempo de interés.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-on-dashboard-last-hour.png" alt-text="Captura de pantalla que muestra el gráfico de métricas.":::

   En este escenario, el rendimiento de mensajes del dispositivo simulado no es lo suficientemente grande como para que IoT Hub limite sus mensajes. En un escenario que realmente conlleve limitación, puede ver que los mensajes de telemetría enviados superan el límite de su centro de IoT durante un tiempo limitado. El motivo de ello es acomodar el tráfico de ráfaga. Para más información, consulte [Modelado de tráfico](iot-hub-devguide-quotas-throttling.md#traffic-shaping).

## <a name="view-the-alerts"></a>Visualización de las alertas

Cuando el número de mensajes enviados supere los límites establecidos en las reglas de alertas, comenzará a recibir alertas por correo electrónico.

Para ver si hay alertas activas, en **Supervisión**, en el panel izquierdo del centro de IoT, seleccione **Alertas**. En el panel **Alertas** se muestra el número de alertas que se han activado durante el intervalo de tiempo especificado, ordenadas por gravedad.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/view-alerts.png" alt-text="Captura de pantalla que muestra el resumen de alertas.":::

Seleccione la fila de gravedad 3. Se abre el panel **Todas las alertas** con las alertas de gravedad 3 que se han activado.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/view-all-alerts.png" alt-text="Captura de pantalla que muestra el panel Todas las alertas.":::

Seleccione una de las alertas para ver los detalles.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/view-individual-alert.png" alt-text="Captura de pantalla que muestra los detalles de la alerta.":::

En la bandeja de entrada, busque los correos electrónicos de Microsoft Azure. En la línea de asunto se describe la alerta que se desencadenó. Por ejemplo, *Azure: Gravedad activada: 3, Enviar alerta si hay más de 1000 mensajes en 5 minutos*. El cuerpo será parecido a la siguiente imagen:

   :::image type="content" source="media/tutorial-use-metrics-and-diags/alert-mail.png" alt-text="Captura de pantalla del correo electrónico con las alertas activadas.":::

## <a name="view-azure-monitor-logs"></a>Visualización de registros de Azure Monitor

En la sección [Recopilación de registros de conexiones y telemetría de dispositivos](#collect-logs-for-connections-and-device-telemetry), creó una configuración de diagnóstico para enviar a los registros de Azure Monitor registros de recursos emitidos por el centro de IoT sobre las operaciones de conexión y telemetría de dispositivos. En esta sección, ejecutará una consulta de Kusto en los registros de Azure Monitor para observar los errores que se produjeron.

1. En Azure Portal, en **Supervisión**, en el panel izquierdo del centro de IoT, seleccione **Registros**. Si está abierta, cierre la ventana inicial de **Consultas**.

1. En el panel Nueva consulta, seleccione la pestaña **Consultas** y, luego, expanda **IoT Hub** para ver la lista de consultas predeterminadas.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/new-query-pane.png" alt-text="Captura de pantalla de las consultas predeterminadas de IoT Hub.":::

1. Seleccione la consulta *Resumen de errores*. La consulta aparece en el panel del Editor de consultas. Seleccione **Ejecutar** en el panel del editor y observe los resultados de la consulta. Expanda una de las filas para ver los detalles.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/logs-errors.png" alt-text="Captura de pantalla de los registros devueltos por la consulta Resumen de errores.":::

   > [!NOTE]
   > Si no ve ningún error, pruebe a ejecutar la consulta *Recently connected devices* (Dispositivos conectados recientemente). Se devolverá una fila para el dispositivo simulado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar todos los recursos que ha creado en este tutorial, elimine el grupo de recursos. Esta acción elimina también todos los recursos del grupo. En este caso, quita el centro de IoT Hub, el área de trabajo de Log Analytics y el propio grupo de recursos. Si ha anclado gráficos de métricas al panel, deberá quitarlas manualmente, para lo cual, tiene que hacer clic en los tres puntos de la esquina superior derecha de cada uno y seleccionar **Quitar**. Después de eliminar los gráficos, asegúrese de guardar los cambios.

Para quitar el grupo de recursos, use el comando [az group delete](/cli/azure/group#az_group_delete).

```azurecli-interactive
az group delete --name ContosoResources
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a usar métricas y registros de IoT Hub mediante las siguientes tareas:

> [!div class="checklist"]
>
> * Usar la CLI de Azure para crear un centro de IoT, registrar un dispositivo simulado y crear un área de trabajo de Log Analytics.  
> * Enviar conexiones de IoT Hub y registros de recursos de telemetría de dispositivos a los registros de Azure Monitor en el área de trabajo de Log Analytics.
> * Usar el explorador de métricas para crear un gráfico basado en métricas seleccionadas y anclarlo al panel.
> * Crear alertas de métricas para recibir notificaciones por correo electrónico cuando se produzcan condiciones importantes.
> * Descargar y ejecutar una aplicación que simule un dispositivo IoT enviando mensajes al centro de IoT.
> * Ver las alertas cuando se produzcan las condiciones.
> * Ver el gráfico de métricas en el panel.
> * Ver los errores y las operaciones de IoT Hub en los registros de Azure Monitor.

En el siguiente tutorial aprender a administrar el estado de un dispositivo IoT. 

> [!div class="nextstepaction"]
> [Configuración de dispositivos desde un servicio back-end](tutorial-device-twins.md)