---
title: Diferencias con la versión original
titleSuffix: Azure Digital Twins
description: Explicación de los cambios de la nueva versión de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 1/28/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: e03552bc3be1a4d124a46c437b60ac2d4e466940
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110476777"
---
# <a name="what-is-the-new-azure-digital-twins-how-is-it-different-from-the-original-version-2018"></a>¿Qué es la nueva versión de Azure Digital Twins? ¿En qué se diferencia de la versión original (2018)?

La primera versión preliminar pública de Azure Digital Twins se publicó en octubre de 2018. Aunque los conceptos básicos de esa versión original se han pasado al servicio actual, muchas de las interfaces y los detalles de implementación han cambiado para que el servicio sea más flexible y accesible. Estos cambios están motivados por los comentarios de los clientes.

> [!IMPORTANT]
> En función de las funcionalidades ampliadas del nuevo servicio, el servicio Azure Digital Twins original se ha retirado. Desde enero de 2021, sus API y datos asociados ya no están disponibles.

Si ha usado la primera versión de Azure Digital Twins durante la primera versión preliminar pública, utilice la información y los procedimientos recomendados de este artículo para aprender a trabajar con el servicio actual y aprovechar sus características.

## <a name="differences-by-topic"></a>Diferencias por tema

En el gráfico siguiente se proporciona una vista en paralelo de los conceptos que han cambiado entre la versión original del servicio y el actual.

| Tema | En la versión original | En la versión actual |
| --- | --- | --- | --- |
| **Modelado**<br>*Más flexible* | La versión original se diseñó en torno a los espacios inteligentes, por lo que incluía un vocabulario integrado para edificios. | La versión actual de Azure Digital Twins es independiente del dominio. Puede definir su propio vocabulario personalizado y modelos personalizados para su solución, a fin de representar más tipos de entornos de maneras más flexibles.<br><br>Obtenga más información en [Conceptos: Modelos personalizados](concepts-models.md). |
| **Topología**<br>*Más flexible*| La versión original admitía una estructura de datos de árbol adaptada a los espacios inteligentes. Los gemelos digitales se conectaban con relaciones jerárquicas. | Con la versión actual, los gemelos digitales se pueden conectar en topologías de grafos arbitrarias, organizados según sea necesario. Esto le proporciona más flexibilidad para expresar las complejas relaciones del mundo real.<br><br>Obtenga más información en [Conceptos: Gemelos digitales y grafo de gemelos](concepts-twins-graph.md). |
| **Proceso**<br>*Más rico y flexible* | En la versión original, la lógica de procesamiento de eventos y telemetría se definía en funciones definidas por el usuario (UDF) de JavaScript. La depuración con UDF era limitada. | La versión actual tiene un modelo de proceso abierto: para proporcionar lógica personalizada debe conectar recursos de proceso externos como [Azure Functions](../azure-functions/functions-overview.md). Esto le permite usar un lenguaje de programación de su elección, acceder a las bibliotecas de código personalizado sin restricciones, y aprovechar los recursos de desarrollo y depuración que el servicio externo pueda tener.<br><br>Obtenga más información en [Instrucciones: Configuración de una función de Azure para procesar datos](how-to-create-azure-function.md). |
| **Administración de dispositivos con IoT Hub**<br>*Más accesible* | En la versión original los dispositivos se administraban con una instancia de [IoT Hub](../iot-hub/about-iot-hub.md) interna del servicio Azure Digital Twins. Este centro de conectividad integrado no era totalmente accesible para los desarrolladores. | En la versión actual, "trae su propio" centro de IoT, para lo que conecta una instancia de IoT Hub creada de forma independiente (junto con los dispositivos que ya administra). Esto le proporciona acceso completo a las funcionalidades de IoT Hub y le permite controlar la administración de dispositivos.<br><br>Obtenga más información en [Instrucciones: Ingesta de telemetría desde IoT Hub](how-to-ingest-iot-hub-data.md). |
| **Seguridad**<br>*Más estándar* | La versión original tenía roles predefinidos que podía usar para administrar el acceso a la instancia. | La versión actual se integra con el mismo servicio de back-end de [control de acceso basado en roles de Azure (Azure RBAC)](../role-based-access-control/overview.md) que utilizan otros servicios de Azure. Esto puede facilitar la autenticación entre otros servicios de Azure en la solución, como IoT Hub, Azure Functions, Event Grid, etc.<br>Con RBAC, puede seguir usando roles predefinidos o puede compilar y configurar roles personalizados.<br><br>Obtenga más información en [Conceptos: Seguridad para las soluciones de Azure Digital Twins](concepts-security.md). |
| **Escalabilidad**<br>*Superior* | La versión original tenía limitaciones de escala para dispositivos, mensajes, grafos y unidades de escalado. Solo se admitía una instancia de Azure Digital Twins por suscripción.  | La versión actual se basa en una nueva arquitectura con una escalabilidad mejorada y tiene una mayor capacidad de proceso. También admite 10 instancias por región y por suscripción.<br><br>Vea [Límites del servicio Azure Digital Twins](reference-service-limits.md) para obtener información sobre los límites de la versión actual. |

## <a name="service-limits"></a>Límites de servicio

Para obtener una lista de los límites de Azure Digital Twins, vea [Límites del servicio Azure Digital Twins](reference-service-limits.md).

## <a name="next-steps"></a>Pasos siguientes

* Comience a trabajar con la versión actual en: [Inicio rápido: Exploración de un escenario de Azure Digital Twins de ejemplo mediante Azure Digital Twins Explorer](quickstart-azure-digital-twins-explorer.md).

* O bien, empiece a obtener información sobre los conceptos clave con [Conceptos: Modelos personalizados](concepts-models.md).