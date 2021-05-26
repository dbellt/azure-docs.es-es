---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 05/25/2021
ms.openlocfilehash: 88d0292048edcc6846d0e1637b79f115d1321e70
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369138"
---
En el modelo multiinquilino, la estructura de recursos de la aplicación lógica solo puede incluir un único flujo de trabajo. Debido a esta relación uno a uno, tanto la aplicación lógica como el flujo de trabajo suelen considerarse y se hace referencia a ellos como sinónimos. Sin embargo, en el modelo de inquilino único, la estructura de recursos de la aplicación lógica puede incluir varios flujos de trabajo. Esta relación uno a varios significa que, en la misma aplicación lógica, los flujos de trabajo pueden compartir y reutilizar otros recursos. Los flujos de trabajo de la misma aplicación lógica e inquilino también ofrecen un rendimiento mejorado debido a este inquilinato compartido y la proximidad entre sí. Esta estructura de recursos tiene un aspecto y funciona de forma similar a Azure Functions, donde una aplicación de funciones puede hospedar muchas funciones.

Para más información y conocer los procedimientos recomendados sobre la organización de flujos de trabajo, el rendimiento y el escalado en la aplicación lógica, revise las instrucciones similares para [Azure Functions](../articles/azure-functions/functions-best-practices.md) que normalmente se pueden aplicar a las aplicaciones de Azure Logic Apps de inquilino único.