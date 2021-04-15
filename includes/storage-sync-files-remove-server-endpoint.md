---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 05/31/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: f730b2abad244132b06c0fa21f07a171559d22f7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96309955"
---
No: eliminar un punto de conexión de servidor no es como reiniciar un servidor. Eliminar y volver a crear el punto de conexión del servidor casi nunca es una solución adecuada para solucionar problemas de sincronización, niveles de nube u otros aspectos de Azure File Sync. Quitar un punto de conexión de servidor es una operación destructiva. Puede producirse una pérdida de datos en caso de que existan archivos en capas fuera del espacio de nombres del punto de conexión de servidor. Consulte [¿Por qué los archivos en capas se encuentran fuera del espacio de nombres del punto de conexión de servidor? ](../articles/storage/files/storage-files-faq.md#afs-tiered-files-out-of-endpoint) para más información. También puede producir archivos inaccesibles para los archivos en capas que existen en el espacio de nombres del punto de conexión de servidor. Estos problemas no se resolverán al volver a crear el punto de conexión en el servidor. Puede haber archivos en capas en el espacio de nombres del punto de conexión de un servidor aunque nunca se hayan habilitado los niveles en la nube. Por eso se recomienda no eliminar el punto de conexión del servidor a menos que desee dejar de usar Azure File Sync con esta carpeta en concreto o un ingeniero de Microsoft le haya solicitado expresamente que lo haga así. Para más información sobre la eliminación de puntos de conexión del servidor, consulte [Eliminación de un punto de conexión de servidor](../articles/storage/files/storage-sync-files-server-endpoint.md#remove-a-server-endpoint).    
