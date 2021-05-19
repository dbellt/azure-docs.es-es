---
title: Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)
description: Aprenda a usar etiquetas de índice de blobs para categorizar, administrar y consultar objetos de blobs.
author: twooley
ms.author: twooley
ms.date: 03/18/2021
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.reviewer: klaasl
ms.custom: references_regions
ms.openlocfilehash: 85aad5435add7ca32de4cf6fcb91944431ab5506
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685178"
---
# <a name="manage-and-find-azure-blob-data-with-blob-index-tags-preview"></a>Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)

A medida que los conjuntos de datos crecen, buscar un objeto específico en un mar de datos puede resultar difícil. Las etiquetas de índice de blob proporcionan funcionalidades de administración y detección de datos mediante atributos de etiquetas de índice de clave-valor. Puede clasificar y buscar objetos en un único contenedor o en todos los contenedores de su cuenta de almacenamiento. A medida que cambian los requisitos de datos, los objetos se pueden clasificar dinámicamente mediante la actualización de sus etiquetas de índice. Los objetos pueden permanecer en contexto con su organización de contenedores actual.

Las etiquetas de índice de blobs le permiten:

- Categorizar dinámicamente los blobs mediante etiquetas de índice de clave-valor
- Buscar rápidamente de blobs etiquetados específicos en toda una cuenta de almacenamiento
- Especificar comportamientos condicionales para las API de blobs en función de la evaluación de etiquetas de índice
- Usar etiquetas de índice para controles avanzados de las características, como la [administración del ciclo de vida de los blobs](storage-lifecycle-management-concepts.md)

Considere un escenario en el que tiene millones de blobs en su cuenta de almacenamiento, donde son muchas aplicaciones diferentes las que acceden a ellos. Quiere buscar todos los datos relacionados de un único proyecto. No está seguro del ámbito exacto, ya que los datos pueden estar distribuidos entre varios contenedores con distintas convenciones de nomenclatura. Sin embargo, las aplicaciones cargan todos los datos con etiquetas basadas en el proyecto. En lugar de buscar a través de millones de blobs y comparar nombres y propiedades, puede usar `Project = Contoso` como criterio de detección. El índice de blobs filtrará todos los contenedores en toda la cuenta de almacenamiento para buscar y devolver rápidamente solo el conjunto de 50 blobs de `Project = Contoso`.

> [!IMPORTANT]
> Actualmente, las etiquetas de índice de blob se encuentran en **versión preliminar** y están disponibles en todas las regiones públicas. Consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

Para empezar a trabajar con ejemplos sobre cómo usar el índice de blobs, consulte [Uso de etiquetas de índice de blobs para administrar y buscar datos](storage-blob-index-how-to.md).

## <a name="blob-index-tags-and-data-management"></a>Etiquetas y administración de datos del índice de blobs

Los prefijos de nombre de contenedores y blobs son una categorización unidimensional. Las etiquetas de índice de blobs permiten la categorización multidimensional para [tipos de datos de blobs (bloque, anexo o página)](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs). Azure Blob Storage indexa la categorización multidimensional de forma nativa para que usted pueda encontrar los datos rápidamente.

Considere los siguientes cinco blobs en su cuenta de almacenamiento:

- *container1/transaction.csv*
- *container2/campaign.docx*
- *photos/bannerphoto.png*
- *archives/completed/2019review.pdf*
- *logs/2020/01/01/logfile.txt*

Estos blobs se separan mediante un prefijo de *contenedor/carpeta virtual/nombre de blob*. Puede establecer un atributo de etiqueta de índice de `Project = Contoso` en estos cinco blobs para categorizarlos juntos, a la vez que mantiene su organización de prefijos actual. Agregar etiquetas de índice elimina la necesidad de mover los datos al exponer la capacidad de filtrar y buscar datos mediante el índice.

## <a name="setting-blob-index-tags"></a>Configuración de las etiquetas del índice de blobs

Las etiquetas del índice de blobs son atributos de clave-valor que se pueden aplicar a objetos nuevos o existentes dentro de la cuenta de almacenamiento. Puede especificar etiquetas de índice durante el proceso de carga mediante las operaciones [Put Blob](/rest/api/storageservices/put-blob), [Put Block List](/rest/api/storageservices/put-block-list) o [Copy Blob](/rest/api/storageservices/copy-blob) y el encabezado `x-ms-tags` opcional. Si ya tiene blobs en la cuenta de almacenamiento, llame a [SetBlobTags](/rest/api/storageservices/set-blob-tags) pasando un documento XML con formato con las etiquetas de índice en el cuerpo de la solicitud.

> [!IMPORTANT]
> El [propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner), así como cualquier persona con una firma de acceso compartido que tenga permiso para acceder a las etiquetas de blobs (el permiso `t` de SAS), puede establecer las etiquetas de índice de blobs.
>
> Además, los usuarios de RBAC con el permiso `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` pueden realizar esta operación.

Puede aplicar una única etiqueta al blob para describir cuándo se han terminado de procesar los datos.

> "processedDate" = '2020-01-01'

Puede aplicar varias etiquetas en el blob para una mayor descripción de los datos.

> "Project" = 'Contoso'  
> "Classified" = 'True'  
> "Status" = 'Unprocessed'  
> "Priority" = '01'

Para modificar los atributos existentes de las etiquetas de índice, recupere los atributos existentes las etiquetas, modifique los atributos de las etiquetas y reemplace por la operación [Set Blob Tags](/rest/api/storageservices/set-blob-tags). Para quitar todas las etiquetas de índice del blob, llame a la operación `Set Blob Tags` sin especificar atributos de etiqueta. Como las etiquetas de índice de blobs son un subrecurso del contenido de los datos del blob, `Set Blob Tags` no modifica ningún contenido subyacente y no cambia la hora de la última modificación ni ETag del blob. Puede crear o modificar las etiquetas de índice para todos los blobs base actuales y versiones anteriores. Sin embargo, las etiquetas de las instantáneas o los blobs eliminados temporalmente no se pueden modificar.

Los límites siguientes se aplican a las etiquetas del índice de blobs:

- Cada blob puede tener hasta 10 etiquetas de índice del blob
- Las claves de etiqueta deben tener entre 1 y 128 caracteres.
- Los valores de etiqueta deben tener entre 0 y 256 caracteres.
- Las claves y los valores de etiqueta no distinguen entre mayúsculas y minúsculas.
- Las claves y los valores de etiqueta solo admiten tipos de datos de cadena. Los números, las fechas, las horas o los caracteres especiales se guardan como cadenas.
- Las claves y los valores de etiqueta deben cumplir las siguientes reglas de nomenclatura:
  - Caracteres alfanuméricos:
    - **a** a la **z** (letras minúsculas)
    - **A** a la **Z** (letras mayúsculas)
    - **0** a **9** (números)
  - Caracteres especiales válidos: espacio, más, menos, punto, dos puntos, igual, guion bajo, barra diagonal (` +-.:=_/`)

## <a name="getting-and-listing-blob-index-tags"></a>Obtención y enumeración de etiquetas de índice de blobs

Las etiquetas de índice de blobs se almacenan como subrecurso junto con los datos de blobs y se pueden recuperar independientemente del contenido subyacente de los datos del blob. Las etiquetas de índice de blobs para un único blob se pueden recuperar con la operación [Get Blob Tags](/rest/api/storageservices/get-blob-tags). La operación [List Blobs](/rest/api/storageservices/list-blobs) con el parámetro `include:tags` también devolverá todos los blobs de un contenedor, junto con las etiquetas de índice de blobs.

> [!IMPORTANT]
> El [propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner), así como cualquier persona con una firma de acceso compartido que tenga permiso para acceder a las etiquetas de blobs (el permiso `t` de SAS), puede obtener y enumerar las etiquetas de índice de blobs.
>
> Además, los usuarios de RBAC con el permiso `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` pueden realizar esta operación.

En el caso de los blobs con al menos una etiqueta de índice de blob, se devuelve `x-ms-tag-count` en las operaciones [List Blobs](/rest/api/storageservices/list-blobs), [Get Blob](/rest/api/storageservices/get-blob) y [Get Blob Properties](/rest/api/storageservices/get-blob-properties), lo que indica que el recuento de etiquetas de índice del blob.

## <a name="finding-data-using-blob-index-tags"></a>Búsqueda de datos mediante etiquetas de índice de blobs

El motor de indexación expone esos atributos de clave-valor en un índice multidimensional. Después de establecer las etiquetas de índice, existen en el blob y se pueden recuperar de inmediato. Puede tardar algún tiempo antes de que se actualice el índice de blobs. Después de actualizado el índice de blobs, puede aprovechar las funcionalidades nativas de consulta y detección que ofrece Blob Storage.

La operación [Find Blobs By Tags](/rest/api/storageservices/find-blobs-by-tags) le permite obtener un conjunto filtrado de blobs cuyas etiquetas de índice coinciden con una expresión de consulta determinada. `Find Blobs by Tags` admite el filtrado entre todos los contenedores de la cuenta de almacenamiento, o puede limitar el filtrado a un único contenedor. Dado que todas las claves y valores de las etiquetas de índice son cadenas, los operadores relacionales usan una ordenación lexicográfica.

> [!IMPORTANT]
> El [propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner), así como cualquier persona con una firma de acceso compartido que tenga permiso para buscar los blobs por etiquetas (el permiso `f` de SAS), puede buscar datos mediante etiquetas de índice de blobs.
>
> Además, los usuarios de RBAC con el permiso `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action` pueden realizar esta operación.

Los criterios siguientes se aplican al filtrado de índices de blobs:

- Las claves de etiqueta deben ir entre comillas dobles (").
- Los valores de etiqueta y los nombres de contenedor deben ir entre comillas simples (').
- El carácter @ solo se permite para filtrar por nombre de un contenedor específico (por ejemplo, `@container = 'ContainerName'`).
- Los filtros se aplican con la ordenación lexicográfica a las cadenas.
- Las operaciones de intervalo del mismo lado en la misma clave no son válidas (por ejemplo, `"Rank" > '10' AND "Rank" >= '15'`).
- Al usar REST para crear una expresión de filtro, los caracteres deben estar codificados por URI.
- Las consultas de etiquetas están optimizadas para las coincidencias de igualdad de una sola etiqueta (por ejemplo, StoreID = "100").  Las consultas de intervalo que usan una sola etiqueta que incluye >, >=, <, <= también son eficaces. Cualquier consulta que use AND con más de una etiqueta no será tan eficaz.  Por ejemplo, Cost > "01" AND Cost <= "100" es eficaz. Cost > "01 AND StoreID = "2" no es tan eficaz.

En la tabla siguiente se muestran todos los operadores válidos para `Find Blobs by Tags`:

|  Operador  |  Descripción  | Ejemplo |
|------------|---------------|---------|
|     =      |     Igual     | `"Status" = 'In Progress'` |
|     >      |  Mayor que | `"Date" > '2018-06-18'` |
|     >=     |  Mayor o igual que | `"Priority" >= '5'` |
|     <      |  Menor que   | `"Age" < '32'` |
|     <=     |  Menor o igual que  | `"Company" <= 'Contoso'` |
|    y     |  Y lógico  | `"Rank" >= '010' AND "Rank" < '100'` |
| @container | Ámbito de un contenedor específico | `@container = 'videofiles' AND "status" = 'done'` |

> [!NOTE]
> Familiarícese con la ordenación lexicográfica al establecer y realizar consultas en etiquetas.
>
> - Los números se ordenan antes que las letras. Los números se ordenan en función del primer dígito.
> - Las mayúsculas se ordenan antes que las minúsculas.
> - Los símbolos no son estándar. Algunos símbolos se ordenan antes que los valores numéricos. Otros símbolos se ordenan antes o después que las letras.

## <a name="conditional-blob-operations-with-blob-index-tags"></a>Operaciones de blob condicionales con etiquetas de índice de blobs

En las versiones de REST 2019-10-10 y posteriores, la mayoría de las [API de Blob service](/rest/api/storageservices/operations-on-blobs) ahora admiten un encabezado condicional, `x-ms-if-tags`, de modo que la operación solo se completará correctamente si se cumple la condición especificada del índice de blobs. Si no se cumple la condición, obtendrá `error 412: The condition specified using HTTP conditional header(s) is not met`.

El encabezado `x-ms-if-tags` puede combinarse con los demás encabezados condicionales HTTP existentes (If-Match, If-None-Match, etc.). Si en una solicitud se proporcionan varios encabezados condicionales, todos deben evaluarse como verdaderos para que la operación se complete correctamente. Todos los encabezados condicionales se combinan eficazmente con AND lógico.

En la tabla siguiente se muestran los operadores válidos para las operaciones condicionales:

|  Operador  |  Descripción  | Ejemplo |
|------------|---------------|---------|
|     =      |     Igual     | `"Status" = 'In Progress'` |
|     <>     |   No igual a   | `"Status" <> 'Done'` |
|     >      |  Mayor que | `"Date" > '2018-06-18'` |
|     >=     |  Mayor o igual que | `"Priority" >= '5'` |
|     <      |  Menor que   | `"Age" < '32'` |
|     <=     |  Menor o igual que  | `"Company" <= 'Contoso'` |
|    y     |  Y lógico  | `"Rank" >= '010' AND "Rank" < '100'` |
|     O BIEN     | OR lógico   | `"Status" = 'Done' OR "Priority" >= '05'` |

> [!NOTE]
> Hay dos operadores adicionales, Not Equal y OR lógico, que se permiten en el encabezado `x-ms-if-tags` condicional para la operación de blobs, pero no existen en la operación `Find Blobs by Tags`.

## <a name="platform-integrations-with-blob-index-tags"></a>Integraciones de la plataforma con etiquetas de índice de blobs

Las etiquetas del Índice de blobs no solo le ayudan a categorizar, administrar y buscar en los datos de blobs, sino que también proporcionan integraciones con otras características de Blob Storage, como la [administración del ciclo de vida](storage-lifecycle-management-concepts.md).

### <a name="lifecycle-management"></a>Administración del ciclo de vida

Con `blobIndexMatch` como filtro de reglas en la administración del ciclo de vida, puede mover los datos a los niveles de acceso esporádico o eliminar los datos en función de las etiquetas de índice aplicadas a los blobs. Puede obtener una mayor granularidad en las reglas y solo mover o eliminar blobs si coinciden con los criterios de etiquetas especificados.

Puede establecer una coincidencia del índice de blobs como un conjunto de filtros independiente en una regla del ciclo de vida para aplicar acciones a los datos etiquetados. O bien, puede combinar tanto un prefijo como un índice de blobs para que coincidan con conjuntos de datos más específicos. Al especificar varios filtros en una regla de ciclo de vida se aplica una operación AND lógica. La acción solo se aplicará si coinciden *todos* los criterios de filtro.

El siguiente ejemplo de regla de administración del ciclo de vida se aplica a los blobs en bloques en un contenedor denominado *videofiles*. La regla segmenta los blobs al almacenamiento de archivos solo si los datos coinciden con los criterios de las etiquetas de índice de blobs de `"Status" == 'Processed' AND "Source" == 'RAW'`.

# <a name="portal"></a>[Portal](#tab/azure-portal)

![Ejemplo de regla de coincidencia del índice de blobs para la administración del ciclo de vida en Azure Portal](media/storage-blob-index-concepts/blob-index-lifecycle-management-example.png)

# <a name="json"></a>[JSON](#tab/json)

```json
{
    "rules": [
        {
            "enabled": true,
            "name": "ArchiveProcessedSourceVideos",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "tierToArchive": {
                            "daysAfterModificationGreaterThan": 0
                        }
                    }
                },
                "filters": {
                    "blobIndexMatch": [
                        {
                            "name": "Status",
                            "op": "==",
                            "value": "Processed"
                        },
                        {
                            "name": "Source",
                            "op": "==",
                            "value": "RAW"
                        }
                    ],
                    "blobTypes": [
                        "blockBlob"
                    ],
                    "prefixMatch": [
                        "videofiles/"
                    ]
                }
            }
        }
    ]
}
```

---

## <a name="permissions-and-authorization"></a>Permisos y autorización

Puede autorizar el acceso a las etiquetas de índice de blobs mediante uno de los métodos siguientes:

- Mediante un control de acceso basado en roles (RBAC) para conceder permisos a una entidad de seguridad de Azure Active Directory (Azure AD). Use Azure AD para mayor seguridad y facilidad de uso. Para obtener más información sobre el uso de Azure AD con las operaciones de blobs, consulte [Autorización del acceso a los blobs y las colas con Azure Active Directory](../common/storage-auth-aad.md).
- Mediante la firma de acceso compartido (SAS) para delegar el acceso al índice de blobs. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).
- Mediante claves de acceso de cuenta para autorizar las operaciones con Clave compartida. Para más información, consulte el artículo sobre la [Autorización con clave compartida](/rest/api/storageservices/authorize-with-shared-key).

Las etiquetas de índice de blobs son un subrecurso de los datos de blobs. Un usuario con permisos o un token de SAS para leer o escribir blobs puede no tener acceso a las etiquetas de índice de blobs.

### <a name="role-based-access-control"></a>Control de acceso basado en rol

A los autores de llamadas que usan una [identidad de Azure AD](../common/storage-auth-aad.md) se les pueden conceder los permisos siguientes para operar con etiquetas de índice de blobs.

| Operaciones de etiquetas de índice de blobs                                          | Acción de RBAC de Azure                                                             |
|--------------------------------------------------------------------|-------------------------------------------------------------------------------|
| [Establecer etiquetas de blobs](/rest/api/storageservices/set-blob-tags)           | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write    |
| [Obtener etiquetas de blobs](/rest/api/storageservices/get-blob-tags)           | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read     |
| [Buscar blobs por etiquetas](/rest/api/storageservices/find-blobs-by-tags) | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action |

Se requieren permisos adicionales independientes de los datos de blobs subyacentes para las operaciones con etiquetas de índice. El rol [propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) tiene permisos para las tres operaciones de etiquetas de índice de blobs. El [lector de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) solo tiene permisos para las operaciones `Find Blobs by Tags` y `Get Blob Tags`.

### <a name="sas-permissions"></a>Permisos de SAS

A los autores de llamadas que usan una [firma de acceso compartido (SAS)](../common/storage-sas-overview.md) se les puede conceder permisos limitados para operar con etiquetas de índice de blobs.

#### <a name="blob-sas"></a>SAS de blobs

Se pueden conceder los siguientes permisos en una SAS de blob para permitir el acceso a las etiquetas de índice de blobs. Los permisos de lectura y escritura de blobs por sí solos no son suficientes para permitir la lectura o escritura de sus etiquetas de índice.

| Permiso | Símbolo de URI | Operaciones permitidas                |
|------------|------------|-----------------------------------|
| Etiquetas de índice |     t      | Obtener y establecer etiquetas de índice para un blob |

#### <a name="container-sas"></a>SAS de contenedor

Se pueden conceder los siguientes permisos en una SAS de contenedor para permitir el filtrado de las etiquetas de blobs. El permiso `Blob List` no es suficiente para permitir el filtrado de blobs por sus etiquetas de índice.

| Permiso | Símbolo de URI | Operaciones permitidas         |
|------------|------------|----------------------------|
| Etiquetas de índice |     f      | Buscar blobs con etiquetas de índice |

## <a name="choosing-between-metadata-and-blob-index-tags"></a>Elección entre metadatos y etiquetas de índice de blobs

Tanto los metadatos como las etiquetas de índice de blobs ofrecen la capacidad de almacenar propiedades arbitrarias de clave-valor definidas por el usuario, junto con un recurso de blob. Ambos se pueden recuperar y establecer directamente, sin devolver ni modificar el contenido del blob. Es posible usar tanto los metadatos como las etiquetas de índice.

Solo las etiquetas de índice se indexan automáticamente y se pueden consultar mediante el servicio nativo de Blob Storage. Los metadatos no se pueden indexar ni consultar de forma nativa. Debe usar un servicio independiente, como [Azure Search](../../search/search-blob-ai-integration.md). Las etiquetas de índice de blobs tienen permisos adicionales de lectura, filtrado y escritura que son independientes de los datos del blob subyacentes. Los metadatos usan los mismos permisos que el blob y se devuelven como encabezados HTTP en las operaciones [Get Blob](/rest/api/storageservices/get-blob) y [Get Blob Properties](/rest/api/storageservices/get-blob-properties). Las etiquetas de índice de blobs se cifran en reposo mediante una [clave administradas por Microsoft](../common/storage-service-encryption.md). Los metadatos se cifran en reposo con la misma clave de cifrado especificada para los datos de blobs.

En la tabla siguiente se resumen las diferencias entre los metadatos y las etiquetas de índice de blobs:

|              |   Metadatos   |   Etiquetas de índice de blobs  |
|--------------|--------------|--------------------|
| **Límites**      | Sin límite numérico, 8 KB en total, no distingue mayúsculas de minúsculas | 10 etiquetas por blob máx., 768 bytes por etiqueta, distingue mayúsculas de minúsculas |
| **Actualizaciones**    | No se permiten en el nivel de archivo, `Set Blob Metadata` reemplaza todos los metadatos existentes, `Set Blob Metadata` cambia la hora de la última modificación del blob | Se permiten para los niveles de acceso, `Set Blob Tags` reemplaza todas las etiquetas existentes, `Set Blob Tags` no cambia la hora de la última modificación del blob |
| **Storage**     | Se almacenan con los datos del blob | Subrecurso de los datos de blobs |
| **Indexación y consulta** | Debe usar un servicio independiente, como Azure Search | Funcionalidades nativas de indexación y consulta integradas en Blob Storage |
| **Cifrado** | Cifrado en reposo con la misma clave de cifrado que se usa para los datos del blob | Cifrado en reposo con una clave de cifrado administrada por Microsoft |
| **Precios** | El tamaño de los metadatos se incluye en los costos de almacenamiento de un blob | Costo fijo por etiqueta de índice |
| **Respuesta del encabezado** | Metadatos devueltos como encabezados en `Get Blob` y `Get Blob Properties` | Recuento de etiquetas devuelto por `Get Blob` o `Get Blob Properties`, etiquetas devueltas solo por `Get Blob Tags` y `List Blobs` |
| **Permisos**  | Los permisos de lectura o escritura para los datos del blob se extienden a los metadatos | Se requieren permisos adicionales para leer, filtrar o escribir etiquetas de índice |
| **Nomenclatura** | Los nombres de metadatos deben cumplir las reglas de nomenclatura para los identificadores de C#. | Las etiquetas de índice de blobs admiten una mayor variedad de caracteres alfanuméricos. |

## <a name="pricing"></a>Precios

Los precios del índice de blobs se encuentran en versión preliminar pública y están sujetos a cambios para disponibilidad general. Se le cobrará por el número promedio mensual de etiquetas de índice en una cuenta de almacenamiento. El motor de indexación no tiene ningún costo. Las solicitudes para establecer etiquetas de blog, obtenerlas o buscarlas se cobran según las tarifas de transacción correspondientes actuales. Tenga en cuenta que el número de transacciones de lista consumidas al realizar una transacción para buscar blobs por etiqueta es igual al número de cláusulas de la solicitud. Por ejemplo, la consulta (StoreID = 100) es una transacción de lista.  La consulta (StoreID = 100 AND SKU = 10010) son dos transacciones de lista. Para obtener más información, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="regional-availability-and-storage-account-support"></a>Disponibilidad región y compatibilidad para cuentas de almacenamiento

Las etiquetas de índice de blobs solo están disponible en cuentas De uso general v2 (GPv2) con un espacio de nombres jerárquico (HNS) deshabilitado. No se admiten las cuentas De uso general (GPv1), pero puede actualizar cualquier cuenta GPv1 a una cuenta GPv2.

Las etiquetas de índice no se admiten en las cuentas de Premium Storage. Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).

Actualmente, las etiquetas de índice de blob están disponibles en todas las regiones públicas.

Para empezar, consulte [Uso de etiquetas de índice de blobs para administrar y buscar datos](storage-blob-index-how-to.md).

> [!IMPORTANT]
> Debe registrar la suscripción para poder usar el índice de blobs en versión preliminar en las cuentas de almacenamiento. Consulte la sección [Condiciones y problemas conocidos](#conditions-and-known-issues) de este artículo.

### <a name="register-your-subscription-preview"></a>Registro de la suscripción (versión preliminar)

Dado que las etiquetas de índice de blobs solo están en versión preliminar pública, deberá registrar su suscripción para poder usar la característica. Para enviar una solicitud, ejecute los siguientes comandos de PowerShell o de la CLI.

#### <a name="register-by-using-powershell"></a>Registro mediante PowerShell

```powershell
Register-AzProviderFeature -FeatureName BlobIndex -ProviderNamespace Microsoft.Storage
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

#### <a name="register-by-using-azure-cli"></a>Registro mediante la CLI de Azure

```azurecli
az feature register --namespace Microsoft.Storage --name BlobIndex
az provider register --namespace 'Microsoft.Storage'
```

## <a name="conditions-and-known-issues"></a>Condiciones y problemas conocidos

En esta sección se describen los problemas conocidos y las condiciones de la versión preliminar pública de las etiquetas de índice de blobs. Esta característica no debe usarse para cargas de trabajo de producción hasta que se lance la versión de disponibilidad general (GA), ya que el comportamiento puede cambiar.

- En el caso de la versión preliminar, primero debe registrar la suscripción para poder usar el índice de blobs en la cuenta de almacenamiento en las regiones de versión preliminar.
- Solo las cuentas GPv2 se admiten en la versión preliminar. Las cuentas de Blob, BlockBlobStorage y DataLake Gen2 habilitadas para HNS no son compatibles. No se admitirán las cuentas GPv1.
- La carga de blobs en páginas con etiquetas de índice no conserva las etiquetas. Establezca las etiquetas después de cargar un blob en páginas.
- Cuando el filtrado se limita a un único contenedor, `@container` solo se puede pasar si todas las etiquetas de índice de la expresión de filtro son comprobaciones de igualdad (clave=valor).
- Al usar el operador de intervalo con la condición `AND`, solo puede especificar el mismo nombre de clave de etiqueta de índice (`"Age" > '013' AND "Age" < '100'`).
- El control de versiones y el índice de blobs no se admiten. Las etiquetas de índice de blobs se conservan para las versiones, pero no se pasan al motor de índices de blobs.
- No hay ninguna API para determinar si las etiquetas de índice están indizadas.
- No se admite la conmutación por error de cuentas. Es posible que el índice de blobs no se actualice correctamente después de la conmutación por error.
- La administración del ciclo de vida solo admite comprobaciones de igualdad con coincidencia del índice de blobs.
- `Copy Blob` no copia las etiquetas de índice de blobs del blob de origen en el nuevo blob de destino. Puede especificar las etiquetas que desea que se apliquen al blob de destino durante la operación de copia.
- `Copy Blob` (copia asincrónica) de otra cuenta de almacenamiento con etiquetas aplicadas en el blob de destino hace que el motor de índices de blobs no devuelva el blob y sus etiquetas en el conjunto de filtros. Use `Copy Blob` de la dirección URL (copia sincrónica).
- Las etiquetas se conservan al crear la instantánea. Sin embargo, no se admite la promoción de una instantánea, ya que puede dar lugar a un conjunto de etiquetas vacío.

## <a name="faq"></a>Preguntas más frecuentes

**¿El índice de blobs puede ayudarme a filtrar y consultar el contenido de mis blobs?**

No, si tiene que buscar dentro de los datos de blobs, use la aceleración de consultas o Azure Search.

**¿Hay algún requisito en los valores de etiquetas de índice?**

Las etiquetas de índice de blobs solo admiten tipos de datos de cadena y las consultas devuelven resultados con ordenación lexicográfica. En el caso de los números, rellene el número con ceros. Para las fechas y horas, almacénelas como formato compatible con ISO 8601.

**¿Las etiquetas de índice de blobs y las etiquetas de Azure Resource Manager están relacionadas?**

No, las etiquetas de Resource Manager ayudan a organizar los recursos del plano de control, como suscripciones, grupos de recursos y cuentas de almacenamiento. Las etiquetas de índice permiten la administración y detección de blobs en el plano de datos.

## <a name="next-steps"></a>Pasos siguientes

Para ver un ejemplo de cómo se usa el índice de blobs, consulte [Uso del índice de blobs para administrar y buscar datos](storage-blob-index-how-to.md).

Obtenga información sobre la [administración del ciclo de vida](storage-lifecycle-management-concepts.md) y establezca una regla con coincidencia de índice de blobs.
