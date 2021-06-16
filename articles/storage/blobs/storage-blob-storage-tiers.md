---
title: 'Niveles de acceso de Azure Blob Storage: frecuente, esporádico y archivo'
description: Lea información sobre los niveles de acceso frecuente, esporádico y de archivo para Azure Blob Storage. Revise las cuentas de almacenamiento que admiten niveles.
author: tamram
ms.author: tamram
ms.date: 03/18/2021
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.reviewer: klaasl
ms.openlocfilehash: 810a62dee277453d4cc2f41c72d0868549d711cf
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110653226"
---
# <a name="access-tiers-for-azure-blob-storage---hot-cool-and-archive"></a>Niveles de acceso de Azure Blob Storage: frecuente, esporádico y archivo

Azure Storage ofrece distintos niveles de acceso, lo que permite almacenar datos de objeto de blob de la manera más rentable. Los niveles de acceso disponibles incluyen:

- **Frecuente**: Optimizado para almacenar datos que se consultan con frecuencia.
- **Esporádico**: Optimizado para almacenar datos a los que se accede con poca frecuencia y al menos durante 30 días.
- **Archivo**: optimizado para almacenar datos a los que se accede muy pocas veces y almacenados durante al menos 180 días con requisitos de latencia flexibles, del orden de horas.

Las siguientes consideraciones se aplican a los distintos niveles de acceso:

- El nivel de acceso se puede establecer en un blob durante o después de la carga.
- Solo los niveles de acceso frecuente y esporádico se pueden establecer en el nivel de cuenta. El nivel de acceso de archivo solo se puede establecer en el nivel de blob.
- Los datos del nivel de acceso esporádico tienen una disponibilidad ligeramente inferior, pero aun así ofrecen una gran durabilidad, latencia de recuperación y unas características de rendimiento similares a las de los datos de acceso frecuente. En el caso de los datos de acceso esporádico, una disponibilidad ligeramente inferior y unos costos de acceso mayores son aceptables a cambio de unos costos de almacenamiento menores en comparación con los datos de acceso frecuente. Para más información, consulte [Contrato de nivel de servicio para Storage](https://azure.microsoft.com/support/legal/sla/storage/v1_5/).
- Los datos del nivel de acceso de archivo se almacenan sin conexión. El nivel de archivo ofrece los costos de almacenamiento más bajos, pero también los costos de acceso más altos.
- Los niveles de acceso frecuente y esporádico son compatibles con todas las opciones de redundancia. El nivel de archivo solo admite las opciones de LRS, GRS y RA-GRS.
- Los límites de almacenamiento de datos se establecen en el nivel de cuenta y no por nivel de acceso. Puede elegir usar todo el límite en un nivel o entre los tres niveles.

Los datos almacenados en la nube están creciendo a un ritmo exponencial. Para administrar los costos de las crecientes necesidades de almacenamiento, resulta útil organizar los datos en función de atributos como frecuencia de acceso y el período de retención planeado para optimizar costos. Los datos almacenados en la nube pueden ser diferentes según la forma en que se generan, se procesan y se accede a ellos a lo largo de su vigencia. A algunos datos se accede y se modifican activamente a lo largo de su duración. A algunos datos se accede con frecuencia al principio de su duración, mientras que el acceso cae drásticamente a medida que envejecen los datos. Algunos datos permanecen inactivos en la nube y, después de que se almacenan, no se accede a ellos prácticamente nunca.

Cada uno de los escenarios de acceso a los datos se beneficia de un distinto nivel de acceso que está optimizado para un patrón de acceso concreto. Con los niveles de acceso frecuente, esporádico y de archivo, Azure Blob Storage satisface esta necesidad de que haya niveles de acceso diferenciados con modelos de precios independientes.

Las siguientes herramientas y bibliotecas cliente son compatibles con el almacenamiento por niveles de blob y el almacenamiento de archivo.

- Azure portal
- PowerShell
- Herramientas de la CLI de Azure
- Biblioteca cliente de .NET
- Biblioteca cliente de Java
- Biblioteca cliente de Python
- Biblioteca cliente de Node.js

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="storage-accounts-that-support-tiering"></a>Cuentas de almacenamiento que admiten niveles

El almacenamiento de objetos por niveles de acceso frecuente, esporádico y de archivo se admite en cuentas de Blob Storage y de uso general v2 (GPv2). Las cuentas de uso general v1 (GPv1) no admiten niveles. Es posible convertir fácilmente las cuentas de GPv1 o de Blob Storage existentes en cuentas de GPv2 mediante Azure Portal. GPv2 proporciona nuevos precios y características para blobs, archivos y colas. Algunas características y reducciones de precios solo se ofrecen en cuentas de GPv2. Algunas cargas de trabajo pueden ser más caras en GPv2 que en GPv1. Para más información, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

Las cuentas de Blob Storage y GPv2 exponen el atributo **access tier** en el nivel de cuenta. Este atributo permite especificar el nivel de acceso predeterminado de cualquier blob que no lo tenga establecido explícitamente en el nivel de objeto. En el caso de objetos con el nivel establecido explícitamente, el nivel de cuenta no se aplica. El nivel de archivo solo puede aplicarse en el nivel de objeto. Puede cambiar entre niveles de acceso en cualquier momento.

Use cuentas de GPv2 en lugar de cuentas de Blob Storage para el almacenamiento por niveles. GPv2 admite todas las características de las cuentas de Blob Storage y muchas más. Los precios de Blob Storage y GPv2 son prácticamente idénticos, pero algunas características nuevas y reducciones de precios solo están disponibles en cuentas de GPv2.

La estructura de precios entre las cuentas de GPv1 y GPv2 es diferente y los clientes deben evaluar con cuidado ambas antes de decidirse a usar las cuentas de GPv2. Puede convertir fácilmente una cuenta existente de Blob Storage o de GPv1 en GPv2 con un simple clic. Para más información, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

## <a name="hot-access-tier"></a>Nivel de acceso frecuente

El nivel de acceso frecuente tiene mayores costos de almacenamiento que los niveles esporádico y de archivo, pero menores costos de acceso. Entre los ejemplos de escenarios de uso del nivel de acceso frecuente se incluyen:

- Datos que están en uso o que se espera que se lean y se escriban con frecuencia
- Datos almacenados de manera provisional para su procesamiento y posterior migración al nivel de acceso esporádico

## <a name="cool-access-tier"></a>Nivel de acceso esporádico

El nivel de acceso esporádico tiene menores costos de almacenamiento y mayores costos de acceso, en comparación con el almacenamiento frecuente. Este nivel está diseñado para datos que van a permanecer en el nivel de acceso esporádico durante al menos 30 días. Entre los ejemplos de escenarios de uso del nivel de acceso esporádico se incluyen:

- Copia de seguridad y recuperación ante desastres a corto plazo
- Datos más antiguos que no se usan con frecuencia pero que se espera que estén disponibles de inmediato cuando se accede a ellos
- Conjuntos de datos de gran tamaño que se deben almacenar de forma rentable mientras se recopilan más datos para el procesamiento futuro

## <a name="archive-access-tier"></a>Nivel de acceso de archivo

El nivel de acceso de archivo tiene el menor costo de almacenamiento, pero los mayores costos de recuperación de datos en comparación con los niveles de acceso frecuente y esporádico. Los datos deben permanecer en el nivel de archivo durante al menos 180 días o estar sujetos a un cargo por eliminación temprana. Los datos del nivel de archivo pueden tardar varias horas en recuperarse en función de la prioridad de rehidratación especificada. En el caso de los objetos pequeños, una rehidratación de alta prioridad puede recuperar el objeto del archivo en menos de una hora. Consulte [Rehidratación de los datos de blob desde el nivel de archivo](storage-blob-rehydration.md) para obtener más información.

Mientras un blob está en almacenamiento de archivo, sus datos están sin conexión y no se pueden leer ni modificar. Para leer o descargar un blob en un archivo, primero debe rehidratarlo en un nivel en línea. No puede tomar instantáneas de un blob en almacenamiento de archivo. Sin embargo, los metadatos de blob permanecen en línea y se mantienen disponibles, lo que permite enumerar el blob, sus propiedades, los metadatos y las etiquetas de índice de blob. No se permite establecer ni modificar los metadatos de blob mientras están en almacenamiento de archivo, aunque se pueden establecer y modificar las etiquetas de índice de blob. En el caso de los blobs en almacenamiento de archivo, las únicas operaciones válidas son [Get Blob Properties](/rest/api/storageservices/get-blob-properties), [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata), [Set Blob Tags](/rest/api/storageservices/set-blob-tags), [Get Blob Tags](/rest/api/storageservices/get-blob-tags), [Find Blobs by Tags](/rest/api/storageservices/find-blobs-by-tags), [List Blobs](/rest/api/storageservices/list-blobs), [Set Blob Tier](/rest/api/storageservices/set-blob-tier), [Copy Blob](/rest/api/storageservices/copy-blob) y [Delete Blob](/rest/api/storageservices/delete-blob).

Entre los ejemplos de escenarios de uso del nivel de acceso de archivo se incluyen:

- Copia de seguridad a largo plazo, copia de seguridad secundaria y conjuntos de datos de archivado
- Datos originales (sin procesar) que deben conservarse, incluso después de que se hayan procesado en un formato útil final
- Datos de cumplimiento y archivado que se deben almacenar durante un largo período de tiempo y a los que casi nunca se accede

> [!NOTE]
> El nivel de archivo no se admite en las cuentas de ZRS, GZRS o RA-GZRS. Se admite la migración de LRS a GRS siempre que no se hayan trasladado blobs al nivel de archivo mientras la cuenta estaba establecida en LRS. Una cuenta se puede devolver a GRS si la actualización se realiza en menos de 30 días desde el momento en que la cuenta se convirtió en LRS y no se ha movido ningún blob al nivel de archivo mientras la cuenta estaba establecida en LRS.

## <a name="account-level-tiering"></a>Almacenamiento por niveles de cuenta

Dentro de la misma cuenta pueden coexistir blobs de los tres niveles de acceso. Los blobs que no tienen un nivel asignado explícitamente infieren el nivel de la configuración del nivel de acceso de la cuenta. Si el nivel de acceso procede de la cuenta, verá que la propiedad de blob **Access Tier Inferred** se establece en "true", y que la propiedad de blob **Access Tier** coincide con el nivel de cuenta. En Azure Portal, la propiedad _access tier inferred_ se muestra con el nivel de acceso de blob como **Frecuente (inferido)** o **Esporádico (inferido)** .

El cambio del nivel de acceso de cuenta se aplica a todos los objetos _access tier inferred_ almacenados en la cuenta que no tengan un nivel explícito establecido. Si cambia el nivel de acceso de la cuenta de frecuente a esporádico, solo se le cobrarán las operaciones de escritura (por 10 000) de todos los blobs sin un nivel establecido en las cuentas de GPv2. En las cuentas de Blob Storage no se realiza ningún cargo por este cambio. Si cambia el nivel de acceso de su cuenta de Blob Storage o de GPv2 de esporádico a frecuente, se le cobran tanto las operaciones de lectura (por 10 000) como las de recuperación de datos (por GB).

Solo los niveles de almacenamiento de acceso frecuente y esporádico se pueden establecer como nivel de acceso de cuenta predeterminado. El nivel de acceso de archivo solo puede establecerse en el nivel de objeto. Al cargar blobs, puede especificar el nivel de acceso que prefiera para que sea frecuente, esporádico o de archivo, independientemente del nivel de cuenta predeterminado. Esta funcionalidad permite escribir datos directamente en el nivel de archivo para ahorrar en costos desde el momento en que se crean los datos en el almacenamiento de blobs.

## <a name="blob-level-tiering"></a>Almacenamiento por niveles de blob

El almacenamiento por niveles de blob permite cargar datos en el nivel de acceso preferido mediante las operaciones [Put Blob](/rest/api/storageservices/put-blob) o [Put Block List](/rest/api/storageservices/put-block-list), y cambiar el nivel de los datos en el nivel de objeto mediante la operación [Set Blob Tier](/rest/api/storageservices/set-blob-tier) o la característica de [administración del ciclo de vida](#blob-lifecycle-management). Puede cargar datos a su nivel de acceso necesario y, a continuación, cambiar el nivel de acceso de un blob entre los niveles de acceso frecuente, esporádico y de archivo, a medida que cambien los patrones de uso, sin tener que mover los datos entre las cuentas. Todas las solicitudes de cambio de nivel se producen inmediatamente y los cambios entre el nivel de acceso frecuente y esporádico son instantáneos. La rehidratación de un blob de un nivel de acceso de archivo puede tardar varias horas.

La hora del último cambio de nivel de blob se expone a través de la propiedad de blob **access tier change time**. La propiedad **access tier change time** es de nivel de blob y no se actualiza cuando se cambia el nivel de cuenta predeterminado. Las propiedades de cuenta y de blob son independientes. Sería muy costoso actualizar la propiedad **access tier change time** en todos los blobs de una cuenta de almacenamiento cada vez que se cambia el nivel de acceso predeterminado de la cuenta.

Al sobrescribir un blob en el nivel de acceso frecuente o esporádico, el blob recién creado hereda el nivel del blob que se ha sobrescrito a menos que el nuevo nivel de acceso del blob se establezca explícitamente en la creación. Si un blob está en el nivel de archivo, no se puede sobrescribir y, por tanto, la carga del mismo blob no se permite en este escenario.

> [!NOTE]
> El almacenamiento de archivo y el almacenamiento por niveles de blob solo admiten blobs en bloques.

### <a name="blob-lifecycle-management"></a>Administración del ciclo de vida de blobs

La administración del ciclo de vida de Blob Storage ofrece una directiva enriquecida basada en reglas que se puede usar para la transición de los datos al mejor nivel de acceso y para hacer que los datos expiren cuando finalice su ciclo de vida. Vea [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](storage-lifecycle-management-concepts.md) para obtener más información.

> [!NOTE]
> Los datos almacenados en una cuenta de almacenamiento de blobs en bloques (rendimiento Premium) no se pueden organizar en niveles de acceso frecuente, esporádico o de archivo con [Set Blob Tier](/rest/api/storageservices/set-blob-tier) ni mediante la administración del ciclo de vida de Azure Blob Storage.
> Para mover datos, debe copiar los blobs de forma sincrónica desde la cuenta de almacenamiento de blobs en bloques en el nivel de acceso frecuente de otra cuenta mediante la [API Put Block From URL](/rest/api/storageservices/put-block-from-url) o una versión de AzCopy que admita esta API.
> **Put Block From URL** API copia datos de forma asíncrónica en el servidor, lo que significa que la llamada solo se completa una vez que todos los datos se han movido de la ubicación original del servidor a la ubicación de destino.

### <a name="blob-level-tiering-billing"></a>Facturación del almacenamiento por niveles de blob

Cuando un blob se carga o se mueve entre niveles, se cobra a la tarifa correspondiente inmediatamente después de la carga o el cambio de nivel.

Cuando un blob se mueve a un nivel de almacenamiento de acceso más esporádico (frecuente -> esporádico, frecuente -> archivo o esporádico -> archivo), la operación se factura como una operación de escritura del nivel de destino, donde se aplican los cargos de la operación de escritura (por 10 000) y de la escritura de datos (por GB) del nivel de destino.

Cuando un blob se mueve a un nivel más frecuente (archivo->esporádico, archivo->frecuente, o esporádico->frecuente), la operación se factura como una lectura desde el nivel de origen, donde se aplican los cargos de la operación de lectura (por 10 000) y de la recuperación de datos (por GB) del nivel de origen. También podrían aplicarse cargos por la [eliminación temprana](#cool-and-archive-early-deletion) de cualquier blob que se haya trasladado desde el nivel de acceso esporádico o de archivo. La [Rehidratación de los datos desde el nivel de archivo](storage-blob-rehydration.md) lleva tiempo, y los datos se cobran a los precios de archivo hasta que se restauran en línea y el nivel de blob cambia a acceso frecuente o esporádico.

La tabla siguiente resume cómo se facturan los cambios de nivel:

| | **Cargos de escritura (operación + acceso)** | **Cargos de lectura (operación + acceso)** |
| ---- | ----- | ----- |
| **Set Blob Tier** | frecuente -> esporádico<br> frecuente -> archivo<br> esporádico -> archivo | archivo -> esporádico<br> archivo -> frecuente<br> esporádico -> frecuente

### <a name="cool-and-archive-early-deletion"></a>Eliminación temprana en los niveles de acceso esporádico y de archivo

Cualquier blob que se mueva al nivel de acceso esporádico (solo cuentas de GPv2) está sujeto a un período de eliminación temprana del acceso esporádico de 30 días. Cualquier blob que se mueva al nivel de acceso esporádico (solo cuentas de GPv2) está sujeto a un período de eliminación temprana del acceso esporádico de 180 días. Este cargo se prorratea. Por ejemplo, si un blob se mueve al nivel de acceso de archivo y, a continuación, se elimina o se mueve al nivel de acceso frecuente al cabo de 45 días, se le cobrará una cuota de eliminación temprana equivalente a 135 (180 menos 45) días a partir del almacenamiento de ese blob en el nivel de acceso de archivo.

Hay algunos detalles a la hora de moverse entre los niveles de acceso esporádico y de archivo:

1. Si se infiere un blob como esporádico en el nivel de acceso predeterminado de la cuenta de almacenamiento y se mueve al nivel de acceso de archivo, no hay ningún cargo de eliminación temprana.
1. Si un blob se mueve explícitamente al nivel de acceso esporádico y luego se mueve al nivel de acceso de archivo, se aplica el cargo de eliminación temprana.

Para calcular la hora de eliminación temprana, use la propiedad de blob **Last-Modified**, siempre que no haya habido cambios en el nivel de acceso. De lo contrario, use la hora de la última modificación del nivel de acceso a esporádico o de archivo; para ello, consulte la propiedad de blob **access-tier-change-time**. Para más información sobre las propiedades de blob, consulte el artículo sobre cómo [obtener las propiedades de blob](/rest/api/storageservices/get-blob-properties).

## <a name="comparing-block-blob-storage-options"></a>Comparar opciones de almacenamiento de blobs en bloques

En la siguiente tabla se muestra una comparación del almacenamiento de blobs en bloques de rendimiento Premium, así como los niveles de acceso frecuente, esporádico y de archivo.

|                                           | **Rendimiento Premium**   | **Nivel frecuente** | **Nivel esporádico**       | **Nivel de archivo**  |
| ----------------------------------------- | ------------------------- | ------------ | ------------------- | ----------------- |
| **Disponibilidad**                          | 99,9 %                     | 99,9 %        | 99%                 | Sin conexión           |
| **Disponibilidad** <br> **(lecturas de RA-GRS)**  | N/D                       | 99,99%       | 99,9 %               | Sin conexión           |
| **Cargos de uso**                         | Costos de almacenamiento más elevados y costos de acceso y transacción más bajos | Mayores costos de almacenamiento, menores costos de acceso y transacciones | Menores costos de almacenamiento, mayores costos de acceso y transacciones | Menores costos de almacenamiento, mayores costos de acceso y transacciones |
| **Duración mínima del almacenamiento**              | N/D                       | N/D          | 30 días<sup>1</sup> | 180 días
| **Latency** <br> **(Tiempo hasta el primer byte)** | Milisegundos de un solo dígito | milisegundos | milisegundos        | horas<sup>2</sup> |

<sup>1</sup> Los objetos del nivel de acceso esporádico en las cuentas de GPv2 tienen una duración mínima de la retención de 30 días. Las cuentas de Blob Storage no tienen ninguna duración mínima de la retención para el nivel esporádico.

<sup>2</sup> Archive Storage admite actualmente dos prioridades de rehidratación, alta y estándar, que ofrecen diferentes latencias y costos de recuperación. Para más información, consulte [Rehidratación de los datos de blob desde el nivel de archivo](storage-blob-rehydration.md).

> [!NOTE]
> Las cuentas de Blob Storage admiten los mismos objetivos de rendimiento y escalabilidad que las cuentas de almacenamiento de uso general v2 (GPv2). Para más información, consulte [Objetivos de escalabilidad y rendimiento de Blob Storage](scalability-targets.md).

## <a name="pricing-and-billing"></a>Precios y facturación

Todas las cuentas de almacenamiento usan un modelo de precios para el almacenamiento de blobs en bloques basado en el nivel de cada blob. Tenga en cuenta las siguientes consideraciones de facturación:

- **Costos de almacenamiento**: Además de la cantidad de datos almacenados, el costo varía en función del nivel de acceso. El costo por gigabyte disminuye a medida que el nivel es más esporádico.
- **Costos de acceso a datos**: los gastos de acceso a los datos aumentan a medida que el nivel es más esporádico. En el nivel de acceso esporádico y de archivo se cobra un cargo de acceso a datos por gigabyte por las operaciones de lectura.
- **Costos de transacciones**: hay un cargo por transacción para todos los niveles, que aumenta a medida que el nivel es más esporádico.
- **Costos de transferencia de datos de replicación geográfica**: este cargo solo se aplica a las cuentas con replicación geográfica configurada, incluidas GRS y RA-GRS. La transferencia de datos de replicación geográfica incurre en un cargo por gigabyte.
- **Costos de transferencia de datos salientes**: las transferencias de datos salientes (los datos que se transfieren fuera de una región de Azure) conllevan un cargo por el uso del ancho de banda por gigabyte, lo que es coherente con las cuentas de almacenamiento de uso general.
- **Cambio del nivel de acceso**: el cambio del nivel de acceso de la cuenta comporta cargos por cambio de nivel para todos los blobs que no tengan un nivel explícito establecido. Para obtener más información sobre el cambio del nivel de acceso de un solo blob, vea [Facturación del almacenamiento por niveles de blob](#blob-level-tiering-billing).

    Cambiar el nivel de acceso de un blob cuando el control de versiones está habilitado, o si el blob tiene instantáneas, puede dar lugar a cargos adicionales. Para obtener información sobre los blobs con el control de versiones habilitado, vea [Precios y facturación](versioning-overview.md#pricing-and-billing) en la documentación sobre el control de versiones de blob. Para obtener información sobre los blobs con instantáneas, vea [Precios y facturación](snapshots-overview.md#pricing-and-billing) en la documentación sobre instantáneas de blob.

> [!NOTE]
> Para obtener más información sobre los precios de los blobs en bloques, vea [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/). Para obtener más información sobre los cargos por transferencia de datos salientes, vea la página [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).

## <a name="availability"></a>Disponibilidad

Los diferentes niveles de acceso, junto con el almacenamiento por niveles de blob, están disponibles en regiones seleccionadas. Para obtener una lista completa, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage).

## <a name="next-steps"></a>Pasos siguientes

Aprenda a administrar blobs y cuentas en niveles de acceso.

- [Administración del nivel de acceso de un blob en una cuenta de Azure Storage](manage-access-tier.md)
- [Administración del nivel de acceso predeterminado de una cuenta de Azure Storage](../common/manage-account-default-access-tier.md)
- [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](storage-lifecycle-management-concepts.md)
