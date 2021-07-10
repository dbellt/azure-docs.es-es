---
title: 'Tutorial: Implementación del análisis espacial de IoT | Microsoft Azure Maps'
description: Tutorial sobre cómo integrar IoT Hub con las API del servicio Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/01/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 37aa8c954f847002ad69fa17ee1f025049ec9bb6
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110785781"
---
# <a name="tutorial-implement-iot-spatial-analytics-by-using-azure-maps"></a>Tutorial: Implementación del análisis espacial de IoT mediante Azure Maps

En un escenario de IoT, son típicos el seguimiento y la captura de los eventos pertinentes que se producen en el espacio y en el tiempo. Entre los ejemplos, se incluyen aplicaciones de administración de flotas, seguimiento de activos, movilidad y ciudad inteligente. Este tutorial le guía por una solución que supervisa la circulación de vehículos de alquiler usados mediante la utilización de las API de Azure Maps.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Crear una cuenta de Azure Storage para registrar los datos de seguimiento de automóviles.
> * Cargar una geovalla en el servicio Azure Maps Data mediante Data Upload API.
> * Crear un centro en Azure IoT Hub y registrar un dispositivo.
> * Crear una función en Azure Functions, para lo que debe implementar una lógica de negocios basada en el análisis espacial de Azure Maps.
> * Suscribirse a eventos de telemetría de dispositivos IoT desde la función de Azure a través de Azure Event Grid.
> * Filtrar los eventos de telemetría mediante el enrutamiento de mensajes de IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).

3. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción. Para obtener más información, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

4. [Cree un grupo de recursos](../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups). En este tutorial, denominaremos el grupo de recursos *ContosoRental*, pero puede elegir el nombre que quiera.

5. Descargue el [proyecto rentalCarSimulation de C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation).

En este tutorial se usa la aplicación [Postman](https://www.postman.com/), pero se puede elegir otro entorno de desarrollo de API.

## <a name="use-case-rental-car-tracking"></a>Caso de uso: seguimiento de automóviles de alquiler

Supongamos que una empresa de alquiler de vehículos desea registrar la información de la ubicación, la distancia recorrida y el estado operativo de sus vehículos de alquiler. Además, la empresa desea almacenar esta información cada vez que un automóvil abandone la región geográfica autorizada correcta.

Los vehículos de alquiler están equipados con dispositivos IoT que envían periódicamente datos de telemetría a IoT Hub. Los datos de telemetría incluyen la ubicación actual e indican si el motor del vehículo está en funcionamiento. El esquema de ubicación del dispositivo cumple el [esquema IoT Plug and Play para datos geoespaciales](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v1-preview/schemas/geospatial.md). El esquema de los datos de telemetría del dispositivo del vehículo de alquiler tiene un aspecto similar al código JSON siguiente:

```JSON
{
    "data": {
        "properties": {
            "Engine": "ON"
        },
        "systemProperties": {
            "iothub-content-type": "application/json",
            "iothub-content-encoding": "utf-8",
            "iothub-connection-device-id": "ContosoRentalDevice",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "636959817064335548",
            "iothub-enqueuedtime": "2019-06-18T00:17:20.608Z",
            "iothub-message-source": "Telemetry"
        },
        "body": {
            "location": {
                "type": "Point",
                "coordinates": [ -77.025988698005662, 38.9015330523316 ]
            }
        }
    }
}
```

En este tutorial, solo se hará el seguimiento de un vehículo. Después de configurar los servicios de Azure, debe descargar el [proyecto rentalCarSimulation de C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation) para ejecutar el simulador de vehículos. El proceso completo, desde el evento hasta la ejecución de la función, se resume en los pasos siguientes:

1. El dispositivo del vehículo envía los datos de telemetría a IoT Hub.

2. Si el motor del automóvil está en marcha, el centro publica los datos de telemetría en Event Grid.

3. Una función de Azure se desencadena debido a su suscripción de eventos a los eventos de telemetría de dispositivo.

4. La función registra las coordenadas de ubicación del dispositivo del vehículo, la hora del evento y el identificador del dispositivo. A continuación, usará [Spatial Geofence Get API](/rest/api/maps/spatial/getgeofence) para determinar si el automóvil ha circulado fuera de la geovalla. Si ha circulado fuera de los límites de la geovalla, la función almacenará los datos de ubicación recibidos del evento en un contenedor de blobs. La función también consulta [Get Search Address Reverse](/rest/api/maps/search/getsearchaddressreverse) para convertir las coordenadas en una dirección y almacenarla con los demás datos de ubicación del dispositivo.

En el siguiente diagrama se proporciona información general sobre el sistema.

   :::image type="content" source="./media/tutorial-iot-hub-maps/system-diagram.png" border="false" alt-text="Diagrama de información general del sistema.":::

La siguiente ilustración resalta el área de geovalla en azul. La ruta del vehículo de alquiler se indica mediante una línea verde.

   :::image type="content" source="./media/tutorial-iot-hub-maps/geofence-route.png" border="false" alt-text="Figura que muestra la ruta de geovalla.":::

## <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

Para almacenar datos de seguimiento de infracciones de automóvil, cree una [cuenta de almacenamiento de uso general v2](../storage/common/storage-account-overview.md) en el grupo de recursos. Si no ha creado un grupo de recursos, siga las instrucciones que se indican en [Crear grupos de recursos](../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups). En este tutorial, el grupo de recursos se llamará *ContosoRental*.

Para crear una cuenta de almacenamiento, siga las instrucciones de [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md?tabs=azure-portal). En este tutorial, la cuenta de almacenamiento se llamará *contosorentalstorage*, pero puede asignarle el nombre que quiera.

Una vez que haya creado la cuenta de almacenamiento correctamente, es necesario crear un contenedor para almacenar los datos de registro.

1. Vaya a la cuenta de almacenamiento recién creada. En la sección **Essentials**, haga clic en el vínculo **Contenedores**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/containers.png" alt-text="Captura de pantalla de los contenedores para el almacenamiento de blobs.":::

2. En la esquina superior izquierda, seleccione **+ Container** (+ Contenedor). Aparece un panel en el lado derecho del explorador. Asigne al contenedor el nombre *contoso-rental-logs* y seleccione **Crear**.

     :::image type="content" source="./media/tutorial-iot-hub-maps/container-new.png" alt-text="Captura de pantalla del cuadro de diálogo de creación de un contenedor de blobs.":::

3. Vaya al panel **Claves de acceso** de la cuenta de almacenamiento y copie los valores de **Nombre de la cuenta de almacenamiento** y **Clave** en la sección **key1**. Necesitará ambos valores en la sección "Creación de una función de Azure y adición de una suscripción a Event Grid".

    :::image type="content" source="./media/tutorial-iot-hub-maps/access-keys.png" alt-text="Captura de pantalla de la copia del nombre y la clave de la cuenta de almacenamiento.":::

## <a name="upload-a-geofence"></a>Carga de una geovalla

A continuación, use la [aplicación Postman](https://www.getpostman.com) para [cargar la geovalla](./geofence-geojson.md) en Azure Maps. La geovalla define el área geográfica autorizada para nuestro vehículo de alquiler. La geovalla se usará en la función de Azure para determinar si un automóvil se ha salido del área de la geovalla.

Siga estos pasos para cargar la geovalla mediante Upload API de Azure Maps: 

1. Abra la aplicación Postman y seleccione **New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección). Asigne un nombre a la colección y seleccione **Create** (Crear).

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create new** (Crear nuevo), seleccione **Request** (Solicitud) y escriba un nombre para la solicitud. Seleccione la colección que creó en el paso anterior y haga clic en **Save** (Guardar).

3. Seleccione el método HTTP **POST** en la pestaña del generador y escriba la siguiente dirección URL para cargar la geovalla en Data Upload API. Asegúrese de reemplazar `{subscription-key}` por la clave de suscripción principal.

    ```HTTP
    https://us.atlas.microsoft.com/mapData?subscription-key={subscription-key}&api-version=2.0&dataFormat=geojson
    ```

    En la ruta de acceso URL, el valor `geojson` del parámetro `dataFormat` representa el formato de los datos que se están cargando.

4. Seleccione **Body** > **raw** (Cuerpo > sin procesar) para el formato de entrada y elija **JSON** en la lista desplegable. [Abra el archivo de datos JSON](https://raw.githubusercontent.com/Azure-Samples/iothub-to-azure-maps-geofencing/master/src/Data/geofence.json?token=AKD25BYJYKDJBJ55PT62N4C5LRNN4) y cópielo en la sección del cuerpo. Haga clic en **Send** (Enviar).

5. Seleccione **Send** (Enviar) y espere a que se procese la solicitud. Una vez finalizada la solicitud, vaya a la pestaña **Headers** (Encabezados) de la respuesta. Copie el valor de la clave de **Operation-Location**, que es `status URL`.

    ```http
    https://us.atlas.microsoft.com/mapData/operations/<operationId>?api-version=2.0
    ```

6. Para comprobar el estado de la llamada de API, cree una solicitud HTTP **GET** en el elemento `status URL`. Tendrá que anexar la clave de suscripción principal a la dirección URL para realizar la autenticación. La solicitud **GET** debe ser como la siguiente dirección URL:

   ```HTTP
   https://us.atlas.microsoft.com/mapData/<operationId>/status?api-version=2.0&subscription-key={subscription-key}
   ```

7. Cuando la solicitud se complete correctamente, seleccione la pestaña **Encabezados** en la ventana de respuesta. Copie el valor de la clave **Resource-Location**, que es `resource location URL`.  `resource location URL` contiene el identificador único (`udid`) de los datos cargados. Copie la `udid` para su uso posterior en este tutorial.

    :::image type="content" source="./media/tutorial-iot-hub-maps/resource-location-url.png" alt-text="Copie la dirección URL de la ubicación del recurso.":::

## <a name="create-an-iot-hub"></a>Creación de un centro de IoT

IoT Hub permite una comunicación bidireccional confiable y segura entre una aplicación de IoT y los dispositivos que administra. Este tutorial tiene como finalidad obtener información del dispositivo en el vehículo para determinar la ubicación del vehículo de alquiler. En esta sección, va a crear un centro de IoT en el grupo de recursos *ContosoRental*. Este centro será responsable de publicar los eventos de telemetría de su dispositivo.

> [!NOTE]
> La característica para publicar eventos de telemetría del dispositivo en Event Grid se encuentra actualmente en versión preliminar. Está característica está disponible en todas las regiones, salvo en las siguientes: Este de EE. UU., Oeste de EE. UU., Oeste de Europa, Azure Government, Azure China 21Vianet y Azure Alemania.

Para crear un centro de IoT en el grupo de recursos *ContosoRental*, siga los pasos descritos en [Creación de un centro de IoT](../iot-hub/quickstart-send-telemetry-dotnet.md#create-an-iot-hub).

## <a name="register-a-device-in-your-iot-hub"></a>Registro de un dispositivo en su centro de IoT

Los dispositivos no pueden conectarse al centro de IoT a menos que estén registrados en el registro de identidades del centro de IoT. En este caso, va a crear un único dispositivo denominado *InVehicleDevice*. Para crear y registrar el dispositivo en el centro de IoT, siga los pasos descritos en [Registro de un nuevo dispositivo en el centro de IoT](../iot-hub/iot-hub-create-through-portal.md#register-a-new-device-in-the-iot-hub). Asegúrese de copiar la cadena de conexión principal del dispositivo. Lo necesitará más adelante.

## <a name="create-a-function-and-add-an-event-grid-subscription"></a>Creación de una función y adición de una suscripción a Event Grid

Azure Functions es un servicio de proceso sin servidor que permite ejecutar pequeños fragmentos de código ("funciones") sin necesidad de aprovisionar ni administrar explícitamente la infraestructura de proceso. Para más información, consulte [Azure Functions](../azure-functions/functions-overview.md).

Una función la desencadena un determinado evento. En este caso, va a crear una función desencadenada por Event Grid. Para establecer la relación entre el desencadenador y la función, cree una suscripción a los eventos de telemetría del dispositivo del centro de IoT. Cuando se produce un evento de telemetría del dispositivo, se llama a la función como un punto de conexión y recibe los datos pertinentes para el dispositivo que se registró anteriormente en el centro de IoT.

Este es el [código de script de C# que contendrá la función](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx).

Configure ahora la función de Azure.

1. En el panel de Azure Portal, seleccione **Crear un recurso**. Escriba **Function App** en el cuadro de búsqueda. Seleccione **Function App** > **Create** (Aplicación de funciones > Crear).

1. En la página de creación de la **Aplicación de funciones**, asigne un nombre a la aplicación de funciones. En **Grupo de recursos**, seleccione **ContosoRental** en la lista desplegable. Seleccione **.NET Core** como la **pila en tiempo de ejecución**. En la parte inferior de la página, seleccione **Next: Hosting >** (Siguiente > Hospedaje).

    :::image type="content" source="./media/tutorial-iot-hub-maps/rental-app.png" alt-text="Captura de pantalla de la creación de una aplicación de funciones.":::

1. En **Cuenta de almacenamiento**, seleccione la cuenta de almacenamiento que creó en [Crear una cuenta de almacenamiento](#create-an-azure-storage-account). Seleccione **Revisar + crear**.

1. Revise los detalles de la aplicación de funciones y seleccione **Create**.

1. Una vez creada la aplicación, es preciso agregarle una función. Vaya a la aplicación de función. Seleccione el panel **Funciones**. En la parte superior de la página, seleccione **+ Agregar**. Aparecerá el panel de plantillas de función. Desplácese hacia abajo por el panel y seleccione **Azure Event Grid Trigger** (Desencadenador de Azure Event Grid).

     >[!IMPORTANT]
    > Las plantillas del **desencadenador de Azure Event Hub** y el **desencadenador de Azure Event Grid** tienen nombres similares. Asegúrese de seleccionar la plantilla **Azure Event Grid Trigger** (Desencadenador de Azure Event Grid).

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create.png" alt-text="Captura de pantalla de la creación de una función.":::

1. Asigne un nombre a la función. En este tutorial, usará el nombre *GetGeoFunction*, pero puede emplear otro si lo desea. Seleccione **Crear función**.

1. En el menú de la izquierda, seleccione el panel **Código + prueba**. Copie y pegue el [script de C#](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/blob/master/src/Azure%20Function/run.csx) en la ventana de código.

     :::image type="content" source="./media/tutorial-iot-hub-maps/function-code.png" alt-text="Captura de pantalla de la copia y pegado de código en la ventana de función.":::

1. En el código de C#, reemplace los siguientes parámetros:
    * Reemplace **SUBSCRIPTION_KEY** por la clave de suscripción principal de su cuenta de Azure Maps.
    * Reemplace **UDID** por el `udid` de la geovalla que cargó en [Carga de geovalla](#upload-a-geofence).
    * La función `CreateBlobAsync` del script crea un blob por evento en la cuenta de almacenamiento de datos. Reemplace **ACCESS_KEY**, **ACCOUNT_NAME** y **STORAGE_CONTAINER_NAME** por la clave de acceso de la cuenta de almacenamiento, el nombre de la cuenta y el contenedor de almacenamiento de datos. Estos valores se generaron al crear la cuenta de almacenamiento en [Crear una cuenta de almacenamiento de Azure](#create-an-azure-storage-account).

1. En el menú de la izquierda, seleccione el panel **Integración**. Seleccione **Desencadenador de Event Grid** en el diagrama. Escriba un nombre para el desencadenador, *eventGridEvent*, y seleccione **Crear suscripción de Event Grid**.

     :::image type="content" source="./media/tutorial-iot-hub-maps/function-integration.png" alt-text="Captura de pantalla de la adición de una suscripción a eventos.":::

1. Rellene los detalles de la suscripción. Asigne un nombre a la suscripción del evento. En **Esquema de eventos**, seleccione **Esquema de Event Grid**. En **Tipos de tema**, seleccione **Cuentas de Azure IoT Hub**. En **Grupos de recursos**, seleccione el grupo de recursos que creó al principio de este tutorial. En **Recurso**, seleccione el centro de IoT que creó en "Creación de un centro de IoT". En **Filtro para tipos de evento**, seleccione **Telemetría del dispositivo**.

   Después de elegir estas opciones, verá que **Tipo de tema** cambia a **IoT Hub**. En **Nombre del tema del sistema**, puede usar el mismo nombre del recurso. Por último, en la sección **Detalles del punto de conexión**, elija **Selección de un punto de conexión**. Acepte toda la configuración y elija **Confirmar selección**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create-event-subscription.png" alt-text="Captura de pantalla de la creación de una suscripción a eventos.":::

1. Revise la configuración. Asegúrese de que el punto de conexión especifica la función que creó al principio de esta sección. Seleccione **Crear**.

    :::image type="content" source="./media/tutorial-iot-hub-maps/function-create-event-subscription-confirm.png" alt-text="Captura de pantalla de confirmación de la creación de una suscripción de eventos.":::

1. Ahora vuelva al panel **Editar desencadenador**. Seleccione **Guardar**.

## <a name="filter-events-by-using-iot-hub-message-routing"></a>Filtrado de eventos mediante el enrutamiento de mensajes de IoT Hub

Cuando se agrega una suscripción a Event Grid para la función de Azure, se crea automáticamente una ruta de mensajería en el centro de IoT especificado. El enrutamiento de mensajes le permite enrutar distintos tipos de datos a varios puntos de conexión. Por ejemplo, puede enrutar los mensajes de telemetría del dispositivo, los eventos del ciclo de vida del dispositivo y los eventos de cambio de dispositivo gemelo. Para más información, consulte [Uso del enrutamiento de mensajes de IoT Hub](../iot-hub/iot-hub-devguide-messages-d2c.md).

:::image type="content" source="./media/tutorial-iot-hub-maps/hub-route.png" alt-text="Captura de pantalla del enrutamiento de mensajes en IoT Hub.":::

En el escenario de ejemplo, solo desea recibir mensajes cuando el vehículo de alquiler está circulando. Cree una consulta de enrutamiento para filtrar los eventos en los que la propiedad `Engine` sea igual a **"ON"** . Para crear una consulta de enrutamiento, seleccione la ruta **RouteToEventGrid** y reemplace la **consulta de enrutamiento** por **"Engine='ON'"** . Después, seleccione **Guardar**. Ahora, el centro de IoT solo publicará los datos de telemetría del dispositivo cuando el valor para Engine sea ON.

:::image type="content" source="./media/tutorial-iot-hub-maps/hub-filter.png" alt-text="Captura de pantalla del filtro de mensajes de enrutamiento.":::

>[!TIP]
>Hay varias maneras de consultar mensajes del dispositivo a la nube de IoT. Para obtener más información sobre la sintaxis del enrutamiento de mensajes, consulte [Enrutamiento de mensajes de IoT Hub](../iot-hub/iot-hub-devguide-routing-query-syntax.md).

## <a name="send-telemetry-data-to-iot-hub"></a>Envío de datos de telemetría a IoT Hub

Cuando la función de Azure esté ejecutándose, podrá enviar los datos de telemetría al centro de IoT, que los enrutará a Event Grid. Use una aplicación de C# para simular los datos de ubicación de un dispositivo instalado en un vehículo de alquiler. Para ejecutar la aplicación, se necesita el SDK de .NET Core 2.1.0, o una versión posterior, en el equipo de desarrollo. Siga los pasos que se indican a continuación para enviar datos de telemetría simulados al centro de IoT.

1. Si todavía no lo ha hecho, descargue el proyecto [rentalCarSimulation](https://github.com/Azure-Samples/iothub-to-azure-maps-geofencing/tree/master/src/rentalCarSimulation) de C#.

2. Abra el archivo `connectionString` en un editor de texto de su elección, reemplace el valor de `simulatedCar.cs` por el que guardó al registrar el dispositivo. Guarde los cambios realizados en el archivo.

3. Asegúrese de que tiene .NET Core instalado en la máquina. En la ventana del terminal local, vaya a la carpeta raíz del proyecto de C# y ejecute el siguiente comando para instalar los paquetes necesarios para la aplicación de dispositivo simulado:

    ```cmd/sh
    dotnet restore
    ```

4. En el mismo terminal, ejecute el comando siguiente para compilar la aplicación de simulación de automóvil de alquiler y ejecutarla:

    ```cmd/sh
    dotnet run
    ```


  El terminal local tendrá el aspecto siguiente.

:::image type="content" source="./media/tutorial-iot-hub-maps/terminal.png" alt-text="Captura de pantalla de la salida del terminal.":::

Si abre el contenedor de almacenamiento de blobs ahora, puede ver cuatro blobs, que corresponden a las ubicaciones en las que el vehículo estuvo fuera de la geovalla.

:::image type="content" source="./media/tutorial-iot-hub-maps/blob.png" alt-text="Captura de pantalla de visualización de blobs dentro del contenedor.":::

En el mapa siguiente se muestran cuatro puntos de ubicación del vehículo fuera de la geovalla. Cada ubicación se registró en intervalos de tiempo regulares.

:::image type="content" source="./media/tutorial-iot-hub-maps/violation-map.png" alt-text="Captura de pantalla del mapa de infracciones.":::

## <a name="explore-azure-maps-and-iot"></a>Explorar Azure Maps e IoT

Para explorar las API de Azure Maps que se usan en este tutorial, consulte:

* [Obtener la dirección de búsqueda inversa](/rest/api/maps/search/getsearchaddressreverse)
* [Get Geofence](/rest/api/maps/spatial/getgeofence)

Para obtener una lista completa de las API REST de Azure Maps, consulte:

* [API REST de Azure Maps](/rest/api/maps/spatial/getgeofence)

* [IoT Plug and Play](../iot-pnp/index.yml)

Para obtener una lista de los dispositivos que tienen la certificación de Azure para IoT, visite:

* [Dispositivos certificados de Azure](https://devicecatalog.azure.com/)

## <a name="clean-up-resources"></a>Limpieza de recursos

No hay recursos que requieran limpieza.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo enviar datos de telemetría del dispositivo a la nube y viceversa, consulte:


> [!div class="nextstepaction"]
> [Envío de datos de telemetría desde un dispositivo](../iot-hub/quickstart-send-telemetry-dotnet.md)