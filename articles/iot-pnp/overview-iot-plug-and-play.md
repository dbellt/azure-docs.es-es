---
title: Introducción a IoT Plug and Play | Microsoft Docs
description: Más información acerca de IoT Plug and Play. IoT Plug and Play se basa en un lenguaje de modelado abierto que permite a los dispositivos IoT inteligentes declarar sus funcionalidades. Los dispositivos de IoT presentan esa declaración, denominada "modelo de dispositivo", cuando se conectan a soluciones en la nube. Luego, la solución en la nube puede comprender automáticamente el dispositivo y empezar a interactuar con él, todo ello sin necesidad de escribir código.
author: rido-min
ms.author: rmpablos
ms.date: 03/21/2021
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
manager: eliotgra
ms.custom: references_regions
ms.openlocfilehash: 91a09db16524ebc7e4c04069b69b1c42c67538c6
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739717"
---
# <a name="what-is-iot-plug-and-play"></a>¿Qué es IoT Plug and Play?

IoT Plug and Play permite a los creadores de soluciones integrar dispositivos inteligentes en sus soluciones sin necesidad de configuración manual. IoT Plug and Play se basa en un modelo de _dispositivos_ que un dispositivo usa para anunciar sus funcionalidades a una aplicación compatible con IoT Plug and Play. Este modelo se estructura como un conjunto de elementos que definen:

- _Propiedades_ que representan el estado de solo lectura y grabable de un dispositivo o de otra entidad. Por ejemplo, el número de serie de un dispositivo puede ser una propiedad de solo lectura, y la temperatura objetivo de un termostato puede ser una propiedad grabable.
- _Datos de telemetría_, que son los datos que emite un dispositivo, independientemente de que sean una secuencia normal de lecturas de un sensor, un error ocasional o un mensaje informativo.
- _Comandos_ que describen una función u operación que se puede realizar en un dispositivo. Por ejemplo, un comando puede reiniciar una puerta de enlace o tomar una imagen mediante una cámara remota.

Puede agrupar estos elementos en interfaces para reutilizarlos en los distintos modelos para facilitar la colaboración y acelerar el desarrollo.

Para que IoT Plug and Play funcione sin problemas con [Azure Digital Twins](../digital-twins/overview.md), se definen modelos e interfaces mediante el [lenguaje de definición de gemelos digitales (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl). IoT Plug and Play y DTDL están abiertos a la comunidad y Microsoft agradece la colaboración de clientes, asociados y del sector. Ambos se basan en estándares de W3C abiertos, como JSON-LD y RDF, lo que facilita la adopción entre servicios y herramientas.

No hay costo adicional por el uso de IoT Plug and Play y DTDL. Las tarifas estándar de [Azure IoT Hub](../iot-hub/about-iot-hub.md) y otros servicios de Azure siguen siendo las mismas.

En este artículo se describen:

- Los roles típicos asociados a un proyecto que usa IoT Plug and Play.
- Cómo usar dispositivos de IoT Plug and Play en una aplicación.
- Cómo desarrollar una aplicación de dispositivo IoT que admita IoT Plug and Play.

## <a name="user-roles"></a>Roles de usuario

IoT Plug and Play es útil para dos tipos de desarrolladores:

- Un _generador de soluciones_ es responsable del desarrollo de una solución de IoT mediante Azure IoT Hub y otros recursos de Azure, y de indicar los dispositivos IoT que se van a integrar.
- Un _generador de dispositivos_ crea el código que se ejecuta en un dispositivo conectado a la solución.

## <a name="use-iot-plug-and-play-devices"></a>Uso de dispositivos IoT Plug and Play

Como generador de soluciones, puede usar [IoT Central](../iot-central/core/overview-iot-central.md) o [IoT Hub](../iot-hub/about-iot-hub.md) para desarrollar una solución IoT hospedada en la nube que use dispositivos IoT Plug and Play.

La interfaz de usuario web de IoT Central le permite supervisar las condiciones del dispositivo, crear reglas y administrar millones de dispositivos y sus datos a lo largo de su ciclo de vida. Los dispositivos IoT Plug and Play se conectan directamente a las aplicaciones IoT Central en las que se pueden usar paneles personalizables para supervisar y controlar los dispositivos. También se pueden usar plantillas de dispositivo en la interfaz de usuario web de IoT Central para crear y editar modelos DTDL.

IoT Hub: un servicio en la nube administrado que actúa como centro de mensajes para que haya una comunicación bidireccional segura entre la aplicación de IoT y los dispositivos. Al conectar un dispositivo IoT Plug and Play a un centro de IoT, puede usar la herramienta [Azure IoT Explorer](./howto-use-iot-explorer.md) para ver los datos de telemetría, las propiedades y los comandos definidos en el modelo de DTDL.

Si tiene sensores acoplados a una puerta de enlace de Windows o Linux, puede usar un [puente IoT Plug and Play](./concepts-iot-pnp-bridge.md) para conectar estos sensores y crear dispositivos IoT Plug and Play sin necesidad de escribir software o firmware de dispositivo (para [protocolos admitidos](./concepts-iot-pnp-bridge.md#supported-protocols-and-sensors)).

## <a name="develop-an-iot-device-application"></a>Desarrollo de una aplicación de dispositivo IoT

Como generador de dispositivos, puede desarrollar un producto de hardware de IoT que admita IoT Plug and Play. El proceso incluye tres pasos clave:

1. Definir el modelo del dispositivo. Cree un conjunto de archivos JSON que definan las funcionalidades del dispositivo mediante el [DTDL](https://github.com/Azure/opendigitaltwins-dtdl). Un modelo describe una entidad completa, como un producto físico, y define el conjunto de interfaces que implementa esa entidad. Las interfaces son contratos compartidos que identifican de forma única la telemetría, las propiedades y los comandos que admite un dispositivo. Las interfaces se pueden reutilizar en otros modelos.

1. Crear el software o firmware del dispositivo de forma que los datos de telemetría, las propiedades y los comandos sigan las convenciones de IoT Plug and Play. Si va a conectar los sensores existentes acoplados a una puerta de enlace Windows o Linux, el [puente de IoT Plug and Play](./concepts-iot-pnp-bridge.md) puede simplificar este paso.

1. El dispositivo anuncia el identificador del modelo como parte de la conexión de MQTT. El SDK de Azure IoT incluye nuevas construcciones para proporcionar el identificador de modelo en el momento de la conexión.

> [!Important]
> Los dispositivos IoT Plug and Play deben usar MQTT o MQTT sobre WebSockets. Otros protocolos, como AMQP o HTTP, no son válidos para implementar dispositivos IoT Plug and Play.

## <a name="device-certification"></a>Certificación de dispositivos

El [programa de certificación de dispositivos IoT Plug and Play](../certification/program-requirements-pnp.md) comprueba que un dispositivo cumple los requisitos de certificación de IoT Plug and Play. Puede agregar un dispositivo certificado al [catálogo de dispositivos certificados para IoT de Azure](https://aka.ms/devicecatalog) público.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una visión general de IoT Plug and Play, el siguiente paso que se recomienda es probar uno de los tutoriales de inicio rápido:

- [Conexión de un dispositivo a IoT Hub](./quickstart-connect-device.md)
- [Interacción con un dispositivo desde su solución](./quickstart-service.md)
