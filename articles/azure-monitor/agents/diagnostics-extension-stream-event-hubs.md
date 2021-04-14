---
title: Envío de datos de Windows Azure Diagnostics Extension a Azure Event Hubs
description: Configure Diagnostics Extension en Azure Monitor para que envíe datos a Azure Event Hubs, de modo que pueda reenviarlos a ubicaciones fuera de Azure.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/18/2020
ms.openlocfilehash: 6aa960eb7aed7a579c3fc95748b6e32601201a61
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305130"
---
# <a name="send-data-from-windows-azure-diagnostics-extension-to-azure-event-hubs"></a>Envío de datos de Windows Azure Diagnostics Extension a Azure Event Hubs
Azure Diagnostics Extension es un agente de Azure Monitor que recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de las máquinas virtuales de Azure y de otros recursos de proceso. En este artículo, se explica cómo puede enviar datos desde Windows Azure Diagnostic Extension (WAD) a [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) para reenviarlos después fuera de Azure.

## <a name="supported-data"></a>Datos admitidos

Los datos recopilados del sistema operativo invitado que pueden enviarse a Event Hubs son los siguientes. Otros orígenes de datos recopilados por WAD, como los volcados de memoria y los registros de IIS, no se pueden enviar a Event Hubs.

* Eventos de Seguimiento de eventos para Windows (ETW)
* Contadores de rendimiento
* Registros de eventos de Windows, incluidos los registros de aplicaciones en el registro de eventos de Windows
* Registros de infraestructura de diagnóstico de Azure

## <a name="prerequisites"></a>Requisitos previos

* Windows Diagnostics Extension 1.6 o versiones posteriores. Consulte el historial de versiones en [Historial y versiones de esquemas de configuración de Azure Diagnostics Extension](diagnostics-extension-versions.md) y los recursos admitidos en la [introducción a Azure Diagnostics Extension](diagnostics-extension-overview.md).
* Siempre debe aprovisionarse un espacio de nombres de Event Hubs. Consulte la [introducción a Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md) para más información.


## <a name="configuration-schema"></a>Esquema de configuración
Consulte las diferentes opciones para habilitar y configurar Diagnostics Extension en [Instalación y configuración de Windows Azure Diagnostics Extension (WAD)](diagnostics-extension-windows-install.md) y una referencia del esquema de configuración en [Esquema de configuración de Azure Diagnostics](diagnostics-extension-schema-windows.md). De aquí hasta el final del artículo, se describe cómo se utiliza esta configuración para enviar datos a un centro de eventos. 

Azure Diagnostics siempre envía registros y métricas a una cuenta de Azure Storage. Puede configurar uno o varios *receptores de datos* para enviar datos a otros destinos. Cada receptor de datos se define en el [elemento SinksConfig](diagnostics-extension-schema-windows.md#sinksconfig-element) de la configuración pública con información confidencial en la configuración privada. Esta configuración de los centros de eventos utiliza los valores de la tabla siguiente.

| Propiedad | Descripción |
|:---|:---|
| Nombre | Nombre descriptivo del receptor. Se utiliza en la configuración para especificar qué orígenes de datos van a enviar información al receptor. |
| Url  | Dirección URL del centro de eventos con el formato \<event-hubs-namespace\>.servicebus.windows.net/\<event-hub-name\>.          |
| SharedAccessKeyName | Nombre de la directiva de acceso compartido del centro de eventos que tiene, como mínimo, el permiso **Enviar**. |
| SharedAccessKey     | Clave principal o secundaria de la directiva de acceso compartido del centro de eventos. |

A continuación, se incluyen ejemplos de la configuración pública y privada. Se trata de una configuración mínima con un solo contador de rendimiento y un solo registro de eventos que ilustra cómo se configura y utiliza el receptor de datos del centro de eventos. Puede ver un ejemplo más complejo en [Esquema de configuración de Azure Diagnostics](diagnostics-extension-schema-windows.md).

### <a name="public-configuration"></a>Configuración pública

```JSON
{
    "WadCfg": {
        "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": 5120,
            "PerformanceCounters": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "myEventHub",
                "PerformanceCounterConfiguration": [
                    {
                        "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                        "sampleRate": "PT3M"
                    }
                ]
            },
            "WindowsEventLog": {
                "scheduledTransferPeriod": "PT1M",
                "sinks": "myEventHub",
                    "DataSource": [
                    {
                        "name": "Application!*[System[(Level=1 or Level=2 or Level=3)]]"
                    }
                ]
            }
        },
        "SinksConfig": {
            "Sink": [
                {
                    "name": "myEventHub",
                    "EventHub": {
                        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
                        "SharedAccessKeyName": "SendRule"
                    }
                }
            ]
        }
    },
    "StorageAccount": "mystorageaccount",
}
```


### <a name="private-configuration"></a>Configuración privada

```JSON
{
    "storageAccountName": "mystorageaccount",
    "storageAccountKey": "{base64 encoded key}",
    "storageAccountEndPoint": "https://core.windows.net",
    "EventHub": {
        "Url": "https://diags-mycompany-ns.servicebus.windows.net/diageventhub",
        "SharedAccessKeyName": "SendRule",
        "SharedAccessKey": "{base64 encoded key}"
    }
}
```



## <a name="configuration-options"></a>Opciones de configuración
Para enviar datos a un receptor de datos, debe especificar el atributo **sinks** en el nodo del origen de datos. El lugar en el que incluya el atributo **sinks** determinará el ámbito de la asignación. En el ejemplo siguiente, el atributo **sinks** está definido en el nodo **PerformanceCounters**, lo que hará que todos los contadores de rendimiento secundarios se envíen al centro de eventos.

```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "sinks": "MyEventHub",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        }
    ]
}
```


En el ejemplo siguiente, el atributo **sinks** se aplica directamente a tres contadores, por lo que solo se enviarán estos contadores de rendimiento al centro de eventos. 

```JSON
"PerformanceCounters": {
    "scheduledTransferPeriod": "PT1M",
    "PerformanceCounterConfiguration": [
        {
            "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        },
        {
            "counterSpecifier": "\\Memory\\Available MBytes",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\Web Service(_Total)\\ISAPI Extension Requests/sec",
            "sampleRate": "PT3M"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Rejected",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        },
        {
            "counterSpecifier": "\\ASP.NET\\Requests Queued",
            "sampleRate": "PT3M",
            "sinks": "MyEventHub"
        }
    ]
}
```

## <a name="validating-configuration"></a>Validación de la configuración
Puede utilizar diferentes métodos para comprobar que los datos se están enviando al centro de eventos. El método más sencillo es utilizar una captura de Event Hubs, tal y como se describe en [Captura de eventos a través de Azure Event Hubs en Azure Blob Storage o Azure Data Lake Storage](../../event-hubs/event-hubs-capture-overview.md). 


## <a name="troubleshoot-event-hubs-sinks"></a>Solución de problemas con los receptores de Event Hubs

- Consulte la tabla **WADDiagnosticInfrastructureLogsTable** de Azure Storage, que contiene registros y errores de Azure Diagnostics. Una opción es usar una herramienta como el [Explorador de Azure Storage](https://www.storageexplorer.com) para conectarse a esta cuenta de almacenamiento, ver esta tabla y agregar una consulta para TimeStamp en las últimas 24 horas. Puede usar la herramienta para exportar un archivo .csv y abrirlo en una aplicación como Microsoft Excel. Excel facilita la búsqueda de cadenas de tarjeta de llamadas, como **EventHubs**, para ver qué error se notifica.  

- Compruebe que el centro de eventos esté correctamente aprovisionado. Toda la información de conexión de la sección **PrivateConfig** de la configuración debe coincidir con los valores del recurso que se muestran en el portal. No olvide que debe tener una directiva SAS definida (en este ejemplo es *SendRule*) en el portal y que debe haberse concedido el permiso *Enviar*.  

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Event Hubs](../../event-hubs/event-hubs-about.md)
* [Creación de un centro de eventos](../../event-hubs/event-hubs-create.md)
* [Preguntas más frecuentes sobre Event Hubs](../../event-hubs/event-hubs-faq.yml)

<!-- Images. -->
[0]: ../../event-hubs/media/event-hubs-streaming-azure-diags-data/dashboard.png



