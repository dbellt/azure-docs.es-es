---
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 8/12/2020
ms.author: sngun
ms.openlocfilehash: 42d1e0f056457ba54e0102a4d23f42a81ebea08d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "90068749"
---
**¿Cómo se me notificará de la retirada del SDK?**

Microsoft le notificará 12 meses antes de la finalización del soporte técnico del SDK que se retira para facilitar una transición sin problemas a un SDK compatible. Se le notificará a través de diversos canales de comunicación: Azure Portal, actualizaciones de Azure y comunicación directa a los administradores de servicios asignados.

**¿Puedo crear aplicaciones con un SDK de Azure Cosmos DB que se retirará durante el período de 12 meses?** 

Sí, podrá crear, implementar y modificar aplicaciones mediante el SDK de Azure Cosmos DB que se retirará durante el período de aviso de 12 meses. Se recomienda que migre a una versión compatible más reciente del SDK de Azure Cosmos DB durante el período de aviso de 12 meses, según corresponda. 

**Tras la fecha de retirada, ¿qué ocurre con las aplicaciones que usan el SDK de Azure Cosmos DB no compatible?** 

Después de la fecha de retirada, Azure Cosmos DB ya no hará correcciones de errores, ni agregará nuevas características, ni proporcionará soporte técnico para las versiones del SDK retiradas. Si prefiere no realizar la actualización, el servicio Azure Cosmos DB seguirá atendiendo las solicitudes enviadas desde las versiones retiradas del SDK. 

**¿Qué versiones del SDK tendrán las características y actualizaciones más recientes?**

Se agregarán nuevas características y actualizaciones solo a la versión secundaria más reciente de la versión del SDK principal compatible más reciente. Se recomienda que use siempre la versión más reciente para aprovechar las ventajas de las nuevas características, mejoras de rendimiento y correcciones de errores. Si usa una versión anterior del SDK que no se haya retirado, las solicitudes realizadas a Azure Cosmos DB seguirán funcionando, pero no tendrá acceso a las nuevas funcionalidades.  

**¿Qué debo hacer si no puedo actualizar la aplicación antes de una fecha límite?**

Se recomienda que actualice al SDK más reciente tan pronto como sea posible. Después de etiquetar un SDK para su retirada, tendrá 12 meses para actualizar la aplicación. Si no puede realizar la actualización en la fecha de retirada, Azure Cosmos DB seguirá atendiendo las solicitudes enviadas desde las versiones retiradas del SDK, por lo que las aplicaciones en ejecución seguirán funcionando. Aunque Azure Cosmos DB ya no hará correcciones de errores, ni agregará nuevas características ni proporcionará soporte técnico para las versiones del SDK retiradas. 

Si tiene un plan de soporte técnico y requiere soporte técnico, [póngase en contacto con nosotros](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) al abrir una incidencia de soporte técnico.
    


