---
title: Solución de problemas de recursos de vínculo privado compartidos
titleSuffix: Azure Cognitive Search
description: Guía de solución de problemas comunes al administrar recursos de vínculo privado compartidos.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/30/2021
ms.openlocfilehash: 82a5135f23293d0fe9bbaaf0eeb0543b4fdb598f
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111744780"
---
# <a name="troubleshooting-common-issues-with-shared-private-link-resources"></a>Solución de problemas comunes con recursos de vínculo privado compartidos

Los recursos de vínculo privado compartidos permiten a Azure Cognitive Search realizar conexiones salientes seguras para acceder a los recursos del cliente. Sin embargo, durante el proceso de administración (crear, eliminar o actualizar) de estos recursos pueden producirse algunos tipos diferentes de errores.

## <a name="creating-a-shared-private-link-resource"></a>Creación de un recurso de vínculo privado compartido

Hay cuatro pasos distintos implicados en la creación de un recurso de vínculo privado compartido:

1. El cliente invoca la [API CreateOrUpdate](/rest/api/searchmanagement/sharedprivatelinkresources/createorupdate) del plano de administración en el proveedor de recursos (RP) de Azure Cognitive Search con detalles del recurso de vínculo privado compartido que se va a crear.

2. El RP de Azure Cognitive Search valida la solicitud y, si se valida correctamente, inicia una operación asincrónica de Azure Resource Manager (cuyo progreso puede consultar el cliente).

3. Azure Cognitive Search consulta la finalización de la operación (que normalmente tarda unos minutos). En este momento, el recurso de vínculo privado compartido tendría un estado de aprovisionamiento de "Actualización en curso".

4. Una vez que la operación se completa correctamente, se crea un punto de conexión privado (junto con las zonas DNS y las asignaciones). En este momento, si el cliente consulta el estado del recurso de vínculo privado compartido, tendría un estado de aprovisionamiento de "Operación correcta".

![Pasos necesarios para crear recursos de vínculo privado compartidos ](media\troubleshoot-shared-private-link-resources\shared-private-link-states.png)

A continuación se enumeran algunos errores comunes que se producen durante la fase de creación.

### <a name="request-validation-failures"></a>Errores de validación de la solicitud

+ SKU no admitida: solo se pueden crear recursos de vínculo privado compartido para las SKU de pago, no se admiten servicios de nivel gratuito.

+ Validación de nombres: los nombres de los recursos de vínculo privado compartidos están restringidos solo a un determinado conjunto de caracteres. Si el nombre del recurso contiene caracteres no válidos, no se aceptará la solicitud para crear el recurso.
Las reglas para asignar un nombre a un recurso de vínculo privado compartido son:
  
  + La longitud debe estar entre 1 y 60 caracteres.
  + Solo debe contener caracteres alfanuméricos o caracteres de subrayado (_), punto (.) o guion (-).

+ Validación de `groupId`: el elemento `groupId` especificado como parte de la solicitud para crear un recurso de vínculo privado compartido debe coincidir (en ortografía y mayúsculas y minúsculas) con la tabla siguiente:

| Recurso de Azure | Identificador de grupo | Primera versión de API disponible |
| --- | --- | --- |
| Azure Storage: Blob (o) ADLS Gen2 | `blob`| `2020-08-01` |
| Azure Storage: Tablas | `table`| `2020-08-01` |
| Azure Cosmos DB: SQL API | `Sql`| `2020-08-01` |
| Azure SQL Database | `sqlServer`| `2020-08-01` |
| Azure Database for MySQL (versión preliminar) | `mysqlServer`| `2020-08-01-Preview` |
| Azure Key Vault | `vault` | `2020-08-01` |
| Azure Functions (versión preliminar) | `sites` | `2020-08-01-Preview` |

Los recursos marcados con "(versión preliminar)" solo están disponibles en las versiones preliminares de la API del plano de administración y aún no están disponibles con carácter general. Cualquier otro valor de `groupId` (o el uso de un elemento `groupId` en una versión de API que no lo admita) produciría un error de validación.

+ Validación de tipos de`privateLinkResourceId`: de forma similar a `groupId`, Azure Cognitive Search valida que se haya especificado el tipo de recurso "correcto" en el elemento `privateLinkResourceId`. Los siguientes son tipos de recursos válidos:

| Recurso de Azure | Tipo de recurso | Primera versión de API disponible |
| --- | --- | --- |
| Azure Storage | `Microsoft.Storage/storageAccounts`| `2020-08-01` |
| Azure Cosmos DB | `Microsoft.DocumentDb/databaseAccounts`| `2020-08-01` |
| Azure SQL Database | `Microsoft.Sql/servers`| `2020-08-01` |
| Azure Database for MySQL (versión preliminar) | `Microsoft.DBforMySQL/servers`| `2020-08-01-Preview` |
| Azure Key Vault | `Microsoft.KeyVault/vaults` | `2020-08-01` |
| Azure Functions (versión preliminar) | `Microsoft.Web/sites` | `2020-08-01-Preview` |

Además, el elemento `groupId` especificado debe ser válido para el tipo de recurso especificado. Por ejemplo, un elemento `groupId` con el valor "blob" es válido para el tipo "Microsoft.Storage/storageAccounts", pero no se puede usar con ningún otro tipo de recurso. Para una versión determinada de la API de administración de Azure Cognitive Search, los clientes pueden averiguar los detalles del tipo de recurso y de los elementos `groupId` admitidos mediante la [API List supported](/rest/api/searchmanagement/privatelinkresources/listsupported).

+ Cumplimiento del límite de cuota: los servicios de Azure Cognitive Search tienen cuotas impuestas en el número de recursos de vínculo privado compartidos que se pueden crear y en el número de diversos tipos de recursos de destino que se usan (en función de `groupId`). Estos se documentan en la sección [Límites de los recursos de vínculo privado compartidos](search-limits-quotas-capacity.md#shared-private-link-resource-limits) de la página de límites del servicio Azure Cognitive Search.

### <a name="azure-resource-manager-deployment-failures"></a>Errores de implementación de Azure Resource Manager

Una vez que Azure Cognitive Search ha aceptado la solicitud para crear un recurso de vínculo privado compartido, la implementación de Azure Resource Manager que inicia también puede producir un error por varios motivos. En todos los casos, cuando los clientes consulten el estado de la operación asincrónica (que se describe [aquí](search-indexer-howto-access-private.md#step-1-create-a-shared-private-link-resource-to-the-storage-account)), se mostrará un mensaje de error adecuado y los detalles disponibles.

Los recursos de vínculo privado compartidos que hayan tenido errores en la implementación de Azure Resource Manager se mostrarán en las llamadas de API [List](/rest/api/searchmanagement/sharedprivatelinkresources/listbyservice) y [Get](/rest/api/searchmanagement/sharedprivatelinkresources/get), pero tendrán el "estado de aprovisionamiento" `Failed` (Con errores). Una vez que se haya determinado el motivo del error de implementación de Azure Resource Manager, elimine el recurso en estado `Failed` y vuelva a crearlo después de aplicar la resolución adecuada a partir de la tabla siguiente.

| Motivo del error de implementación | Descripción | Solución |
| --- | --- | --- |
| El proveedor de recursos de red no está registrado en la suscripción del recurso de destino. | Se crea un punto de conexión privado (y las asignaciones DNS asociadas) para el recurso de destino (cuenta de almacenamiento, CosmosDB, SQL Server, etc.) mediante el proveedor de recursos (RP) `Microsoft.Network`. Si la suscripción que hospeda el recurso de destino ("suscripción de destino") no está registrada con el RP `Microsoft.Network`, se puede producir un error en la implementación de Azure Resource Manager. | Los clientes deben registrar este RP en la suscripción de destino. Normalmente, esto se puede hacer mediante Azure Portal, PowerShell o la CLI, como se documenta en [esta guía](../azure-resource-manager/management/resource-providers-and-types.md). |
| Elemento `groupId` no válido para el recurso de destino | Cuando se crean cuentas de CosmosDB, los clientes pueden especificar el tipo de API para la cuenta de base de datos. Aunque CosmosDB ofrece varios tipos de API diferentes, Azure Cognitive Search solo admite "Sql" como valor de `groupId` para los recursos de vínculo privado compartidos. Cuando se crea un recurso de vínculo privado compartido de tipo "Sql" para un elemento `privateLinkResourceId` que apunta a una cuenta de base de datos que no es SQL, se producirá un error en la implementación de Azure Resource Manager debido a la falta de coincidencia de `groupId`. El identificador de recurso de Azure de una cuenta de CosmosDB no es suficiente para determinar el tipo de API que se está utilizando. Azure Cognitive Search intenta crear el punto de conexión privado, que CosmosDb deniega. | Los clientes deben asegurarse de que el elemento `privateLinkResourceId` del recurso de CosmosDb especificado sea para una cuenta de base de datos del tipo de API "Sql". |
| Recurso de destino no encontrado | La existencia del recurso de destino especificado en `privateLinkResourceId` solo se comprueba durante el inicio de la implementación de Azure Resource Manager. Si el recurso de destino ya no está disponible, se producirá un error en la implementación. | El cliente debe asegurarse de que el recurso de destino esté presente en la suscripción y el grupo de recursos especificados y de que no se mueva ni elimine. |
| Errores transitorios y otros errores | La implementación de Azure Resource Manager puede producir un error si se produce una interrupción en la infraestructura o debido a otros motivos inesperados. Esto debe ser poco frecuente y normalmente indica un estado transitorio. | Vuelva a intentar crear este recurso más adelante. Si el problema persiste, póngase en contacto con el equipo de soporte técnico de Azure. |

### <a name="resource-stuck-in-updating-or-incomplete-state"></a>Recurso bloqueado en estado "Actualización en curso" o "Incompleto"

Normalmente, un recurso de vínculo privado compartido debe pasar a un estado terminal (`Succeeded` o `Failed`) en unos minutos después de que el RP de Azure Cognitive Search haya aceptado la solicitud.

En raras circunstancias, Azure Cognitive Search puede no marcar correctamente el estado del recurso de vínculo privado compartido en un estado terminal (`Succeeded` o `Failed`). Esto suele ocurrir debido a un error inesperado o catastrófico en el RP de Azure Cognitive Search. Los recursos de vínculo privado compartidos cambian automáticamente al estado `Failed` si han estado "bloqueados" en un estado no terminal durante más de 8 horas.

Si observa que el recurso de vínculo privado compartido no ha pasado a un estado terminal, espere 8 horas para asegurarse de que pasa al estado `Failed` antes de eliminarlo y volver a crearlo. Como alternativa, en lugar de esperar, puede intentar crear otro recurso de vínculo privado compartido con otro nombre (manteniendo todos los demás parámetros iguales).

## <a name="updating-a-shared-private-link-resource"></a>Actualización de un recurso de vínculo privado compartido

Un recurso de vínculo privado compartido existente se puede actualizar mediante la [API Create or Update](/rest/api/searchmanagement/sharedprivatelinkresources/createorupdate). El RP de Azure Cognitive Search solo permite actualizaciones restringidas en el recurso de vínculo privado compartido: solo se puede modificar el mensaje de solicitud mediante esta API.

+ No es posible actualizar ninguna de las propiedades "principales" de un recurso de vínculo privado compartido existente (como `privateLinkResourceId` o `groupId`) y esto siempre será incompatible. Si es necesario cambiar cualquier otra propiedad que no sea el mensaje de solicitud, recomendamos a los clientes que eliminen y vuelvan a crear el recurso de vínculo privado compartido.

+ El intento de actualizar el mensaje de solicitud de un recurso de vínculo privado compartido solo es posible si ha alcanzado el estado de aprovisionamiento `Succeeded`.

## <a name="deleting-a-shared-private-link-resource"></a>Eliminación de un recurso de vínculo privado compartido

Los clientes pueden eliminar un recurso de vínculo privado compartido existente mediante la [API Delete](/rest/api/searchmanagement/sharedprivatelinkresources/delete). De forma similar al proceso de creación (o de actualización), también se trata de una operación asincrónica con cuatro pasos:

1. El cliente solicita al RP de Azure Cognitive Search la eliminación del recurso de vínculo privado compartido.

2. El RP de Azure Cognitive Search valida que el recurso exista y se encuentre en un estado válido para su eliminación. Si es así, inicia una operación de eliminación de Azure Resource Manager para quitar el recurso.

3. Azure Cognitive Search consulta la finalización de la operación (que normalmente tarda unos minutos). En este momento, el recurso de vínculo privado compartido tendría un estado de aprovisionamiento de "Eliminación en curso".

4. Una vez que la operación se completa correctamente, se quitan el punto de conexión privado de respaldo y las asignaciones de DNS asociadas. El recurso no se mostrará como parte de la operación [List](/rest/api/searchmanagement/sharedprivatelinkresources/listbyservice) y si se intenta una operación [Get](/rest/api/searchmanagement/sharedprivatelinkresources/get) en este recurso, se mostrará un error 404 No encontrado.

![Pasos necesarios para eliminar recursos de vínculo privado compartidos ](media\troubleshoot-shared-private-link-resources\shared-private-link-delete-states.png)

A continuación se enumeran algunos errores comunes que se producen durante la fase de eliminación.

| Tipo de error | Descripción | Solución |
| --- | --- | --- |
| El recurso está en estado no terminal | No se puede eliminar un recurso de vínculo privado compartido que no esté en un estado terminal (`Succeeded` o `Failed`). Es posible (poco frecuente) que un recurso de vínculo privado compartido se bloquee en un estado no terminal durante un máximo de 8 horas. | Espere hasta que el recurso haya alcanzado un estado terminal y vuelva a intentar la solicitud de eliminación. |
| Error en la operación de eliminación con el error "Conflicto&quot; | La operación de Azure Resource Manager para eliminar un recurso de vínculo privado compartido llega al proveedor de recursos del recurso de destino especificado en `privateLinkResourceId` (&quot;RP de destino") antes de poder quitar el punto de conexión privado y las asignaciones de DNS. Los clientes pueden usar [bloqueos de recursos de Azure](../azure-resource-manager/management/lock-resources.md) para evitar cambios en sus recursos. Cuando Azure Resource Manager accede al RP de destino, requiere que el RP de destino modifique el estado del recurso de destino (para quitar los detalles sobre el punto de conexión privado de sus metadatos). Cuando el recurso de destino tiene un bloqueo configurado (o su grupo de recursos o suscripción), se produce un error en la operación de Azure Resource Manager con el error "Conflicto" (y los detalles adecuados). El recurso de vínculo privado compartido no se eliminará. | Los clientes deben quitar el bloqueo en el recurso de destino antes de reintentar la operación de eliminación. **Nota:** Este problema también se puede producir cuando los clientes intentan eliminar un servicio de Azure Cognitive Search con recursos de vínculo privado compartidos que apuntan a recursos de destino "bloqueados". |
| Error de la operación de eliminación | La operación de eliminación asincrónica de Azure Resource Manager puede producir un error en raras ocasiones. Cuando se produce un error en esta operación, al consultar el estado de la operación asincrónica se mostrará a los clientes un mensaje de error y los detalles adecuados. | Vuelva a intentar la operación más adelante o póngase en contacto con el equipo de soporte técnico de Azure si el problema persiste.
| Recurso bloqueado en el estado "Eliminación en curso" | En raras ocasiones, un recurso de vínculo privado compartido podría estar bloqueado en el estado "Eliminación en curso" durante un máximo de 8 horas, probablemente debido a algún error catastrófico en el RP de Azure Cognitive Search. | Espere 8 horas, después de las cuales el recurso pasaría al estado `Failed` y, a continuación, vuelva a emitir la solicitud.|

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los recursos de vínculo privado compartidos y cómo usarlos para proteger el acceso al contenido protegido.

+ [Establecimiento de conexiones del indexador a través de un punto de conexión privado](search-indexer-howto-access-private.md)
+ [Referencia de API de REST](/rest/api/searchmanagement/sharedprivatelinkresources)