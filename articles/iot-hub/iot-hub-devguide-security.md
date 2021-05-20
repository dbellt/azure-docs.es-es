---
title: Control de acceso y seguridad en IoT Hub | Microsoft Docs
description: Información general sobre cómo controlar el acceso a IoT Hub, que incluye vínculos a artículos detallados sobre la integración de AAD y las opciones de SAS.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/15/2021
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: Operations'
- devx-track-js
- devx-track-csharp
ms.openlocfilehash: 617a68a36e0f4673ed89753d1e48f529725d1648
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109484188"
---
# <a name="control-access-to-iot-hub"></a>Control del acceso a IoT Hub

En esta sección se describen las opciones para proteger su instancia de IoT Hub. IoT Hub usa *permisos* para conceder acceso a cada uno de los puntos de conexión de IoT Hub. Los permisos limitan el acceso a un centro de IoT basado en la funcionalidad.

Hay tres maneras diferentes de controlar el acceso a IoT Hub:

- **Integración de Azure Active Directory (Azure AD)** para API de servicio. Azure proporciona autenticación basada en identidades con AAD y autorización específica con el control de acceso basado en rol de Azure (Azure RBAC). La integración de Azure AD y RBAC solo es compatible con las API de servicio de IoT Hub. Para más información, vea [Control del acceso a IoT Hub mediante Azure Active Directory](iot-hub-dev-guide-azure-ad-rbac.md).
- Las **firmas de acceso compartido** le permiten agrupar permisos y concederlos a las aplicaciones mediante claves de acceso y tokens de seguridad firmados. Para más información, vea [Control del acceso a IoT Hub mediante la firma de acceso compartido](iot-hub-dev-guide-sas.md). 
- **Credenciales de seguridad de cada dispositivo**. Cada instancia de IoT Hub contiene un [registro de identidades](iot-hub-devguide-identity-registry.md), y para cada dispositivo de este registro de identidades, puede configurar credenciales de seguridad que otorgan permisos DeviceConnect con ámbito a los puntos de conexión de dispositivo. Para más información, vea [Autenticación de un dispositivo en IoT Hub](iot-hub-dev-guide-sas.md#authenticating-a-device-to-iot-hub).

## <a name="next-steps"></a>Pasos siguientes

- [Control del acceso a IoT Hub mediante Azure Active Directory](iot-hub-dev-guide-azure-ad-rbac.md)
- [Control del acceso a IoT Hub mediante la firma de acceso compartido](iot-hub-dev-guide-sas.md)
- [Autenticación de un dispositivo en IoT Hub](iot-hub-dev-guide-sas.md#authenticating-a-device-to-iot-hub)
