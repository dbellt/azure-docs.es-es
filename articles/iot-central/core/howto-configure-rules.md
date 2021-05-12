---
title: Configuración de reglas y acciones en Azure IoT Central | Microsoft Docs
description: En este artículo paso a paso se muestra cómo puede, como generador, configurar las reglas y las acciones basadas en la telemetría en la aplicación de Azure IoT Central.
author: vavilla
ms.author: vavilla
ms.date: 12/23/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: philmea
ms.openlocfilehash: c78b95e7651117f85f3cf38f5cbff1b890145dae
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759552"
---
# <a name="configure-rules"></a>Configuración de reglas

Las reglas de IoT Central sirven como herramienta de respuesta personalizable que se desencadena en eventos supervisados activamente desde dispositivos conectados. En las secciones siguientes se describe cómo se evalúan las reglas.

## <a name="select-target-devices"></a>Seleccionar dispositivos de destino

Use la sección de dispositivos de destino para seleccionar el tipo de dispositivos que se aplicará a esta regla. Asimismo, los filtros le permiten restringir aún más los dispositivos que deben incluirse. Tenga en cuenta que los filtros usan propiedades en la plantilla de dispositivos para así poder filtrar el conjunto de dispositivos. Los filtros no desencadenan una acción. En la siguiente captura de pantalla, los dispositivos de destino son el tipo de plantilla de dispositivo **Frigorífico**. El filtro indica que la regla solo debe incluir **Frigoríficos** en los que la propiedad **Estado de fabricación** sea igual a **Washington**.

![Condiciones](media/howto-configure-rules/filters.png)

## <a name="use-multiple-conditions"></a>Uso de varias condiciones

Las reglas se activan según las condiciones. Actualmente, cuando se agregan varias condiciones a una regla, se agrupan lógicamente mediante AND. En otras palabras, se deben cumplir todas las condiciones para que la regla se evalúe como "true".  

En la siguiente captura de pantalla, las condiciones comprueban si la temperatura es mayor que 70 &deg;F y la humedad es menor que 10. Cuando ambas instrucciones son "true", la regla se evalúa como "true" y se desencadena una acción.

![Captura de pantalla que muestra el monitor de un refrigerador con las condiciones especificadas para la temperatura y la humedad.](media/howto-configure-rules/conditions.png)

### <a name="use-a-cloud-property-in-a-value-field"></a>Uso de una propiedad en la nube en un campo de valor

Puede hacer referencia a una propiedad en la nube desde la plantilla de dispositivo en el campo **Valor** de una condición. La propiedad en la nube y el valor de telemetría deben tener tipos similares. Por ejemplo, si **Temperatura** es un valor de tipo Double, solo se mostrarán como opciones en la lista desplegable **Valor** las propiedades en la nube de tipo Double.

Si elige un valor de telemetría de tipo de evento, la lista desplegable **Valor** incluye la opción **Cualquiera**. La opción **Cualquier** significa que la regla se desencadena cuando la aplicación recibe un evento de ese tipo, sea cual sea la carga.

## <a name="use-aggregate-windowing"></a>Uso de la opción para agregar funciones de ventana

Las reglas evalúan ventanas de tiempo de agregado como, como las ventanas de saltos de tamaño constante. En la captura de pantalla siguiente, la ventana de tiempo es de cinco minutos. Cada cinco minutos, la regla se evalúa en función los últimos cinco minutos de datos. Los datos solo se evalúan una vez en la ventana a la que correspondan.

![Ventana de saltos de tamaño constante](media/howto-configure-rules/tumbling-window.png)

## <a name="use-rules-with-iot-edge-modules"></a>Usar reglas con módulos de IoT Edge

Una restricción se aplica a las reglas que se aplican a los módulos de IoT Edge. Las reglas en la telemetría de módulos diferentes no se evalúan como reglas válidas. Intente lo siguiente como ejemplo: La primera condición de la regla se encuentra en la telemetría de la temperatura del módulo A. La segunda condición de la regla se encuentra en la telemetría de la humedad en el módulo B. Dado que las dos condiciones provienen de módulos diferentes, se trata de un conjunto no válido de condiciones. Esta regla no es válida y producirá un error al intentar guardarla.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a configurar una regla en su aplicación de Azure IoT Central, puede aprender a [configurar reglas avanzadas](howto-configure-rules-advanced.md) mediante Power Automate o Azure Logic Apps.
