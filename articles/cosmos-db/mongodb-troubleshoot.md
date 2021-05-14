---
title: Solución de errores habituales de la API de Azure Cosmos DB para MongoDB
description: En este documento se describen las formas de solucionar los problemas habituales detectados en la API de Azure Cosmos DB para MongoDB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: troubleshooting
ms.date: 07/15/2020
ms.author: chrande
ms.openlocfilehash: c2d86af956ff60bba22effe059996cf499e4b282
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108134094"
---
# <a name="troubleshoot-common-issues-in-azure-cosmos-dbs-api-for-mongodb"></a>Solución de errores habituales de la API de Azure Cosmos DB para MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

En el siguiente artículo se describen los errores comunes y las soluciones para las implementaciones usan la API de Azure Cosmos DB para MongoDB.

>[!Note]
> Azure Cosmos DB no hospeda el motor de MongoDB. Proporciona una implementación del [protocolo de conexión 4.0](mongodb-feature-support-40.md) y [3.6](mongodb-feature-support-36.md) de MongoDB y compatibilidad heredada con la [versión 3.2 del protocolo de conexión](mongodb-feature-support.md). Por lo tanto, algunos de estos errores solo se encuentran en la API de Azure Cosmos DB para MongoDB.

## <a name="common-errors-and-solutions"></a>Errores habituales y soluciones

| Código       | Error                | Descripción  | Solución  |
|------------|----------------------|--------------|-----------|
| 2 | BadValue | Una causa común es que la ruta de acceso del índice correspondiente al elemento order-by especificado se excluye o la consulta order by no tiene un índice compuesto correspondiente desde el que se puede atender. La consulta solicita que se ordene por un campo que no está indexado. | Cree un índice coincidente (o índice compuesto) para la consulta de ordenación que se intenta realizar. |
| 2 | La transacción no está activa | La transacción de varios documentos ha sobrepasado el límite de tiempo fijo de 5 segundos. | Vuelva a intentar la transacción de varios documentos o limite el ámbito de las operaciones dentro de la transacción de varios documentos para que se complete en el límite de 5 segundos. |
| 13 | No autorizado | La solicitud no tiene los permisos necesarios para completarse. | Asegúrese de que está utilizando las claves correctas.  |
| 26 | NamespaceNotFound | No se encuentra la base de datos o la colección a la que se hace referencia en la consulta. | Asegúrese de que el nombre de la base de datos o colección coincida exactamente con el nombre de la consulta.|
| 50 | ExceededTimeLimit | La solicitud ha superado el tiempo de espera de 60 segundos de ejecución. |  Este error puede deberse a muchas causas. Una de las causas es cuando la capacidad de las unidades de solicitud asignada actualmente no es suficiente para completar la solicitud. Para solucionar esto, se pueden aumentar las unidades de solicitud de esa colección o base de datos. En otros casos, este error se puede solucionar dividiendo una solicitud grande en otras más pequeñas. Si se vuelve a intentar una operación de escritura que ha recibido este error, se puede producir una operación de escritura duplicada. <br><br>Si intenta eliminar grandes cantidades de datos sin afectar a las unidades de solicitud: <br>- Considere la posibilidad de usar TTL (basado en la marca de tiempo): [Expiración de los datos con la API de Azure Cosmos DB para MongoDB](mongodb-time-to-live.md) <br>- Use el tamaño del cursor o del lote para realizar la eliminación. Puede capturar un solo documento cada vez y eliminarlo mediante un bucle. Esto le ayudará a eliminar los datos lentamente sin afectar a la aplicación de producción.|
| 61 | ShardKeyNotFound | El documento de la solicitud no contenía la clave de partición de la colección (clave de partición de Azure Cosmos DB). | Asegúrese de que la clave de partición de la colección se use en la solicitud.|
| 66 | ImmutableField | La solicitud está intentando cambiar un campo inmutable. | Los campos "_id" son inmutables. Asegúrese de que la solicitud no trate de actualizar el campo o el campo de clave de partición. |
| 67 | CannotCreateIndex | No se puede completar la solicitud para crear un índice. | Se pueden crear hasta 500 índices de un solo campo en cada contenedor. Se pueden incluir hasta ocho campos en un índice compuesto (los índices compuestos se admiten en la versión 3.6 y posteriores). |
| 112 | WriteConflict | Error en la transacción de varios documentos debido a una transacción de varios documentos en conflicto | Vuelva a intentar la transacción de varios documentos hasta que se complete correctamente. |
| 115 | CommandNotSupported | No se admite el intento de solicitud. | Se deben especificar los detalles adicionales en el error. Si esta funcionalidad es importante para sus implementaciones, cree una incidencia de soporte técnico en [Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). El equipo de Azure Cosmos DB le responderá. |
| 11000 | DuplicateKey | La clave de partición (clave de partición de Azure Cosmos DB) del documento que se está insertando ya existe en la colección, o se infringió alguna restricción de un campo de índice único. | Use la función update() para actualizar un documento existente. Si se infringió la restricción de un campo de índice único, inserte o actualice el documento con un valor de campo que no exista todavía en la partición. Otra opción sería usar un campo que contenga una combinación de los campos identificador y clave de partición. |
| 16500 | TooManyRequests  | El número total de unidades de solicitud consumidas es mayor que la tasa de unidades de solicitud aprovisionadas para la colección y se ha limitado. | Considere la posibilidad de escalar el rendimiento asignado a un contenedor o un conjunto de contenedores desde Azure Portal o vuelva a intentarlo. Si [habilita SSR (reintento en el servidor)](prevent-rate-limiting-errors.md), Azure Cosmos DB reintenta automáticamente las solicitudes que no se envían debido a este error. |
| 16501 | ExceededMemoryLimit | Como se trata de un servicio de varios inquilinos, la operación ha superado la asignación de memoria del cliente. Esto solo se aplica a la API de Azure Cosmos DB para MongoDB versión 3.2. | Reduzca el ámbito de la operación a través de criterios de consulta más restrictivos o póngase en contacto con soporte técnico desde [Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Ejemplo: `db.getCollection('users').aggregate([{$match: {name: "Andy"}}, {$sort: {age: -1}}]))` |
| 40324 | Nombre de fase de canalización no reconocido. | No se reconoció el nombre de fase en la solicitud de canalización de agregación. | Asegúrese de que todos los nombres de canalización de agregación de la solicitud sean válidos. |
| - | Problemas de versión de la conexión de MongoDB | Las versiones anteriores de los controladores de MongoDB son no detectan el nombre de la cuenta de Azure Cosmos en las cadenas de conexión. | Anexe `appName=@accountName@` al final de la cadena de conexión, donde `accountName` es el nombre de la cuenta de Azure Cosmos DB. |
| - | Problemas de red del cliente de MongoDB (como excepciones de socket o endOfStream)| Error en la solicitud de red. Esto suele deberse a una conexión TCP inactiva que el cliente de MongoDB está tratando de usar. Los controladores de MongoDB suelen usar la agrupación de conexiones, que da como resultado una conexión aleatoria elegida del grupo que se usa para una solicitud. Las conexiones inactivas suelen agotar el tiempo de espera de Azure Cosmos DB después de cuatro minutos. | Puede volver a intentar estas solicitudes con error en el código de la aplicación, cambiar la configuración del cliente de MongoDB (controlador) para desmontar las conexiones TCP inactivas antes de que se agote el tiempo de espera de cuatro minutos, o configurar las opciones `keepalive` del sistema operativo para mantener las conexiones TCP en estado activo.<br><br>Para evitar mensajes de conectividad, es posible que quiera cambiar la cadena de conexión para establecer `maxConnectionIdleTime` en 1 o 2 minutos.<br>-Controlador de Mongo: configure `maxIdleTimeMS=120000` <br>- Node.JS: configure `socketTimeoutMS=120000`, `autoReconnect` = true, `keepAlive` = true, `keepAliveInitialDelay` = 3 minutos
| - | El shell de Mongo no funciona en Azure Portal. | Cuando el usuario intenta abrir un shell de Mongo, no sucede nada y la pestaña se queda en blanco.  | Compruebe el firewall. El firewall de no es compatible con el shell de Mongo en Azure Portal. <br>- Instale el shell de Mongo en el equipo local dentro de las reglas de firewall. <br>- Utilice el shell de Mongo heredado.
| - | No se puede conectar con la cadena de conexión.  | La cadena de conexión ha cambiado al actualizar desde 3.2-> 3.6 | Tenga en cuenta que cuando se usa la API de Azure Cosmos DB para las cuentas de MongoDB, la versión 3.6 de las cuentas tiene el punto de conexión en el formato `*.mongo.cosmos.azure.com`, mientras que la versión 3.2 de las cuentas tiene el punto de conexión en el formato `*.documents.azure.com`.  

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [usar Studio 3T](mongodb-mongochef.md) con la API de Azure Cosmos DB para MongoDB.
- Aprenda a [usar Robo 3T](mongodb-robomongo.md) con la API de Azure Cosmos DB para MongoDB.
- Explore [ejemplos](mongodb-samples.md) de MongoDB con la API de Azure Cosmos DB para MongoDB.
