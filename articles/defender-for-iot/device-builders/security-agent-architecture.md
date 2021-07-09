---
title: 'Inicio rápido: Introducción a los agentes de seguridad'
description: En este inicio rápido, conocerá la arquitectura de agente de seguridad de los agentes que se usan en el servicio Azure Defender para IoT.
ms.topic: quickstart
ms.date: 4/4/2021
ms.openlocfilehash: 0937cccb0335f4eee16ca3590babe9574320b89f
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113011484"
---
# <a name="quickstart-security-agent-reference-architecture"></a>Inicio rápido: Arquitectura de referencia de los agentes de seguridad

Azure Defender para IoT proporciona una arquitectura de referencia a agentes de seguridad que registran, procesan, agregan y envían datos de seguridad mediante IoT Hub.

Los agentes de seguridad están diseñados para funcionar en un entorno de IoT restringido, y son tremendamente personalizables en cuanto a todo el valor que aportan en comparación con los recursos que consumen.

Los agentes de seguridad permiten realizar lo siguiente:

- Autenticarse con la identidad del dispositivo existente, o con una identidad de módulo dedicada. Para obtener información, consulte  [Métodos de autenticación del agente de seguridad](concept-security-agent-authentication-methods.md).

- Recopilar eventos de seguridad sin procesar del sistema operativo subyacente (Linux o Windows). Vea [Configuración del agente de Defender para IoT](how-to-agent-configuration.md) para obtener más información sobre los recopiladores de datos de seguridad disponibles.

- Agregar eventos de seguridad sin procesar a los mensajes enviados mediante IoT Hub.

- Establecer configuraciones de forma remota mediante el módulo gemelo **azureiotsecurity**. Para obtener más información, consulte [Configuración de un agente de Defender para IoT](how-to-agent-configuration.md).

Los agentes de seguridad de Defender para IoT se desarrollan como proyectos de código abierto y están disponibles en GitHub:

- [Agente basado en C de Defender para IoT](https://github.com/Azure/Azure-IoT-Security-Agent-C)
- [Agente basado en C# de Defender para IoT](https://github.com/Azure/Azure-IoT-Security-Agent-CS)

## <a name="prerequisites"></a>Requisitos previos

- Ninguno

## <a name="agent-supported-platforms"></a>Plataformas compatibles con el agente

Defender para IoT ofrece diferentes instaladores de agente de 32 y 64 bits de Windows, y lo mismo para 32 bits y 64 bits de Linux. Asegúrese de que tiene el instalador de agente correcto para cada uno de los dispositivos, según la siguiente tabla:

| Architecture | Linux | Windows | Detalles |
|--|--|--|--|
| 32 bits | C | C# |  |
| 64 bits | C# o C | C# | Se recomienda usar el agente de C para dispositivos con recursos de dispositivo mínimos o más restringidos. |


## <a name="next-steps"></a>Pasos siguientes

En este artículo encontrará información general de alto nivel sobre la arquitectura de Defender-IoT-micro-agent de Defender para IoT y los instaladores disponibles.
Para más información sobre la implementación de Defender para IoT, consulte: 

> [!div class="nextstepaction"]
> [Métodos de autenticación del agente de seguridad](concept-security-agent-authentication-methods.md)
