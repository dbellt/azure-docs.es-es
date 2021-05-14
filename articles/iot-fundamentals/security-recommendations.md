---
title: Recomendaciones de seguridad para Azure IoT | Microsoft Docs
description: En este artículo se resumen los pasos adicionales para garantizar la seguridad en la solución Azure IoT Hub.
author: dsk-2015
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: article
ms.date: 11/13/2019
ms.author: dkshir
ms.custom:
- security-recommendations
- amqp
- mqtt
ms.openlocfilehash: eeb8a8132b905254c02c86460c376d69948b9264
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109481416"
---
# <a name="security-recommendations-for-azure-internet-of-things-iot-deployment"></a>Recomendaciones de seguridad para la implementación de Azure Internet of Things (IoT)

Este artículo contiene recomendaciones de seguridad para IoT. La implementación de estas recomendaciones le ayudará a cumplir sus obligaciones de seguridad, tal y como se describe en nuestro modelo de responsabilidad compartida. Para más información sobre lo que Microsoft hace para cumplir las responsabilidades del proveedor de servicios, consulte [Responsabilidades compartidas de la informática en la nube](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91).

Algunas de las recomendaciones incluidas en este artículo se pueden supervisar automáticamente mediante Azure Security Center. Azure Security Center es la primera línea de defensa en la protección de los recursos de Azure. Se encarga de analizar el estado de seguridad de los recursos de Azure para identificar posibles vulnerabilidades de seguridad. Después, proporciona recomendaciones sobre cómo abordarlas.

- Para más información sobre las recomendaciones de seguridad de Azure Security Center, consulte [Recomendaciones de seguridad en Azure Security Center](../security-center/security-center-recommendations.md).
- Para más información sobre Azure Security Center, consulte [¿Qué es Azure Security Center?](../security-center/security-center-introduction.md)

## <a name="general"></a>General

| Recomendación | Comentarios | Compatible con ASC |
|-|----|--|
| Manténgase al día | Use las versiones más recientes de las plataformas compatibles, los lenguajes de programación, los protocolos y los marcos. | - |
| Mantenga seguras las claves de autenticación | Mantenga los id. de dispositivo y sus claves de autenticación físicamente seguros tras la implementación. De esta forma, evitará que un dispositivo malintencionado se enmascare como un dispositivo registrado. | - |
| Use los SDK de dispositivo cuando sea posible | Los SDK de dispositivo implementan varias características de seguridad como, por ejemplo, el cifrado, la autenticación, etc., que le ayudarán a desarrollar una aplicación de dispositivos sólida y segura. Para obtener más información, consulte [Información y uso de los SDK de Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md). | - |

## <a name="identity-and-access-management"></a>Administración de identidades y acceso 

| Recomendación | Comentarios | Compatible con ASC |
|-|----|--|
| Defina el control de acceso para el centro | [Comprenda y defina el tipo de acceso](iot-security-deployment.md#securing-the-cloud) que tendrá cada componente en su solución de IoT Hub, según la funcionalidad. Los permisos permitidos son *Registry Read*, *RegistryReadWrite*, *ServiceConnect* y *DeviceConnect*. Las [directivas de acceso compartido en el centro de IoT](../iot-hub/iot-hub-dev-guide-sas.md#access-control-and-permissions) predeterminadas también pueden ayudarle a definir los permisos para cada componente según su rol. | - |
| Defina el control de acceso para los servicios de back-end | Los datos que introduce la solución de IoT Hub pueden aprovecharse en otros servicios de Azure, como, por ejemplo, [Cosmos DB](../cosmos-db/index.yml), [Stream Analytics](../stream-analytics/index.yml), [App Service](../app-service/index.yml), [Logic Apps](../logic-apps/index.yml) y [Blob Storage](../storage/blobs/storage-blobs-introduction.md). Asegúrese de comprender y conceder los permisos de acceso adecuados tal como se documenta para estos servicios. | - |

## <a name="data-protection"></a>Protección de los datos

| Recomendación | Comentarios | Compatible con ASC |
|-|----|--|
| Autenticación de dispositivos segura | Garantice una comunicación segura entre sus dispositivos y el centro de IoT, ya sea mediante [una clave de identidad única o un token de seguridad](iot-security-deployment.md#iot-hub-security-tokens) o bien mediante [un certificado X.509 en el dispositivo](iot-security-deployment.md#x509-certificate-based-device-authentication) para cada uno de los dispositivos. Utilice el método adecuado para [utilizar tokens de seguridad basados en el protocolo elegido (MQTT, AMQP o HTTPS)](../iot-hub/iot-hub-dev-guide-sas.md). | - |
| Comunicación segura con los dispositivos | IoT Hub protege la conexión con los dispositivos mediante el estándar de seguridad de la capa de transporte (TLS), cuyas versiones admitidas son 1.2 y 1.0. Use [TLS 1.2](https://tools.ietf.org/html/rfc5246) para garantizar la máxima seguridad. | - |
| Comunicación segura del servicio | IoT Hub proporciona puntos de conexión para conectarse a servicios back-end como, por ejemplo, [Azure Storage](../storage/index.yml) o [Event Hubs](../event-hubs/index.yml), usando únicamente el protocolo TLS, de forma que ningún punto de conexión se expone en un canal no cifrado. Una vez que estos datos lleguen a estos servicios de back-end para el almacenamiento o el análisis, asegúrese de emplear los métodos de seguridad y cifrado adecuados para ese servicio y de proteger la información confidencial en el back-end. | - |

## <a name="networking"></a>Redes

| Recomendación | Comentarios | Compatible con ASC |
|-|----|--|
| Protección del acceso a los dispositivos | Mantenga los puertos de hardware de los dispositivos a la mínima expresión para evitar el acceso no deseado. Asimismo, cree mecanismos para prevenir o detectar la alteración física del dispositivo. Consulte los [procedimientos recomendados sobre seguridad de IoT](iot-security-best-practices.md) para obtener más información. | - |
| Compilación de hardware segura | Incorpore características de seguridad como, por ejemplo, el almacenamiento cifrado o el Módulo de plataforma segura (TPM), para proteger todavía más los dispositivos y la infraestructura. Mantenga el sistema operativo y los controladores del dispositivo actualizados a las versiones más recientes y, si el espacio lo permite, instale funcionalidades de antivirus y antimalware. Consulte la [arquitectura de seguridad de IoT](iot-security-architecture.md) para comprender cómo estas medidas pueden ayudar a mitigar varias amenazas de seguridad. | - |

## <a name="monitoring"></a>Supervisión

| Recomendación | Comentarios | Compatible con ASC |
|-|----|--|
| Supervisión del acceso no autorizado a los dispositivos |  Use la característica de registro del sistema operativo para supervisar las posibles vulneraciones de seguridad o alteraciones físicas del dispositivo o sus puertos. | - |
| Supervisión de la solución de IoT desde la nube | Supervise el estado general de la solución de IoT Hub mediante las [métricas de Azure Monitor](../iot-hub/monitor-iot-hub.md). | - |
| Configuración de diagnósticos | Para vigilar sus operaciones de cerca, registre los eventos de la solución y, a continuación, envíe los registros de diagnóstico a Azure Monitor para visualizar mejor el rendimiento. Consulte [Supervisión y diagnóstico de problemas en el centro de IoT](../iot-hub/monitor-iot-hub.md) para obtener más información. | - |

## <a name="next-steps"></a>Pasos siguientes

Para escenarios avanzados que implican Azure IoT, quizá haya otros requisitos de seguridad que deban tenerse en consideración. Consulte la [arquitectura de seguridad de IoT](iot-security-architecture.md) para obtener una guía.