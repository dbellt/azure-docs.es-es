---
author: laujan
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 06/27/2019
ms.author: lajanuar
ms.openlocfilehash: fbf4fc405c92700f993a488d94334338ab2757b4
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111887514"
---
Form Recognizer trabaja con documentos de entrada que cumplen estos requisitos:

* El formato debe ser JPG, PNG o PDF (texto o digitalizado) o TIFF. Los PDF insertados de texto son mejores porque no hay posibilidad de error en la extracción de caracteres y en la ubicación.
* El tamaño del archivo debe ser inferior a 50 MB.
* Las imágenes deben tener unas dimensiones entre 50 x 50 píxeles y 10 000 x 10 000 píxeles.
* Los archivos PDF deben tener unas dimensiones de 17 x 17 pulgadas como máximo, lo que se corresponde con los tamaños de papel Legal o A3 y más pequeños.
* En el caso de PDF y TIFF, solo se procesan las primeras 200 páginas (con una suscripción de nivel gratuitos, solo se procesan las dos primeras páginas).
* El tamaño total del conjunto de datos de aprendizaje puede ser de 500 páginas o menos.
* Si los archivos PDF están bloqueados con contraseña, debe quitar el bloqueo antes de enviarlos.
* Si se digitalizan desde documentos en papel, los formularios deben ser digitalizaciones de alta calidad.
* En el caso del aprendizaje sin supervisión (sin datos etiquetados), los datos deben contener claves y valores.
* En el caso del aprendizaje sin supervisión (sin datos etiquetados), las claves deben aparecer por encima o a la izquierda de los valores; no pueden aparecer por abajo ni a la derecha.
