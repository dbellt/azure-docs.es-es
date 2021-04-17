---
title: archivo de inclusión
description: archivo de inclusión
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: include
ms.date: 11/19/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: c37e2c28eb6aee9edf4bdd97066ce5f15e7447c1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96005644"
---
## <a name="message-headers"></a>Encabezados de mensaje
Estas son las propiedades que recibe en los encabezados de mensaje:

| Nombre de propiedad | Descripción |
| ------------- | ----------- | 
| aeg-subscription-name | Nombre de la suscripción de eventos. |
| aeg-delivery-count | Número de intentos realizados para el evento. |
| aeg-event-type | <p>Tipo de evento.</p><p>Puede ser uno de los siguientes valores:</p><ul><li>SubscriptionValidation</li><li>Notificación</li><li>SubscriptionDeletion</li></ul> | 
| aeg-metadata-version | <p>Versión de metadatos del evento.<p> Esta propiedad representa la versión de los metadatos en el **esquema de eventos de Event Grid** y la **versión de la especificación** en el **esquema de eventos en la nube**. </p>|
| aeg-data-version | <p>Versión de datos del evento.</p><p>Esta propiedad representa la versión de los datos en el **esquema de eventos de Event Grid** y la no se aplica en el **esquema de eventos en la nube**.</p> |
| aeg-output-event-id | Id. del evento de Event Grid. |


