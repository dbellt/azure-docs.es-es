---
title: SDK y recursos de Azure Video Analyzer
description: Obtenga información sobre los SDK de Azure Video Analyzer.
author: bennage
ms.author: christb
ms.topic: reference
ms.date: 05/14/2021
ms.openlocfilehash: 480d3fbadbac6dcf7ec56f92e45e7c2e65653195
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388637"
---
# <a name="azure-video-analyzer-sdks"></a>SDK de Azure Video Analyzer

Azure Video Analyzer incluye dos grupos de SDK. Los SDK de administración se usan para administrar el recurso de Azure y los SDK de cliente se usan para interactuar con los módulos perimetrales.

## <a name="management-sdks"></a>SDK de administración

Los SDK de administración permiten interactuar con los recursos expuestos por Azure Resource Manager. Puede crear una cuenta de Video Analyzer, generar tokens de aprovisionamiento para módulos perimetrales, administrar directivas de acceso para vídeos y mucho más. Los SDK se basan en una [API REST] subyacente.

Se admiten las siguientes plataformas:

- [.NET](https://aka.ms/ava/sdk/mgt/net)
- [Java](https://aka.ms/ava/sdk/mgt/java)
- [Python](https://aka.ms/ava/sdk/mgt/python)

## <a name="client-sdks"></a>SDK de cliente

Los SDK de cliente permiten interactuar con los [métodos directos][docs-direct-methods] de un módulo perimetral de Video Analyzer. Estos SDK están diseñados para su uso con los [SDK de Azure IoT Hub][docs-iot-hub-sdks]. Admiten la construcción de objetos que representan los métodos directos que luego se pueden enviar al módulo perimetral mediante los SDK de IoT Hub.

Se admiten las siguientes plataformas:

- [.NET](https://aka.ms/ava/sdk/client/net)
- [Python](https://aka.ms/ava/sdk/client/python)
- [Java](https://aka.ms/ava/sdk/client/java)

<!-- links -->
[docs-direct-methods]: direct-methods.md
[docs-iot-hub-sdks]: /azure/iot-hub/iot-hub-devguide-sdks

[REST API]: https://aka.ms/ava/api/rest
