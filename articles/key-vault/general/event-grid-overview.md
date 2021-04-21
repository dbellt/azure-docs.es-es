---
title: Supervisión de Key Vault con Azure Event Grid
description: Uso de Azure Event Grid para suscribirse a eventos de Key Vault
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: mbaldwin
ms.openlocfilehash: 4fb6d57bb84f4a3b4c5c138be9306489191bfce8
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107753369"
---
# <a name="monitoring-key-vault-with-azure-event-grid"></a>Supervisión de Key Vault con Azure Event Grid

La integración de Key Vault con Event Grid permite a los usuarios recibir una notificación cuando cambia el estado de un secreto almacenado en el almacén de claves. Un cambio de estado se define como un secreto que está a punto de expirar (30 días antes de la fecha de expiración), un secreto que ha expirado o un secreto que tiene una nueva versión disponible. Se admiten las notificaciones para los tres tipos de secretos (clave, certificado y secreto).

Las aplicaciones pueden reaccionar a estos eventos mediante las modernas arquitecturas sin servidor y sin necesidad de usar código complejo o servicios de sondeo costosos e ineficaces. Los eventos se insertan mediante [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) en controladores de eventos como [Azure Functions](https://azure.microsoft.com/services/functions/), [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) o incluso en su propio webhook y solo se paga por lo que se usa. Para información sobre los precios, consulte [Precios de Event Grid](https://azure.microsoft.com/pricing/details/event-grid/).

## <a name="key-vault-events-and-schemas"></a>Eventos y esquemas de Key Vault

Event Grid usa las [suscripciones a eventos](../../event-grid/concepts.md#event-subscriptions) para enrutar los mensajes de eventos a los suscriptores. Los eventos de Key Vault contienen toda la información necesaria para responder a cualquier cambio que se produzca en los datos. Puede identificar un evento de Key Vault porque la propiedad eventType comienza por "Microsoft.KeyVault".

Para más información, consulte el [esquema de eventos de Key Vault](../../event-grid/event-schema-key-vault.md).

> [!WARNING]
> Los eventos de notificación solo se desencadenan en las nuevas versiones de secretos, claves y certificados y primero debe suscribirse al evento en el almacén de claves para recibir estas notificaciones.

## <a name="practices-for-consuming-events"></a>Prácticas para consumir eventos

Las aplicaciones que controlan los eventos de Key Vault deben seguir algunos procedimientos recomendados:

* Se pueden configurar varias suscripciones para enrutar eventos al mismo controlador de eventos. Es importante no suponer que los eventos provienen de un origen determinado y comprobar el tema del mensaje para asegurarse de que proviene del almacén de claves que se espera.
* De igual forma, compruebe que eventType es uno de los que está preparado para procesar y no asuma que todos los eventos que reciba van a ser los tipos que espera.
* Ignore los campos que no comprenda.  Esta práctica le ayudará a mantenerse resistente a las nuevas características que puedan agregarse en el futuro.
* Use las coincidencias de prefijo y sufijo "subject" para limitar los eventos a un evento determinado.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a Azure Key Vault](overview.md)
- [Introducción a Azure Event Grid](../../event-grid/overview.md)
- Procedimientos: [Enrutar eventos de Key Vault al Runbook de Automation](event-grid-tutorial.md).
- Procedimientos: [Recibir un correo electrónico al cambiar un secreto del almacén de claves](event-grid-logicapps.md)
- [Esquema de eventos Azure Event Grid para Azure Key Vault](../../event-grid/event-schema-key-vault.md)
- [Información general sobre Azure Automation](../../automation/index.yml)
