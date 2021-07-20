---
title: 'Inicio rápido: Creación y uso de una aplicación de Azure IoT Central | Microsoft Docs'
description: 'Inicio rápido: Cree una nueva aplicación de Azure IoT Central y conecte su primer dispositivo. En esta guía de inicio rápido, se usa una aplicación de smartphone de Google Play o Apple App Store como un dispositivo IoT.'
author: dominicbetts
ms.author: dobett
ms.date: 05/27/2021
ms.topic: quickstart
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 4e40c079c50ccb7f83a4e03d47cf3e9b419870ef
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112288617"
---
# <a name="quickstart---create-an-azure-iot-central-application-and-use-your-smartphone-to-send-telemetry"></a>Inicio rápido: Creación de una aplicación de Azure IoT Central y uso de un smartphone para enviar datos de telemetría

En este inicio rápido, se muestra cómo crear una aplicación de Azure IoT Central y conectar su primer dispositivo. Para empezar a trabajar rápidamente, instale una aplicación en el smartphone para que actúe como el dispositivo. La aplicación envía datos de telemetría, notifica las propiedades y responde a comandos:

:::image type="content" source="media/quick-deploy-iot-central/overview.png" alt-text="Información general sobre el escenario de inicio rápido que conecta una aplicación de smartphone a IoT Central." border="false":::

## <a name="prerequisites"></a>Prerrequisitos

Una suscripción de Azure activa. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

> [!TIP]
> Debe tener como mínimo acceso de **colaborador** en la suscripción a Azure. Si creó la suscripción usted mismo, automáticamente será un administrador con acceso suficiente. Para más información, consulte [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../../role-based-access-control/overview.md)

Un teléfono Android o iOS en el que pueda instalar una aplicación gratuita desde una de las tiendas de aplicaciones oficiales.

## <a name="create-an-application"></a>Crear una aplicación

Navegue al sitio de [Compilación Azure IoT Central](https://aka.ms/iotcentral). A continuación, inicie sesión con la cuenta personal, profesional o educativa de Microsoft asociada a la suscripción de Azure.

IoT Central proporciona varias plantillas de aplicación centradas en el sector para ayudarle a empezar a trabajar. En este inicio rápido, se usa la plantilla **Aplicación personalizada** para crear una aplicación desde cero:

1. Vaya a la página **Crear** y seleccione **Crear aplicación** en el icono **Aplicación personalizada**:

    :::image type="content" source="media/quick-deploy-iot-central/iotcentralcreate-new-application.png" alt-text="Página Compilación de una aplicación de IoT":::

1. En la página **Nueva aplicación**, asegúrese de que **Aplicación personalizada** está seleccionada en **Plantilla de aplicación**.

1. Azure IoT Central sugiere automáticamente un **nombre de aplicación** en función de la plantilla de aplicación que ha seleccionado. Escriba su propio nombre de aplicación, como *Contoso quickstart app* (Aplicación de inicio rápido de Contoso).

1. Azure IoT Central también genera un prefijo de dirección **URL** único, en función del nombre de la aplicación. Esta dirección URL se usa para tener acceso a la aplicación. Si quiere, cambie el prefijo de esta URL por algo más fácil de recordar. Esta dirección URL debe ser única.

    :::image type="content" source="media/quick-deploy-iot-central/iotcentralcreate-custom.png" alt-text="Página de creación de una aplicación de Azure IoT Central":::

1. Para esta guía de inicio rápido, deje el plan de precios establecido en **Estándar 2**.

1. Seleccione la suscripción en el desplegable **Suscripción de Azure**.

1. Seleccione la ubicación más cercana en la lista desplegable **Ubicación**.

1. Revise los Términos y Condiciones y seleccione **Crear** en la parte inferior de la página. Después de unos segundos, la aplicación de IoT Central aplicación lista para usarse:

    :::image type="content" source="media/quick-deploy-iot-central/iotcentral-application.png" alt-text="Aplicación de Azure IoT Central":::

## <a name="register-a-device"></a>Registrar un dispositivo

Para conectar un dispositivo a la aplicación de IoT Central, necesita la información de conexión. Una manera fácil de obtener esta información de conexión es registrar el dispositivo.

Para registrar el dispositivo:

1. En IoT Central, vaya a la página **Dispositivos** y seleccione **Crear un dispositivo**:

    :::image type="content" source="media/quick-deploy-iot-central/create-device.png" alt-text="Captura de pantalla que muestra la creación de un dispositivo en IoT Central.":::

1. En la página **Crear un dispositivo**, acepte los valores predeterminados y, a continuación, seleccione **Crear**.

1. En la lista de dispositivos, haga clic en el nombre de dispositivo.

    :::image type="content" source="media/quick-deploy-iot-central/device-name.png" alt-text="Captura de pantalla que muestra el nombre del dispositivo resaltado que puede seleccionar.":::

1. En la página del dispositivo, seleccione **Conectar** y, a continuación, seleccione **Código QR**:

    :::image type="content" source="media/quick-deploy-iot-central/device-registration.png" alt-text="Captura de pantalla que muestra el código QR que puede usar para conectar la aplicación del teléfono.":::

Mantenga esta página abierta. En la sección siguiente, examinará este código QR mediante la aplicación del teléfono para conectarlo a IoT Central.

## <a name="connect-your-device"></a>Conexión del dispositivo

Para empezar a trabajar rápidamente, en este artículo se usa la aplicación de smartphone **IoT Plug and Play** como un dispositivo IoT. La aplicación envía los datos de telemetría recopilados de los sensores del teléfono, responde a los comandos invocados desde IoT Central e informa de los valores de las propiedades a IoT Central.

[!INCLUDE [iot-phoneapp-install](../../../includes/iot-phoneapp-install.md)]

Para conectar la aplicación **IoT Plug and Play** a la aplicación de IoT Central:

1. Abra la aplicación **IoT PnP** en el smartphone.

1. En la página principal, seleccione **Scan QR code** (Escanear código QR). Apunte la cámara del teléfono al código QR. A continuación, espere unos segundos mientras se establece la conexión.

1. En la página de telemetría de la aplicación, puede ver los datos que la aplicación envía a IoT Central. En la página de registros, puede ver el dispositivo que se conecta y varios mensajes de inicialización.

Para ver los datos de telemetría de la aplicación de smartphone en IoT Central:

1. En IoT Central, vaya a la página **Dispositivos**.

1. En la lista de dispositivos, haga clic en el nombre del dispositivo y seleccione **Información general**:

    :::image type="content" source="media/quick-deploy-iot-central/iotcentral-telemetry.png" alt-text="Captura de pantalla de la página de información general con trazados de telemetría.":::

> [!TIP]
> La aplicación de smartphone solo envía datos cuando la pantalla está activada.
## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una aplicación de IoT Central y ha conectado un dispositivo que envía datos de telemetría. En este inicio rápido, ha usado una aplicación de smartphone como dispositivo IoT que se conecta a IoT Central. Este es el siguiente paso sugerido para obtener más información sobre IoT Central:

> [!div class="nextstepaction"]
> [Inicio rápido: Configuración de reglas y acciones para el dispositivo en Azure IoT Central](./quick-configure-rules.md)
