---
title: Información general sobre la replicación de objetos
titleSuffix: Azure Storage
description: La replicación de objetos copia asincrónicamente los blobs en bloques entre una cuenta de almacenamiento de origen y una de destino. Utilice la replicación de objetos para minimizar la latencia de las solicitudes de lectura, aumentar la eficacia de las cargas de trabajo de proceso, optimizar la distribución de los datos y minimizar los costos.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/11/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 503c146584ddf05f12ada3dfc103c86b28bd797a
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790836"
---
# <a name="object-replication-for-block-blobs"></a>Replicación de objetos para blobs en bloques

La replicación de objetos copia asincrónicamente los blobs en bloques entre una cuenta de almacenamiento de origen y una de destino. Algunos escenarios que admite la replicación de objetos son:

- **Minimización de latencia.** La replicación de objetos puede reducir la latencia de las solicitudes de lectura al permitir a los clientes consumir datos de una región que esté más cerca físicamente.
- **Aumentar la eficiencia de las cargas de trabajo de proceso.** Con la replicación de objetos, las cargas de trabajo de proceso pueden procesar los mismos conjuntos de blob en bloques en diferentes regiones.
- **Optimizar la distribución de datos.** Puede procesar o analizar los datos en una sola ubicación y luego replicar sólo los resultados en regiones adicionales.
- **Optimizar los costos.** Después de replicar los datos, puede reducir los costos moviéndolos al nivel de archivo mediante las directivas de administración del ciclo de vida.

El siguiente diagrama muestra cómo la replicación de objetos replica blob en bloques de una cuenta de almacenamiento de origen en una región a cuentas de destino en dos regiones diferentes.

:::image type="content" source="media/object-replication-overview/object-replication-diagram.svg" alt-text="Diagrama que muestra cómo funciona la replicación de objetos":::

Para información sobre cómo configurar la replicación de objetos, consulte [Configuración de la replicación de objetos](object-replication-configure.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="prerequisites-for-object-replication"></a>Requisitos previos de la replicación de objetos

La replicación de objetos requiere que también estén habilitadas las características de Azure Storage siguientes:

- [Fuente de cambios](storage-blob-change-feed.md): debe estar habilitada en la cuenta de origen. Para información sobre cómo habilitar la fuente de cambios, consulte [Habilitar y deshabilitar la fuente de cambios](storage-blob-change-feed.md#enable-and-disable-the-change-feed).
- [Control de versiones de blobs](versioning-overview.md): debe estar habilitado en la cuenta de origen y en la cuenta de destino. Para información sobre cómo habilitar el control de versiones, consulte [Habilitación y administración del control de versiones de blobs](versioning-enable.md).

Habilitar la fuente de cambios y el control de versiones de blob puede suponer costos adicionales. Para más información, consulte la página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

La replicación de objetos sólo es compatible con las cuentas de almacenamiento v2 de uso general. Tanto la cuenta de origen como la de destino deben ser de uso general v2. 

## <a name="how-object-replication-works"></a>Funcionamiento de la replicación de objetos

La replicación asincrónica de objetos copia blobs en bloques en un contenedor según las reglas que configure. El contenido del blob, las versiones asociadas al blob y los metadatos y las propiedades del blob se copian desde el contenedor de origen al contenedor de destino.

> [!IMPORTANT]
> Debido a que los datos de blob en bloques se replican asincrónicamente, la cuenta de origen y la de destino no están inmediatamente sincronizadas. Actualmente no hay ningún acuerdo de nivel de servicio sobre cuánto tiempo se tarda en replicar los datos en la cuenta de destino. Puede comprobar el estado de replicación en el blob de origen para determinar si se ha completado la replicación. Para más información, consulte el apartado [Comprobación del estado de replicación de un blob](object-replication-configure.md#check-the-replication-status-of-a-blob).

### <a name="blob-versioning"></a>Control de versiones de blobs

La replicación de objetos requiere que el control de versiones de los blobs esté habilitado en las cuentas de origen y de destino. Cuando se modifica un blob replicado de la cuenta de origen, se crea una nueva versión del blob en la cuenta de origen que refleja el estado anterior del mismo antes de la modificación. La versión actual (o el blob base) de la cuenta de origen refleja las actualizaciones más recientes. Tanto la versión actual actualizada como la nueva versión anterior se replican en la cuenta de destino. Para más información sobre la forma en que las operaciones de escritura afectan a las versiones del blob, consulte [Control de versiones en operaciones de escritura](versioning-overview.md#versioning-on-write-operations).

Cuando se elimina un blob de la cuenta de origen, la versión actual del blob se captura en una versión anterior y, después, se elimina. Todas las versiones anteriores del blob se conservan incluso después de que se elimine la versión actual. Este estado se replica en la cuenta de destino. Para más información sobre la forma en que las operaciones de eliminación afectan a las versiones del blob, consulte [Control de versiones en operaciones de eliminación](versioning-overview.md#versioning-on-delete-operations).

### <a name="snapshots"></a>Instantáneas

La replicación de objetos no admite instantáneas de blobs. Las instantáneas de un blob de la cuenta de origen no se replican en la cuenta de destino.

### <a name="blob-tiering"></a>Niveles de blob

La replicación de objetos se admite cuando las cuentas de origen y de destino se encuentran en el nivel de acceso frecuente o esporádico. Las cuentas de origen y destino pueden estar en niveles diferentes. Sin embargo, se producirá un error en la replicación de objetos si un blob de las cuentas de origen o destino se ha movido al nivel de archivo. Para más información sobre los niveles de blobs, consulte [Niveles de acceso de Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

### <a name="immutable-blobs"></a>Blobs inalterables

La replicación de objetos no admite blobs inmutables. Si un contenedor de origen o de destino tiene una directiva de retención basada en tiempo o una retención legal, se produce un error en la replicación de objetos. Para más información sobre los blobs inmutables, vea [Almacenamiento de datos críticos para la empresa con almacenamiento inmutable](storage-blob-immutable-storage.md).

## <a name="object-replication-policies-and-rules"></a>Reglas y directivas de replicación de objetos

Al configura la replicación de objetos, se crea una política de replicación que especifica la cuenta de almacenamiento de origen y la cuenta de destino. Una directiva de replicación incluye una o más reglas que especifican un contenedor de origen y un contenedor de destino e indican qué blob en bloques del contenedor de origen se replicarán.

Después de configurar la replicación de objetos, Azure Storage comprueba periódicamente la fuente de cambios de la cuenta de origen y replica asincrónicamente cualquier operación de escritura o borrada a la cuenta de destino. La latencia de replicación depende del tamaño del blob en bloques que se está replicando.

### <a name="replication-policies"></a>Directivas de replicación

Al configurar la replicación de objetos, se crea una directiva de replicación tanto en la cuenta de origen como en la de destino a través del proveedor de recursos de Azure Storage. La directiva de replicación se identifica mediante un id. de directiva. La directiva en las cuentas de origen y de destino debe tener el mismo id. de directiva para que tenga lugar la replicación.

Una cuenta de origen se puede replicar en un máximo de dos cuentas de destino, con una directiva para cada cuenta de destino. De forma similar, una cuenta de destino puede tener como máximo dos cuentas de origen. Cada directiva de replicación puede tener hasta 10 reglas de replicación, lo que significa que pueden participar en cada directiva de replicación un máximo de 10 contenedores de origen y 10 contenedores de destino.

Las cuentas de origen y de destino pueden estar en la misma región o en regiones diferentes. También pueden residir en diferentes suscripciones y en distintos inquilinos de Azure Active Directory (Azure AD). Solo se puede crear una directiva de replicación para cada par de cuentas de origen/cuentas de destino.

### <a name="replication-rules"></a>Reglas de replicación

Las reglas de replicación especifican el modo en que Azure Storage replicará los blobs de un contenedor de origen a un contenedor de destino. Puede especificar un máximo de 10 reglas de replicación para cada directiva de replicación. Cada regla de replicación define un único contenedor de origen y de destino, y cada contenedor de origen y de destino puede utilizarse en una sola regla.

Al crear una regla de replicación, de forma predeterminada solo se copian los blobs en bloques nuevos que se agreguen posteriormente al contenedor de origen. Puede especificar que se copien tanto los blob en bloques nuevos como los ya existentes, o bien puede definir un ámbito de copia personalizado que copie los blobs en bloques creados a partir de un momento determinado.

También puede especificar uno o varios filtros como parte de una regla de replicación para filtrar los blobs en bloques por prefijo. Cuando se especifica un prefijo, solo los blobs que coinciden con ese prefijo en el contenedor de origen se copiarán en el contenedor de destino.

Los contenedores de origen y de destino deben existir antes de poder especificarlos en una regla. Después de crear la directiva de replicación, no se permiten las operaciones de escritura en el contenedor de destino. Cualquier intento de escribir en el contenedor de destino producirá un error con el código de error 409 (Conflicto). Para escribir en un contenedor de destino para el que esté configurada una regla de replicación, debe eliminar la regla que está configurada para ese contenedor o quitar la directiva de replicación. Las operaciones de lectura y eliminación en el contenedor de destino se permiten cuando la directiva de replicación está activa.

Puede llamar a la operación [Establecer el nivel del blob](/rest/api/storageservices/set-blob-tier) en un blob en el contenedor de destino para moverla al nivel de archivo. Para más información sobre el nivel de archivo, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md#archive-access-tier).

## <a name="replication-status"></a>Estado de replicación

Puede comprobar el estado de replicación de un blob en la cuenta de origen. Para más información, consulte el apartado [Comprobación del estado de replicación de un blob](object-replication-configure.md#check-the-replication-status-of-a-blob).

Si el estado de replicación de un blob en la cuenta de origen indica un error, investigue las posibles causas que se mencionan a continuación:

- Asegúrese de que la directiva de replicación de objetos está configurada en la cuenta de destino.
- Compruebe que el contenedor de destino aún existe.
- Si el blob de origen se cifró con una clave proporcionada por el cliente como parte de una operación de escritura, se producirá un error en la replicación de objetos. Para más información sobre las claves proporcionadas por el cliente, consulte [Especificación de una clave de cifrado en una solicitud a Blob Storage](encryption-customer-provided-keys.md).

## <a name="billing"></a>Facturación

La replicación de objetos incurre en costos adicionales por las transacciones de lectura y escritura en las cuentas de origen y de destino, así como los cargos de salida por la replicación de datos de la cuenta de origen a la cuenta de destino y cargos de lectura para procesar la fuente de cambios.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la replicación de objetos](object-replication-configure.md)
- [Compatibilidad con la fuente de cambios en Azure Blob Storage](storage-blob-change-feed.md)
- [Habilitar y administrar las versiones de blob](versioning-enable.md)
