---
title: Agentes de seguridad
description: Introducción a la configuración, implementación y uso de los agentes del servicio de seguridad de Azure Defender para IoT en dispositivos de IoT.
ms.topic: conceptual
ms.date: 1/24/2021
ms.openlocfilehash: 117572c8b5e2d42b24c1f3af9dbb0eab474647a9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949162"
---
# <a name="get-started-with-azure-defender-for-iot-device-micro-agents"></a>Introducción a los microagentes de dispositivo de Azure Defender para IoT

Los agentes de seguridad de Defender para IoT ofrecen funcionalidades de seguridad mejoradas, como la supervisión de los procedimientos recomendados de configuración del sistema operativo. Tome el control de la protección contra amenazas del campo de dispositivo y la postura de seguridad con un único servicio.

Los agentes de Defender para IoT controlan la recopilación de eventos sin procesar procedentes del sistema operativo del dispositivo, la agregación de eventos para reducir el costo y la configuración mediante un módulo gemelo del dispositivo. Los mensajes de seguridad se envían a través del IoT Hub a los servicios de análisis de Defender para IoT.

Use el flujo de trabajo a continuación para implementar y probar los agentes de seguridad de Defender para IoT:

1. [Habilite el servicio Defender para IoT en IoT Hub](quickstart-onboard-iot-hub.md).

1. Si IoT Hub no tiene ningún dispositivo registrado, [registre uno nuevo](/previous-versions/azure/iot-accelerators/iot-accelerators-device-simulation-overview).

1. [Cree un módulo gemelo DefenderIotMicroAgent](quickstart-create-micro-agent-module-twin.md) para sus dispositivos.

1. Para instalar el agente en un dispositivo simulado de Azure en lugar de hacerlo en un dispositivo real, [prepare una máquina virtual de Azure nueva](../virtual-machines/linux/quick-create-portal.md) en una zona de disponibilidad.

1. [Implemente un agente de seguridad de Defender para IoT](how-to-deploy-linux-cs.md) en el dispositivo IoT o en una máquina virtual nueva.

1. Siga las instrucciones de [trigger_events](https://aka.ms/iot-security-github-trigger-events) para ejecutar un evento de línea de base del sistema operativo.

1. Compruebe las recomendaciones de Defender para IoT en respuesta al error de comprobación de línea de base del sistema operativo simulado en el paso anterior. No las compruebe hasta que hayan transcurrido 30 minutos desde la ejecución del script.

## <a name="next-steps"></a>Pasos siguientes

- Configurar la [solución](quickstart-configure-your-solution.md)
- [Creación de microagentes de Defender para IoT](quickstart-create-security-twin.md)
- Configurar [alertas personalizadas](quickstart-create-custom-alerts.md)
- [Implementación de un agente de seguridad](how-to-deploy-agent.md)