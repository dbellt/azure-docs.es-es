---
title: Introducción a los aceleradores de soluciones de Azure IoT en Azure | Microsoft Docs
description: Información acerca de los aceleradores de soluciones de IoT de Azure. Los aceleradores de soluciones de IoT son elementos completos y preparados para implementar soluciones de IoT.
author: dominicbetts
ms.author: dobett
ms.date: 03/09/2019
ms.topic: overview
ms.custom: mvc
ms.service: iot-accelerators
services: iot-accelerators
manager: timlt
ms.openlocfilehash: c966051ed5699d408fe83f1e9c862ca78b3282c4
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714549"
---
# <a name="what-are-azure-iot-solution-accelerators"></a>¿Qué son los aceleradores de soluciones de Azure IoT?

Una solución IoT basada en la nube normalmente usa código personalizado y servicios en la nube para administrar la conectividad de los dispositivos, el procesamiento y el análisis de los datos y su presentación.

Los aceleradores de soluciones IoT son soluciones IoT completas y listas para implementar escenarios comunes de IoT. Entre los escenarios se incluyen la factoría conectada y la simulación de dispositivos. Al implementar un acelerador de soluciones, la implementación incluye todos los servicios basados en la nube necesarios, junto con el código de aplicación requerido.

Los aceleradores de soluciones son puntos de partida para sus propias soluciones de IoT. El código fuente para todos los aceleradores de soluciones es código abierto y está disponible en GitHub. Le recomendamos que descargue y personalice los aceleradores de soluciones para cumplir sus requisitos.

También puede utilizarlos como herramientas de aprendizaje antes de crear una solución de IoT personalizada desde cero. Implementan prácticas probadas para soluciones de IoT basadas en la nube de modo que pueda realizar su seguimiento.

El código de la aplicación de cada acelerador de soluciones incluye una aplicación web que le permite administrar dicho acelerador.

> [!NOTE]
> Las soluciones de supervisión remota y mantenimiento predictivo se han quitado del sitio [Aceleradores de soluciones de Azure IoT](https://www.azureiotsolutions.com/Accelerators). Para más información, consulte [¿Qué son los aceleradores de soluciones de Azure IoT? (versión anterior)](/previous-versions/azure/iot-accelerators/about-iot-accelerators).

## <a name="supported-iot-scenarios"></a>Escenarios de IoT admitidos

Actualmente, hay dos aceleradores de soluciones disponibles para implementar:

### <a name="connected-factory"></a>Fábrica conectada

Use el [acelerador de soluciones Factoría conectada](iot-accelerators-connected-factory-features.md) para recopilar datos de telemetría de recursos industriales con una interfaz de [OPC Unified Architecture](https://opcfoundation.org/about/opc-technologies/opc-ua/) y para controlarlos. Los recursos industriales podrían incluir estaciones de prueba y ensamblado en una línea de producción de una fábrica.

Puede utilizar el panel de fábrica conectada para supervisar y administrar sus dispositivos industriales:

:::image type="content" source="./media/about-iot-accelerators/cf-dashboard-inline.png" alt-text="Captura de pantalla que muestra el panel de la solución de factoría conectada." lightbox="./media/about-iot-accelerators/cf-dashboard-expanded.png":::

### <a name="device-simulation"></a>Simulación de dispositivo

Use el [acelerador de soluciones Simulación de dispositivos](iot-accelerators-device-simulation-overview.md) para ejecutar dispositivos simulados que generan datos de telemetría realistas. Puede utilizar este acelerador de soluciones para probar el comportamiento de los demás aceleradores de soluciones o para probar sus propias soluciones personalizadas de IoT.

Puede usar la aplicación web de simulación de dispositivos para configurar y ejecutar simulaciones:

:::image type="content" source="./media/about-iot-accelerators/ds-dashboard-inline.png" alt-text="Captura de pantalla que muestra el panel de soluciones de simulación de dispositivos" lightbox="./media/about-iot-accelerators/ds-dashboard-expanded.png":::

## <a name="design-principles"></a>Principios de diseño

Todos los aceleradores de soluciones siguen los mismos principios de diseño y objetivos. Están diseñados para ser:

* **Escalables**, lo que le permite conectar y administrar millones de dispositivos conectados.
* **Extensibles**, lo que le permite personalizarlos para satisfacer sus requisitos.
* **Comprensibles**, lo que le permite comprender cómo funcionan y cómo se implementan.
* **Modulares**, lo que le permite intercambiar servicios alternativos.
* **Seguros**, combinando la seguridad de Azure con características de seguridad integradas de dispositivos y de conectividad.

## <a name="architectures-and-languages"></a>Arquitecturas y lenguajes

Los aceleradores de soluciones originales se escribieron con .NET usando una arquitectura Model-View-Controller (MVC). Microsoft está actualizando los aceleradores de soluciones con una nueva arquitectura basada en microservicios. En la siguiente tabla se muestra el estado actual de los aceleradores de soluciones con vínculos a los repositorios de GitHub:

| Acelerador de soluciones   | Architecture  | Lenguajes     |
| ---------------------- | ------------- | ------------- |
| Fábrica conectada      | MVC           | [.NET](https://github.com/Azure/azure-iot-connected-factory)          |
| Simulación de dispositivo      | Microservicios | [.NET](https://github.com/Azure/azure-iot-pcs-device-simulation)          |

Para más información sobre la arquitectura de microservicios, consulte [Introducción a la arquitectura de referencia de Azure IoT](/azure/architecture/reference-architectures/iot/).

## <a name="deployment-options"></a>Opciones de implementación

Puede implementar los aceleradores de soluciones desde el sitio [Aceleradores de soluciones de Microsoft Azure IoT](https://www.azureiotsolutions.com/Accelerators#) o mediante la línea de comandos.

El costo que supone ejecutar un acelerador de soluciones es una combinación del [costo de ejecutar los servicios de Azure subyacentes](https://azure.microsoft.com/pricing). Consulte los detalles de los servicios de Azure utilizados al elegir las opciones de implementación.

## <a name="next-steps"></a>Pasos siguientes

Para probar uno de los aceleradores de soluciones de IoT, consulte el inicio rápido [Prueba de una solución de factoría conectada](quickstart-connected-factory-deploy.md).
