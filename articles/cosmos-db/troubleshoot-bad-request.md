---
title: Solución de problemas de excepciones de solicitudes erróneas de Azure Cosmos DB
description: Obtenga información sobre cómo diagnosticar y corregir excepciones de solicitudes erróneas. Por ejemplo, cuando el contenido de entrada o la clave de partición no son válidos, la clave de partición no coincide en Azure Cosmos DB.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 04/06/2021
ms.author: maquaran
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: e0ce00331eb524d064fd6e7c5205be8b66d4dbc2
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106555935"
---
# <a name="diagnose-and-troubleshoot-bad-request-exceptions-in-azure-cosmos-db"></a>Diagnosticar y solucionar problemas de excepciones de solicitudes erróneas en Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

El código de estado HTTP 400 indica que la solicitud contiene datos no válidos o que faltan parámetros necesarios.

## <a name="missing-the-id-property"></a><a name="missing-id-property"></a>Falta la propiedad de identificador
En este escenario, es habitual ver el siguiente error:

*The input content is invalid because the required properties - 'id; ' - are missing* (El contenido de entrada no es válido porque faltan las propiedades requeridas: 'id; ')

Si una respuesta muestra este error significa que el documento JSON que se envía al servicio no tiene la propiedad de identificador necesaria.

### <a name="solution"></a>Solución
Especifique una propiedad `id` con un valor de cadena que siga la [especificación de REST](/rest/api/cosmos-db/documents) como parte del documento. Los SDK no generan automáticamente valores para esta propiedad.

## <a name="invalid-partition-key-type"></a><a name="invalid-partition-key-type"></a>Tipo de clave de partición no válido
En este escenario, es habitual ver errores como los siguientes:

*Partition key ... is invalid* (La clave de partición no es válida)

Si una respuesta muestra este error significa que el valor de la clave de partición no es válido.

### <a name="solution"></a>Solución
El valor de la clave de partición debe ser una cadena o un número. Asegúrese de que el valor sea uno de estos.

## <a name="wrong-partition-key-value"></a><a name="wrong-partition-key-value"></a>Valor incorrecto de la clave de partición
En este escenario, es habitual ver el siguiente error:

*PartitionKey extracted from document doesn’t match the one specified in the header* (El valor de PartitionKey extraído del documento no coincide con el especificado en el encabezado)

Una respuesta que muestre este error significa que se está ejecutando una operación y pasando un valor de clave de partición que no coincide con el valor de cuerpo del documento para la propiedad prevista. Si la ruta de la clave de partición de la recopilación es `/myPartitionKey`, el documento tiene una propiedad denominada `myPartitionKey`, con un valor que no coincide con el que se proporcionó como valor de clave de partición al llamar al método del SDK.

### <a name="solution"></a>Solución
Envíe el parámetro de valor de la clave de partición que coincida con el valor de la propiedad del documento.

## <a name="next-steps"></a>Pasos siguientes
* [Diagnóstico y solución de problemas](troubleshoot-dot-net-sdk.md) al utilizar el SDK de Azure Cosmos DB para .NET.
* Más información sobre las directrices de rendimiento de [.NET v3](performance-tips-dotnet-sdk-v3-sql.md) y [.NET v2](performance-tips.md).
* [Diagnóstico y solución de problemas](troubleshoot-java-sdk-v4-sql.md) al utilizar el SDK de Azure Cosmos DB para Java v4.
* Obtenga información sobre las instrucciones de rendimiento del [SDK para Java v4](performance-tips-java-sdk-v4-sql.md).
