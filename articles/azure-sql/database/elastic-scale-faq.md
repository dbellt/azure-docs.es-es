---
title: Preguntas más frecuentes sobre el Escalado elástico
description: Preguntas más frecuentes sobre el Escalado elástico de Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: d2a9e78dcc81a16941a775618ac4e221a5a1357c
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110705509"
---
# <a name="elastic-database-tools-frequently-asked-questions-faq"></a>Preguntas frecuentes (P+F) sobre las herramientas de base de datos elástica
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="if-i-have-a-single-tenant-per-shard-and-no-sharding-key-how-do-i-populate-the-sharding-key-for-the-schema-info"></a>Si tengo un solo inquilino por partición y ninguna clave de particionamiento, ¿cómo relleno la clave de particionamiento para la información de esquema?

El objeto de información del esquema solo se usa en escenarios de combinación o división. Si una aplicación es inherentemente de un solo inquilino, no requiere la herramienta de combinación o división y, por lo tanto, no es necesario rellenar el objeto de información del esquema.

## <a name="ive-provisioned-a-database-and-i-already-have-a-shard-map-manager-how-do-i-register-this-new-database-as-a-shard"></a>He aprovisionado una base de datos y ya tengo una instancia de Shard Map Manager, ¿cómo registro esta nueva base de datos como partición?

Consulte [Incorporación de una partición a una aplicación mediante la biblioteca de cliente de Elastic Database](elastic-scale-add-a-shard.md).

## <a name="how-much-do-elastic-database-tools-cost"></a>¿Cuánto cuestan las herramientas de base de datos elástica?

El uso de la biblioteca cliente de Base de datos elástica no incurre en costos. Los costos se acumulan solo para las bases de datos de Azure SQL Database que usa para particiones y el Administrador de asignación de particiones, así como los roles web/de trabajo aprovisionados para la herramienta de combinación o división.

## <a name="why-are-my-credentials-not-working-when-i-add-a-shard-from-a-different-server"></a>¿Por qué mis credenciales no funcionan cuando agrego una partición de un servidor diferente?

No use credenciales con el formato "Id. de usuario username@servername"; en su lugar, simplemente use "Id. de Usuario=nombre de usuario".  Además, asegúrese de que el inicio de sesión de "nombre de usuario" tiene permisos en la partición.

## <a name="do-i-need-to-create-a-shard-map-manager-and-populate-shards-every-time-i-start-my-applications"></a>¿Tengo que crear una instancia de Shard Map Manager y rellenar las particiones cada vez que inicie las aplicaciones?

No, la creación de la instancia de Shard Map Manager (por ejemplo, [ShardMapManagerFactory.CreateSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager)) es una operación que se realiza una sola vez.  La aplicación debe usar la llamada [ShardMapManagerFactory.TryGetSqlShardMapManager()](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager) en el momento de iniciar la aplicación.  Solo debería haber una de estas llamadas por dominio de aplicación.

## <a name="i-have-questions-about-using-elastic-database-tools-how-do-i-get-them-answered"></a>Tengo dudas sobre el uso de las herramientas de base de datos elásticas, ¿cómo puedo obtener ayuda?

Póngase en contacto con nosotros en la [página de preguntas y respuestas de Microsoft sobre SQL Database](/answers/topics/azure-sql-database.html).

## <a name="when-i-get-a-database-connection-using-a-sharding-key-i-can-still-query-data-for-other-sharding-keys-on-the-same-shard--is-this-by-design"></a>Al obtener una conexión de base de datos mediante una clave de particionamiento, puedo consultar los datos de otras claves de particionamiento en la misma partición.  ¿Esto se debe al diseño?

Las API de Escalado elástico ofrecen una conexión a la base de datos correcta para su clave de particionamiento, pero no proporcionan filtrado de claves de particionamiento.  Agregue cláusulas **WHERE** a su consulta para restringir el ámbito a la clave de particionamiento proporcionada, si es necesario.

## <a name="can-i-use-a-different-sql-database-edition-for-each-shard-in-my-shard-set"></a>¿Puedo usar una edición diferente de SQL Database en cada partición de mi conjunto de particiones?

Sí, una partición es una base de datos individual y, por lo tanto, una partición podría ser una edición Premium y otra una edición Standard. Además, la edición de una partición puede escalar verticalmente o reducirse verticalmente varias veces durante la duración de la partición.

## <a name="does-the-split-merge-tool-provision-or-delete-a-database-during-a-split-or-merge-operation"></a>¿La herramienta de combinación o división aprovisiona (o elimina) una base de datos durante una operación de combinación o división?

No. En el caso de las operaciones de **división** , la base de datos de destino debe existir con el esquema apropiado y registrarse con el Administrador de asignación de particiones.  En el caso de las operaciones de **combinación** , debe eliminar la partición desde el Administrador de asignación de particiones y, luego, eliminar la base de datos.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]