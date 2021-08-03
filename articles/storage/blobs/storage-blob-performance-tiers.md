---
title: 'Niveles de rendimiento del almacenamiento de blobs en bloques: Azure Storage'
description: Describe las diferencias entre los niveles de rendimiento Premium y Estándar para el almacenamiento de blobs en bloques de Azure.
author: normesta
ms.author: normesta
ms.date: 05/17/2021
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.reviewer: clausjor
ms.openlocfilehash: faf26ce78e0d6137c93e150d31bd49b9e72b6b8b
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110655108"
---
# <a name="performance-tiers-for-block-blob-storage"></a>Niveles de rendimiento para el almacenamiento de blobs en bloques

A medida que las empresas implementan aplicaciones nativas en la nube sensibles al rendimiento, es importante disponer de opciones para el almacenamiento rentable de datos en diferentes niveles de rendimiento.

El almacenamiento de blobs en bloques de Azure ofrece dos niveles de rendimiento diferentes:

- **Premium**: optimizado para altas tasas de transacciones y latencia de almacenamiento coherente de un solo dígito
- **Estándar**: optimizado para una alta capacidad y un alto rendimiento

Las siguientes consideraciones se aplican a los distintos niveles de rendimiento:

| Área | Rendimiento Estándar | Rendimiento Premium |
|--|--|--|
| Disponibilidad en regiones | Todas las regiones | En [regiones seleccionadas](https://azure.microsoft.com/global-infrastructure/services/?products=storage) |
| [Tipos de cuenta de almacenamiento](../common/storage-account-overview.md#types-of-storage-accounts) admitidos | Uso general v2, uso general v1, blob heredado | Blobs en bloques Premium |
| Admite [blobs en bloques de alto rendimiento](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/) | Sí, con tamaños de PutBlock o PutBlob superiores a 4 MiB | Sí, con tamaños de PutBlock o PutBlob superiores a 256 KiB |
| Redundancia | Consulte [Tipos de cuentas de almacenamiento](../common/storage-account-overview.md#types-of-storage-accounts). | Actualmente solo se admite almacenamiento con redundancia local (LRS) y con redundancia de zona (ZRS).<div role="complementary" aria-labelledby="zone-redundant-storage"><sup>1</sup></div> |

<div id="zone-redundant-storage"><sup>1</sup>El almacenamiento con redundancia de zona (ZRS) está disponible en algunas regiones para cuentas de almacenamiento de blobs en bloques prémium.</div>

En relación con el costo, el rendimiento Premium ofrece precios optimizados para aplicaciones con altas tasas de transacciones para ayudar a [reducir el costo de almacenamiento total](https://azure.microsoft.com/blog/reducing-overall-storage-costs-with-azure-premium-blob-storage/) de estas cargas de trabajo.

## <a name="premium-performance"></a>Rendimiento Premium

El almacenamiento de blobs en bloques de rendimiento Premium hace que los datos estén disponibles a través de un hardware de alto rendimiento. Los datos se almacenan en unidades de estado sólido (SSD) que están optimizadas para baja latencia. Las SSD ofrecen mayor rendimiento en comparación con las unidades de disco duro tradicionales.

El almacenamiento de rendimiento Premium es perfecto para cargas de trabajo que requieren tiempos de respuesta rápidos y coherentes. Es mejor para las cargas de trabajo que realizan muchas transacciones pequeñas. Entre las cargas de trabajo de ejemplo se incluyen:

- **Cargas de trabajo interactivas**. Estas cargas de trabajo requieren actualizaciones instantáneas y comentarios de los usuarios como es el caso de las aplicaciones de comercio electrónico y de mapas. Por ejemplo, en una aplicación de comercio electrónico, es probable que los elementos que se ven con menos frecuencia no estén almacenados en caché. Sin embargo, deben mostrarse al cliente a petición de forma instantánea.

- **Análisis**. En un escenario de IoT, se podría insertar un gran número de operaciones más pequeñas en la nube cada segundo. Se podrían tomar grandes cantidades de datos, agregarlos con fines de análisis y, finalmente, eliminarlos de forma casi inmediata. Las funcionalidades de alto nivel de ingesta del almacenamiento de blob en bloques Premium lo hacen especialmente eficaz para este tipo de carga de trabajo.

- **Inteligencia artificial y Aprendizaje automático (IA y ML)** . Este tema trata sobre el uso y procesamiento de diferentes tipos de datos como objetos visuales, voz y texto. Este tipo de informática de alto rendimiento de la carga de trabajo se ocupa de grandes cantidades de datos que requieren una respuesta rápida y tiempos de ingesta eficientes para el análisis de datos.

- **Transformación de datos**. Los procesos que requieren edición, modificación y conversión constante de datos requieren actualizaciones instantáneas. Para una representación de los datos precisa, los consumidores de estos datos deben ver los cambios reflejados inmediatamente.

## <a name="standard-performance"></a>Rendimiento Estándar

El rendimiento Estándar admite distintos [niveles de acceso](storage-blob-storage-tiers.md) para almacenar los datos de la manera más rentable. Está optimizado para una alta capacidad y un alto rendimiento en grandes conjuntos de datos.

- **Conjuntos de datos de copia de seguridad y recuperación ante desastres**. El almacenamiento de rendimiento estándar ofrece niveles de servicio rentables, por lo que es un caso de uso perfecto para los conjuntos de datos de recuperación ante desastres a corto y largo plazo, las copias de seguridad secundarias y el archivado de datos de cumplimiento.

- **Contenido multimedia**. A menudo se accede frecuentemente a imágenes y vídeos cuando se crean y almacenan por primera vez, pero este tipo de contenido se usa con menos frecuencia a medida que se vuelve más antiguo. El almacenamiento de rendimiento estándar ofrece los niveles de servicio adecuados para los requisitos del contenido multimedia. 

- **Procesamiento de datos por lotes**. Estos tipos de cargas de trabajo son adecuados para el almacenamiento estándar porque requieren un almacenamiento rentable de alto rendimiento en lugar de una latencia baja constante. Los conjuntos de datos sin procesar de gran tamaño se almacenan provisionalmente para su procesamiento y, finalmente, se migran a los niveles de servicio de acceso esporádico.

## <a name="migrate-from-standard-to-premium"></a>Migración de Estándar a Premium

No se puede convertir una cuenta de almacenamiento de rendimiento Estándar existente en una cuenta de almacenamiento de blobs en bloques con rendimiento Premium. Para migrar a una cuenta de almacenamiento de rendimiento prémium, debe crear una cuenta de blob en bloques prémium y migrar los datos a la nueva cuenta. Para más información, consulte [Creación de una cuenta de BlockBlobStorage](../common/storage-account-create.md).

Para copiar blobs entre cuentas de almacenamiento, puede usar la versión más reciente de la herramienta [AzCopy](../common/storage-use-azcopy-v10.md#transfer-data) de la línea de comandos. Otras herramientas, como Azure Data Factory, también están disponibles para el movimiento y la transformación de datos.

## <a name="blob-lifecycle-management"></a>Administración del ciclo de vida de blobs

La administración del ciclo de vida de Blob Storage ofrece una directiva completa basada en reglas:

- **Premium**: los datos expiran al final de su ciclo de vida.
- **Estándar**: los datos experimentan una transición al mejor nivel de acceso y expiran al final de su ciclo de vida.

Para más información, consulte [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md).

Los datos que se almacenan en una cuenta de almacenamiento de blobs en bloques de rendimiento Premium no se pueden mover entre los niveles de acceso frecuente, esporádico y de archivo. Sin embargo, puede copiar blobs desde una cuenta de almacenamiento de blobs en bloques al nivel de acceso frecuente en una cuenta *diferente*. Use [Put Block From URL](/rest/api/storageservices/put-block-from-url) API o [AzCopy v10](../common/storage-use-azcopy-v10.md) para copiar los datos en una cuenta diferente. **Put Block From URL** API realiza una copia sincrónica de los datos en el servidor. La llamada se completa solo después de que todos los datos se han movido de la ubicación del servidor original a la ubicación de destino.

## <a name="next-steps"></a>Pasos siguientes

Evaluación de los niveles de acceso frecuente, esporádico y de archivo en cuentas de GPv2 y Blob Storage.

- [Obtenga información sobre la rehidratación de datos de blob desde el nivel de archivo](storage-blob-rehydration.md)
- [Evaluación del uso de las cuentas de almacenamiento actuales mediante la habilitación de las métricas de Azure Storage](./monitor-blob-storage.md)
- [Comprobación de los precios de los niveles de acceso frecuente, esporádico y de archivo en cuentas de Blob Storage y de GPv2 por región](https://azure.microsoft.com/pricing/details/storage/)
- [Detalles de precios de Transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/)