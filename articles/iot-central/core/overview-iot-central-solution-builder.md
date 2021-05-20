---
title: Compilación de soluciones para Azure IoT Central | Microsoft Docs
description: Azure IoT Central es una plataforma de aplicaciones de IoT que simplifica la creación de soluciones de IoT. En este artículo se proporciona información general sobre la creación de soluciones integradas con IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 02/11/2021
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: dc8ca48d11f31f69ca9db5dce53cce4d9d6e8f25
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108772340"
---
# <a name="iot-central-solution-builder-guide"></a>Guía del creador de soluciones de IoT Central

Una aplicación IoT Central le permite supervisar y administrar millones de dispositivos a lo largo de su ciclo de vida. Esta guía está orientada a los creadores de soluciones que usan IoT Central para crear soluciones integradas. Una aplicación IoT Central le permite administrar dispositivos, analizar su telemetría e integrarlos con otros servicios de back-end.

Un creador de soluciones:

- Configura los paneles y las vistas de la interfaz de usuario web de IoT Central.
- Usa las reglas y las herramientas de análisis integradas para extraer conclusiones empresariales de los dispositivos conectados.
- Usa las funcionalidades de exportación de datos y reglas para integrar IoT Central con otros servicios de back-end.

## <a name="configure-dashboards-and-views"></a>Configuración de paneles y vistas

Una aplicación IoT Central puede tener uno o más paneles que los operadores usan para ver la aplicación e interactuar con ella. Como creador de soluciones, puede personalizar el panel predeterminado y crear paneles especializados:

- Para ver algunos ejemplos de paneles personalizados, consulte [Plantillas centradas en el sector](concepts-app-templates.md#industry-focused-templates).
- Para más información sobre los paneles, consulte [Creación y administración de varios paneles](howto-create-personal-dashboards.md) y [Configuración del panel de la aplicación](howto-add-tiles-to-your-dashboard.md).

Cuando un dispositivo se conecta a una instancia de IoT Central, se asocia con una plantilla correspondiente al tipo de dispositivo. Una plantilla de dispositivo tiene vistas personalizables que un operador utiliza para administrar dispositivos individuales. Como desarrollador de soluciones, puede crear y personalizar las vistas disponibles para un tipo de dispositivo. Para más información, consulte [Adición de vistas](howto-set-up-template.md#add-views).

## <a name="use-built-in-rules-and-analytics"></a>Uso de reglas y análisis integrados

Un desarrollador de soluciones puede agregar reglas a una aplicación IoT Central que ejecute acciones personalizables. Las reglas evalúan condiciones, en función de los datos procedentes de un dispositivo, para determinar cuándo se debe ejecutar una acción. Para más información sobre las reglas, consulte:

- [Tutorial: Creación de una regla y configuración de las notificaciones en la aplicación de Azure IoT Central](tutorial-create-telemetry-rules.md)
- [Creación de acciones de webhook para reglas en Azure IoT Central](howto-create-webhooks.md)
- [Agrupación de varias acciones para ejecutar desde una o varias reglas](howto-use-action-groups.md)

IoT Central cuenta con funcionalidades de análisis integradas que un operador puede usar para analizar los datos que fluyen desde los dispositivos conectados. Para más información, consulte [Cómo usar análisis para analizar los datos del dispositivo](howto-create-analytics.md).

## <a name="integrate-with-other-services"></a>Integración con otros servicios

Como creador de soluciones, puede usar las funcionalidades de exportación de datos y reglas de IoT Central para realizar la integración con otro servicio. Para obtener más información, consulte:

- [Exportación de datos de IoT a destinos en la nube mediante la característica de exportación de datos](howto-export-data.md)
- [Transformación de datos para IoT Central](howto-transform-data.md)
- [Uso de flujos de trabajo para integrar la aplicación de Azure IoT Central con otros servicios en la nube](howto-configure-rules-advanced.md)
- [Extensión de Azure IoT Central con reglas personalizadas mediante Stream Analytics, Azure Functions y SendGrid](howto-create-custom-rules.md)
- [Ampliación de Azure IoT Central con análisis personalizados mediante Azure Databricks](howto-create-custom-analytics.md)
- [Visualización y análisis de los datos de Azure IoT Central en un panel de Power BI](howto-connect-powerbi.md)

## <a name="apis"></a>API existentes

La API de IoT Central permite crear integraciones profundas con otros servicios de la solución IoT. Las API disponibles se clasifican como API de *plano de datos* o de *plano de control*.

Las API del plano de datos se usan para acceder a las entidades y a las funcionalidades de la aplicación IoT Central. Por ejemplo, administración de dispositivos, plantillas de dispositivo, usuarios y roles. Las API de REST de IoT Central son operaciones de *plano de datos*. Para obtener más información, consulte [Uso de la API de REST de IoT Central para administrar usuarios y roles](howto-manage-users-roles-with-rest-api.md).

El *plano de control* se utiliza para administrar los recursos de la suscripción de Azure relacionados con IoT Central. Puede usar las plantillas de CLI de Azure y de Resource Manager para las operaciones del plano de control. Por ejemplo, la CLI de Azure se puede utilizar para crear una aplicación de IoT Central. Para obtener más información, consulte [Administración de IoT Central desde la CLI de Azure](howto-manage-iot-central-from-cli.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el uso de IoT Central, los siguientes pasos sugeridos son probar los inicios rápidos, empezando por [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md).
