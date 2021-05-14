---
title: Cómo crear reglas de patrón de conjunto de recursos
description: Aprenda a crear una regla de patrón de recursos para anular cómo se agrupan los recursos en conjuntos
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 04/15/2021
ms.openlocfilehash: 61de2cf2e3ad9175d97378234d62f72ab3517b51
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587840"
---
# <a name="create-resource-set-pattern-rules"></a>Creación de reglas de patrón de conjunto de recursos

Los sistemas de procesamiento de datos a escala normalmente almacenan una sola tabla en un disco en varios archivos. Este concepto se representa en Azure Purview mediante conjuntos de recursos. Un conjunto de recursos es un objeto simple del catálogo de datos que representa una gran cantidad de recursos en almacenamiento. Para más información, consulte [Descripción de los conjuntos de recursos](concept-resource-sets.md).

Al analizar una cuenta de almacenamiento, Azure Purview usa un conjunto de patrones definidos para determinar si un grupo de recursos es un conjunto de recursos. En algunos casos, es posible que la agrupación de un conjunto de recursos de Azure Purview no refleje adecuadamente el estado de los datos. Las reglas de patrón del conjunto de recursos le permiten personalizar o invalidar la forma en que Azure Purview detecta qué recursos se agrupan como conjuntos de recursos, además de cómo se muestran dentro del catálogo.

Las reglas de patrón se admiten actualmente en los siguientes tipos de origen:
- Azure Data Lake Storage Gen2
- Azure Blob Storage
- Azure Files


## <a name="how-to-create-a-resource-set-pattern-rule"></a>Cómo crear una regla de patrón de conjunto de recursos

Siga los pasos que se indican a continuación para crear una nueva regla de patrón de conjunto de recursos:

1. Vaya al centro de administración. Seleccione **Pattern rules** (Reglas de patrón) en el menú bajo el encabezado de conjuntos de recursos. Para crear una regla, seleccione **+ New** (+ Nueva).

   :::image type="content" source="media/how-to-resource-set-pattern-rules/create-new-scoped-resource-set-rule.png" alt-text="Creación de reglas de patrón de conjunto de recursos" border="true":::

1. Introduzca el ámbito de la regla de patrón de conjunto de recursos. Seleccione el tipo y el nombre de la cuenta de almacenamiento en la que desea crear un conjunto de reglas. Cada conjunto de reglas se aplica en relación con el ámbito de una ruta de acceso de una carpeta especificada en el campo **Ruta de acceso de la carpeta**.

   :::image type="content" source="media/how-to-resource-set-pattern-rules/create-new-scoped-resource-set-scope.png" alt-text="Creación de configuraciones de reglas de patrón de conjunto de recursos" border="true":::

1. Para especificar una regla para un ámbito de configuración, seleccione **+ New Rule** (+ Nueva regla).

1. Escriba en los campos siguientes para crear una regla:

   1. **Nombre de regla:** el nombre de la regla de configuración. Este campo no tiene ningún efecto en los recursos a los que se aplica la regla.

   1. **Nombre completo:** una ruta de acceso completa que usa una combinación de texto, sustitutos dinámicos y sustitutos estáticos para hacer coincidir los recursos con la regla de configuración. Esta ruta de acceso está relacionada con el ámbito de la regla de configuración. Consulte la sección de [Sintaxis](#syntax) que aparece a continuación para obtener instrucciones detalladas sobre cómo especificar nombres completos.

   1. **Nombre para mostrar:** el nombre para mostrar del recurso. Este campo es opcional. Use texto sin formato y sustitutos estáticos para personalizar el modo en que se muestra un recurso en el catálogo. Para obtener instrucciones más detalladas, consulte la sección [Sintaxis](#syntax) que aparece a continuación.

   1. **Do not group as resource set** (No agrupar como conjunto de recursos): si está habilitado, el recurso coincidente no se agrupará en un conjunto de recursos.

      :::image type="content" source="media/how-to-resource-set-pattern-rules/scoped-resource-set-rule-example.png" alt-text="Creación de una nueva regla de configuración." border="true":::

1. Haga clic en **Agregar** para guardar la regla.

> [!NOTE]
> Después de crear una regla de patrón, todos los exámenes nuevos aplicarán la regla durante la ingesta. Los recursos existentes en el catálogo de datos se actualizarán a través de un proceso en segundo plano que puede tardar hasta varias horas. 

## <a name="pattern-rule-syntax"></a><a name="syntax"></a> Sintaxis de reglas de patrón

Al crear reglas de patrón de conjuntos de recursos, use la siguiente sintaxis para especificar a qué recursos se aplican las reglas.

### <a name="dynamic-replacers-single-brackets"></a>Sustitutos dinámicos (corchetes simples)

Los corchetes simples se usan como **sustitutos dinámicos** en las reglas de patrón. Para especificar un sustituto dinámico en el nombre completo, use el formato `{<replacerName:<replacerType>}`. Si coinciden, los sustitutos dinámicos se utilizan como una condición de agrupación que indica que los recursos se deben representar como un conjunto de recursos. Si los recursos se agrupan en un conjunto de recursos, la ruta de acceso completa del conjunto de recursos incluiría `{replacerName}` donde se especificaría el sustituto.

Por ejemplo, si dos recursos `folder1/file-1.csv` y `folder2/file-2.csv` coinciden con la regla `{folder:string}/file-{NUM:int}.csv`, el conjunto de recursos sería una sola entidad `{folder}/file-{NUM}.csv`.

#### <a name="special-case-dynamic-replacers-when-not-grouping-into-resource-set"></a>Caso especial: sustitutos dinámicos cuando no hay agrupación en un conjunto de recursos

Si la opción *Do not group as resource set* (No agrupar como conjunto de recursos) está habilitada para una regla de patrón, el nombre del sustituto es un campo opcional. `{:<replacerType>}` es una sintaxis válida. Por ejemplo, `file-{:int}.csv` puede coincidir correctamente con `file-1.csv` y `file-2.csv` y crear dos recursos diferentes en lugar de un conjunto de recursos.

### <a name="static-replacers-double-brackets"></a>Sustitutos estáticos (corchetes dobles)

Los corchetes dobles se usan como **sustitutos estáticos** en el nombre completo de una regla de patrón. Para especificar un sustituto estático en el nombre completo, use el formato `{{<replacerName>:<replacerType>}}`. Si coincide, cada conjunto de valores únicos del sustituto estático creará diferentes agrupaciones de conjuntos de recursos.

Por ejemplo, si dos recursos `folder1/file-1.csv` y `folder2/file-2.csv` coinciden con la regla `{{folder:string}}/file-{NUM:int}.csv`, se crearán dos conjuntos de recursos: `folder1/file-{NUM}.csv` y `folder2/file-{NUM}.csv`.

Los sustitutos estáticos se pueden usar para especificar el nombre para mostrar de un recurso que coincida con una regla de patrón. Si usa `{{<replacerName>}}` en el nombre para mostrar de una regla, se usará el valor coincidente en el nombre del recurso.

### <a name="available-replacement-types"></a>Tipos de sustitutos disponibles

A continuación se muestran los tipos disponibles que se pueden usar en los sustitutos estáticos y dinámicos:

| Tipo | Estructura |
| ---- | --------- |
| string | Una serie de uno o varios caracteres Unicode, incluidos delimitadores como los espacios. |
| int | Una serie de uno o varios caracteres ASCII del 0-9, se puede usar 0 como prefijo (por ejemplo, 0001). |
| guid | Una serie de 32 representaciones de cadenas (o 8-4-4-4-12) de un UUID como se definen en [RFC 4122](https://tools.ietf.org/html/rfc4122). |
| date | Una serie de 6 u 8 caracteres ASCII del 0-9 que, opcionalmente, puede incluir separadores: aaaammdd, aaaa-mm-dd, aammdd, aa-mm-dd, como se especifica en [RFC 3339](https://tools.ietf.org/html/rfc3339). |
| time | Una serie de 4 o 6 caracteres ASCII del 0-9 con separadores opcionales: HHmm, HH:mm, HHmmss, HH:mm:ss como se especifica en [RFC 3339](https://tools.ietf.org/html/rfc3339). |
| timestamp | Una serie de 12 o 14 caracteres ASCII del 0-9 con separadores opcionales: aaaa-mm-ddTHH:mm, aaaammddhhmm, aaaa-mm-ddTHH:mm:ss, aaaammddHHmmss como se especifica en [RFC 3339](https://tools.ietf.org/html/rfc3339). |
| boolean | Puede contener "true" o "false", sin distinción entre mayúsculas y minúsculas. |
| number | Una serie de 0 o más caracteres ASCII 0-9, puede tener un prefijo 0 (por ejemplo, 0001) seguido de un punto "." y una serie de 1 o más caracteres ASCII 0-9, puede ser un postfijo 0 (por ejemplo,. 100) |
| hex | Una serie de uno o varios caracteres ASCII del conjunto 0-1 y A-F, el valor puede tener el prefijo 0 |
| locale | Una cadena que coincide con la sintaxis especificada en [RFC 5646](https://tools.ietf.org/html/rfc5646). |

## <a name="order-of-resource-set-pattern-rules-getting-applied"></a>Orden de las reglas de patrón de conjunto de recursos que se aplican

A continuación se muestra el orden de las operaciones para aplicar las reglas de patrón:

1. Los ámbitos más específicos tendrán prioridad si un recurso coincide con dos reglas. Por ejemplo, las reglas de un ámbito `container/folder` se aplicarán antes que las reglas del ámbito `container`.

1. Orden de las reglas dentro de un ámbito específico. Esto se puede editar en la experiencia del usuario.

1. Si un recurso no coincide con ninguna regla especificada, se aplica la heurística predeterminada del conjunto de recursos.

## <a name="examples"></a>Ejemplos

### <a name="example-1"></a>Ejemplo 1

Extracción de datos de SAP en cargas completas y diferenciales

#### <a name="inputs"></a>Entradas

Archivos:

- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/13/saptable_customer_20200101_20200102_01.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/13/saptable_customer_20200101_20200102_02.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/delta/2020/01/15/saptable_customer_20200101_20200102_01.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/17/saptable_customer_20200101_20200102_01.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/17/saptable_customer_20200101_20200102_02.txt`

#### <a name="pattern-rule"></a>Regla de patrón

**Ámbito:** `https://myazureblob.blob.core.windows.net/bar/`

**Nombre para mostrar:** "Cliente externo"

**Nombre completo:** `customer/{extract:string}/{year:int}/{month:int}/{day:int}/saptable_customer_{date_from:date}_{date_to:time}_{sequence:int}.txt`

**Conjunto de recursos:** true

#### <a name="output"></a>Output

Un recurso del conjunto de recursos

**Nombre para mostrar:** Cliente externo

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/customer/{extract}/{year}/{month}/{day}/saptable_customer_{date_from}_{date_to}_{sequence}.txt`

### <a name="example-2"></a>Ejemplo 2

Datos de IoT en formato Avro

#### <a name="inputs"></a>Entradas

Archivos:

- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/02-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

#### <a name="pattern-rules"></a>Reglas de patrón

**Ámbito:** `https://myazureblob.blob.core.windows.net/bar/`

Regla 1

**Nombre para mostrar:** "machine-89"

**Nombre completo:** `raw/machinename-89/{date:date}/{time:time}-{id:int}.avro`

**Conjunto de recursos:** true

Regla 2

**Nombre para mostrar:** "machine-90"

**Nombre completo:** `raw/machinename-90/{date:date}/{time:time}-{id:int}.avro`

**Conjunto de recursos:** true

#### <a name="outputs"></a>Salidas

2 conjuntos de recursos

Conjunto de recursos 1

**Nombre para mostrar:** machine-89

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/{date}/{time}-{id}.avro`

Conjunto de recursos 2

**Nombre para mostrar:** machine-90

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/{date}/{time}-{id}.avro`

### <a name="example-3"></a>Ejemplo 3

Datos de IoT en formato Avro

#### <a name="inputs"></a>Entradas

Archivos:

- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`
- `https://myazureblob.blob.core.windows.netbar/raw/machinename-89/02-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

#### <a name="pattern-rule"></a>Regla de patrón

**Ámbito:** `https://myazureblob.blob.core.windows.net/bar/`

**Nombre para mostrar:** "Machine-{{machineid}}"

**Nombre completo:** `raw/machinename-{{machineid:int}}/{date:date}/{time:time}-{id:int}.avro`

**Conjunto de recursos:** true

#### <a name="outputs"></a>Salidas

Conjunto de recursos 1

**Nombre para mostrar:** machine-89

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/{date}/{time}-{id}.avro`

Conjunto de recursos 2

**Nombre para mostrar:** machine-90

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/{date}/{time}-{id}.avro`

### <a name="example-4"></a>Ejemplo 4

No agrupar en conjuntos de recursos

#### <a name="inputs"></a>Entradas

Archivos:

- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/02-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

#### <a name="pattern-rule"></a>Regla de patrón

**Ámbito:** `https://myazureblob.blob.core.windows.net/bar/`

**Nombre para mostrar:** `Machine-{{machineid}}`

**Nombre completo:** `raw/machinename-{{machineid:int}}/{{:date}}/{{:time}}-{{:int}}.avro`

**Conjunto de recursos:** false

#### <a name="outputs"></a>Salidas

4 recursos individuales

Recurso 1

**Nombre para mostrar:** machine-89

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`

Recurso 2

**Nombre para mostrar:** machine-89

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`

Recurso 3

**Nombre para mostrar:** machine-89

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/02-01-2020/22:33:22-001.avro`

Recurso 4

**Nombre para mostrar:** machine-90

**Nombre completo:** `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

## <a name="next-steps"></a>Pasos siguientes

Introducción al [registro y análisis de una cuenta de almacenamiento de Azure Data Lake Gen2](register-scan-adls-gen2.md).