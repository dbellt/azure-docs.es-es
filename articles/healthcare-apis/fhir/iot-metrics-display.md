---
title: Visualización y configuración de las métricas de Azure IoT Connector for FHIR (versión preliminar)
description: En este artículo se explica cómo mostrar y configurar las métricas del conector de Azure IoT para FHIR (versión preliminar).
services: healthcare-apis
author: msjasteppe
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: how-to
ms.date: 11/13/2020
ms.author: jasteppe
ms.openlocfilehash: 08d572970cc96eeb28b831293b82ab6ab3257cce
ms.sourcegitcommit: 0beea0b1d8475672456da0b3a4485d133283c5ea
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2021
ms.locfileid: "112991224"
---
# <a name="display-and-configure-azure-iot-connector-for-fhir-preview-metrics"></a>Visualización y configuración de las métricas de Azure IoT Connector for FHIR (versión preliminar) 

En este artículo, aprenderá a mostrar y configurar el conector de Azure IoT para métricas de Recursos Rápidos de Interoperabilidad en Salud (FHIR&#174;)*.

> [!TIP]
> Para aprender a configurar la exportación de los datos de métricas, siga las instrucciones que aparecen en [Exportación de las métricas de Azure IoT Connector for FHIR (versión preliminar) mediante la configuración de diagnóstico](iot-metrics-diagnostics-export.md).

## <a name="display-metrics-for-azure-iot-connector-for-fhir-preview"></a>Visualización de las métricas del conector de Azure IoT para FHIR (versión preliminar)

1. Inicie sesión en Azure Portal y seleccione el servicio Azure API for FHIR. 

2. En el panel izquierdo, seleccione **Métricas**. 

3. Seleccione la pestaña **Conector de IoT**.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-main.png" alt-text="Captura de pantalla del panel &quot;Conector de IoT&quot;, con gráficos de líneas que muestran el número de mensajes entrantes y normalizados." lightbox="media/iot-metrics-display/iot-metrics-main.png"::: 

4. Seleccione un conector de IoT para ver sus métricas. Por ejemplo, hay cuatro conectores de IoT (conector *1*, *conector 2*, etc.) asociados a este servicio Azure API for FHIR.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-select-connector.png" alt-text="Captura de pantalla del panel &quot;Conector de IoT&quot; que muestra las pestañas de los conectores 1, 2, 3 y 4." lightbox="media/iot-metrics-display/iot-metrics-select-connector.png"::: 

5. Seleccione el período de tiempo (por ejemplo, **1 hora**, **24 horas**, **7 días** o **Personalizado**) de las métricas del conector de IoT que desea mostrar. La pestaña **Personalizado** permite crear combinaciones específicas de fecha y hora para ver las métricas de los conectores de IoT.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-select-time.png" alt-text="Captura de pantalla del panel &quot;Conector de IoT&quot;, que muestra un gráfico de líneas del período de tiempo &quot;1 hora&quot; para &quot;conector 1&quot;." lightbox="media/iot-metrics-display/iot-metrics-select-time.png"::: 
 
## <a name="metric-types-for-azure-iot-connector-for-fhir-preview"></a>Tipos de métricas para el conector de Azure IoT para FHIR (versión preliminar) 

> [!TIP]
> Para obtener información sobre el flujo de datos en el conector de Azure IoT para FHIR, vea [Flujo de datos del conector de Azure IoT para FHIR (versión preliminar)](iot-data-flow.md), y [Guía de solución de problemas del conector de Azure IoT para FHIR (versión preliminar)](iot-troubleshoot-guide.md) para obtener más información sobre los mensajes de error y las correcciones.

Las métricas del conector de IoT que se pueden mostrar se enumeran en la tabla siguiente:

|Tipo de métrica|Finalidad de la métrica| 
|-----------|--------------|
|Número de mensajes entrantes|Muestra el número de mensajes entrantes recibidos sin procesar (por ejemplo, los eventos de dispositivo).|
|Número de mensajes normalizados|Muestra el número de mensajes normalizados.|
|Número de grupos de mensajes|Muestra el número de grupos que tienen mensajes agregados en la ventana de tiempo designada.|
|Latencia de la fase normalizada promedio|Muestra la latencia promedio de la fase normalizada. La fase normalizada realiza la normalización de los mensajes entrantes sin procesar.|
|Latencia de la fase de grupo promedio|Muestra la latencia promedio de la fase de grupo. La fase de grupo realiza el almacenamiento en búfer, la agregación y la agrupación en mensajes normalizados.| 
|Recuento total de errores|Muestra el número total de errores.| 

## <a name="focus-on-and-configure-azure-iot-connector-for-fhir-preview-metrics"></a>Enfoque en las métricas de Azure IoT Connector for FHIR (versión preliminar) y configuración de las mismas

En este ejemplo, nos centraremos en la métrica **Number of Incoming Messages** (Número de mensajes entrantes).

1. Seleccione un momento dado en el que quiera centrarse.

   :::image type="content" source="media/iot-metrics-display/iot-metrics-focus.png" alt-text="Captura de pantalla del panel de métricas &quot;Number of Incoming Messages&quot; (Número de mensajes entrantes), que resalta un único punto en el tiempo en el gráfico de líneas." lightbox="media/iot-metrics-display/iot-metrics-focus.png"::: 

2. En el panel **Number of Incoming Messages** (Número de mensajes entrantes), puede personalizar aún más la métrica; para ello, seleccione **Agregar métrica**, **Agregar filtro** o **Apply splitting** (Aplicar división). 

   :::image type="content" source="media/iot-metrics-display/iot-metrics-add-options.png" alt-text="Captura de pantalla del panel de métricas &quot;Number of Incoming Messages&quot; (Número de mensajes entrantes), donde se resaltan los botones &quot;Agregar métrica&quot;, &quot;Agregar filtro&quot; y &quot;Apply splitting&quot; (Aplicar división)." lightbox="media/iot-metrics-display/iot-metrics-add-options.png"::: 

## <a name="conclusion"></a>Conclusión 
Tener acceso a las métricas del plano de datos es fundamental para las tareas de supervisión y solución de problemas. Conector de Azure IoT para FHIR lo ayuda con estas acciones mediante las métricas. 

## <a name="next-steps"></a>Pasos siguientes

Obtenga respuestas a las preguntas más frecuentes sobre el conector de Azure IoT para FHIR.

>[!div class="nextstepaction"]
>[P+F del conector de Azure IoT para FHIR](fhir-faq.yml)

*En Azure Portal, el conector de Azure IoT para FHIR se conoce como conector de IoT (versión preliminar). FHIR es una marca registrada de HL7 y se usa con el permiso de HL7. 
