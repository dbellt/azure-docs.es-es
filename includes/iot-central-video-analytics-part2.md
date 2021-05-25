---
title: archivo de inclusión
description: archivo de inclusión
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 10/06/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: e99dcb0d7edaf79356f54c930e24e35153c26c3f
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109718886"
---
## <a name="deploy-and-configure-azure-media-services"></a>Implementación y configuración de Azure Media Services

La solución usa una cuenta de Azure Media Services para almacenar los clips de vídeo de detección de objetos realizados por el dispositivo de puerta de enlace IoT Edge.

Al crear la cuenta de Media Services:

- Debe proporcionar un nombre de cuenta, una suscripción de Azure, una ubicación, un grupo de recursos y una cuenta de almacenamiento. Cree una nueva cuenta de almacenamiento con la configuración predeterminada mientras crea la cuenta de Media Services.

- Elija la región **Este de EE. UU.** para la ubicación.

- Cree un nuevo grupo de recursos llamado *lva-rg* en la región **Este de EE. UU.** para las cuentas de Media Services y de almacenamiento. Cuando termine los tutoriales, será fácil la limpieza todos los recursos mediante la eliminación el grupo de recursos *lva-rg*.

Cree la [cuenta de Media Services en Azure Portal](https://portal.azure.com/?r=1#create/Microsoft.MediaService).

> [!TIP]
> Estos tutoriales usan la región **Este de EE. UU.** en todos los ejemplos. Si lo prefiere, puede usar la región más cercana.

Anote el nombre de la cuenta de **Media Services** en el archivo *scratchpad.txt*.

Una vez completada la implementación, abra Cloud Shell y ejecute el siguiente comando para recuperar el valor de **Id. de recurso** de la cuenta de Media Services:

```azurecli
az resource list --resource-group lva-rg --resource-type microsoft.media/mediaservices --output table --query "[].{ResourceID:id}"
```

:::image type="content" source="media/iot-central-video-analytics-part2/get-resource-id.png" alt-text="Uso de Cloud Shell para obtener el identificador de recurso.":::

Anote el valor de **Id. del recurso** en el archivo *scratchpad.txt*, usará este valor más adelante al configurar el módulo IoT Edge.

A continuación, configure una entidad de servicio de Azure Active Directory para el recurso de Media Services. Seleccione **Acceso de API** y, a continuación, seleccione **Autenticación de la entidad de servicio**. Cree una nueva aplicación de Azure Active Directory con el mismo nombre que el recurso de Media Services y cree un secreto con la descripción *IoT Edge Access* (Acceso a IoT Edge).

:::image type="content" source="./media/iot-central-video-analytics-part2/media-service-authentication.png" alt-text="Configuración de la aplicación de Azure A D para Azure Media Services":::

Cuando se genere el secreto, desplácese hacia abajo hasta la sección **Copie las credenciales para conectarse a la aplicación de la entidad de servicio.** . A continuación, seleccione **JSON**. Puede copiar toda la información de credenciales desde aquí en un solo paso. Anote esta información en el archivo *scratchpad.txt*, la usará más adelante al configurar el módulo IoT Edge.

> [!WARNING]
> Esta es la única oportunidad de ver y guardar el secreto. Si lo pierde, tendrá que generar otro secreto.

## <a name="create-the-azure-iot-central-application"></a>Creación de la aplicación de Azure IoT Central

En esta sección, creará una nueva aplicación de Azure IoT Central a partir de una plantilla. Usará esta aplicación a lo largo de la serie de tutoriales para crear una solución completa.

Para crear una nueva aplicación de Azure IoT Central:

1. Vaya al sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral).

1. Inicie sesión con las credenciales que usa para acceder a la suscripción de Azure.

1. Para empezar a crear una nueva aplicación de Azure IoT Central, seleccione **New Application** (Nueva aplicación) en la página **Build** (Compilar).

1. Seleccione **Retail** (Comercio minorista). En la página de comercio minorista se muestran varias plantillas de aplicación de venta al por menor.

Para crear una nueva aplicación de análisis de vídeo:

1. Seleccione la plantilla de aplicación **Video analytics - object and motion detection** (Análisis de vídeo: detección de objetos y movimiento). Esta plantilla incluye plantillas de dispositivo para los dispositivos usados en el tutorial. La plantilla incluye paneles de ejemplo que los operadores pueden usar para realizar tareas como la supervisión y la administración de las cámaras.

1. Opcionalmente, elija un valor descriptivo para **Application name** (Nombre de la aplicación). Esta aplicación se basa en un comercio minorista ficticio llamado Northwind Traders. En el tutorial se usa el **Nombre de aplicación** *Northwind Traders video analytics*.

    > [!NOTE]
    > Aunque use un **Nombre de aplicación** descriptivo, debe usar un valor único para la **dirección URL** de la aplicación.

1. Si tiene una suscripción de Azure, seleccione el **Directorio**, la **suscripción de Azure** y **United States** (Estados Unidos) para la **Location** (Ubicación). Si no tiene una suscripción, puede activar **7-day free trial** (Evaluación gratuita de 7 días) y completar la información de contacto necesaria. En este tutorial se usan tres dispositivos: dos cámaras y un dispositivo IoT Edge, por lo que si no usa la evaluación gratuita, se le facturará el uso.

    Para más información sobre los directorios, las suscripciones y las ubicaciones, consulte [Creación de una aplicación de IoT Central](../articles/iot-central/core/howto-create-iot-central-application.md).

1. Seleccione **Crear**.

    :::image type="content" source="./media/iot-central-video-analytics-part2/new-application.png" alt-text="Página de creación de una aplicación de Azure IoT Central":::

### <a name="retrieve-the-configuration-data"></a>Recuperación de los datos de configuración

Más adelante en este tutorial, al configurar la puerta de enlace IoT Edge, necesitará algunos datos de configuración de la aplicación de IoT Central. El dispositivo IoT Edge necesita esta información para registrarse y conectarse a la aplicación.

En la sección **Administration** (Administración), seleccione **su aplicación** y anote el valor de **Application URL** (Dirección URL de la aplicación) y de **Application ID** (Id. de la aplicación) en el archivo *scratchpad.txt*:

:::image type="content" source="./media/iot-central-video-analytics-part2/administration.png" alt-text="La captura de pantalla muestra el panel de administración de una página de análisis de vídeo, con la dirección URL y el identificador de aplicación resaltados.":::

Seleccione **API Tokens** (Tokens de API) y genere un nuevo token llamado **LVAEdgeToken** para el rol **Operator** (Operador):

:::image type="content" source="./media/iot-central-video-analytics-part2/token.png" alt-text="Generar token":::

Anote el token en el archivo *scratchpad.txt* para más adelante. Cuando el cuadro de diálogo se cierre, no podrá volver a ver el token.

En la sección **Administration** (Administración), seleccione **Device connection** (Conexión del dispositivo) y, a continuación, seleccione **SAS-IoT-Devices** (Dispositivos-IoT-SAS).

Anote el valor de **Primary key** (Clave principal) de los dispositivos en el archivo *scratchpad.txt*. Usará este *token de firma de acceso compartido de grupo principal* más adelante cuando configure el dispositivo IoT Edge.
