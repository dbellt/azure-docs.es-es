---
title: 'Configuración de protocolos de API: Event Grid en IoT Edge | Microsoft Docs'
description: Obtenga información sobre las configuraciones de protocolo posibles de un módulo de Event Grid.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 05/10/2021
ms.subservice: iot-edge
ms.topic: article
ms.openlocfilehash: 2bd533cb59fd661e3ac6dff9ff6aa3fa2d646889
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370725"
---
# <a name="configure-event-grid-api-protocols"></a>Configuración de protocolos de API en Event Grid

En esta guía se proporcionan ejemplos de las posibles configuraciones de protocolos de un módulo de Event Grid. El módulo de Event Grid expone la API en sus operaciones de administración y tiempo de ejecución. En la tabla siguiente se capturan los protocolos y puertos.

| Protocolo | Port | Descripción |
| ---------------- | ------------ | ------------ |
| HTTP | 5888 | Desactivado de forma predeterminada. Solo es útil durante las pruebas. No es adecuado para cargas de trabajo de producción.
| HTTPS | 4438 | Valor predeterminado

Consulte la guía [Seguridad y autenticación](security-authentication.md) para ver todas las configuraciones posibles.

## <a name="expose-https-to-iot-modules-on-the-same-edge-network"></a>Exposición de HTTPS en módulos IoT en la misma red perimetral

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ]
}
 ```

## <a name="enable-https-to-other-iot-modules-and-non-iot-workloads"></a>Habilitación de HTTPS para otros módulos IoT y cargas de trabajo no IoT

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
 ```

>[!NOTE]
> La sección **PortBindings** permite asignar puertos internos a los puertos del host del contenedor. Esta característica posibilita el acceso al módulo de Event Grid desde fuera de la red del contenedor de IoT Edge, si el dispositivo IoT Edge es accesible públicamente.

## <a name="expose-http-and-https-to-iot-modules-on-the-same-edge-network"></a>Exposición de HTTP y HTTPS a módulos IoT en la misma red perimetral

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=enabled",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ]
}
 ```

## <a name="enable-http-and-https-to-other-iot-modules-and-non-iot-workloads"></a>Habilitación de HTTP y HTTPS para otros módulos IoT y cargas de trabajo no IoT

```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=enabled",
    "inbound__serverAuth__serverCert__source=IoTEdge"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ],
      "5888/tcp": [
        {
          "HostPort": "5888"
        }
      ]
    }
  }
}
 ```

>[!NOTE]
> De forma predeterminada, cada módulo IoT es parte del entorno de ejecución de IoT Edge creado por la red de puente. Esto permite la comunicación entre diferentes módulos IoT de la misma red. **PortBindings** permite asignar un puerto interno del contenedor a la máquina host, de modo que cualquier usuario puede acceder al puerto del módulo de Event Grid desde fuera.

>[!IMPORTANT]
> Aunque los puertos pueden hacerse accesibles fuera de la red de IoT Edge, la autenticación de cliente determina quién puede realmente realizar llamadas en el módulo.
