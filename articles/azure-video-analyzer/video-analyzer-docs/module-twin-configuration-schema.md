---
title: Esquema JSON del módulo gemelo de Azure Video Analyzer
description: En este artículo se proporciona información general sobre un esquema JSON de módulo gemelo de Azure Video Analyzer.
ms.topic: how-to
ms.date: 04/30/2021
ms.openlocfilehash: b87074040a50f0b3a68a71e987e4cac3c3b48398
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388669"
---
# <a name="module-twin-configuration-schema"></a>Esquema de configuración de módulos gemelos

Los dispositivos gemelos son documentos JSON que almacenan información acerca del estado del dispositivo, incluidos metadatos, configuraciones y condiciones. Azure IoT Hub mantiene un dispositivo gemelo para cada dispositivo que se conecta a IoT Hub. Si desea una explicación detallada, consulte [Uso de módulos gemelos en IoT Hub](../../iot-hub/iot-hub-devguide-module-twins.md).

En este tema se describe el esquema JSON de módulo gemelo de Azure Video Analyzer.

## <a name="module-twin-properties"></a>Propiedad del módulo gemelo

Azure Video Analyzer expone estas propiedades del módulo gemelo.

| Propiedad                    | Obligatorio | Dinámica | Descripción                                                  |
| :-------------------------- | :------- | :------ | :----------------------------------------------------------- |
| `ProvisioningToken`          | Sí      | No      | Token de autenticación para validar el módulo perimetral y aprovisionar los servicios en la nube (incluido el acceso a la cuenta de Video Analyzer) |
| `ApplicationDataDirectory`    | Sí      | No      | Ruta de acceso dentro del sistema de archivos del módulo que se asigna a un volumen montado para conservar la configuración.       |
| `DiagnosticsEventsOutputName` | No       | Sí     | Salida del centro para los eventos de diagnóstico. (Vacío significa que los diagnósticos no están publicados) |
| `OperationalEventsOutputName` | No       | Sí     | Salida del concentrador para eventos operativos. (Vacío significa que los eventos operativos no se publican) |
| `LogLevel`                    | No       | Sí     | Uno de los siguientes: · Detallado · Información (valor predeterminado) · Advertencia · Error · Ninguno |
| `LogCategories`               | No       | Sí     | Lista separada por comas de los siguientes elementos: Application, MediaPipeline, Events Valor predeterminado: Application, Events |
| `AllowUnsecuredEndpoints`     | No       | Sí     | Valor booleano para permitir la creación de topologías con puntos de conexión no seguros, como `#Microsoft.VideoAnalyzer.UnsecuredEndpoint`; valor predeterminado -true        |
| `TelemetryOptOut`             | No       | No     | Valor booleano para no participar en el envío de datos de telemetría; valor predeterminado -false       |
| `DebugLogsDirectory`          | No       | Sí     | Directorio para los registros de depuración. Si está presente, se generan registros. Si no están presentes, los registros de depuración están deshabilitados.       |

Las propiedades dinámicas se pueden actualizar sin necesidad de reiniciar el módulo. 

Consulte el artículo sobre [la supervisión y el registro](monitor-log-edge.md) para obtener más información sobre el rol de la configuración de diagnóstico opcional.

```json
{
    "properties.desired": {
        "ProvisioningToken": "$AVA_PROVISIONING_TOKEN",
        "ApplicationDataDirectory": "/var/lib/videoanalyzer",
        "DiagnosticsEventsOutputName": "diagnostics",
        "OperationalEventsOutputName": "operational",
        "LogLevel": "information",
        "LogCategories": "Application,Events",
        "DebugLogsDirectory": "/tmp/logs",
        "AllowUnsecuredEndpoints": true,
        "TelemetryOptOut": false    
     
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

[Métodos directos](direct-methods.md)
