---
author: laujan
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 03/15/2021
ms.author: lajanuar
ms.openlocfilehash: 734b548953c4ede5e7d09feb0042509ae5528e82
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108026340"
---
Vaya a Azure Portal y <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer" title="cree un nuevo recurso de Form Recognizer" target="_blank">create a new Form Recognizer resource </a>. En el panel **Crear**, proporcione la siguiente información:

| Detalles del proyecto   | Descripción   |
|--|--|
| **Suscripción** | Seleccione la suscripción de Azure a la que se le ha concedido acceso. |
| **Grupos de recursos** | [Grupo de recursos de Azure](/azure/cloud-adoption-framework/govern/resource-consistency/resource-access-management#what-is-an-azure-resource-group) que contendrá su recurso. Puede crear un nuevo grupo o agregarlo a uno ya existente. |
| **Región** | Ubicación de la instancia de Cognitive Services. Las diferentes ubicaciones pueden crear latencias, pero no tienen ningún impacto en la disponibilidad del tiempo de ejecución del recurso. |
| **Nombre** | Un nombre descriptivo para su recurso. Se recomienda usar un nombre descriptivo, como *MyNameFormRecognizer*. |
| **Plan de tarifa** | El costo del recurso depende el plan de tarifa elegido y del uso. Para obtener más información, consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/cognitive-services/).

> [!NOTE]
> Al crear un recurso de Cognitive Services en Azure Portal, tiene la opción de crear una clave de suscripción para varios servicios (que se usa en varios servicios de Cognitive Services) o una clave de suscripción para un solo servicio (que se usa solo con un servicio específico de Cognitive Services). Actualmente, Form Recognizer no está incluido en la suscripción de varios servicios.

Cuando el recurso de Form Recognizer termine la implementación, búsquelo y selecciónelo en la lista **Todos los recursos** del portal. Encontrará su clave y punto de conexión en la página de clave y punto de conexión del recurso, en Administración de recursos. Guarde ambos en una ubicación temporal antes de continuar.
