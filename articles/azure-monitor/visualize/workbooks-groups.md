---
title: Grupos en los libros de Azure Monitor
description: Cómo usar grupos en los libros de Azure Monitor.
services: azure-monitor
author: lgayhardt
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: lagayhar
ms.openlocfilehash: b5a599a17a82a7f0798318c10d8dce61d9f39c6a
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108017380"
---
# <a name="how-to-use-groups-in-workbooks"></a>Uso de grupos en los libros

Un elemento de grupo en un libro le permite agrupar de forma lógica un conjunto de pasos en un libro.

Los grupos en los libros son útiles para varios fines:

- Diseño
  - En los escenarios en los que quiere que los elementos se organicen de manera vertical, puede crear un grupo de elementos que se apilarán y establecer que el estilo del grupo sea un ancho porcentual, en lugar de establecer un ancho porcentual en todos los elementos de forma individual.
- Visibilidad
  - En los escenarios en los que quiera ocultar o mostrar muchos elementos juntos, puede establecer la visibilidad de todo el grupo de elementos, en lugar de establecer la configuración de visibilidad en cada elemento individual. Esto puede ser útil en las plantillas que usan pestañas, ya que puede usar un grupo como contenido de la pestaña y todo el grupo se podrá ocultar o mostrar en función de un parámetro que haya establecido la pestaña seleccionada.
- Rendimiento
  - En los casos en los que tenga una plantilla de gran tamaño con muchas secciones o pestañas, puede convertir cada sección en su propia subplantilla y usar grupos para cargar todas las subplantillas dentro de la plantilla de nivel superior. El contenido de las subplantillas no se cargará ni se ejecutará hasta que un usuario haga visibles esos grupos. Obtenga más información sobre [cómo dividir una plantilla de gran tamaño en muchas plantillas](#how-to-split-a-large-template-into-many-templates).

## <a name="using-groups"></a>Uso de grupos

Para agregar un grupo al libro, seleccione *Agregar* y, después, *Agregar grupo*.

![Seleccione Agregar y después Agregar grupo.](./media/workbooks-groups/add-group.png)

A continuación se muestra una captura de pantalla de un grupo en modo de lectura con dos elementos: un elemento de texto y un elemento de consulta.  

![Grupo en modo de lectura.](./media/workbooks-groups/groups-view.png)

Al editar el libro, puede ver que ambos elementos en realidad están dentro de un elemento de grupo:

![Grupo en modo de edición. ](./media/workbooks-groups/groups-edit.png)

En la captura de pantalla anterior, el grupo está en el modo de edición, que muestra que contiene dos elementos (dentro del área de contorno discontinuo). Cada paso puede estar en modo de edición o de lectura, independientes entre sí. Por ejemplo, el paso de texto está en modo de edición, mientras que el paso de consulta está en modo de lectura.

## <a name="scoping"></a>Ámbito

En este momento, un grupo se trata como un nuevo ámbito en el libro. Los parámetros creados en el grupo solo son visibles dentro del grupo. Esto también se aplica a las combinaciones; solo puede ver los datos dentro de su grupo o en el nivel superior.

## <a name="group-types"></a>Tipos de grupo

El elemento Grupo de libro permite agregar un grupo de elementos a un libro. Como creador de un libro, especifique qué tipo de grupo será. Hay dos tipos de grupos:

- Modificable
  - El grupo del libro le permite agregar, quitar o editar el contenido de los elementos en el grupo. Se suele usar con fines de diseño y visibilidad.
- Desde una plantilla
  - El grupo del libro se carga a partir del contenido de otro libro por su identificador. El contenido de ese libro se carga y se combina en el libro en tiempo de ejecución. En el modo de edición, no se puede modificar el contenido del grupo, ya que solo se volverá a cargar desde la plantilla la próxima vez que se cargue el elemento. Al cargar un grupo desde una plantilla, use el identificador de recurso de Azure completo de un libro existente.

## <a name="load-types"></a>Tipos de carga

Existen varias maneras diferentes de cargar el contenido de un grupo. Como creador de un libro, puede especificar cuándo y cómo se cargará el contenido de un grupo.

### <a name="lazy-the-default"></a>Diferido (valor predeterminado)

El grupo solo se cargará cuando el elemento sea visible. Esto permite que los elementos de una pestaña usen un grupo. Si la pestaña nunca se selecciona, el grupo nunca se vuelve visible, por lo que el contenido no se carga.

En el caso de los grupos creados a partir de una plantilla, no se recupera el contenido de la plantilla y los elementos del grupo no se crean hasta que el grupo se vuelva visible. El usuario verá los indicadores de progreso para todo el grupo mientras se recupera el contenido.

### <a name="explicit"></a>Explícito

En este modo, se muestra un botón en la posible ubicación del grupo y no se recupera ni se crea ningún contenido hasta que el usuario hace clic explícitamente en el botón para cargar el contenido. Resulta útil en los escenarios en los que puede ser costoso procesar el contenido, o este se usa con poca frecuencia. El creador puede especificar el texto que aparecerá en el botón.

A continuación se muestra una captura de pantalla de la opción de carga explícita que muestra un botón "Load more" (Cargar más) configurado.

![Configuración de carga explícita](./media/workbooks-groups/groups-explicitly-loaded.png)

El grupo antes de cargarse en el libro:

![Grupo explícito antes de cargarse en el libro](./media/workbooks-groups/groups-explicitly-loaded-before.png)

El grupo después de cargarse en el libro:

![Grupo explícito después de cargarse en un libro](./media/workbooks-groups/groups-explicitly-loaded-after.png)

### <a name="always"></a>Siempre

En este modo, el contenido del grupo siempre se carga y se crea tan pronto como se carga el libro. Se usa con más frecuencia cuando se está usando un grupo solo con fines de diseño, en el que el contenido siempre estará visible.

## <a name="using-templates-inside-a-group"></a>Uso de plantillas dentro de un grupo

Cuando un grupo está configurado para cargarse a partir de una plantilla, el contenido se cargará de manera diferida de forma predeterminada y solo se cargará cuando el grupo sea visible.

Cuando se carga una plantilla en un grupo, el libro intenta combinar todos los parámetros declarados en la plantilla que se está cargando con los parámetros que ya existen en el grupo. Los parámetros que ya existan en el libro con nombres idénticos se combinarán con los de la plantilla que se está cargando. Si se combinan todos los parámetros de un paso de parámetro, este desaparecerá completo.

### <a name="example-1-all-parameters-have-identical-names"></a>Ejemplo 1: todos los parámetros tienen nombres idénticos

Imagine una plantilla con dos parámetros en el nivel superior.

- `TimeRange`: un parámetro de intervalo de tiempo.
- `Filter`: un parámetro de texto.

![Edición del elemento de parámetros: "parámetros de nivel superior"](./media/workbooks-groups/groups-top-level-params.png)

Después, un elemento de grupo carga una segunda plantilla que tiene dos parámetros propios y un paso de texto; en ella, los parámetros tienen el mismo nombre:

![Edición de un elemento de parámetro para la segunda plantilla](./media/workbooks-groups/groups-merged-away.png)

Cuando se carga la segunda plantilla en el grupo, se combinan los parámetros duplicados. Dado que todos los parámetros se combinan, el paso de los parámetros internos también se combina. como resultado, el grupo contendrá solo el paso de texto.

### <a name="example-2-one-parameter-has-an-identical-name"></a>Ejemplo 2: un parámetro tiene un nombre idéntico

Imagine una plantilla de grupo con dos parámetros en el nivel superior.

- `TimeRange`: un parámetro de intervalo de tiempo.
- `FilterB`: un parámetro de texto. Tenga en cuenta que no se trata de `Filter`, como en la plantilla anterior.

![Edición de un elemento de grupo con el resultado de la combinación de parámetros](./media/workbooks-groups/groups-wont-merge-away.png)

Cuando se cargue la plantilla de elemento del grupo, el parámetro `TimeRange` se combinará fuera del grupo. A continuación, el libro tendrá el paso de parámetros inicial con `TimeRange` y `Filter`, y el paso de parámetros del grupo solo incluirá `FilterB`

![resultado de los parámetros que no se combinarán](./media/workbooks-groups/groups-wont-merge-away-result.png)

Si la plantilla que se cargó hubiera contenido `TimeRange` y `Filter` (en lugar de `FilterB`), el libro resultante tendría un paso de parámetros y un grupo que solo incluiría el paso de texto.

## <a name="how-to-split-a-large-template-into-many-templates"></a>División de una plantilla de gran tamaño en muchas plantillas

Para mejorar el rendimiento, resulta ventajoso dividir una plantilla de gran tamaño en varias plantillas más pequeñas que cargan el contenido de forma diferida o a petición del usuario. De esta manera la carga inicial es más rápida, ya que la plantilla de nivel superior puede ser mucho más pequeña.

Al dividir una plantilla en partes, tendrá que dividir la plantilla en muchas plantillas (subplantillas) que funcionen de forma individual. Por tanto, si la plantilla de nivel superior tiene un parámetro `TimeRange` que otros pasos usan, la subplantilla también deberá incluir un paso de parámetros que defina un parámetro con el mismo nombre exacto. Así, las subplantillas podrán funcionar de forma independiente y podrán cargarse dentro de plantillas más grandes en grupos.

Para convertir una plantilla más grande en varias subplantillas, haga lo siguiente:

1.  Cree un nuevo grupo vacío cerca del principio del libro, después de los parámetros compartidos. Este nuevo grupo se convertirá, con el tiempo, en una subplantilla.
2. Cree una copia del paso de parámetros compartidos y, a continuación, use *mover al grupo* para trasladar la copia al grupo que creó en el paso 1. Este paso de parámetros permitirá que la subplantilla funcione independientemente de la plantilla externa y se combinará cuando se cargue dentro de dicha plantilla externa.
    > [!NOTE]
    > Técnicamente, no es necesario que la subplantilla incluya estos parámetros que se combinarán si no pretende que las subplantillas sean visibles por sí mismas. Sin embargo, dificultará su edición o depuración si necesita hacerlo más adelante.

3. Mueva todos los elementos del libro que quiera incluir en la subplantilla al grupo que creó en el paso 1.
4. Si los pasos individuales que se movieron en el paso 3 tenían una visibilidad condicional, esta será la nueva visibilidad del grupo externo (como se usa en las pestañas). Quite dicha visibilidad de los elementos dentro del grupo y agregue la configuración de visibilidad al grupo. Guarde e este momento para evitar la pérdida de cambios o exporte y guarde una copia del contenido JSON.
5. Si quiere que ese grupo se cargue desde una plantilla, puede usar el botón *Editar* de la barra de herramientas del grupo. Se abrirá únicamente el contenido de ese grupo como un libro en una ventana nueva. Después, podrá guardarlo como corresponda y cerrar la vista de libro (no cierre el explorador, solo esa vista para volver al libro anterior que estaba editando).
6. Después, podrá cambiar el paso de grupo para que se cargue desde la plantilla y establecer el campo "Id. de plantilla" en el libro o plantilla que creó en el paso 5. Para trabajar con los identificadores de libro, el origen debe ser el identificador de recurso de Azure de un libro compartido. Presione *Cargar* y el contenido de ese grupo ahora se cargará desde esa subplantilla, en lugar de guardarse dentro de este libro externo.

## <a name="next-steps"></a>Pasos siguientes
- [Información general sobre los libros](./workbooks-overview.md)
- [Uso de JSONPath con libros](workbooks-jsonpath.md)