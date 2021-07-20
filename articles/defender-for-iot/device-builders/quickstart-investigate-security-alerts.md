---
title: 'Inicio rápido: Investigación de alertas de seguridad'
description: Conozca, explore en profundidad e investigue las alertas de seguridad de Defender para IoT en sus dispositivos de IoT.
ms.topic: quickstart
ms.date: 06/21/2021
ms.openlocfilehash: 02005680ca1c5d140ffdb101ede359594b28a840
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113011500"
---
# <a name="quickstart-investigate-security-alerts"></a>Inicio rápido: Investigación de alertas de seguridad

La mejor manera de garantizar el cumplimiento y la protección en toda la solución IoT es la investigación programada y la corrección de las alertas que emite Defender para IoT.

## <a name="investigate-new-security-alerts"></a>Investigación de nuevas alertas de seguridad

La lista de alertas de seguridad de IoT Hub muestra todas las alertas de seguridad agregadas para una instancia de IoT Hub. 

1. En Azure Portal, abra la instancia de **IoT Hub** en el que desee investigar nuevas alertas.

1. En el menú **Seguridad**, seleccione **Alertas**. Se mostrarán todas las alertas de seguridad para la instancia de IoT Hub y se agregará la marca **Nueva** a las alertas de las últimas 24 horas.

    :::image type="content" source="media/quickstart/investigate-new-security-alerts.png" alt-text="Investigación de las nuevas alertas de seguridad de IoT con la marca de nueva alerta":::

1. Seleccione una alerta de la lista para abrir los detalles de la alerta y conocer los detalles específicos de la alerta. 

## <a name="security-alert-details"></a>Detalles de alertas de seguridad

Al abrir cada alerta agregada, se muestra su descripción detallada, los pasos de corrección y el identificador de dispositivo de cada dispositivo que desencadenó una alerta. La gravedad de la alerta y la investigación directa son accesibles mediante Log Analytics. 

1. Vaya a **IoT Hub** > **Seguridad** > **Alertas**. 

1. Seleccione cualquier alerta de seguridad de la lista para abrirla. 

1. Revise la **descripción**, **gravedad**, **origen de la detección** y **detalles del dispositivo** de la alerta para todos los dispositivos que la emitieron en el período de agregación.

    :::image type="content" source="media/quickstart/drill-down-iot-alert-details.png" alt-text="Investigación y revisión de los detalles de cada dispositivo en una alerta agregada"::: 

1. Después de revisar los detalles de la alerta, siga las instrucciones del **paso de corrección manual** para ayudar a corregir y resolver el problema que provocó la alerta.

    :::image type="content" source="media/quickstart/iot-alert-manual-remediation-steps.png" alt-text="Siga los pasos de corrección manual para ayudar a resolver o corregir las alertas de seguridad del dispositivo.":::

1. Si se requiere una investigación adicional, **investigue las alertas en Log Analytics** mediante el vínculo.
 
    :::image type="content" source="media/quickstart/investigate-iot-alert-log-analytics.png" alt-text="Para investigar más una alerta, use el vínculo para investigar con Log Analytics que aparece en la pantalla":::

## <a name="next-steps"></a>Pasos siguientes

Continúe con el siguiente artículo para más información sobre los tipos de alertas de Defender y las posibles personalizaciones.

> [!div class="nextstepaction"]
> [Descripción de las alertas de seguridad de IoT](concept-security-alerts.md)
