---
author: baanders
description: 'archivo de inclusión para Azure Digital Twins: formas de administrar la instancia'
ms.service: digital-twins
ms.topic: include
ms.date: 11/11/2020
ms.author: baanders
ms.openlocfilehash: 1f66101fc1231be2e5ce36dc348b1ca850113867
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107303771"
---
En este artículo se describe cómo completar diferentes operaciones de administración con el [**SDK** para .NET de Azure Digital Twins (C#)](/dotnet/api/overview/azure/digitaltwins/management). También puede crear estas mismas llamadas de administración con los otros SDK de lenguaje descritos en [*Paso a paso: las API y los SDK de Azure Digital Twins*](../articles/digital-twins/how-to-use-apis-sdks.md).

> [!TIP] 
> Recuerde que todos los métodos del SDK cuentan con versiones sincrónicas y asincrónicas. En las llamadas de paginación, los métodos asincrónicos devuelven `AsyncPageable<T>`, mientras que las versiones sincrónicas devuelven `Pageable<T>`.

Otra opción de administración es llamar a las [**API de REST**](/rest/api/azure-digitaltwins/) de Azure Digital Twins para esta cuestión directamente a través de un cliente de REST, como Postman. Para obtener instrucciones sobre cómo hacerlo, consulte [*Procedimiento: Realización de solicitudes con Postman*](../articles/digital-twins/how-to-use-postman.md).

Por último, puede completar las mismas operaciones de administración mediante la **CLI** de Azure Digital Twins. Para obtener más información sobre el uso de la CLI, consulte [*Paso a paso: Uso de la CLI de Azure Digital Twins*](../articles/digital-twins/how-to-use-cli.md).