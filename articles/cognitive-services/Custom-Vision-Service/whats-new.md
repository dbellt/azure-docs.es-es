---
title: Novedades de Custom Vision
titleSuffix: Azure Cognitive Services
description: Este artículo describe las novedades de Custom Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: overview
ms.date: 05/24/2021
ms.author: pafarley
ms.openlocfilehash: 78986eca3d1db65afb8ac4e0ca8ff76a0e3201eb
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451969"
---
# <a name="whats-new-in-custom-vision"></a>Novedades de Custom Vision

Conozca las novedades del servicio. Estos elementos pueden ser notas de la versión, vídeos, entradas de blogs y otros tipos de información. Marque esta página para mantenerse actualizado con el servicio.


## <a name="october-2020"></a>Octubre de 2020 

### <a name="custom-base-model"></a>Modelo base personalizado

- Algunas aplicaciones tienen una gran cantidad de datos de entrenamiento conjunto, pero necesitan ajustar sus modelos por separado. El resultado es un rendimiento mejorado de las imágenes de otros orígenes con pequeñas diferencias. En este caso, puede entrenar el primer modelo como de costumbre con un gran volumen de datos de entrenamiento. Después, llame a **TrainProject** en la API de versión preliminar pública 3.4 con _CustomBaseModelInfo_ en el cuerpo de la solicitud para usar el modelo entrenado de la primera fase como modelo base para los proyectos de nivel inferior. Si el proyecto de origen y el proyecto de destino de nivel inferior tienen características similares, cabe esperar un mejor rendimiento. 

### <a name="new-domain-information"></a>Nueva información del dominio

- La información de dominio devuelta desde **GetDomains** en la API de versión preliminar pública de Custom Vision 3.4 ahora incluye plataformas exportables admitidas, una breve descripción de la arquitectura del modelo y el tamaño del modelo para dominios compactos.

### <a name="training-divergence-feedback"></a>Comentarios sobre la divergencia de entrenamiento

- La API de versión preliminar pública de Custom Vision 3.4 ahora devuelve **TrainingErrorDetails** desde la llamada a **GetIteration**. En las iteraciones con errores, este valor revela si el error se debió a la divergencia de entrenamiento, lo que puede solucionarse con más datos de entrenamiento y de mejor calidad.

## <a name="july-2020"></a>Julio de 2020

### <a name="azure-role-based-access-control"></a>Control de acceso basado en roles de Azure

* Custom Vision admite el control de acceso basado en roles de Azure (Azure RBAC), un sistema de autorización que permite administrar el acceso individual a los recursos de Azure. Para aprender a administrar el acceso a los proyectos de Custom Vision, consulte [Control de acceso basado en roles de Azure](./role-based-access-control.md).

### <a name="subset-training"></a>Entrenamiento de subconjuntos

* Al entrenar un proyecto de detección de objetos, puede entrenar opcionalmente en solo un subconjunto de las etiquetas aplicadas. Es posible que desee hacer esto si aún no ha aplicado suficientes etiquetas de un tipo determinado, pero tiene bastantes de las otras. Siga el [inicio rápido de la biblioteca cliente](./quickstarts/object-detection.md) para C# o Python para más información.

### <a name="azure-storage-notifications"></a>Notificaciones de almacenamiento de Azure

* Puede integrar el proyecto de Custom Vision con una cola de almacenamiento de blobs de Azure para obtener notificaciones de envío de actividad de entrenamiento/exportación del proyecto y copias de seguridad de los modelos publicados. Esta característica resulta útil para evitar el sondeo continuo del servicio en busca de resultados cuando se ejecutan operaciones largas. En su lugar, puede integrar las notificaciones de la cola de almacenamiento en el flujo de trabajo. Para más información, consulte la guía de [integración del almacenamiento](./storage-integration.md).

### <a name="copy-and-move-projects"></a>Copia y traslado de proyectos

* Ahora puede copiar proyectos de una cuenta de Custom Vision en otras. Puede que desee trasladar un proyecto desde un entorno de desarrollo a un entorno de producción, o realizar una copia de seguridad de un proyecto en una cuenta de otra región de Azure para aumentar la seguridad de los datos. Para más información, consulte la guía [Copia y traslado de proyectos](./copy-move-projects.md).

## <a name="september-2019"></a>Septiembre de 2019

### <a name="suggested-tags"></a>Etiquetas sugeridas

* La herramienta Smart Labeler del [sitio web de Custom Vision](https://www.customvision.ai/) genera etiquetas sugeridas para las imágenes de entrenamiento. Esto le permite etiquetar un gran número de imágenes más rápidamente al entrenar un modelo de Custom Vision. Para obtener instrucciones sobre cómo usar esta característica, consulte [Suggested tags](./suggested-tags.md) (Etiquetas sugeridas).

## <a name="cognitive-service-updates"></a>Actualizaciones de Cognitive Service

[Anuncios de actualización de Azure para Cognitive Services](https://azure.microsoft.com/updates/?product=cognitive-services)