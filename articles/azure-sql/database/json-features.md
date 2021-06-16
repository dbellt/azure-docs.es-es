---
title: Trabajo con datos JSON
description: Azure SQL Database e Instancia administrada de Azure SQL permiten analizar datos, consultarlos y darles formato en notación de objetos JavaScript (JSON).
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: uc-msft
ms.author: umajay
ms.reviewer: mathoma
ms.date: 04/19/2020
ms.openlocfilehash: 4597380c0bae96c67c38d36d162fdf740ba119a2
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111590259"
---
# <a name="getting-started-with-json-features-in-azure-sql-database-and-azure-sql-managed-instance"></a>Introducción a las características de JSON en Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database e Instancia administrada de Azure AD permiten analizar y consultar datos representados en formato de notación de objetos JavaScript [(JSON)](https://www.json.org/), y exportar los datos relacionales como texto JSON. Los escenarios JSON siguientes están disponibles:

- [Aplicación del formato JSON a datos relacionales](#formatting-relational-data-in-json-format) mediante la cláusula `FOR JSON`.
- [Trabajo con datos JSON](#working-with-json-data)
- [Consulta de datos JSON](#querying-json-data) mediante funciones escalares JSON.
- [Transformación de JSON en formato tabular](#transforming-json-into-tabular-format) mediante la función `OPENJSON`.

## <a name="formatting-relational-data-in-json-format"></a>Aplicación del formato JSON a datos relacionales

Si tiene un servicio web que toma datos del nivel de base de datos y proporciona una respuesta en formato JSON o marcos JavaScript del lado cliente o bibliotecas que aceptan datos con formato JSON, puede dar formato JSON al contenido de la base de datos directamente en una consulta SQL. Ya no tiene que escribir código de aplicación para dar formato JSON a los resultados de Azure SQL Database o Instancia administrada de Azure SQL ni incluir una biblioteca de serialización de JSON para convertir los resultados tabulares de la consulta y después serializar los objetos en formato JSON. En su lugar, puede usar la cláusula FOR JSON para dar formato JSON a resultados de consultas SQL y usarlos directamente en su aplicación.

En el ejemplo siguiente, se aplica el formato JSON a las filas de la tabla Sales.Customers mediante la cláusula FOR JSON:

```sql
select CustomerName, PhoneNumber, FaxNumber
from Sales.Customers
FOR JSON PATH
```

La cláusula FOR JSON PATH da formato de texto JSON a los resultados de la consulta. Los nombres de columna se utilizan como claves, mientras que los valores de celda se generan como valores JSON:

```json
[
{"CustomerName":"Eric Torres","PhoneNumber":"(307) 555-0100","FaxNumber":"(307) 555-0101"},
{"CustomerName":"Cosmina Vlad","PhoneNumber":"(505) 555-0100","FaxNumber":"(505) 555-0101"},
{"CustomerName":"Bala Dixit","PhoneNumber":"(209) 555-0100","FaxNumber":"(209) 555-0101"}
]
```

Se da al conjunto de resultados formato de matriz JSON, donde cada fila tiene formato de objeto JSON independiente.

PATH indica que puede personalizar el formato de salida de su resultado JSON utilizando la notación de puntos en los alias de columna. La consulta siguiente cambia el nombre de la clave "CustomerName" en el formato JSON de salida y coloca los números de teléfono y fax en el subobjeto "Contact":

```sql
select CustomerName as Name, PhoneNumber as [Contact.Phone], FaxNumber as [Contact.Fax]
from Sales.Customers
where CustomerID = 931
FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
```

La salida de esta consulta tiene el siguiente aspecto:

```json
{
    "Name":"Nada Jovanovic",
    "Contact":{
           "Phone":"(215) 555-0100",
           "Fax":"(215) 555-0101"
    }
}
```

En este ejemplo, se devuelve un único objeto JSON en lugar de una matriz al especificarse la opción [WITHOUT_ARRAY_WRAPPER](/sql/relational-databases/json/remove-square-brackets-from-json-without-array-wrapper-option). Puede usar esta opción si sabe que va a devolver un solo objeto como resultado de la consulta.

El valor principal de la cláusula FOR JSON es que permite devolver datos jerárquicos complejos desde la base de datos con formato de objetos o matrices JSON anidados. En el ejemplo siguiente se muestra cómo incluir las filas de la tabla `Orders` que pertenecen a `Customer` como matriz anidada de `Orders`:

```sql
select CustomerName as Name, PhoneNumber as Phone, FaxNumber as Fax,
        Orders.OrderID, Orders.OrderDate, Orders.ExpectedDeliveryDate
from Sales.Customers Customer
    join Sales.Orders Orders
        on Customer.CustomerID = Orders.CustomerID
where Customer.CustomerID = 931
FOR JSON AUTO, WITHOUT_ARRAY_WRAPPER
```

En lugar de enviar consultas diferentes para obtener los datos del cliente y después recuperar una lista de pedidos relacionados, puede obtener todos los datos necesarios con una única consulta, como se muestra en la siguiente salida de ejemplo:

```json
{
  "Name":"Nada Jovanovic",
  "Phone":"(215) 555-0100",
  "Fax":"(215) 555-0101",
  "Orders":[
    {"OrderID":382,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":395,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":1657,"OrderDate":"2013-01-31","ExpectedDeliveryDate":"2013-02-01"}
  ]
}
```

## <a name="working-with-json-data"></a>Trabajo con datos JSON

Si no tiene datos con una estructura estricta, si tiene subobjetos complejos, matrices o datos jerárquicos, o si sus estructuras de datos evolucionan con el tiempo, el formato JSON puede ayudarlo a representar cualquier estructura de datos compleja.

JSON es un formato de texto que se puede usar como cualquier otro tipo de cadena en Azure SQL Database e Instancia administrada de Azure SQL. Puede enviar o almacenar datos JSON como un tipo NVARCHAR estándar:

```sql
CREATE TABLE Products (
  Id int identity primary key,
  Title nvarchar(200),
  Data nvarchar(max)
)
go
CREATE PROCEDURE InsertProduct(@title nvarchar(200), @json nvarchar(max))
AS BEGIN
    insert into Products(Title, Data)
    values(@title, @json)
END
```

Los datos JSON usados en este ejemplo se representan mediante el tipo NVARCHAR(MAX). Se puede insertar JSON en esta tabla o proporcionarlo como argumento del procedimiento almacenado mediante sintaxis estándar de Transact-SQL, tal como se muestra en el ejemplo siguiente:

```sql
EXEC InsertProduct 'Toy car', '{"Price":50,"Color":"White","tags":["toy","children","games"]}'
```

Cualquier lenguaje del lado cliente o biblioteca que funcione con datos de cadena en Azure SQL Database e Instancia administrada de Azure SQL también funciona con datos JSON. JSON se puede almacenar en cualquier tabla que admita el tipo NVARCHAR, como una tabla optimizado para memoria o una tabla con versiones del sistema. JSON no introduce ninguna restricción en el código del lado cliente ni en el nivel de base de datos.

## <a name="querying-json-data"></a>Consulta de datos JSON

Si tiene datos con formato JSON almacenados en tablas SQL de Azure, las funciones JSON permiten usarlos en cualquier consulta SQL.

Las funciones JSON que están disponibles en Azure SQL Database e Instancia administrada de Azure SQL permiten tratar los datos con formato JSON como cualquier otro tipo de datos SQL. Puede extraer fácilmente valores del texto JSON y usar datos JSON en cualquier consulta:

```sql
select Id, Title, JSON_VALUE(Data, '$.Color'), JSON_QUERY(Data, '$.tags')
from Products
where JSON_VALUE(Data, '$.Color') = 'White'

update Products
set Data = JSON_MODIFY(Data, '$.Price', 60)
where Id = 1
```

La función JSON_VALUE extrae un valor del texto JSON almacenado en la columna Data. Esta función usa una ruta de acceso de estilo JavaScript para hacer referencia a un valor en el texto JSON para extraerlo. El valor extraído se puede usar en cualquier parte de la consulta SQL.

La función JSON_QUERY es similar a JSON_VALUE. A diferencia de JSON_VALUE, esta función extrae subobjetos complejos, como matrices u objetos ubicados en texto JSON.

La función JSON_MODIFY permite especificar la ruta de acceso del valor en el texto JSON que debe actualizarse, así como un nuevo valor que sobrescribirá el anterior. De esta forma, puede actualizar fácilmente el texto JSON sin volver a analizar la estructura completa.

Dado que JSON se almacena en texto estándar, no existe ninguna garantía de que los valores almacenados en las columnas de texto tengan el formato correcto. Puede comprobar que el texto almacenado en la columna JSON tenga un formato correcto con las restricciones CHECK estándar de Azure SQL Database y la función ISJSON:

```sql
ALTER TABLE Products
    ADD CONSTRAINT [Data should be formatted as JSON]
        CHECK (ISJSON(Data) > 0)
```

Si el texto de entrada tiene un formato JSON correcto, la función ISJSON devuelve el valor 1. Con cada inserción o actualización de la columna JSON, esta restricción comprobará que el nuevo valor de texto no tenga un formato JSON incorrecto.

## <a name="transforming-json-into-tabular-format"></a>Transformación de JSON en formato tabular

Azure SQL Database e Instancia administrada de Azure SQL también permiten transformar las colecciones JSON en formato tabular y cargar o consultar datos JSON.

OPENJSON es una función de valores de tabla que analiza texto JSON, busca una matriz de objetos JSON, itera en los elementos de la matriz y devuelve en el resultado de salida una fila para cada elemento de la matriz.

![JSON tabular](./media/json-features/image_2.png)

En el ejemplo anterior, se puede especificar dónde ubicar la matriz JSON que se debe abrir (en la ruta de acceso $.Orders), qué columnas se deben devolver como resultado y dónde encontrar los valores JSON que se devolverán como celdas.

Se puede transformar una matriz JSON de la variable @orders en un conjunto de filas, analizar este conjunto de resultados o insertar filas en una tabla estándar:

```sql
CREATE PROCEDURE InsertOrders(@orders nvarchar(max))
AS BEGIN

    insert into Orders(Number, Date, Customer, Quantity)
    select Number, Date, Customer, Quantity
    FROM OPENJSON (@orders)
     WITH (
            Number varchar(200),
            Date datetime,
            Customer varchar(200),
            Quantity int
     )
END
```

La colección de pedidos con formato de matriz JSON que se proporciona como parámetro del procedimiento almacenado se puede analizar e insertar en la tabla Orders.

## <a name="next-steps"></a>Pasos siguientes

Para aprender a integrar JSON en la aplicación, consulte estos recursos:

Para más información sobre diversos escenarios para integrar JSON en su aplicación, vea las demostraciones en este [vídeo de Channel 9](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/JSON-as-a-bridge-betwen-NoSQL-and-relational-worlds) o busque un escenario que coincida con su caso de uso en las [publicaciones del blog sobre JSON](/archive/blogs/sqlserverstorageengine/json-in-sql-server-use-cases).