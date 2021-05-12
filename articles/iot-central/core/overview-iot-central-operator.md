---
title: Guía del operador de Azure IoT Central
description: Azure IoT Central es una plataforma de aplicaciones de IoT que simplifica la creación de soluciones de IoT. En este artículo se proporciona información general el rol del operador en IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 03/19/2021
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: 0b29c9ceb5ad8cd7adb38582d087e91c2fe6e8b1
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743478"
---
# <a name="iot-central-operator-guide"></a>Guía del operador de IoT Central

Una aplicación IoT Central le permite supervisar y administrar millones de dispositivos a lo largo de su ciclo de vida. Esta guía es para los operadores que usan una aplicación de IoT Central para administrar dispositivos IoT.

El operador:

- Supervisa y administra los dispositivos conectados a la aplicación.
- Soluciona y corrige problemas con los dispositivos.
- Aprovisiona nuevos dispositivos.

## <a name="monitor-and-manage-devices"></a>Supervisión y administración de dispositivos

:::image type="content" source="media/overview-iot-central-operator/simulated-telemetry.png" alt-text="Captura de pantalla que muestra la vista de un dispositivo.":::

Para supervisar los dispositivos, el operador puede usar las vistas de dispositivo definidas por el generador de soluciones como parte de la plantilla de dispositivo. Estas vistas pueden mostrar los valores de propiedad y la telemetría del dispositivo. Un ejemplo es la vista **Información general** que se muestra en la captura de pantalla anterior.

Para información más detallada, el operador puede usar grupos de dispositivos y las características de análisis integradas. Para más información, consulte [Cómo usar análisis para analizar los datos del dispositivo](howto-create-analytics.md).

Para administrar dispositivos individuales, el operador puede usar las vistas de dispositivo para establecer las propiedades de la nube y del dispositivo, y llamar a los comandos del dispositivo. Entre los ejemplos se incluyen las vistas **Administrar dispositivo** y **Comandos** de la captura de pantalla anterior.

Para administrar dispositivos de forma masiva, el operador puede crear y programar trabajos. Los trabajos pueden actualizar propiedades y ejecutar comandos en varios dispositivos. Para más información, consulte [Crear y ejecutar trabajos en la aplicación de Azure IoT Central](howto-run-a-job.md).

## <a name="troubleshoot-and-remediate-issues"></a>Solución y corrección de problemas

El operador es responsable del mantenimiento de la aplicación y de sus dispositivos. La [guía de solución de problemas](troubleshoot-connection.md) ayuda a los operadores a diagnosticar y corregir problemas comunes. El operador puede usar la página **Dispositivos** para bloquear los dispositivos que parezca que no funcionan correctamente hasta que el problema se resuelva.

## <a name="add-and-remove-devices"></a>Adición y eliminación de dispositivos

El operador puede agregar dispositivos a la aplicación de IoT Central de forma individual o masiva, así como eliminarlos de esta. Para más información, consulte [Administración de dispositivos en la aplicación de Azure IoT Central](howto-manage-devices.md).

## <a name="personalize"></a>Personalizar

Los operadores pueden crear paneles personalizados en una aplicación IoT Central que contengan vínculos a los recursos que usan con mayor frecuencia. Para más información, consulte [Administración de paneles](howto-create-personal-dashboards.md#manage-dashboards).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el uso de IoT Central, los siguientes pasos sugeridos son probar los inicios rápidos, empezando por [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md).
