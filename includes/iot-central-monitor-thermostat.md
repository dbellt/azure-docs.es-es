---
title: archivo de inclusión
description: archivo de inclusión
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 03/12/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 26aefab9b4637c3f3f6b0f0d1195068e2cd770ab
ms.sourcegitcommit: 12f15775e64e7a10a5daebcc52154370f3e6fa0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108001831"
---
<!-- All needs updating -->
Como operador de la aplicación de Azure IoT Central, puede:

* Ver la telemetría enviada por los dos componentes del termostato en la página **Overview** (Información general):

    :::image type="content" source="media/iot-central-monitor-thermostat/view-telemetry.png" alt-text="Visualización de datos de telemetría de dispositivo":::

* Consulte las propiedades del dispositivo en la página **About** (Acerca de). En esta página se muestran las propiedades del componente de información del dispositivo y los dos componentes del termostato:

    :::image type="content" source="media/iot-central-monitor-thermostat/about-properties.png" alt-text="Visualización de las propiedades del dispositivo":::

## <a name="customize-the-device-template"></a>Personalización de la plantilla de dispositivo

Como desarrollador de soluciones, puede personalizar la plantilla de dispositivo que IoT Central creó automáticamente al conectar el dispositivo de control de la temperatura.

Para agregar una propiedad de nube para almacenar el nombre de cliente asociado al dispositivo:

1. En la aplicación de IoT Central, vaya a la plantilla del dispositivo **Temperature Controller** (Controlador de temperatura) en la página **Device templates** (Plantillas de dispositivo).

1. Seleccione **Cloud Properties** (Propiedades de la nube) en la plantilla del dispositivo **Temperature Controller** (Controlador de temperatura).

1. Seleccione **Add Cloud Property** (Agregar propiedad de la nube). Especifique un valor en *Customer name* (Nombre del cliente) que será el **nombre para mostrar** y elija **Cadena** como el valor de **esquema**. Después, seleccione **Guardar**.

Para personalizar cómo se muestran los comandos **Get Max-Min report** cen la aplicación de IoT Central:

1. En la plantilla de dispositivo, seleccione **Customize** (Personalizar).

1. Para **getMaxMinReport (thermostat1)** , reemplace *Get Max-Min report* por *Get thermostat1 status report*.

1. Para **getMaxMinReport (thermostat2)** , reemplace *Get Max-Min report* por *Get thermostat2 status report*.

1. Seleccione **Guardar**.

Para personalizar cómo se muestran las propiedades de escritura **Target Temperature** en la aplicación de IoT Central:

1. En la plantilla de dispositivo, seleccione **Customize** (Personalizar).

1. Para **targetTemperature (thermostat1)** , reemplace *Target Temperature* por *Target Temperature (1)* .

1. Para **targetTemperature (thermostat2)** , reemplace *Target Temperature* por *Target Temperature (2)* .

1. Seleccione **Guardar**.

Los componentes del termostato del modelo **Thermostat Controller** (Controlador del termostato) incluyen la propiedad de escritura **Target Temperature**; la plantilla de dispositivo incluye la propiedad de nube **Customer Name**. Cree una vista que pueda usar el operador para editar estas propiedades:

1. Seleccione **Views** (Vistas) y el icono **Editing device and cloud data** (Edición de los datos del dispositivo y de la nube).

1. Escriba _Properties_ (Propiedades) como nombre del formulario.

1. Seleccione las propiedades **Target Temperature (1)** , **Target Temperature (2)** y **Customer Name**. Después, seleccione **Add section** (Agregar sección).

1. Guarde los cambios.

:::image type="content" source="media/iot-central-monitor-thermostat/properties-view.png" alt-text="Vista para actualizar los valores de propiedad":::

## <a name="publish-the-device-template"></a>Publicación de la plantilla de dispositivo

Para que un operador pueda ver y usar las personalizaciones realizadas, debe publicar la plantilla de dispositivo.

Seleccione **Publish** (Publicar) en la plantilla del dispositivo **Thermostat**. En el panel **Publicar esta plantilla de dispositivo en la aplicación**, seleccione **Publicar**.

Ahora los operadores podrán usar la vista **Properties** (Propiedades) para actualizar los valores de propiedad y llamar a comandos denominados **Get thermostat1 status report** y **Get thermostat2 status report** en la página de comandos del dispositivo:

* Actualizar los valores de las propiedades con posibilidad de escritura en la página **Properties** (Propiedades):

    :::image type="content" source="media/iot-central-monitor-thermostat/update-properties.png" alt-text="Actualización de las propiedades del dispositivo.":::

* Llame al comando desde la página **Commands** (Comandos). Si ejecuta el comando de informe de estado, seleccione una fecha y hora para el parámetro **Desde** antes de la ejecución del comando:

    :::image type="content" source="media/iot-central-monitor-thermostat/call-command.png" alt-text="Llamada al comando.":::

    :::image type="content" source="media/iot-central-monitor-thermostat/command-response.png" alt-text="Visualización de la respuesta del comando.":::
