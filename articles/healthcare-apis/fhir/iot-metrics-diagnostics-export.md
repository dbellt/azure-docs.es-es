---
title: Exportación de las métricas de Azure IoT Connector for FHIR (versión preliminar) mediante la configuración de diagnóstico
description: En este artículo se explica cómo exportar las métricas de Azure IoT Connector for FHIR (versión preliminar) mediante la configuración de diagnóstico
services: healthcare-apis
author: msjasteppe
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: how-to
ms.date: 11/13/2020
ms.author: jasteppe
ms.openlocfilehash: efdf932a26e783ff4e7fc22820e061fb9b2e26ba
ms.sourcegitcommit: 0beea0b1d8475672456da0b3a4485d133283c5ea
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2021
ms.locfileid: "112992160"
---
# <a name="export-azure-iot-connector-for-fhir-preview-metrics-through-diagnostic-settings"></a>Exportación de las métricas de Azure IoT Connector for FHIR (versión preliminar) mediante la configuración de diagnóstico

En este artículo, aprenderá a exportar el Conector de Azure IoT para registros de métricas de Recursos Rápidos de Interoperabilidad en Salud (FHIR&#174;)*. La característica que permite el registro de métricas es la [**configuración de diagnóstico**](../../azure-monitor/essentials/diagnostic-settings.md) de Azure Portal. 

> [!TIP]
> Para configurar el registro de auditoría, siga las instrucciones que se indican en [Habilitación del registro de diagnóstico en Azure API for FHIR y Azure IoT Connector for FHIR](enable-diagnostic-logging.md#enable-diagnostic-logging-in-azure-api-for-fhir).

## <a name="enable-metrics-logging-for-the-azure-iot-connector-for-fhir-preview"></a>Habilitación del registro de métricas para Azure IoT Connector for FHIR (versión preliminar)
1. Para habilitar el registro de métricas para Azure IoT Connector for FHIR, seleccione el servicio Azure API for FHIR en Azure Portal. 

2. Vaya a **Configuración de diagnóstico**. 

3. Seleccione **+ Agregar configuración de diagnóstico**.

   :::image type="content" source="media/iot-metrics-export/diagnostic-settings-main.png" alt-text="IoT Connector1" lightbox="media/iot-metrics-export/diagnostic-settings-main.png"::: 

4. Escriba un nombre en el cuadro de diálogo **Diagnostic setting name** (Nombre de la configuración de diagnóstico).

5. Seleccione el método que desea usar para acceder a los registros de diagnóstico:

    1. **Archivar en una cuenta de almacenamiento** para auditoría o inspección manual. Es necesario que la cuenta de almacenamiento que desea usar ya esté creada.
    2. **Transmitir a un centro de eventos** para la ingesta en un servicio de terceros o una solución de análisis personalizado. Tendrá que crear un espacio de nombres de centro de eventos y una directiva de centro de eventos para poder configurar este paso.
    3. **Transmitir al área de trabajo de Log Analytics** en Azure Monitor. Tendrá que crear el área de trabajo de Log Analytics para poder seleccionar esta opción.

6. Seleccione **Errors, Traffic, and Latency** (Errores, tráfico y latencia) para Azure IoT Connector for FHIR.  Seleccione todas las categorías de métricas adicionales que quiera capturar para Azure API for FHIR.

7. Seleccione **Guardar**.

   :::image type="content" source="media/iot-metrics-export/diagnostic-setting-add.png" alt-text="IoT Connector2" lightbox="media/iot-metrics-export/diagnostic-setting-add.png":::

> [!Note] 
> Los primeros registros de métricas pueden tardar hasta 15 minutos en mostrarse en el repositorio de su elección.  
 
Para más información sobre cómo trabajar con los registros de diagnóstico, consulte la [documentación del registro de recursos de Azure](../../azure-monitor/essentials/platform-logs-overview.md).

## <a name="conclusion"></a>Conclusión 
Tener acceso a los registros de métricas es fundamental para las tareas de supervisión y solución de problemas.  Azure IoT Connector for FHIR le permite realizar estas acciones mediante registros de métricas. 

## <a name="next-steps"></a>Pasos siguientes

Consulte las preguntas más frecuentes sobre el conector de Azure IoT para FHIR

>[!div class="nextstepaction"]
>[P+ del conector de Azure IoT para FHIR](fhir-faq.yml)

*En Azure Portal, el Conector de Azure IoT para FHIR se conoce como Conector de IoT (versión preliminar). FHIR es una marca registrada de HL7 y se usa con el permiso de HL7.