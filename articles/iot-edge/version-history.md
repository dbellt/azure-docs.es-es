---
title: 'Navegación e historial de versiones de IoT Edge: Azure IoT Edge'
description: Descubra las novedades de IoT Edge con información sobre las nuevas características y funcionalidades de las versiones más recientes.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 04/07/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 1d3473e975e7c69a83c359b040a2de0defaac69b
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310487"
---
# <a name="azure-iot-edge-versions-and-release-notes"></a>Versiones de Azure IoT Edge y notas de la versión

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Azure IoT Edge es un producto creado a partir del proyecto de IoT Edge de código abierto hospedado en GitHub. Todas las versiones nuevas están disponibles en el [proyecto de Azure IoT Edge](https://github.com/Azure/azure-iotedge). Las contribuciones y los informes de errores se pueden realizar en el [proyecto de IoT Edge de código abierto](https://github.com/Azure/iotedge).

## <a name="documented-versions"></a>Versiones documentadas

La documentación de IoT Edge de este sitio está disponible para dos versiones diferentes del producto, de modo que pueda elegir el contenido que se aplica a su entorno de IoT Edge. Actualmente, las versiones compatibles son:

* **IoT Edge 1.2** incluye contenido adicional para nuevas características y funcionalidades disponibles en la versión estable más reciente.
* **IoT Edge 1.1 (LTS)** es la primera versión de compatibilidad a largo plazo (LTS) de IoT Edge. La documentación de esta versión cubre todas las características y capacidades de todas las versiones anteriores hasta 1.1. Esta versión de la documentación será estable a través de la duración admitida de la versión 1.1 y no reflejará las nuevas características publicadas en versiones posteriores.

Para más información sobre las versiones de IoT Edge, consulte [Sistemas compatibles con Azure IoT Edge](support.md).

## <a name="version-history"></a>Historial de versiones

En esta tabla se proporciona el historial de versiones recientes de los paquetes de IoT Edge, y se destacan las actualizaciones de documentación realizadas para cada versión.

| Notas de la versión y recursos | Tipo | Date | Aspectos destacados |
| ------------------------ | ---- | ---- | ---------- |
| [1.2](https://github.com/Azure/azure-iotedge/releases/tag/1.2.0) | Stable | Abril de 2021 | [Dispositivos de IoT Edge detrás de puertas de enlace](how-to-connect-downstream-iot-edge-device.md?view=iotedge-2020-11&preserve-view=true)<br>[Agente MQTT de IoT Edge (versión preliminar)](how-to-publish-subscribe.md?view=iotedge-2020-11&preserve-view=true)<br>Se introdujeron nuevos paquetes de IoT Edge, con nuevos pasos de instalación y configuración. Para obtener más información, consulte [Actualización de 1.0 o 1.1 a 1.2](how-to-update-iot-edge.md#special-case-update-from-10-or-11-to-12).
| [1.1](https://github.com/Azure/azure-iotedge/releases/tag/1.1.0) | Soporte técnico a largo plazo (LTS) | Febrero de 2021 | [Plan de soporte técnico a largo plazo y actualizaciones de sistemas compatibles](support.md) |
| [1.0.10](https://github.com/Azure/azure-iotedge/releases/tag/1.0.10) | Stable | Octubre de 2020 | [Método directo UploadSupportBundle](how-to-retrieve-iot-edge-logs.md#upload-support-bundle-diagnostics)<br>[Carga de métricas en tiempo de ejecución](how-to-access-built-in-metrics.md)<br>[Enrutamiento de prioridad y período de vida](module-composition.md#priority-and-time-to-live)<br>[Orden de inicio de módulos](module-composition.md#configure-modules)<br>[Aprovisionamiento manual de X.509](how-to-register-device.md) |
| [1.0.9](https://github.com/Azure/azure-iotedge/releases/tag/1.0.9) | Stable | Marzo de 2020 | [Aprovisionamiento automático de X.509 con DPS](how-to-auto-provision-x509-certs.md)<br>[Método directo RestartModule](how-to-edgeagent-direct-method.md#restart-module)<br>[Comando support-bundle](troubleshoot.md#gather-debug-information-with-support-bundle-command) |

## <a name="next-steps"></a>Pasos siguientes

* [Consulte todas las versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases)
* [Cree o revise solicitudes de características en el foro de comentarios](https://feedback.azure.com/forums/907045-azure-iot-edge)