---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: 9447cec55c53861ca57d5416a91ffefd35fdd20b
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108753375"
---
## <a name="repeatability-during-copy"></a>Repetibilidad durante la copia
Cuando se copian datos en Azure SQL o SQL Server desde otros almacenes de datos, es necesario tener en mente la repetibilidad para evitar resultados imprevistos. 

Al copiar datos en la Base de datos SQL de Azure, la actividad de copia anexará el conjunto de datos a la tabla del receptor de forma predeterminada. Por ejemplo, cuando se copian datos desde un origen de archivo CSV (datos de valores separados por comas) que contiene dos registros a la Base de datos SQL de Azure y SQL Server, la tabla es como sigue:

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
```

Supongamos que encontró errores en el archivo de origen y actualizó la cantidad de Down Tube de 2 a 4 en el archivo de origen. Si vuelve a ejecutar el segmento de datos durante ese período, encontrará dos nuevos registros que se anexarán a Base de datos SQL de Azure y SQL Server. Lo siguiente supone que ninguna de las columnas de la tabla tienen la restricción de clave principal.

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

Para evitar esto, tendrá que especificar la semántica UPSERT aprovechando uno de los dos mecanismos siguientes indicados a continuación.

> [!NOTE]
> Un segmento se puede volver a ejecutar automáticamente en Azure Data Factory según la directiva de reintento especificada.
> 
> 

### <a name="mechanism-1"></a>Mecanismo 1
Puede aprovechar la propiedad **sqlWriterCleanupScript** para realizar primero la acción de limpieza cuando se ejecuta un segmento. 

```json
"sink":  
{ 
  "type": "SqlSink", 
  "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM table WHERE ModifiedDate >= \\'{0:yyyy-MM-dd HH:mm}\\' AND ModifiedDate < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
}
```

El script de limpieza debe ejecutarse primero durante la copia de un segmento dado que eliminará los datos de la tabla SQL correspondiente a dicho segmento. La actividad de copia insertará posteriormente los datos en la tabla de SQL. 

Si el segmento se vuelve a ejecutar, se encontrará que la cantidad se actualiza tal como se desea.

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

Supongamos que se quita el registro Flat Washer desde el archivo csv original. Después si vuelve a ejecutar el segmento, se producirá el siguiente resultado: 

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
7     Down Tube    4            2015-05-01 00:00:00
```
No hay que hacer nada nuevo. La actividad de copia ejecutó el script de limpieza para eliminar los datos correspondientes para ese segmento. Después leyó la entrada en el archivo csv (que entonces contenía solo un registro) y lo insertó en la tabla. 

### <a name="mechanism-2"></a>Mecanismo 2
> [!IMPORTANT]
> sliceIdentifierColumnName no se admite en este momento para Azure Synapse Analytics. 

Otro mecanismo para lograr la repetibilidad es tener una columna dedicada (**sliceIdentifierColumnName**) en la tabla de destino. Esta columna debe usarAzure Data Factory para asegurarse de que el origen y el destino estén sincronizados. Este enfoque funciona cuando hay flexibilidad para cambiar o definir el esquema de la tabla SQL de destino. 

Esta columna debe usarla Azure Data Factory con fines de repetibilidad y en el proceso Azure Data Factory no realizará ningún cambio de esquema en la tabla. Forma de usar este enfoque:

1. Defina una columna de tipo binario (32) en la tabla SQL de destino. No debería haber ninguna restricción en esta columna. Llamemos a esta columna 'ColumnForADFuseOnly' para este ejemplo.
2. Úselo en la actividad de copia de la forma siguiente:
   
    ```json
    "sink":  
    { 
   
        "type": "SqlSink", 
        "sliceIdentifierColumnName": "ColumnForADFuseOnly"
    }
    ```

Azure Data Factory rellenará esta columna según sus necesidades para asegurarse de que el origen y el destino estén sincronizados. Los valores de esta columna no deben usarse fuera de este contexto por el usuario. 

De manera similar al mecanismo 1, la actividad de copia limpiará primero automáticamente los datos para el segmento especificado del tabla SQL de destino y después ejecutará la actividad de copia normalmente para insertar los datos del origen al destino para dicho segmento. 

