---
title: Control de versiones de blobs
titleSuffix: Azure Storage
description: El control de versiones de almacenamiento de blobs conserva automáticamente las versiones anteriores de un objeto y las identifica con marcas de tiempo. Puede restaurar las versiones anteriores de un blob para recuperar los datos si se modifican o eliminan erróneamente.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 04/08/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 268de3e8ea168ac721362d42149389b9f37c86fe
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107305062"
---
# <a name="blob-versioning"></a>Control de versiones de blobs

Puede habilitar el control de versiones de almacenamiento de blobs para conservar automáticamente las versiones anteriores de un objeto. Cuando el control de versiones de blobs está habilitado, puede restaurar una versión anterior de un blob para recuperar los datos si se modifican o eliminan por error.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="recommended-data-protection-configuration"></a>Configuración de protección de datos recomendada

El control de versiones de blobs forma parte de una exhaustiva estrategia para proteger los datos de los blobs. Para obtener una protección óptima de los datos de blobs, Microsoft recomienda habilitar las siguientes características de protección de datos:

- Control de versiones de blobs, para conservar automáticamente las versiones anteriores de un blob. Cuando el control de versiones de blobs está habilitado, puede restaurar una versión anterior de un blob para recuperar los datos si se modifican o eliminan por error. Para obtener información sobre cómo habilitar el control de versiones de blobs, consulte [Habilitación y administración del control de versiones de blobs](versioning-enable.md).
- Eliminación temporal de contenedores, para restaurar un contenedor que se ha eliminado. Para obtener información sobre cómo habilitar la eliminación temporal de contenedores, consulte [Habilitación y administración de la eliminación temporal de contenedores](soft-delete-container-enable.md).
- Eliminación temporal de blobs, para restaurar un blob, una instantánea o una versión que se han eliminado. Para obtener información sobre cómo habilitar la eliminación temporal de blobs, consulte [Habilitación y administración de la eliminación temporal para blobs](soft-delete-blob-enable.md).

Para obtener más información sobre las recomendaciones de Microsoft con respecto a la protección de datos, consulte [Información general sobre la protección de datos](data-protection-overview.md).

## <a name="how-blob-versioning-works"></a>Funcionamiento del control de versiones de blobs

Una versión captura el estado de un blob en un momento dado. Cada versión se identifica mediante un identificador. Si el control de versiones de blobs está habilitado para una cuenta de almacenamiento, Azure Storage crea automáticamente una versión con un identificador único cuando se crea por primera vez un blob y cada vez que este se modifica posteriormente.

Un identificador de versión puede identificar la versión actual u otra anterior. Un blob solo puede tener una versión actual a la vez.

Cuando se crea un blob, existe una única versión, que es la versión actual. Al modificar un blob existente, la versión actual se convierte en otra anterior. Se crea una versión para capturar el estado actualizado y esa nueva versión pasa a ser la actual. Cuando elimina un blob, la versión actual se convierte en otra anterior y deja de haber una versión actual. Las versiones anteriores del blob se conservan.

En el siguiente diagrama se muestra cómo se crean las versiones en operaciones de escritura y cómo se puede promover una versión anterior para que pase a ser la actual:

:::image type="content" source="media/versioning-overview/blob-versioning-diagram.png" alt-text="Diagrama que muestra el funcionamiento del control de versiones de blobs":::

Las versiones del blob son inmutables. No puede modificar el contenido ni los metadatos de la versión de un blob existente.

Tener un gran número de versiones por cada blob puede aumentar la latencia de las operaciones de enumeración de blobs. Microsoft recomienda mantener menos de 1000 versiones por blob. Puede usar la administración del ciclo de vida para eliminar automáticamente las versiones anteriores. Para obtener más información sobre la administración del ciclo de vida, consulte [Optimización de los costos mediante la automatización de los niveles de acceso de Azure Blob Storage](storage-lifecycle-management-concepts.md).

El control de versiones de blobs está disponible para cuentas estándar de uso general V2, cuentas premium de blob en bloques y cuentas heredadas de almacenamiento de blobs. Actualmente, no se admiten las cuentas de almacenamiento con un espacio de nombres jerárquico habilitado para usarse con Azure Data Lake Storage Gen2.

La versión 2019-10-10 y las posteriores de la API REST de Azure Storage admiten el control de versiones de blobs.

> [!IMPORTANT]
> El control de versiones de blobs no puede ayudarle a recuperar una eliminación accidental de una cuenta de almacenamiento o un contenedor. Para evitar dicha eliminación, configure un bloqueo en el recurso de la cuenta de almacenamiento. Para obtener más información sobre cómo bloquear una cuenta de almacenamiento, consulte [Aplicación de un bloqueo de Azure Resource Manager a una cuenta de almacenamiento](../common/lock-account-resource.md).

### <a name="version-id"></a>Id. de la versión

Cada versión de blob se identifica mediante un identificador único. El valor del identificador de la versión es la marca de tiempo en la que se ha actualizado el blob. El identificador de la versión se asigna en el momento en que se crea la versión.

Para realizar operaciones de lectura o eliminación en una versión específica de un blob, puede proporcionar su identificador de versión. Si omite el identificador de la versión, la operación actúa con respecto a la versión actual.

Cuando llama a una operación de escritura para crear o modificar un blob, Azure Storage devuelve el encabezado *x-ms-version-id* en la respuesta. Este encabezado contiene el identificador de la versión actual del blob creado mediante la operación de escritura.

El identificador de la versión es el mismo para toda la vigencia de esta.

### <a name="versioning-on-write-operations"></a>Control de versiones en operaciones de escritura

Cuando se activa el control de versiones de blobs, con cada operación de escritura en un blob se crea una nueva versión. Las operaciones de escritura incluyen [Put Blob](/rest/api/storageservices/put-blob), [Put Block List](/rest/api/storageservices/put-block-list), [Copy Blob](/rest/api/storageservices/copy-blob) y [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata).

Si la operación de escritura crea un nuevo blob, el resultante es su versión actual. Si la operación modifica un blob ya existente, la versión actual se convierte en otra anterior, y se crea una nueva versión actual para capturar el blob actualizado.

En el diagrama siguiente se muestra cómo las operaciones de escritura afectan a las versiones de los blobs. Por motivos de simplicidad, en los diagramas que se incluyen en este artículo el identificador de la versión se muestra como un valor entero simple. En realidad, el identificador de la versión es una marca de tiempo. La versión actual se muestra en azul y las anteriores, en gris.

:::image type="content" source="media/versioning-overview/write-operations-blob-versions.png" alt-text="Diagrama que muestra cómo las operaciones de escritura afectan a los blobs con versiones":::

> [!NOTE]
> Un blob creado antes de habilitar el control de versiones para la cuenta de almacenamiento no tiene ningún identificador de versión. Cuando se modifica, se convierte en la versión actual y se crea una versión para guardar el estado del blob antes de la actualización. Se asigna un identificador a la versión, que corresponde a la hora de creación.

Si el control de versiones de blobs está habilitado para una cuenta de almacenamiento, todas las operaciones de escritura en blobs en bloques desencadenan la creación de una versión, con la excepción de la operación [Put Block](/rest/api/storageservices/put-block).

En el caso de los blobs en páginas y anexos, solo un subconjunto de operaciones de escritura desencadena la creación de una versión. Entre las operaciones se incluyen:

- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Block List](/rest/api/storageservices/put-block-list)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Copy Blob](/rest/api/storageservices/copy-blob)

Las siguientes operaciones no la desencadenan. Para capturar los cambios de esas operaciones, realice una instantánea manual:

- [Put Page](/rest/api/storageservices/put-page) (blob en páginas)
- [Append Block](/rest/api/storageservices/append-block) (blob en anexos)

Todas las versiones de un blob deben ser del mismo tipo de blob. Si un blob tiene versiones anteriores, no puede sobrescribirlo con uno de otro tipo, a menos que primero elimine el blob y todas sus versiones.

### <a name="versioning-on-delete-operations"></a>Control de versiones en operaciones de eliminación

Cuando se llama a la operación [Delete Blob](/rest/api/storageservices/delete-blob) sin especificar un identificador de versión, la versión actual se convierte en otra anterior y deja de haber una versión actual. Todas las versiones del blob que ya existieran previamente se conservan.

En el diagrama siguiente se muestra el efecto de una operación de eliminación en un blob con versiones:

:::image type="content" source="media/versioning-overview/delete-versioned-base-blob.png" alt-text="Diagrama que muestra la eliminación de un blob con versiones":::

Para eliminar una versión específica de un blob, proporcione el identificador de la versión en la operación de eliminación. Si la eliminación temporal de blobs también está habilitada para la cuenta de almacenamiento, la versión se mantiene en el sistema hasta que transcurre el período de retención de la eliminación.

Al escribir nuevos datos en el blob, se crea una versión actual de este. Las versiones existentes no se verán afectadas, tal como se muestra en el diagrama siguiente.

:::image type="content" source="media/versioning-overview/recreate-deleted-base-blob.png" alt-text="Diagrama que muestra la nueva creación del blob con versiones después de la eliminación":::

### <a name="access-tiers"></a>Niveles de acceso

Puede trasladar cualquier versión de un blob en bloques, incluida la actual, a otro nivel de acceso de blobs mediante una llamada a la operación [Set Blob Tier](/rest/api/storageservices/set-blob-tier). Para aprovechar los precios de menor capacidad, traslade las versiones anteriores de un blob al nivel de acceso esporádico o de archivo. Para obtener más información, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

Para automatizar el proceso de trasladar blobs en bloques al nivel adecuado, use la administración del ciclo de vida del blob. Para obtener más información sobre la administración del ciclo de vida, consulte [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md).

## <a name="enable-or-disable-blob-versioning"></a>Habilitación o deshabilitación del control de versiones de blobs

Para obtener información sobre cómo habilitar el control de versiones de blobs, consulte [Habilitación y administración del control de versiones de blob (versión preliminar)](versioning-enable.md).

Al deshabilitar el control de versiones de blobs, no se eliminan los blobs, las versiones ni las instantáneas existentes. Cuando desactiva el control de versiones de blobs, todas las versiones existentes permanecen accesibles en la cuenta de almacenamiento. No se crean nuevas versiones posteriormente.

Si un blob se creó o modificó después de que se deshabilitara el control de versiones en la cuenta de almacenamiento, se creará una nueva versión cuando se sobrescriba. El blob actualizado ya no es la versión actual ni tiene ningún identificador de versión. Todas las actualizaciones posteriores del blob sobrescribirán sus datos sin guardar el estado anterior.

Puede leer o eliminar versiones con el identificador de la versión después de deshabilitar el control de versiones. Así mismo, después de deshabilitarlo, también puede enumerar las versiones de un blob.

En el diagrama siguiente se muestra cómo se crea un blob sin versiones cuando se modifica un blob después de deshabilitar el control de versiones. Las versiones existentes asociadas con el blob se conservan.

:::image type="content" source="media/versioning-overview/modify-base-blob-versioning-disabled.png" alt-text="Diagrama que muestra el blob base modificado después de deshabilitar el control de versiones":::

## <a name="blob-versioning-and-soft-delete"></a>Control de versiones de blobs y eliminación temporal

Con el fin de obtener una protección de datos óptima, Microsoft recomienda habilitar el control de versiones y la eliminación temporal de blobs para las cuentas de almacenamiento. Para obtener más información sobre la eliminación temporal, consulte [Eliminación temporal de blobs de Azure Storage](./soft-delete-blob-overview.md).

### <a name="overwriting-a-blob"></a>Sobrescritura de un blob

Si están habilitados el control de versiones y la eliminación temporal de blobs para una cuenta de almacenamiento, al sobrescribir un blob se crea una versión automáticamente. La nueva versión no se elimina de forma temporal y no se quita cuando expira el período de retención de eliminación temporal. No se crea ninguna instantánea eliminada temporalmente.

### <a name="deleting-a-blob-or-version"></a>Eliminación de un blob o una versión

Si están habilitados el control de versiones y la eliminación automática para una cuenta de almacenamiento, al eliminar un blob la versión actual de este se convierte en otra anterior. No se crea ninguna versión ni ninguna instantánea eliminada temporalmente. El período de retención de la eliminación temporal no se aplica al blob eliminado.

La eliminación temporal ofrece protección adicional para eliminar versiones de blobs. Cuando se elimina una versión anterior del blob, esta se elimina de forma temporal. La versión eliminada temporalmente se conserva hasta que transcurre el período de retención, momento en el que se elimina de forma permanente.

Para eliminar una versión anterior de un blob, llame a la operación **Delete Blob** y especifique el identificador de la versión.

En el diagrama siguiente se muestra lo que ocurre cuando elimina un blob o una versión de este.

:::image type="content" source="media/versioning-overview/soft-delete-historical-version.png" alt-text="Diagrama que muestra la eliminación de una versión con la eliminación temporal habilitada":::

### <a name="restoring-a-soft-deleted-version"></a>Restauración de una versión eliminada temporalmente

Puede usar la operación [Undelete Blob](/rest/api/storageservices/undelete-blob) para restaurar versiones eliminadas temporalmente durante el período de retención de la eliminación. La operación **Undelete Blob** siempre restaura todas las versiones eliminadas temporalmente del blob. No es posible restaurar una sola versión eliminada temporalmente.

Al realizar dicha restauración con la operación **Undelete Blob**, no se promueve ninguna versión para que sea la actual. Para restaurar la versión actual, primero restaure todas las versiones eliminadas temporalmente y, después, use la operación [Copy Blob](/rest/api/storageservices/copy-blob) para copiar una versión anterior en otra actual de reciente creación.

En el diagrama siguiente se muestra cómo restaurar las versiones de blobs eliminadas temporalmente con la operación **Undelete blob** y cómo restaurar la versión actual del blob con la operación **Copy blob**.

:::image type="content" source="media/versioning-overview/undelete-version.png" alt-text="Diagrama que muestra cómo restaurar versiones eliminadas temporalmente":::

Una vez transcurrido el período de retención de eliminación temporal, cualquier versión del blob eliminada temporalmente se eliminará de forma permanente.

## <a name="blob-versioning-and-blob-snapshots"></a>Control de versiones e instantáneas de blobs

Una instantánea de blob es una copia de solo lectura de un blob que se ha realizado en un momento dado específico. Las instantáneas y las versiones de blobs son similares, pero, las instantáneas las crea el usuario o la aplicación de forma manual, mientras que una versión de blob se crea automáticamente en una operación de escritura o eliminación cuando el control de versiones de blobs está habilitado para la cuenta de almacenamiento.

> [!IMPORTANT]
> Microsoft recomienda que, después de habilitar el control de versiones de blobs, también actualice la aplicación para que deje de tomar instantáneas de blobs en bloques. Si está habilitado el control de versiones para la cuenta de almacenamiento, las versiones capturan y conservan todas las actualizaciones y eliminaciones de los blobs en bloques. La toma de instantáneas no ofrece ninguna protección adicional a los datos de los blobs en bloques si el control de versiones de blobs está habilitado, y puede aumentar los costos y la complejidad de la aplicación.

### <a name="snapshot-a-blob-when-versioning-is-enabled"></a>Instantánea de un blob cuando el control de versiones está habilitado

Aunque no se recomienda, puede tomar una instantánea de un blob que también tenga versiones. Si no puede actualizar la aplicación para que deje de tomar instantáneas de blobs al habilitar el control de versiones, es posible que la aplicación admita tanto instantáneas como versiones.

Cuando toma una instantánea de un blob con versiones, se crea una nueva versión al mismo tiempo que se crea la instantánea. También se crea una nueva versión actual cuando se toma una instantánea.

En el diagrama siguiente se muestra lo que ocurre cuando toma una instantánea de un blob con versiones. En el diagrama, las versiones del blob y las instantáneas con el identificador de versión 2 y 3 contienen datos idénticos.

:::image type="content" source="media/versioning-overview/snapshot-versioned-blob.png" alt-text="Diagrama que muestra instantáneas de un blob con versiones ":::

## <a name="authorize-operations-on-blob-versions"></a>Autorización de operaciones en versiones de blobs

Puede autorizar el acceso a las versiones de blobs mediante uno de los enfoques siguientes:

- Mediante el uso de un control de acceso basado en roles (RBAC) para conceder permisos a una entidad de seguridad de Azure Active Directory (Azure AD). Microsoft recomienda usar Azure AD para mayor seguridad y facilidad de uso. Para obtener más información sobre el uso de Azure AD con las operaciones de blobs, consulte [Autorización del acceso a los blobs y las colas con Azure Active Directory](../common/storage-auth-aad.md).
- Firma de acceso compartido (SAS), para delegar el acceso a las versiones de blobs. Especifique el identificador de versión del tipo de recurso firmado `bv`, que representa una versión de blob, a fin de crear un token de SAS para las operaciones en una versión específica. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).
- Claves de acceso a la cuenta, para autorizar las operaciones mediante versiones de blobs con la clave compartida. Para más información, consulte el artículo sobre la [Autorización con clave compartida](/rest/api/storageservices/authorize-with-shared-key).

El control de versiones de blobs está diseñado para proteger los datos de la eliminación accidental o malintencionada. A fin de mejorar la protección, la eliminación de una versión de blob requiere permisos especiales. En las secciones siguientes se describen los permisos necesarios para eliminar una versión de un blob.

### <a name="azure-rbac-action-to-delete-a-blob-version"></a>Acción de Azure RBAC para eliminar una versión de un blob

En la tabla siguiente se muestran las acciones de Azure RBAC que admiten la eliminación de un blob o una versión de blob.

| Descripción | Operación de servicio de blob | Acción de datos de Azure RBAC necesaria | Informe de rol integrado de Azure |
|----------------------------------------------|------------------------|---------------------------------------------------------------------------------------|-------------------------------|
| Eliminación de la versión actual | Delete Blob | **Microsoft.Storage/storageAccounts/blobServices/containers/blobs/delete** | Colaborador de datos de blobs de almacenamiento |
| Eliminación de una versión anterior | Delete Blob | **Microsoft.Storage/storageAccounts/blobServices/containers/blobs/deleteBlobVersion/action** | Propietario de datos de blobs de almacenamiento |

### <a name="shared-access-signature-sas-parameters"></a>Parámetros de la Firma de acceso compartido (SAS)

El recurso firmado de una versión de blob es `bv`. Para obtener más información, vea [Creación de un SAS de servicio](/rest/api/storageservices/create-service-sas) o [Creación de un SAS de delegación de usuario](/rest/api/storageservices/create-user-delegation-sas).

En la tabla siguiente se muestra el permiso necesario en un SAS para eliminar una versión de un blob.

| **Permiso** | **Símbolo de URI** | **Operaciones permitidas** |
|----------------|----------------|------------------------|
| Eliminar         | x              | Eliminar una versión de un blob |

## <a name="pricing-and-billing"></a>Precios y facturación

La habilitación del control de versiones de blobs puede dar lugar a cargos de almacenamiento de datos adicionales en su cuenta. Al diseñar una aplicación, es importante tener en cuenta cómo se pueden acumular estos gastos para que pueda de minimizar los costos.

Las versiones de blobs, como las instantáneas de blobs, se facturan con la misma tarifa que los datos activos. La forma en que se facturan las versiones depende de si ha establecido explícitamente el nivel para el blob de base o para cualquiera de sus versiones (o instantáneas). Para más información sobre los niveles de blobs, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

Si no ha cambiado el nivel de un blob o de una versión, se le facturarán los bloques únicos de datos en ese blob, sus versiones y las instantáneas que pueda tener. Para más información, consulte [Facturación cuando el nivel de blob no se ha establecido explícitamente](#billing-when-the-blob-tier-has-not-been-explicitly-set).

Si ha cambiado el nivel de un blob o de una versión, se le facturará por todo el objeto, independientemente de que el blob y la versión estén en el mismo nivel de nuevo. Para más información, consulte [Facturación cuando el nivel de blob se ha establecido explícitamente](#billing-when-the-blob-tier-has-been-explicitly-set).

> [!NOTE]
> Habilitar el control de versiones de los datos que se sobrescriben con frecuencia puede provocar un aumento de los cargos por capacidad de almacenamiento y una mayor latencia durante las operaciones de enumeración. A fin de mitigar estos problemas, almacene los datos que se sobrescriben con frecuencia en una cuenta de almacenamiento independiente con el control de versiones deshabilitado.

Para más información sobre los detalles de facturación de las instantáneas de blobs, consulte [Instantáneas de blob](snapshots-overview.md).

### <a name="billing-when-the-blob-tier-has-not-been-explicitly-set"></a>Facturación cuando el nivel de blob no se ha establecido explícitamente

Si no ha establecido explícitamente el nivel del blob para un blob base o cualquiera de sus versiones, se le cobrará por bloques o páginas únicos en el blob, sus versiones y las instantáneas que pueda tener. Los datos que se comparten entre un blob y sus versiones se cobran solo una vez. Cuando se actualiza un blob, los datos de un blob base difieren de los datos almacenados en sus versiones y se cobran los datos únicos por bloque o página.

Al reemplazar un bloque en un blob en bloques, ese bloque será considerado como un bloque único a la hora de aplicar cargos. Esto es así aunque el bloque tenga el mismo identificador de bloque y los mismos datos que tiene en la versión anterior. Cuando se vuelva a confirmar el bloque, diferirá de su equivalente en la versión anterior y se le aplicarán cargos por sus datos. Lo mismo sucede con una página de un blob en páginas que se haya actualizado con datos idénticos.

Blob Storage no dispone de medios para determinar si dos bloques contienen datos idénticos. Cada bloque que se carga y confirma se trata como único, incluso si tiene los mismos datos y el mismo identificador de bloque. Dado que las cargas se incrementan con los bloques únicos, es importante considerar que, cuando se actualiza un blob con el control de versiones habilitado, se generarán más bloques únicos y se aplicarán cargos adicionales.

Cuando el control de versiones de blobs esté habilitado, diseñe las operaciones de actualización en blobs en bloques para que actualicen el menor número posible de bloques. Las operaciones de escritura que permiten un control más exhaustivo de los bloques son [Put Block](/rest/api/storageservices/put-block) y [Put Block List](/rest/api/storageservices/put-block-list). Por otro lado, la operación [Put Blob](/rest/api/storageservices/put-blob) reemplaza todo el contenido de un blob, lo que puede dar lugar a cargos adicionales.

En los siguientes escenarios se muestra cómo se incrementan los cargos para un blob en bloques y sus versiones cuando el nivel del blob no se ha establecido explícitamente.

#### <a name="scenario-1"></a>Escenario 1

En el escenario 1, el blob tiene una versión anterior. Este no se ha actualizado desde que se creó la versión, así que se generarán cargos únicamente para los bloques únicos 1, 2 y 3.

![Diagrama 1 que muestra la facturación de los bloques únicos en el blob base y la versión anterior.](./media/versioning-overview/versions-billing-scenario-1.png)

#### <a name="scenario-2"></a>Escenario 2

En el escenario 2, se ha actualizado un bloque (bloque 3 en el diagrama) en el blob. Aunque dicho bloque actualizado contiene los mismos datos y el mismo identificador, no es igual que el bloque 3 de la versión anterior. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Diagrama 2 que muestra la facturación de los bloques únicos en el blob base y la versión anterior.](./media/versioning-overview/versions-billing-scenario-2.png)

#### <a name="scenario-3"></a>Escenario 3

En el escenario 3, el blob se ha actualizado, pero la versión no. El bloque 3 se reemplazó por el bloque 4 en el blob base, pero la versión anterior sigue reflejando el bloque 3. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Diagrama 3 que muestra la facturación de los bloques únicos en el blob base y la versión anterior.](./media/versioning-overview/versions-billing-scenario-3.png)

#### <a name="scenario-4"></a>Escenario 4

En la situación 4, el blob de base se ha actualizado totalmente y no contiene ninguno de los bloques originales. Como resultado, los cargos de la cuenta se aplicarán por los ocho bloques únicos, cuatro de los cuales se encuentran en el blob base y, los demás, en la versión anterior. Este escenario puede producirse si escribe en un blob con la operación [Put Blob](/rest/api/storageservices/put-blob), ya que reemplaza todo el contenido del blob base.

![Diagrama 4 que muestra la facturación de los bloques únicos en el blob base y la versión anterior.](./media/versioning-overview/versions-billing-scenario-4.png)

### <a name="billing-when-the-blob-tier-has-been-explicitly-set"></a>Facturación cuando el nivel de blob se ha establecido explícitamente

Si ha establecido explícitamente el nivel de blob para un blob o una versión (o instantánea), se le cobrará la longitud completa del contenido del objeto en el nuevo nivel, con independencia de si comparte bloques con un objeto en el nivel original. También se le cobrará por la longitud de contenido completo de la versión más antigua en el nivel original. Las demás versiones o instantáneas anteriores que permanecen en el nivel original se cobran por los bloques únicos que pueden compartir, tal como se describe en [Facturación cuando el nivel de blob no se ha establecido explícitamente](#billing-when-the-blob-tier-has-not-been-explicitly-set).

#### <a name="moving-a-blob-to-a-new-tier"></a>Traslado de un blob a un nuevo nivel

En la tabla siguiente se describe el comportamiento de facturación de un blob o una versión cuando se mueve a un nuevo nivel.

| Cuando el nivel de blob se establece explícitamente en... | A continuación, se le facturará por... |
|-|-|
| Un blob base con una versión anterior | El blob base en el nuevo nivel y la versión más antigua del nivel original, además de los bloques únicos de otras versiones.<sup>1</sup> |
| Un blob base con una versión anterior y una instantánea | El blob base en el nuevo nivel, la versión más antigua en el nivel original y la instantánea más antigua en el nivel original, además de los bloques únicos en otras versiones o instantáneas<sup>1</sup>. |
| Una versión anterior | La versión del nuevo nivel y el blob base en el nivel original, además de los bloques únicos de otras versiones.<sup>1</sup> |

<sup>1</sup>Si hay otras versiones o instantáneas anteriores que no se han migrado desde su nivel original, esas versiones o instantáneas se cobran en función del número de bloques únicos que contienen, tal como se describe en [Facturación cuando el nivel del blob no se ha establecido explícitamente](#billing-when-the-blob-tier-has-not-been-explicitly-set).

En el diagrama siguiente se muestra cómo se facturan los objetos cuando un blob con versión se mueve a un nivel diferente.

:::image type="content" source="media/versioning-overview/versioning-billing-tiers.png" alt-text="Diagrama que muestra cómo se facturan los objetos cuando un blob con versión está explícitamente organizado en niveles.":::

No se puede deshacer si se establece explícitamente el nivel de un blob, una versión o una instantánea. Si mueve un blob a un nuevo nivel y luego lo vuelve a su nivel original, se le cobrará la longitud completa del contenido del objeto incluso si comparte bloques con otros objetos en el nivel original.

Las operaciones que establecen explícitamente el nivel de un blob, una versión o una instantánea incluyen:

- [Set Blob Tier](/rest/api/storageservices/set-blob-tier)
- [Put Blob](/rest/api/storageservices/put-blob) con el nivel especificado
- [Put Block List](/rest/api/storageservices/put-block-list) con el nivel especificado
- [Copy Blob](/rest/api/storageservices/copy-blob) con el nivel especificado

#### <a name="deleting-a-blob-when-soft-delete-is-enabled"></a>Eliminación de un blob cuando está habilitada la eliminación temporal

Cuando la eliminación temporal del blob está habilitada, si elimina o sobrescribe un blob base que se ha establecido explícitamente en el nivel, las versiones anteriores del blob eliminado temporalmente se facturan con la longitud del contenido completo. Para obtener más información sobre el uso conjunto del control de versiones de blobs y la eliminación temporal, consulte [Control de versiones de blobs y eliminación temporal](#blob-versioning-and-soft-delete).

En la tabla siguiente se describe el comportamiento de facturación de un blob que se elimina temporalmente, en función de si el control de versiones está habilitado o deshabilitado. Cuando se habilita el control de versiones, se crea una versión cuando un blob se elimina temporalmente. Cuando se deshabilita el control de versiones, la eliminación temporal de un blob crea una instantánea de eliminación temporal.

| Cuando sobrescribe un blob base con su nivel establecido explícitamente... | A continuación, se le facturará por... |
|-|-|
| Si la eliminación temporal de blobs y el control de versiones están habilitados | Todas las versiones existentes en la longitud del contenido completo independientemente del nivel. |
| Si la eliminación temporal de blobs está habilitada, pero el control de versiones está deshabilitado | Todas las instantáneas de eliminación temporal existentes con una longitud de contenido completo independientemente del nivel. |

## <a name="see-also"></a>Consulte también

- [Habilitar y administrar las versiones de blob](versioning-enable.md)
- [Creación de una instantánea de un blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob)
- [Eliminación temporal de blobs de Azure Storage](./soft-delete-blob-overview.md)