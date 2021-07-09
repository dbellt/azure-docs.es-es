---
title: Introducción a Azure IoT industrial
description: En este artículo se proporciona información general de IoT industrial. En él se explican los componentes de seguridad y conectividad de la zona de producción en IIoT.
author: jehona-m
ms.author: jemorina
ms.service: industrial-iot
ms.topic: overview
ms.date: 3/22/2021
ms.openlocfilehash: 902cd7c9cf0f682541be86e82a1d51525ad5b3f9
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110678038"
---
# <a name="what-is-industrial-iot-iiot"></a>¿Qué es IoT industrial (IIoT)?

![IoT industrial](media/overview-what-is-Industrial-IoT/icon-255-px.png)

Microsoft Azure IoT industrial (IIoT) es un conjunto de módulos y servicios de Azure que integran la eficacia de la nube en plantas industriales y de fabricación. Mediante interfaces abiertas estándares del sector, como la [arquitectura unificada de comunicaciones de plataforma abierta (OPC UA)](https://opcfoundation.org/about/opc-technologies/opc-ua/), Azure IIoT le proporciona la capacidad de integrar datos de recursos y sensores,incluidos los que ya funcionan en la fábrica, en la nube de Azure. Tener los datos en la nube permite que se utilicen de forma más rápida y flexible como retroalimentación para desarrollar procesos empresariales e industriales transformadores.

## <a name="discover-register-and-manage-your-industrial-assets-with-azure"></a>Detección, registro y administración de los recursos industriales con Azure

La plataforma de Azure Industrial IoT permite a los operarios de la planta detectar los servidores habilitados para OPC UA de una red de la fábrica y registrarlos en Azure IoT Hub. El personal de operaciones puede suscribirse y reaccionar a los eventos de la fábrica desde cualquier lugar del mundo, recibir alertas y alarmas, y reaccionar ante ellos en tiempo real.

IIoT proporciona un conjunto de microservicios que implementan la funcionalidad de OPC UA. Las API REST de microservicios reflejan el lado perimetral de los servicios de OPC UA. Esto permite que las aplicaciones en la nube examinen los espacios de direcciones de servidor o las variables de lectura y escritura, y ejecutar métodos mediante HTTPS y cargas JSON sencillas de OPC UA. Los servicios perimetrales se implementan como módulos de Azure IoT Edge y se ejecutan de forma local. Los microservicios en la nube se implementan como microservicios de ASP.NET con una interfaz de REST y se ejecutan en una instancia administrada de Azure Kubernetes Services o de forma independiente en Azure App Service. Para los servicios perimetrales y en la nube, IIoT proporciona contenedores de Docker creados previamente en Microsoft Container Registry (MCR). Los servicios perimetrales y en la nube se utilizan entre sí y deben usarse juntos. IIoT también proporciona scripts de implementación fáciles de usar que permiten implementar toda la plataforma con un solo comando.

Además, se pueden usar las API REST con cualquier lenguaje de programación mediante la especificación Open API (Swagger) expuesta. Esto significa que, al integrar OPC UA en soluciones de administración en la nube, los desarrolladores pueden elegir la tecnología que coincida con sus aptitudes, intereses y opciones de arquitectura. Por ejemplo, un desarrollador web de pila completa que desarrolle una aplicación para un panel de alarmas y eventos puede escribir la lógica para responder a los eventos en JavaScript o TypeScript sin tener conocimientos del SDK de OPC UA, C, C++, Java o C#.

## <a name="manage-certificates-and-trust-groups"></a>Administración de certificados y grupos de confianza

Azure IoT industrial administra certificados de aplicación y listas de confianza de OPC UA de los sistemas de control y la maquinaria de la fábrica para mantener la comunicación del cliente OPC UA y el servidor de forma segura. Se restringe qué cliente puede hablar con qué servidor. El almacenamiento de claves privadas y la firma de certificados se lleva a cabo mediante Azure Key Vault, que admite seguridad basada en hardware (HSM).

## <a name="industrial-iot-components"></a>Componentes de IoT industrial

Las soluciones de Azure IIoT se crean a partir de componentes específicos. Entre ellos se incluyen los siguientes.

- **Al menos una instancia de Azure IoT Hub.**
- **Dispositivos IoT Edge.**
- **Módulos Edge industriales.**

### <a name="iot-hub"></a>IoT Hub
[Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/) actúa como un centro de mensajes común para la comunicación bidireccional segura entre la aplicación IoT y los dispositivos que administra. Es una oferta de plataforma como servicio (PaaS) en la nube, abierta y flexible que admite SDK de código abierto y diversos protocolos. 

La recopilación de los datos industriales y empresariales en un centro de IoT permite almacenar los datos de forma segura, realizar análisis empresariales y de eficiencia sobre los datos y generar informes a partir de ellos. También puede aplicar servicios y herramientas de Microsoft Azure, como [Power BI](https://powerbi.microsoft.com), en los datos consolidados.

### <a name="iot-edge-devices"></a>Dispositivos de IoT Edge
Los [servicios perimetrales](https://azure.microsoft.com/services/iot-edge/) se implementan como módulos Azure IoT Edge y se ejecutan de forma local. Los microservicios en la nube se implementan como microservicios de ASP.NET con una interfaz de REST y se ejecutan en una instancia administrada de Azure Kubernetes Services o de forma independiente en Azure App Service. En el caso de los servicios perimetrales y en la nube, hemos proporcionado contenedores de Docker creados previamente en Microsoft Container Registry (MCR), lo que elimina este paso para el cliente. Los servicios perimetrales y en la nube se utilizan entre sí y deben usarse juntos. También hemos proporcionado scripts de implementación fáciles de usar que permiten implementar toda la plataforma con un solo comando.

Un dispositivo IoT Edge se compone del entorno de ejecución de Edge y de módulos Edge.
- Los **módulos de Edge** son contenedores de Docker, que son la unidad de cálculo más pequeña, como OPC Publisher y OPC Twin. 
- El **dispositivo de Edge** se usa para implementar estos módulos, que actúan como mediadores entre el servidor de OPC UA e IoT Hub en la nube.

### <a name="industrial-edge-modules"></a>Módulos de Edge industrial
- **OPC Publisher**: OPC Publisher se ejecuta dentro de IoT Edge. Se conecta a los servidores de OPC UA y publica los datos de telemetría codificados en JSON desde estos servidores en formato "pub/sub" de OPC UA para Azure IoT Hub. Se pueden usar todos los protocolos de transporte admitidos por el SDK de cliente de Azure IoT Hub, es decir, HTTPS, AMQP y MQTT.
- **OPC Twin**: consta de microservicios que usan Azure IoT Edge e IoT Hub para conectar la nube y la red de fábrica. OPC Twin proporciona la detección, el registro y el control remoto de dispositivos industriales mediante las API REST. OPC Twin no requiere un SDK de OPC Unified Architecture (OPC UA). Es independiente del lenguaje de programación y puede incluirse en un flujo de trabajo sin servidor.
- **Detección**: el módulo de detección, representado por la identidad del programa de detección, proporciona servicios de detección en el perímetro, que incluyen la detección de servidores de OPC UA. Si la detección está configurada y habilitada, el módulo enviará los resultados de un sondeo de examen a través de la ruta de acceso de telemetría de IoT Edge e IoT Hub al servicio de incorporación. El servicio procesa los resultados y actualiza todas las identidades relacionadas en el Registro.

## <a name="next-steps"></a>Pasos siguientes
Ahora que aprendió qué es Industrial IoT, puede leer más sobre OPC Publisher o empezar a implementar la plataforma de IIoT:

> [!div class="nextstepaction"]
> [¿Qué es OPC Publisher?](overview-what-is-opc-publisher.md)

> [!div class="nextstepaction"]
> [Implementación de la plataforma de Industrial IoT](tutorial-deploy-industrial-iot-platform.md)