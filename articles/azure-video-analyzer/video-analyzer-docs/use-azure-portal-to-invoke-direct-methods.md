---
title: Uso de Azure Portal para invocar métodos directos para Azure Video Analyzer
description: Este artículo es una introducción al uso de Azure Portal para invocar métodos directos para Azure Video Analyzer.
ms.topic: how-to
ms.date: 03/31/2021
ms.openlocfilehash: c41c1b3c901a1ca3a83695994d7c9570e723f8a7
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388709"
---
# <a name="use-azure-portal-to-invoke-direct-methods-for-azure-video-analyzer"></a>Uso de Azure Portal para invocar métodos directos para Azure Video Analyzer

IoT Hub permite invocar [métodos directos](../../iot-hub/iot-hub-devguide-direct-methods.md#method-invocation-for-iot-edge-modules) en dispositivos perimetrales desde la nube. El módulo Azure Video Analyzer (Video Analyzer) expone varios [métodos directos](./direct-methods.md) que están disponibles para definir, implementar y activar distintas canalizaciones para el análisis de vídeo en directo.

En este artículo, obtendrá información sobre cómo invocar llamadas de métodos directos en el módulo Video Analyzer a través de Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

* El módulo Video Analyzer tiene que estar en ejecución en su dispositivo perimetral mediante uno de los métodos descritos en [Inicio rápido: Video Analyzer](get-started-detect-motion-emit-events.md) o mediante el [portal](./deploy-iot-edge-device.md).
* Debe entender [Video Analyzer](overview.md) y el [concepto de canalización](pipeline.md).

## <a name="invoking-direct-methods-via-azure-portal"></a>Invocación de métodos directos a través de Azure Portal

Los [métodos directos](./direct-methods.md) expuestos por el módulo Video Analyzer se puede invocar a través de Azure Portal. Los pasos siguientes incluyen los detalles de un método directo. Es posible invocar otros métodos directos por medio de pasos similares. Sin embargo, cada método directo requiere una carga específica.

Utilice la llamada de método `livePipelineList` para recuperar una lista de todas las topologías de canalización actualmente implementadas en el módulo Video Analyzer. Siga estos pasos para invocar el método directo:

1. Inicie sesión en Azure Portal.
1. Busque el grupo de recursos correspondiente en la página principal del portal para buscar el módulo IoT Hub. Si ya conoce el módulo IoT Hub, selecciónelo.
    ![grupo de recursos en la página principal del portal](./media/use-azure-portal-to-invoke-direct-methods/portal-rg-home.png)
1. Una vez en la página IoT Hub, seleccione IoT Edge en Administración automática de dispositivos para enumerar los distintos identificadores de dispositivo. Seleccione el identificador de dispositivo correspondiente para enumerar los módulos que están ejecutándose en el dispositivo.
    ![página iot hub](./media/use-azure-portal-to-invoke-direct-methods/iot-hub-page.png)
1. Seleccione el módulo Video Analyzer para abrir su página de configuración.<br><br>
    ![Selección del módulo Video Analyzer para abrir su página de configuración](./media/use-azure-portal-to-invoke-direct-methods/modules.png)
1. Seleccione la opción de menú Método directo. <br><br>
    ![Selección de la opción de menú Método directo](./media/use-azure-portal-to-invoke-direct-methods/module-details.png)
    > [!NOTE]
    > Puede que tenga que añadir un valor en las secciones de cadena de conexión, tal como se muestra en la página actual. No es necesario ocultar ni cambiar nada en la sección Nombre del valor. Es correcto dejar que sea pública.

1. Escriba *livePipelineList* en el campo **Nombre del método**.
1. Copie y pegue el siguiente contenido JSON en el campo **Carga**.
    ```json
    {
    "@apiVersion": "1.0"
    }
    ```
1. Seleccione el botón **Invocar método** en la parte superior de la página.<br><br>
    ![botón de invocar método](./media/use-azure-portal-to-invoke-direct-methods/direct-method.png)
1. Debería ver un mensaje de estado 200 en el área de **resultados**.<br><br>
    ![tiempo de espera de conexión agotado](./media/use-azure-portal-to-invoke-direct-methods/connection-timeout.png)

## <a name="responses"></a>Respuestas

| Condición             | Código de estado | Código de error detallado |
|-----------------------|-------------|---------------------|
| Correcto               | 200         | N/D                 |
| Errores de usuario generales   | Intervalo 400   |                     |
| Errores generales del servidor | Intervalo 500   |                     |

## <a name="next-steps"></a>Pasos siguientes

Hay otros métodos directos disponibles en la página [Métodos directos](./direct-methods.md).

> [!NOTE]
> Una canalización activa una topología específica. Por lo tanto, asegúrese de haber establecido la topología correcta antes de activar una canalización.

[Inicio rápido: Detecte eventos Motion Emit](detect-motion-emit-events-quickstart.md) es una buena fuente de referencia para comprender la secuencia exacta de llamadas a métodos directos que se van a realizar.
