---
title: archivo de inclusión
description: archivo de inclusión
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 04/05/2019
ms.author: sngun
ms.custom: include file
ms.openlocfilehash: 4e050f9e96566035bdf0b8aa53aa90c06f4bca90
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112235035"
---
Ahora puede usar consultas en Data Explorer para recuperar y filtrar los datos.

1. En la parte superior de la pestaña **Elementos** de Data Explorer, examine la consulta predeterminada `SELECT * FROM c`. Esta consulta recupera y muestra todos los documentos del contenedor ordenados por identificador. 
   
   :::image type="content" source="./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-query.png" alt-text="La consulta predeterminada del Explorador de datos es SELECT * FROM c":::
   
1. Para cambiar la consulta, seleccione **Editar filtro**, reemplace la consulta predeterminada por `ORDER BY c._ts DESC`y, después, seleccione **Aplicar filtro**.
   
   :::image type="content" source="./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-edit-query.png" alt-text="Cambiar la consulta predeterminada mediante la incorporación de ORDER BY c._ts DESC y haciendo clic en Aplicar filtro":::

   Esta consulta modificada muestra los documentos en orden descendente según su marca de tiempo, por lo que ahora el segundo documento aparece en primer lugar. 
   
   :::image type="content" source="./media/cosmos-db-create-sql-api-query-data/azure-cosmosdb-data-explorer-edited-query.png" alt-text="Consulta cambiada a ORDER BY c._ts DESC y clic en Aplicar filtro":::

Si está familiarizado con la sintaxis de SQL, puede especificar cualquiera de las [consultas SQL](../sql-query-getting-started.md) admitidas en el cuadro de predicado de consulta. También puede usar Data Explorer para crear procedimientos almacenados, UDF y desencadenadores para la lógica de negocios del servidor. 

Data Explorer proporciona un acceso sencillo en Azure Portal a todas las características de acceso a datos mediante programación integrado que está disponible en las API. Use también el portal para escalar el rendimiento, obtener las claves y cadenas de conexión, y revisar las métricas y los contratos de nivel de servicio de la cuenta de Azure Cosmos DB.