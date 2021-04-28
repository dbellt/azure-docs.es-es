---
title: Uso de funciones en consultas de registros de Azure Monitor
description: En este artículo se describe cómo utilizar funciones para llamar a una consulta desde otra consulta de registro de Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 04/19/2021
ms.openlocfilehash: fecede1d582232ebc75878a687a24818031f0d80
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752945"
---
# <a name="functions-in-azure-monitor-log-queries"></a>Uso de funciones en consultas de registros de Azure Monitor
Una función es una consulta de registro en Azure Monitor que se puede usar en otras consultas de registro como si se tratase de un comando. Las funciones permiten a los desarrolladores proporcionar soluciones a distintos clientes y a los usuarios reutilizar la lógica de consulta en su propio entorno. En este artículo se proporcionan detalles sobre cómo usar funciones y cómo crear las suyas propias.

## <a name="types-of-functions"></a>Tipos de funciones
Hay dos tipos de funciones en Azure Monitor:

- **Función de solución:** funciones pregeneradas incluidas con Azure Monitor. Están disponibles en todas las áreas de trabajo de Log Analytics y no se pueden modificar.
- **Funciones del área de trabajo:** funciones instaladas en un área de trabajo de Log Analytics determinada y que el usuario puede modificar y controlar.

## <a name="viewing-functions"></a>Visualización de funciones
Puede ver las funciones de solución y las funciones del área de trabajo en el área de trabajo actual desde la pestaña **Funciones** del panel izquierdo de un área de trabajo de Log Analytics. Use el botón **Filtrar** para filtrar las funciones incluidas en la lista y **Agrupar por** para cambiar su agrupación. Escriba una cadena en el cuadro **Buscar** para buscar una función determinada. Mantenga el puntero sobre una función para ver los detalles sobre ella, incluida una descripción y parámetros.

:::image type="content" source="media/functions/view-functions.png" alt-text="Ver función" lightbox="media/functions/view-functions.png":::

## <a name="use-a-function"></a>Uso de una función
Use una función en una consulta escribiendo su nombre con valores para cualquier parámetro igual que lo haría en un comando. La salida de la función se puede devolver como resultados o canalizarse a otro comando.

Para agregar una función a la consulta actual, haga doble clic en su nombre o mantenga el puntero sobre ella y seleccione **Usar en el editor**. Las funciones del área de trabajo también se incluirán en IntelliSense a medida que escriba una consulta. 

Si una consulta requiere parámetros, puede proporcionarlos mediante la sintaxis: `function_name(param1,param2,...)`.

:::image type="content" source="media/functions/function-use.png" alt-text="Uso de una función" lightbox="media/functions/function-use.png":::

## <a name="create-a-function"></a>Creación de una función
Para crear una función a partir de la consulta actual del editor, seleccione **Guardar** y, a continuación, **Guardar como función**. 

:::image type="content" source="media/functions/function-save.png" alt-text="Creación de una función" lightbox="media/functions/function-save.png":::

Para crear una función con Log Analytics en Azure Portal, haga clic en **Guardar** y, después, proporcione la información de la tabla siguiente.

| Configuración | Descripción |
|:---|:---|
| Nombre de la función  | El nombre de la función. Este no puede incluir un espacio ni ningún carácter especial. Tampoco puede comenzar con un carácter de subrayado (_), ya que este carácter está reservado para las funciones de solución. |
| Categoría heredada | Categoría definida por el usuario para ayudar a filtrar y agrupar funciones.   |
| Guardar como grupo de equipos | Guarde la consulta como un [grupo de equipos](computer-groups.md).  |
| Parámetros | Agregue un parámetro para cada variable de la función que requiera un valor cuando se utilice. Consulte [Parámetros de función](#function-parameters) para más información. |

:::image type="content" source="media/functions/function-details.png" alt-text="Detalles de la función" lightbox="media/functions/function-details.png":::

## <a name="function-parameters"></a>Parámetros de función 
Puede agregar parámetros a una función para que pueda proporcionar valores de determinadas variables al llamarla. Esto permite usar la misma función en consultas diferentes, cada una de las cuales proporciona valores diferentes para los parámetros. Los parámetros se definen mediante las siguientes propiedades.

| Configuración | Descripción |
|:---|:---|
| Tipo  | Tipo de datos para el valor. |
| Nombre  | Nombre del parámetro. Este es el nombre que se debe usar en la consulta para reemplazar por el valor del parámetro.  |
| Valor predeterminado | Valor que se usará para el parámetro si no se proporciona ningún valor. |

Los parámetros se ordenan a medida que se van creando, primero los parámetros sin ningún valor predeterminado y, después, aquellos que sí lo tienen.

## <a name="working-with-function-code"></a>Uso del código de función
Puede ver el código de una función para obtener información sobre cómo funciona o para modificar el código de una función del área de trabajo. Seleccione **Cargar el código de la función** para agregar el código de la función a la consulta actual del editor. Si lo agrega a una consulta vacía o a la primera línea de una consulta existente, agregará el nombre de función a la pestaña. Si se trata de una función del área de trabajo, esto permitirá editar los detalles de la función.

:::image type="content" source="media/functions/function-code.png" alt-text="Cargar el código de la función" lightbox="media/functions/function-code.png":::

## <a name="edit-a-function"></a>Edición de una función
Edite las propiedades o el código de una función mediante la creación de una nueva consulta y, a continuación, mantenga el puntero sobre el nombre de la función y seleccione **Cargar código de función**. Realice las modificaciones que desee en el código y seleccione **Guardar** y, a continuación, **Editar los detalles de la función**. Realice los cambios que desee en las propiedades y parámetros de la función antes de hacer clic en **Guardar**.

:::image type="content" source="media/functions/function-edit.png" alt-text="Editar función" lightbox="media/functions/function-edit.png":::
## <a name="example"></a>Ejemplo
La siguiente función de ejemplo devuelve todos los eventos del registro de actividades de Azure a partir de una fecha determinada y los eventos que coinciden con una categoría determinada. 

Comience con la consulta siguiente mediante valores codificados de forma rígida. Esto comprueba que la consulta funciona según lo previsto.

```Kusto
AzureActivity
| where CategoryValue == "Administrative"
| where TimeGenerated > todatetime("2021/04/05 5:40:01.032 PM")
```

:::image type="content" source="media/functions/example-query.png" alt-text="Función de ejemplo: consulta inicial" lightbox="media/functions/example-query.png":::

A continuación, reemplace los valores codificados de forma rígida por nombres de parámetro y, a continuación, guarde la función seleccionando **Guardar** y, a continuación, **Guardar como función**.

```Kusto
AzureActivity
| where CategoryValue == CategoryParam
| where TimeGenerated > DateParam
```

:::image type="content" source="media/functions/example-save-function.png" alt-text="Función de ejemplo: guardar función" lightbox="media/functions/example-save-function.png":::

 Proporcione los siguientes valores para las propiedades de servidor.

| Propiedad | Value |
|:---|:---|
| Nombre de función | AzureActivityByCategory |
| Categoría heredada | Funciones de demostración |

Defina los parámetros siguientes antes de guardar la función.

| Tipo | Nombre | Valor predeterminado |
|:---|:---|:---|
| string   | CategoryParam | "Administrativa" |
| datetime | DateParam     | |

:::image type="content" source="media/functions/example-function-properties.png" alt-text="Función de ejemplo: propiedades de función" lightbox="media/functions/example-function-properties.png":::

Cree una nueva consulta y mantenga el puntero sobre ella para ver la nueva función. Tenga en cuenta el orden de los parámetros, ya que este es el orden en que se deberán especificar cuando se use la función.

:::image type="content" source="media/functions/example-view-details.png" alt-text="Función de ejemplo: ver detalles" lightbox="media/functions/example-view-details.png":::

Seleccione **Usar en el editor** para agregar la nueva función a una consulta y, a continuación, agregue los valores de los parámetros. Tenga en cuenta que no es necesario especificar un valor para CategoryParam porque tiene un valor predeterminado.

:::image type="content" source="media/functions/example-use-function.png" alt-text="Función de ejemplo: usar función" lightbox="media/functions/example-use-function.png":::



## <a name="next-steps"></a>Pasos siguientes
Consulte otras lecciones para escribir consultas de registro de Azure Monitor:

- [Operaciones de cadena](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#string-operations)

