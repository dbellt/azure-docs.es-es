---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 09/04/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 4604616cd4f2d6c75c272586df1331fc405061cb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "70737498"
---
## <a name="error-conditionheadersnotsupported-from-a-web-application-using-azure-files-from-browser"></a>Error ConditionHeadersNotSupported de una aplicación web al usar Azure Files desde el explorador

El error ConditionHeadersNotSupported se produce cuando se accede a contenido hospedado en Azure Files utilizando una aplicación que emplea encabezados condicionales, como un explorador web. Este error indica que no se admiten encabezados condicionales.

![Error en los encabezados condicionales de Azure Files](media/storage-files-condition-headers/conditionalerror.png)

### <a name="cause"></a>Causa

Los encabezados condicionales todavía no se admiten. Las aplicaciones que los implementen tendrán que solicitar el archivo completo cada vez que se accede al archivo.

### <a name="workaround"></a>Solución alternativa

Cuando se carga un archivo nuevo, la propiedad cache-control predeterminada es "no-cache". Para forzar que la aplicación solicite el archivo cada vez, la propiedad cache-control del archivo se debe actualizar de "no-cache" a "no-cache, no-store, must-revalidate". Esto se puede lograr mediante el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

![Modificación de la memoria caché de contenido del explorador de almacenamiento para los encabezados condicionales de Azure Files](media/storage-files-condition-headers/storage-explorer-cache.png)