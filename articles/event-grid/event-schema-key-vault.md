---
title: Azure Key Vault como origen de Event Grid
description: Describe las propiedades y el esquema que se proporcionan para los eventos de Azure Key Vault con Azure Event Grid
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 48f4df660a17c36030b6b6d2396bd96cfec48edc
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110794092"
---
# <a name="azure-key-vault-as-event-grid-source"></a>Azure Key Vault como origen de Event Grid

En este artículo se proporcionan las propiedades y el esquema de los eventos en [Azure Key Vault](../key-vault/index.yml). Para una introducción a los esquemas de eventos, consulte [Esquema de eventos de Azure Event Grid](event-schema.md).


## <a name="available-event-types"></a>Tipos de eventos disponibles

Una cuenta Azure Key Vault genera los siguientes tipos de eventos:

| Nombre completo del evento | Nombre para mostrar del evento | Descripción |
| ---------- | ----------- |---|
| Microsoft.KeyVault.CertificateNewVersionCreated | Nueva versión de certificado creada | Se desencadena cuando se crea un nuevo certificado o una nueva versión de certificado. |
| Microsoft.KeyVault.CertificateNearExpiry | El certificado está a punto de expirar | Se desencadena cuando la versión actual del certificado está a punto de expirar. (El evento se desencadena 30 días antes de la fecha de expiración). |
| Microsoft.KeyVault.CertificateExpired | Certificado expirado | Se desencadena cuando el certificado ha expirado. |
| Microsoft.KeyVault.KeyNewVersionCreated | Nueva versión de clave creada | Se desencadena cuando se crea una nueva clave o nueva versión de clave. |
| Microsoft.KeyVault.KeyNearExpiry | La clave está a punto de expirar | Se desencadena cuando la versión actual de una clave está a punto de expirar. (El evento se desencadena 30 días antes de la fecha de expiración). |
| Microsoft.KeyVault.KeyExpired | Clave expirada | Se desencadena cuando una clave ha expirado. |
| Microsoft.KeyVault.SecretNewVersionCreated | Secreto nueva versión creada | Se desencadena cuando se crea una nueva versión de secreto o secreto nuevo. |
| Microsoft.KeyVault.SecretNearExpiry | El secreto está a punto de expirar | Se desencadena cuando la versión actual de un secreto está a punto de expirar. (El evento se desencadena 30 días antes de la fecha de expiración). |
| Microsoft.KeyVault.SecretExpired | Secreto expirado | Se desencadena cuando un secreto ha expirado. |
| Microsoft.KeyVault.VaultAccessPolicyChanged | Directiva de acceso de Vault modificada | Se desencadena cuando cambia una directiva de acceso en Key Vault. Incluye un escenario en el que el modelo de permisos de Key Vault se modifica para o desde el control de acceso basado en rol de Azure.   |

## <a name="event-examples"></a>Ejemplos de eventos

# <a name="event-grid-event-schema"></a>[Esquema de eventos de Event Grid](#tab/event-grid-event-schema)

En el ejemplo siguiente se muestra el esquema para **Microsoft.KeyVault.SecretNewVersionCreated**:

```JSON
[
   {
      "id":"00eccf70-95a7-4e7c-8299-2eb17ee9ad64",
      "topic":"/subscriptions/{subscription-id}/resourceGroups/sample-rg/providers/Microsoft.KeyVault/vaults/sample-kv",
      "subject":"newsecret",
      "eventType":"Microsoft.KeyVault.SecretNewVersionCreated",
      "eventTime":"2019-07-25T01:08:33.1036736Z",
      "data":{
         "Id":"https://sample-kv.vault.azure.net/secrets/newsecret/ee059b2bb5bc48398a53b168c6cdcb10",
         "VaultName":"sample-kv",
         "ObjectType":"Secret",
         "ObjectName ":"newsecret",
         "Version":" ee059b2bb5bc48398a53b168c6cdcb10",
         "NBF":"1559081980",
         "EXP":"1559082102"
      },
      "dataVersion":"1",
      "metadataVersion":"1"
   }
]
```

# <a name="cloud-event-schema"></a>[Esquema de eventos en la nube](#tab/cloud-event-schema)

En el ejemplo siguiente se muestra el esquema para **Microsoft.KeyVault.SecretNewVersionCreated**:

```JSON
[
   {
      "id":"00eccf70-95a7-4e7c-8299-2eb17ee9ad64",
      "source":"/subscriptions/{subscription-id}/resourceGroups/sample-rg/providers/Microsoft.KeyVault/vaults/sample-kv",
      "subject":"newsecret",
      "type":"Microsoft.KeyVault.SecretNewVersionCreated",
      "time":"2019-07-25T01:08:33.1036736Z",
      "data":{
         "Id":"https://sample-kv.vault.azure.net/secrets/newsecret/ee059b2bb5bc48398a53b168c6cdcb10",
         "VaultName":"sample-kv",
         "ObjectType":"Secret",
         "ObjectName ":"newsecret",
         "Version":" ee059b2bb5bc48398a53b168c6cdcb10",
         "NBF":"1559081980",
         "EXP":"1559082102"
      },
      "specversion":"1.0"
   }
]
```

---

### <a name="event-properties"></a>Propiedades de evento

# <a name="event-grid-event-schema"></a>[Esquema de eventos de Event Grid](#tab/event-grid-event-schema)
Un evento tiene los siguientes datos de nivel superior:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| `topic` | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| `subject` | string | Ruta al asunto del evento definida por el anunciante. |
| `eventType` | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| `eventTime` | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| `id` | string | Identificador único para el evento |
| `data` | object | Datos del evento de App Configuration. |
| `dataVersion` | string | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| `metadataVersion` | string | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |


# <a name="cloud-event-schema"></a>[Esquema de eventos en la nube](#tab/cloud-event-schema)

Un evento tiene los siguientes datos de nivel superior:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| `source` | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| `subject` | string | Ruta al asunto del evento definida por el anunciante. |
| `type` | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| `time` | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| `id` | string | Identificador único para el evento |
| `data` | object | Datos del evento de App Configuration. |
| `specversion` | string | Versión de especificación del esquema CloudEvents. |

---
 

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| ---------- | ----------- |---|
| `id` | string | El identificador del objeto que desencadenó este evento |
| `VaultName` | string | Nombre del almacén de claves del objeto que desencadenó este evento |
| `ObjectType` | string | El tipo del objeto que desencadenó este evento |
| `ObjectName` | string | El nombre del objeto que desencadenó este evento |
| `Version` | string | La versión del objeto que desencadenó este evento |
| `NBF` | number | La fecha no antes de segundos desde 1970-01-01T00:00:00Z del objeto que desencadenó este evento |
| `EXP` | number | Fecha de expiración en segundos desde 1970-01-01T00:00:00Z del objeto que desencadenó este evento |

## <a name="tutorials-and-how-tos"></a>Tutoriales y procedimientos
|Título  |Descripción  |
|---------|---------|
| [Supervisión de eventos de Key Vault con Azure Event Grid](../key-vault/general/event-grid-overview.md) | Información general sobre la integración de Key Vault con Event Grid. |
| [Tutorial: Creación y supervisión de eventos de Key Vault con Event Grid](../key-vault/general/event-grid-logicapps.md) | Obtenga información sobre cómo configurar notificaciones de Event Grid para Key Vault. |


## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a Azure Event Grid, consulte el artículo de [introducción a Event Grid](overview.md).
* Para más información sobre cómo crear una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
* Para más información sobre Key Vault, consulte [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)

