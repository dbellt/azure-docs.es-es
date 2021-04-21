---
title: 'Creación de un dispositivo simulado personalizado: Azure | Microsoft Docs'
description: En este tutorial, usará Simulación de dispositivo para crear un dispositivo simulado personalizado para su uso en las simulaciones.
author: troyhopwood
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: tutorial
ms.custom: mvc
ms.date: 10/25/2018
ms.author: troyhop
ms.openlocfilehash: ed83fdde4e005b0db76b9edf30faef5e8c732998
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714045"
---
# <a name="tutorial-create-a-custom-simulated-device"></a>Tutorial: Creación de un dispositivo simulado personalizado

En este tutorial, usará Simulación de dispositivo para crear un dispositivo simulado personalizado para su uso en las simulaciones. Para empezar a trabajar con Simulación de dispositivo, puede usar uno de los dispositivos simulados de ejemplo que se incluyen. También puede crear dispositivos simulados personalizados como se describe en este artículo. Para conocer más opciones de personalización, consulte [Create an advanced device model](iot-accelerators-device-simulation-advanced-device.md) (Creación de un modelo de dispositivo avanzado).

En este tutorial, hizo lo siguiente:

>[!div class="checklist"]
> * Ver una lista de los modelos de dispositivos simulados
> * Crear un dispositivo simulado personalizado
> * Clonar un modelo de dispositivo
> * Eliminar un modelo de dispositivo

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

Para seguir este tutorial, necesitará una instancia implementada de Simulación de dispositivo en la suscripción de Azure.

Si aún no ha implementado la simulación de dispositivo, consulte [Implementación de la simulación de dispositivo](https://github.com/Azure/azure-iot-pcs-device-simulation/blob/master/README.md) en GitHub.

## <a name="view-your-device-models"></a>Ver los modelos de dispositivo

Seleccione **Modelos de dispositivo** en la barra de menús. La página **Modelos de dispositivo** muestra todos los modelos de dispositivo disponibles en esta instancia de Simulación de dispositivo:

![Modelos de dispositivos](media/iot-accelerators-device-simulation-create-custom-device/devicemodelnav.png)

## <a name="create-a-device-model"></a>Creación de un modelo de dispositivo

Haga clic en **+ Agregar modelos de dispositivo** en la esquina superior derecha de la página:

![Incorporación del modelo de dispositivo](media/iot-accelerators-device-simulation-create-custom-device/devicemodels.png)

En este tutorial, va a crear un frigorífico simulado que envía datos de temperatura y de humedad.

Rellene el formulario con la siguiente información:

| Configuración             | Value                                                |
| ------------------- | ---------------------------------------------------- |
| Nombre del modelo de dispositivo   | Frigorífico                                         |
| Descripción del modelo   | Un frigorífico con sensores de temperatura y humedad |
| Versión             | 1.0                                                  |

> [!NOTE]
> El nombre del modelo de dispositivo debe ser único.

Haga clic en **+ Add data point** (Agregar punto de datos) para agregar los puntos de datos de temperatura y humedad con los siguientes valores:

| Punto de datos          | Comportamiento        | Valor mínimo | Valor máximo | Unidad |
| ------------------- | --------------- | --------- | --------- | ---- |
| Temperatura         | Random          | -50       | 100       | F    |
| Humedad            | Random          | 0         | 100       | %    |

Haga clic en **Guardar** para guardar el modelo de dispositivo.

![Crear modelo de dispositivo](media/iot-accelerators-device-simulation-create-custom-device/adddevicemodel.png)

Ahora ya está incluido el frigorífico en la lista de modelos de dispositivo. Puede que tenga que hacer clic en **Siguiente** para ir a otra página para ver el frigorífico.

## <a name="clone-a-device-model"></a>Clonar un modelo de dispositivo

La clonación de un modelo de dispositivo le permite crear una copia de un modelo de dispositivo existente. A continuación, puede editar la copia para satisfacer sus necesidades específicas. La clonación le permite ahorrar tiempo a la hora de crear modelos de dispositivo similares.

Para clonar un modelo de dispositivo, active la casilla situada junto al modelo y, a continuación, haga clic en **Clonar** en la barra de acciones:

![Captura de pantalla que resalta el modelo seleccionado y el botón Clonar.](media/iot-accelerators-device-simulation-create-custom-device/clonedevice.png)

## <a name="delete-a-device-model"></a>Eliminar un modelo de dispositivo

Puede eliminar cualquier modelo de dispositivo personalizado. Para eliminar un modelo de dispositivo, active la casilla situada junto al modelo y, a continuación, haga clic en **Eliminar** en la barra de acciones:

![Eliminación de un modelo de dispositivo](media/iot-accelerators-device-simulation-create-custom-device/deletedevice.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear, clonar y eliminar modelos de dispositivos personalizados. Para más información sobre los modelos de dispositivo, consulte el siguiente artículo de procedimientos:

> [!div class="nextstepaction"]
> [Creación de un modelo de dispositivo avanzado](iot-accelerators-device-simulation-advanced-device.md)