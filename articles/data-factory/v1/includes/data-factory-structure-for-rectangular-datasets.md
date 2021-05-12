---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: 7bfca8b9e2894fa3c82457a96ad2d09731a6e01f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108741579"
---
## <a name="specifying-structure-definition-for-rectangular-datasets"></a>Especificación de la definición de la estructura de los conjuntos de datos rectangulares
La sección structure de JSON de los conjuntos de datos es una sección **opcional** para tablas rectangulares (con filas y columnas) y contiene una colección de columnas de la tabla. Se usará la sección structure para proporcionar información de tipo para las conversiones de tipos o para realizar asignaciones de columnas. En las siguientes secciones se describen estas configuraciones con más detalle. 

Cada columna contiene las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| name |Nombre de la columna. |Sí |
| type |Tipo de datos de la columna. Consulte la sección de conversiones de tipos siguiente para más detalles sobre cuándo debe especificar la información de tipo. |No |
| culture |Referencia cultural basada en .NET que se usará cuando se especifica el tipo y sea un tipo .NET Datetime o Datetimeoffset. El valor predeterminado es «en-us». |No |
| format |Cadena de formato que se usará cuando se especifica el tipo y sea un tipo .NET Datetime o Datetimeoffset. |No |

En el ejemplo siguiente se muestra el JSON de la sección structure de una tabla con tres columnas userid, name y lastlogindate.

```json
"structure": 
[
    { "name": "userid"},
    { "name": "name"},
    { "name": "lastlogindate"}
],
```

Utilice las siguientes directrices sobre cuándo incluir la información de la "estructura" y qué incluir en la sección **structure** .

* **Para los orígenes de datos estructurados** que almacenan el esquema de datos e información de tipo junto con los mismos datos (orígenes como SQL Server, Oracle, tabla de Azure etc.), debe especificar la sección "structure" solo si desea realizar una asignación de columna de las columnas de orígenes específicos en columnas específicas de receptor y sus nombres no son iguales (consulte los detalles en la siguiente sección de asignación de columna). 

    Como se ha mencionado antes, la información de tipo es opcional en la sección «structure». En los orígenes estructurados, ya está disponible la información de tipo como parte de la definición del conjunto de datos en el almacén de datos, por lo que no debería incluir la información de tipo cuando se incluye la sección "structure".
* **En cuanto al esquema en los orígenes de datos de lectura (específicamente, el blob de Azure)** , puede elegir guardar datos sin almacenar el esquema o la información de tipo con ellos. Para estos tipos de orígenes de datos debe incluir la sección "structure" en los dos casos siguientes:
  * Quiere realizar una asignación de columna.
  * Cuando el conjunto de datos es un origen en una actividad de copia, puede proporcionar la información de tipo en la «estructura» y Factoría de datos usará esta información de tipo para la conversión a tipos nativos para el receptor. Consulte el artículo [Mover datos a y desde el blob de Azure](../data-factory-azure-blob-connector.md) para obtener más información.

### <a name="supported-net-based-types"></a>Tipos basados en .NET admitidos
Factoría de datos admite los siguientes valores de tipo basados en .NET compatible con CLS para proporcionar información de tipo en la "estructura" del esquema en los orígenes de datos de lectura como blob de Azure.

* Int16
* Int32 
* Int64
* Single
* Double
* Decimal
* Byte[]
* Bool
* String 
* Guid
* Datetime
* Datetimeoffset
* TimeSpan 

Para Datetime y Datetimeoffset, también puede opcionalmente especificar la cadena "culture" y "format" para facilitar el análisis de la cadena de fecha y hora personalizada. Consulte el ejemplo de conversión de tipos siguiente.