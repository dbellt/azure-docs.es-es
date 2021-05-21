---
title: 'Tutorial: Uso de grupos de dispositivos en una aplicación de Azure IoT Central | Microsoft Docs'
description: 'Tutorial: aprenda a usar grupos de dispositivos para analizar los datos de telemetría de los dispositivos en la aplicación de Azure IoT Central.'
author: dominicbetts
ms.author: dobett
ms.date: 11/16/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: ce02c86fff5ccece1528e0d08413acb5c0a7f9c6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743388"
---
# <a name="tutorial-use-device-groups-to-analyze-device-telemetry"></a>Tutorial: Uso de grupos de dispositivos para analizar la telemetría de dispositivo

En este artículo se describe cómo usar grupos de dispositivos para analizar los datos de telemetría del dispositivo en la aplicación de Azure IoT Central.

Un grupo de dispositivos es una lista de dispositivos que se agrupan entre sí porque cumplen algunos criterios especificados. Los grupos de dispositivos ayudan a administrar, visualizar y analizar los dispositivos a escala, agrupando dichos dispositivos en grupos lógicos más pequeños. Por ejemplo, puede crear un grupo de dispositivos para enumerar todos los dispositivos de aire acondicionado de Seattle para que, así, un técnico de Seattle pueda encontrar los dispositivos de los que es responsable.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un grupo de dispositivos
> * Uso de un grupo de dispositivos para analizar la telemetría del dispositivo

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, debe completar los inicios rápidos [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md) e [Incorporación de un dispositivo simulado a la aplicación de IoT Central](./quick-create-simulated-device.md) para crear la plantilla de dispositivo **Sensor Controller** con la que va a trabajar.

## <a name="create-simulated-devices"></a>Creación de dispositivos simulados

Antes de crear un grupo de dispositivos, agregue al menos cinco dispositivos simulados basados en la plantilla de dispositivo **Sensor Controller** que se va a usar en este tutorial:


:::image type="content" source="media/tutorial-use-device-groups/simulated-devices.png" alt-text="Captura de pantalla que muestra cinco dispositivos simulados de Sensor Controller.":::

En cuatro de los dispositivos de sensor simulados, use la vista **Administrar dispositivo** para establecer el nombre del cliente en *Contoso* y seleccione **Guardar**.

:::image type="content" source="media/tutorial-use-device-groups/customer-name.png" alt-text="Captura de pantalla que muestra cómo establecer la propiedad en la nube nombre del cliente":::

## <a name="create-a-device-group"></a>Creación de un grupo de dispositivos

1. Seleccione **Grupos de dispositivos** en el panel izquierdo para ir a la página Grupos de dispositivos.

1. Seleccione **+ Nuevo**.

1. Asigne al grupo de dispositivos el nombre *Contoso devices*. También puede agregar una descripción. Un grupo de dispositivos solo puede contener los dispositivos de una sola plantilla de dispositivo. Elija la plantilla de dispositivo **Sensor Controller** que se va a usar para este grupo.

1. Para personalizar el grupo de dispositivos para incluir solo los dispositivos que pertenecen a **Contoso**, seleccione **+ Filtrar**. Seleccione la propiedad **Nombre del cliente**, el operador de comparación **Es igual a** y **Contoso** como valor. Puede agregar varios filtros y dispositivos que cumplan **todos** los criterios de filtro establecidos en el grupo de dispositivos. El grupo de dispositivos que cree será accesible para todas aquellas personas que tengan acceso a la aplicación, por lo que cualquier usuario podrá ver, modificar o eliminar el grupo de dispositivos.

    > [!TIP]
    > El grupo de dispositivos es una consulta dinámica. Cada vez que vea la lista de dispositivos, puede haber diferentes dispositivos en ella. La lista depende de qué dispositivos cumplen actualmente los criterios de la consulta.

1. Elija **Guardar**.

:::image type="content" source="media/tutorial-use-device-groups/device-group-query.png" alt-text="Captura de pantalla que muestra la configuración de la consulta del grupo de dispositivos":::

> [!NOTE]
> En los dispositivos de Azure IoT Edge, seleccione las plantillas de Azure IoT Edge para crear un grupo de dispositivos.

## <a name="analytics"></a>Análisis

Puede usar **Analytics** con un grupo de dispositivos para analizar la telemetría de los dispositivos del grupo. Por ejemplo, puede trazar la temperatura media notificada por todos los dispositivos Environmental Sensor de Contoso.

Para analizar la telemetría de un grupo de dispositivos:

1. Elija **Analytics** en el panel izquierdo.

1. Seleccione el grupo de dispositivos **Dispositivos de Contoso** que creó. A continuación, agregue los tipos de telemetría **Temperatura** y **Humedad**:

    :::image type="content" source="media/tutorial-use-device-groups/create-analysis.png" alt-text="Captura de pantalla que muestra los tipos de telemetría seleccionados para el análisis":::

    Use los iconos de engranaje situados junto a los tipos de telemetría para seleccionar un tipo de agregación. El valor predeterminado es **Promedio**. Use **Agrupar por** para cambiar el modo en que se muestran los datos agregados. Por ejemplo, si divide por id. del dispositivo, verá un trazado para cada dispositivo al seleccionar **Analizar**.

1. Seleccione **Analizar** para ver los valores de telemetría promedio:

    :::image type="content" source="media/tutorial-use-device-groups/view-analysis.png" alt-text="Captura de pantalla que muestra los valores medios de todos los dispositivos de Contoso":::

    Puede personalizar la vista, cambiar el período de tiempo mostrado y exportar los datos.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora que aprendió a usar grupos de dispositivos en la aplicación de Azure IoT Central, este es el paso siguiente sugerido:

> [!div class="nextstepaction"]
> [Creación de reglas de telemetría](tutorial-create-telemetry-rules.md)
