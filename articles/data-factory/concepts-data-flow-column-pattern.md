---
title: Patrones de columna en el flujo de datos de asignación de Azure Data Factory
description: Cree patrones generalizados de transformación de datos mediante patrones de columna en flujos de datos de asignación de Azure Data Factory
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/21/2021
ms.openlocfilehash: aca49982cf6e25e95002c1fab40b46ef05c842e2
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110454949"
---
# <a name="using-column-patterns-in-mapping-data-flow"></a>Uso de patrones de columnas en el flujo de datos de asignación

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Varias transformaciones de flujo de datos de asignación permiten hacer referencia a columnas de plantilla en función de patrones en lugar de nombres de columna codificados de forma rígida. Esta coincidencia se conoce como *patrones de columna*. Puede definir patrones para buscar la coincidencia de columnas según el nombre, el tipo de datos, la secuencia, el origen o la posición, en lugar de requerir nombres de campo exactos. Hay dos escenarios en los que resultan útiles los patrones de columna:

* Si los campos de origen de entrada cambian a menudo, como en el caso de columnas que cambian en los archivos de texto o bases de datos NoSQL. Este escenario se conoce como [desfase de esquema](concepts-data-flow-schema-drift.md).
* Si desea realizar una operación común en un grupo grande de columnas. Por ejemplo, si desea convertir cada columna que contiene "total" en el nombre de columna en un valor doble.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4Iui1]

## <a name="column-patterns-in-derived-column-and-aggregate"></a>Patrones de columna en columna derivada y de agregado

Para agregar un patrón de columna a una columna derivada, de agregado o de transformación de ventana, haga clic en **Agregar** encima de la lista de columnas o el icono del signo más junto a una columna derivada existente. Seleccione **Add column pattern** (Agregar patrón de columna).

![Captura de pantalla que muestra el icono de suma para agregar el patrón de columna](media/data-flow/add-column-pattern.png "Patrones de columnas")

Use el [generador de expresiones](concepts-data-flow-expression-builder.md) para escribir la condición de coincidencia. Cree una expresión booleana que busque la coincidencia con las columnas en función de los elementos `name`, `type`, `stream`, `origin` y `position` de la columna. El patrón afectará a cualquier columna, desfasada o definida, donde la condición devuelva true.


![Captura de pantalla que muestra la pestaña Derived column's settings (Configuración de la columna derivada).](media/data-flow/edit-column-pattern.png "Patrones de columnas")

El patrón de columna anterior coincide con cada columna de tipo de datos doble y crea una columna derivada por coincidencia. Al indicar `$$` como campo de nombre de columna, todas las columnas coincidentes se actualizan con el mismo nombre. El valor de cada columna es el valor existente redondeado a dos decimales.

Para comprobar que la condición de coincidencia es correcta, puede validar el esquema de salida de las columnas definidas en la pestaña **Inspeccionar** u obtener una instantánea de los datos en la pestaña **Vista previa de los datos**. 

![Captura de pantalla que muestra la pestaña Output schema (Esquema de salida).](media/data-flow/columnpattern3.png "Patrones de columnas")

### <a name="hierarchical-pattern-matching"></a>Coincidencia de patrones jerárquica

También puede crear una coincidencia de patrones dentro en estructuras jerárquicas complejas. Expanda la sección `Each MoviesStruct that matches` en la que se le solicitará cada jerarquía en el flujo de datos. Después, puede crear patrones de coincidencia para las propiedades dentro de la jerarquía elegida.

![La captura de pantalla muestra el patrón de columna en jerarquías.](media/data-flow/patterns-hierarchy.png "Patrones de columnas en jerarquías")

## <a name="rule-based-mapping-in-select-and-sink"></a>Asignación basada en reglas en selección y receptor

Cuando asigna columnas en transformaciones de origen y selección, puede agregar asignación fija o asignación basada en reglas. Busque la coincidencia según los elementos `name`, `type`, `stream`, `origin` y `position` de las columnas. Puede usar cualquier combinación de asignaciones basadas en reglas y fijas. De forma predeterminada, todas las proyecciones con más de 50 columnas tendrán como valor predeterminado una asignación basada en reglas que coincida con todas las columnas y que genere el nombre insertado. 

Para agregar una asignación basada en reglas, haga clic en **Agregar asignación** y seleccione **Rule based mapping** (Asignación basada en reglas).

![Captura de pantalla que muestra la asignación basada en reglas seleccionada en Agregar asignación.](media/data-flow/rule2.png "Asignación basada en reglas")

Cada asignación basada en reglas requiere dos entradas: la condición por la que buscar coincidencias y el nombre de cada columna asignada. Ambos valores se insertaron a través del [generador de expresiones](concepts-data-flow-expression-builder.md). En el cuadro de expresión de la izquierda, escriba la condición de coincidencia booleana. En el cuadro de expresión de la derecha, especifique a qué se asignará la columna coincidente.

![Captura de pantalla que muestra una asignación.](media/data-flow/rule-based-mapping.png "Asignación basada en reglas")

Use la sintaxis de `$$` para hacer referencia al nombre de entrada de una columna coincidente. Utilizando la imagen anterior como ejemplo, supongamos que un usuario desea buscar coincidencias en todas las columnas de cadena cuyos nombres tengan más de 6 caracteres. Si una columna de entrada se denomina `test`, la expresión `$$ + '_short'` cambiará el nombre de la columna `test_short`. Si esta es la única asignación que existe, todas las columnas que no cumplan la condición se quitarán de los datos de salida.

Los patrones coinciden con las columnas desfasadas y definidas. Para ver qué columnas definidas están asignadas mediante una regla, haga clic en el icono de las gafas junto a la regla. Compruebe la salida mediante la vista previa de los datos.

### <a name="regex-mapping"></a>Asignación de expresión regular

Si hace clic en el icono del botón de contenido adicional hacia abajo, puede especificar una condición de asignación de regex. Una condición de asignación de regex coincide con todos los nombres de columna que coinciden con la condición regex especificada. Se puede usar en combinación con las asignaciones estándar basadas en reglas.

![Captura de pantalla que muestra la condición de asignación de regex con Hierarchy level (Nivel de jerarquía) y Name matches (Coincidencias de nombres).](media/data-flow/regex-matching.png "Asignación basada en reglas")

El ejemplo anterior coincide con el patrón regex `(r)` o cualquier nombre de columna que contenga un "r" en minúscula. De forma similar a la asignación basada en reglas estándar, todas las columnas coincidentes se modifican por la condición de la derecha con `$$` sintaxis.

### <a name="rule-based-hierarchies"></a>Jerarquías basadas en reglas

Si la proyección definida tiene una jerarquía, puede usar la asignación basada en reglas para asignar las subcolumnas de las jerarquías. Especifique una condición de coincidencia y la columna compleja cuyas subcolumnas desee asignar. Todas las subcolumnas coincidentes se enviarán con la regla para asignar un nombre de salida especificada a la derecha.

![Captura de pantalla que muestra una asignación basada en reglas usando una jerarquía.](media/data-flow/rule-based-hierarchy.png "Asignación basada en reglas")

En el ejemplo anterior se hace coincidir con todas las subcolumnas de la columna compleja `a`. `a` contiene dos subcolumnas `b` y `c`. El esquema de salida incluirá dos columnas `b` y `c`, ya que la condición para asignar un nombre de salida es `$$`.

## <a name="pattern-matching-expression-values"></a>Valores de expresión de coincidencia de patrones.

* `$$` se traduce al nombre o valor de cada coincidencia en tiempo de ejecución. Piense en `$$` como equivalente a `this`.
* `name` representa el nombre de cada columna de entrada
* `type` representa el tipo de datos de cada columna de entrada. La lista de tipos de datos del sistema de tipos de flujo de datos se puede encontrar [aquí.](concepts-data-flow-overview.md#data-flow-data-types)
* `stream` representa el nombre asociado a cada secuencia o transformación del flujo
* `position` es la posición ordinal de las columnas en el flujo de datos
* `origin` es la transformación en la que se ha originado o se ha actualizado por última vez una columna

## <a name="next-steps"></a>Pasos siguientes
* Obtenga más información sobre el [lenguaje de expresiones](data-flow-expression-functions.md) del flujo de datos de asignación para las transformaciones de datos.
* Uso de patrones de columnas en la [transformación de receptor](data-flow-sink.md) y en la [transformación de selección](data-flow-select.md) con asignación basada en reglas.
