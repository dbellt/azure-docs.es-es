---
title: Disponibilidad de regiones y residencia de datos para Azure Communication Services
description: Obtenga información sobre la residencia de datos y las cuestiones relacionadas con la privacidad en Azure Communication Services.
author: chpalm
manager: anvalent
services: azure-communication-services
ms.author: chpalm
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 9ccc192bb5cf4c5e183e21add8e73d9e096d281c
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136020"
---
# <a name="region-availability-and-data-residency"></a>Disponibilidad de regiones y residencia de datos

Azure Communication Services se compromete a ayudar a nuestros clientes a cumplir sus requisitos sobre privacidad y datos personales. Como desarrollador que usa Communication Services con una relación directa con las personas que usan la aplicación, es posible que actúe como un controlador de sus datos. Dado que Azure Communication Services almacena y cifra estos datos en reposo en su nombre, somo algo así como un procesador de estos datos. En esta página se resume cómo conserva los datos el servicio y cómo puede identificar, exportar y eliminar estos datos.

## <a name="data-residency"></a>Residencia de datos

Al crear un recurso de Communication Services, se especifica una **geografía** (no un centro de datos de Azure). Todos los datos almacenados por Communication Services en reposo se conservarán en esa geografía, en un centro de datos seleccionado internamente por Communication Services. Los datos pueden estar en tránsito o procesarse en otras regiones geográficas. Estos puntos de conexión globales son necesarios para proporcionar una experiencia de alto rendimiento y baja latencia a los usuarios finales, independientemente de su ubicación.

## <a name="data-residency-and-events"></a>Residencia de datos y eventos

Cualquier tema del sistema de Event Grid configurado con Azure Communication Services se creará en una ubicación global. Para respaldar unas entregas confiables, un tema global del sistema de Event Grid puede almacenar los datos de eventos en cualquier centro de datos de Microsoft. Cuando configura Event Grid con Azure Communication Services, está entregando sus datos de eventos a Event Grid, que es un recurso de Azure bajo su control. Aunque Azure Communication Services se puede configurar para que use Azure Event Grid, el usuario es responsable de administrar el recurso de Event Grid y los datos almacenados en él.

## <a name="relating-humans-to-azure-communication-services-identities"></a>Relación de las personas con las identidades de Azure Communication Services

La aplicación administra la relación entre los usuarios humanos y las identidades de Communication Services. Si desea eliminar los datos de un usuario humano, debe eliminar los datos que impliquen todas las identidades de Communication Services correlacionadas con el usuario.

Hay dos categorías de datos de Communication Services:
- **Datos de la API.** Estos datos se crean y administran mediante las API de Communication Services, un ejemplo típico serían los mensajes de chat que se administran mediante las API de chat.
- **Registros de Azure Monitor.** Estos datos los crea el servicio y se administran mediante la plataforma de datos de Azure Monitor. Estos datos incluyen datos de telemetría y métricas para ayudarle a comprender el uso de Communication Services. No se administran mediante las API de Communication Services.

## <a name="api-data"></a>Datos de la API

### <a name="identities"></a>Identities

Azure Communication Services mantiene un directorio de identidades, use la API [DeleteIdentity](/rest/api/communication/communicationidentity/delete) para eliminarlos. Al eliminar una identidad, se revocan todos los tokens de acceso asociados y se eliminan sus mensajes de chat. Para más información sobre cómo eliminar una identidad, [consulte esta página](../quickstarts/access-tokens.md).

- DeleteIdentity

### <a name="azure-resource-manager"></a>Azure Resource Manager

El uso de Azure Portal o las API de Azure Resource Manager con Communication Services puede crear datos personales. [Administración de datos personales asociados a Azure Resource Manager](../../azure-resource-manager/management/resource-manager-personal-data.md)

### <a name="telephone-number-management"></a>Administración de números de teléfono

Azure Communication Services mantiene un directorio de números de teléfono asociados a un recurso de Communication Services. Utilice [Phone Number Administration API](/rest/api/communication/phonenumbers) para recuperar números de teléfono y eliminarlos:

- `Get All Phone Numbers`
- `Release Phone Number`

### <a name="chat"></a>Chat

Los mensajes y los hilos de chat se conservan hasta que se eliminan explícitamente. Un hilo completamente inactivo sin participantes se eliminará automáticamente después de 30 días. Use las [API de chat](/rest/api/communication/chat/chatthread) para obtener, enumerar, actualizar y eliminar mensajes.

- `Get Thread`
- `Get Message`
- `List Messages`
- `Update Message`
- `Delete Thread`
- `Delete Message`

### <a name="sms"></a>SMS

El servicio procesa de forma efímera los mensajes SMS enviados y recibidos y no se conservan.

### <a name="pstn-voice-calling"></a>Llamada de voz RTC

El servicio procesa de forma efímera la comunicación de audio y vídeo, y no se conserva ningún dato en el recurso aparte de los registros de Azure Monitor.

### <a name="internet-voice-and-video-calling"></a>Llamadas de voz y vídeo de Internet

El servicio procesa de forma efímera la comunicación de audio y vídeo, y no se conserva ningún dato en el recurso aparte de los registros de Azure Monitor.

## <a name="azure-monitor-and-log-analytics"></a>Azure Monitor y Log Analytics

Azure Communication Services alimentará los datos de registro de Azure Monitor para conocer el estado operativo y el uso del servicio. Algunos de estos registros incluyen identidades de Communication Services y números de teléfono como datos de campos. Para eliminar cualquier dato potencialmente personal, [utilice estos procedimientos para Azure Monitor](../../azure-monitor/logs/personal-data-mgmt.md). También puede que desee configurar [el período de retención predeterminado de Azure Monitor](../../azure-monitor/logs/manage-cost-storage.md).

## <a name="additional-resources"></a>Recursos adicionales

- [Solicitudes de titulares de los datos de Azure para el RGPD y la CCPA](/microsoft-365/compliance/gdpr-dsr-azure)
- [Microsoft Trust Center](https://www.microsoft.com/trust-center/privacy/data-location)
- [Mapa interactivo de Azure: ¿dónde están los datos de los clientes?](https://azuredatacentermap.azurewebsites.net/)
