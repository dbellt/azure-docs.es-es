---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: fauhse
ms.service: storage
ms.topic: include
ms.date: 2/20/2020
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: 180b615869579752f9a14aa2dcdd34f1676b577b
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109645121"
---
Para completar este paso, necesita las credenciales de la suscripción a Azure.

El recurso principal para configurar Azure File Sync se denomina *servicio de sincronización de almacenamiento*. Se recomienda implementar solo uno para todos los servidores que sincronizan el mismo conjunto de archivos ahora o en el futuro. Solo debe crear varios servicios de sincronización de almacenamiento si tiene distintos conjuntos de servidores que nunca deben intercambiar datos. Por ejemplo, podría tener servidores que nunca deben sincronizar el mismo recurso compartido de archivos de Azure. De lo contrario, el procedimiento recomendado es contar con un único servicio de sincronización de almacenamiento.

Elija una región de Azure para el servicio de sincronización de almacenamiento que esté cerca de su ubicación. Todos los demás recursos de nube se deben implementar en la misma región. Para simplificar el proceso de administración, cree un nuevo grupo de recursos en la suscripción para hospedar los recursos de sincronización y almacenamiento.

Para más información, consulte la [sección sobre la implementación del servicio de sincronización de almacenamiento](../articles/storage/file-sync/file-sync-deployment-guide.md#deploy-the-storage-sync-service) en el artículo sobre la implementación de Azure File Sync. Siga solo esta sección del artículo. En pasos posteriores, tendrá vínculos a otras secciones del artículo.