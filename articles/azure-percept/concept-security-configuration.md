---
title: Recomendaciones de seguridad y configuración del firewall de Azure Percept
description: Obtenga más información sobre las recomendaciones de seguridad y configuración del firewall de Azure Percept
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: conceptual
ms.date: 03/25/2021
ms.openlocfilehash: 9564160698b1f1bf17fc89ec13b1e292aa3b6098
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137552"
---
# <a name="azure-percept-firewall-configuration-and-security-recommendations"></a>Recomendaciones de seguridad y configuración del firewall de Azure Percept

Revise las directrices siguientes para obtener información sobre la configuración de firewalls y los procedimientos recomendados de seguridad generales con Azure Percept.

## <a name="configuring-firewalls-for-azure-percept-dk"></a>Configuración de firewalls para Azure Percept DK

Si la configuración de redes requiere agregar en la lista de permitidos las conexiones hechas desde los dispositivos de Azure Percept DK, revise la siguiente lista de componentes:

Esta lista de comprobación es un punto de partida para las reglas de firewall:

|URL (* = comodín)|Puertos TCP salientes|Uso|
|-------------------|------------------|---------|
|*.auth.azureperceptdk.azure.net|443|Autenticación y autorización del módulo de sistema de Azure Percept DK|
|*.auth.projectsantacruz.azure.net|443|Autenticación y autorización del módulo de sistema de Azure Percept DK|

Además, revise la lista de [conexiones que usa Azure IoT Edge](../iot-edge/production-checklist.md#allow-connections-from-iot-edge-devices).

## <a name="additional-recommendations-for-deployment-to-production"></a>Recomendaciones adicionales para la implementación en producción

Azure Percept DK ofrece una gran variedad de funcionalidades de seguridad listas para usarse. Además de esas eficaces características de seguridad incluidas en la versión actual, Microsoft también sugiere las siguientes directrices al tener en cuenta las implementaciones de producción:

- Protección física sólida del propio dispositivo
- Asegurarse de que el cifrado de datos en reposo esté habilitado
- Supervisión continua de la postura del dispositivo y respuesta rápida a las alertas
- Limitación del número de administradores que tienen acceso al dispositivo

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre la seguridad de Azure Percept](./overview-percept-security.md).