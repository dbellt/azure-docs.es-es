---
title: Conexiones del indexador a través de un punto de conexión privado
titleSuffix: Azure Cognitive Search
description: Configure las conexiones del indexador para acceder a través de un punto de conexión privado al contenido de recursos de Azure protegidos.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: 0de817d2d18105b3f1a27ccd938f85bc62504867
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108770500"
---
# <a name="make-indexer-connections-through-a-private-endpoint"></a>Establecimiento de conexiones del indexador a través de un punto de conexión privado

Muchos recursos de Azure, como las cuentas de Azure Storage, pueden configurarse para que acepten las conexiones de una lista de redes virtuales y rechacen las conexiones externas cuyo origen sea una red pública. Si usa un indexador para indexar datos en Azure Cognitive Search y el origen de datos se encuentra en una red privada, puede crear una [conexión de punto de conexión privado](../private-link/private-endpoint-overview.md) de salida para acceder a los datos.

El método de conexión de este indexador está sujeto a los dos requisitos siguientes:

+ El recurso de Azure que proporciona contenido o código debe registrarse previamente en el servicio [Azure Private Link](https://azure.microsoft.com/services/private-link/).

+ El servicio Azure Cognitive Search debe estar en el nivel Básico o superior. Algunas características no están disponibles en el nivel Gratis. Además, si el indexador tiene un conjunto de aptitudes, el nivel debe ser Estándar 2 (S2) o superior. Para más información, consulte los [límites del servicio](search-limits-quotas-capacity.md#shared-private-link-resource-limits).

## <a name="shared-private-link-resources-management-apis"></a>API de administración de recursos de vínculo privado compartido

Los puntos de conexión privados de recursos protegidos que se crean a través de las API de Azure Cognitive Search se conocen como *recursos de vínculo privado compartido*. Eso se debe a que "comparte" el acceso a un recurso, como una cuenta de almacenamiento, que se ha integrado en el [servicio Azure Private Link](https://azure.microsoft.com/services/private-link/).

A través de su API REST de administración, Azure Cognitive Search proporciona una operación [CreateOrUpdate](/rest/api/searchmanagement/sharedprivatelinkresources/createorupdate) que puede usar para configurar el acceso desde un indexador de Azure Cognitive Search.

Puede crear conexiones de punto de conexión privado solo a algunos recursos mediante la versión preliminar de Search Management API (versión *2020-08-01-preview* o posterior), que se designa como *versión preliminar previa* en la tabla siguiente. Los recursos sin una designación *versión preliminar* se pueden crear con la versión de la API en versión preliminar o disponible con carácter general (*2020-08-01* o posterior).

En la siguiente tabla se enumeran los recursos de Azure para los que puede crear puntos de conexión privados de salida desde Azure Cognitive Search. Para crear un recurso compartido de vínculo privado, escriba los valores de **Id. de grupo** exactamente como están escritos en la API. Los valores distinguen mayúsculas de minúsculas.

| Recurso de Azure | Identificador de grupo |
| --- | --- |
| Azure Storage: Blob (o) ADLS Gen2 | `blob`|
| Azure Storage: Tablas | `table`|
| Azure Cosmos DB: SQL API | `Sql`|
| Azure SQL Database | `sqlServer`|
| Azure Database for MySQL (versión preliminar) | `mysqlServer`|
| Azure Key Vault | `vault` |
| Azure Functions (versión preliminar) | `sites` |

También puede consultar los recursos de Azure para los que se admiten conexiones de punto de conexión privado de salida mediante la [lista de API compatibles](/rest/api/searchmanagement/privatelinkresources/listsupported).

En el resto de este artículo, se usa una combinación de Azure Portal, o la [CLI de Azure](/cli/azure/), si lo prefiere, y [Postman](https://www.postman.com/), o cualquier otro cliente HTTP como [curl](https://curl.se/), si lo prefiere, para demostrar las llamadas API de REST.

> [!NOTE]
> Los ejemplos de este artículo se basan en los siguientes supuestos:
> * El nombre del servicio de búsqueda es _contoso-search_, que existe en el grupo de recursos _contoso_ de una suscripción con el identificador _00000000-0000-0000-0000-000000000000_. 
> * El identificador de recurso de este servicio de búsqueda es _/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search_.

En el resto de los ejemplos se muestra cómo se puede configurar el servicio _contoso-search_ para que sus indexadores puedan acceder a los datos de la cuenta de almacenamiento seguro _/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Storage/storageAccounts/contoso-storage_.

## <a name="secure-your-storage-account"></a>Protección de una cuenta de almacenamiento

Puede configurar la cuenta de almacenamiento para [permitir el acceso solo desde subredes específicas](../storage/common/storage-network-security.md#grant-access-from-a-virtual-network). En Azure Portal, si selecciona esta opción y deja el conjunto vacío, significa que no se permite el tráfico desde las redes virtual.

   ![Captura de pantalla del panel "Firewalls y redes virtuales", que muestra la opción de permitir el acceso a las redes seleccionadas. ](media\search-indexer-howto-secure-access\storage-firewall-noaccess.png)

> [!NOTE]
> Puede usar el [enfoque de servicio de Microsoft de confianza](../storage/common/storage-network-security.md#trusted-microsoft-services) para omitir las restricciones de red virtual o IP en una cuenta de almacenamiento. También puede habilitar el servicio de búsqueda para acceder a los datos de la cuenta de almacenamiento. Para ello, consulte [Acceso a los datos de las cuentas de almacenamiento de forma segura mediante una excepción de servicio de confianza](search-indexer-howto-access-trusted-service-exception.md).
>
> Sin embargo, cuando se usa este enfoque, la comunicación entre Azure Cognitive Search y su cuenta de almacenamiento se produce mediante la dirección IP pública de la cuenta de almacenamiento, a través de la red troncal de Microsoft segura.

Los recursos de Private Link compartidos para el servicio Azure Cognitive Search se pueden administrar en Azure Portal. Vaya al servicio de búsqueda -> Redes -> Shared Private Access (Acceso privado compartido) para administrar estos recursos en el portal.

   ![Captura de pantalla del panel "Redes", en la que se muestra la hoja de administración de vínculos privados compartidos. ](media\search-indexer-howto-secure-access\shared-private-link-portal-blade.png)

### <a name="step-1-create-a-shared-private-link-resource-to-the-storage-account"></a>Paso 1: Crear un recurso de vínculo privado compartido a la cuenta de almacenamiento

Para solicitar a Azure Cognitive Search que cree una conexión de punto de conexión privado saliente a la cuenta de almacenamiento, en la hoja Shared Private Access (Acceso privado compartido), haga clic en "Add Shared Private Access" (Agregar acceso privado compartido). En el cuadro de diálogo que se abre a la derecha, puede elegir "Connect to an Azure resource in my directory" (Conectarse a un recurso de Azure en mi directorio) o "Connect to an Azure resource by resource ID or alias" (Conectarse a un recurso de Azure por identificador de recurso o alias).

Cuando se usa la primera opción (recomendada), el panel de diálogo le ayudará a elegir la cuenta de almacenamiento adecuada y le ayudará a rellenar otras propiedades, como el identificador de grupo del recurso y el tipo de recurso.

   ![Captura de pantalla del panel "Add Shared Private Access" (Agregar acceso privado compartido), en la que se muestra una experiencia guiada para crear un recurso de vínculo privado compartido. ](media\search-indexer-howto-secure-access\new-shared-private-link-resource.png)

Al usar la segunda opción, puede escribir manualmente el identificador de recurso de Azure de la cuenta de almacenamiento de destino y elegir el identificador de grupo adecuado (en este caso, "blob").

![Captura de pantalla del panel "Add Shared Private Access" (Agregar acceso privado compartido), en la que se muestra una experiencia manual para crear un recurso de vínculo privado compartido. ](media\search-indexer-howto-secure-access\new-shared-private-link-resource-manual.png)

Como alternativa, puede realizar la siguiente llamada API con la [CLI de Azure](/cli/azure/):

```dotnetcli
az rest --method put --uri https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe?api-version=2020-08-01 --body @create-pe.json
```

El contenido del archivo *create-pe.json*, que representa el cuerpo de la solicitud a la API, es el siguiente:

```json
{
      "name": "blob-pe",
      "properties": {
        "privateLinkResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Storage/storageAccounts/contoso-storage",
        "groupId": "blob",
        "requestMessage": "please approve"
      }
}
```

Se devuelve la respuesta `202 Accepted` en caso de éxito. El proceso de creación de un punto de conexión privado de salida es una operación de larga duración (asincrónica), Implica la implementación de los siguientes recursos:

+ Un punto de conexión privado asignado con una dirección IP privada con un estado `"Pending"`. La dirección IP privada se obtiene del espacio de direcciones que se asigna a la red virtual del entorno de ejecución del indexador privado específico del servicio de búsqueda. Tras la aprobación del punto de conexión privado, cualquier comunicación desde Azure Cognitive Search a la cuenta de almacenamiento se origina desde la dirección IP privada y un canal seguro de vínculo privado.

+ Una zona DNS privada para el tipo de recurso, que se basa en el valor de `groupId`. Mediante la implementación de este recurso se asegura de que todas las búsquedas DNS en el recurso privado utilizan la dirección IP asociada con el punto de conexión privado.

Asegúrese de especificar el valor de `groupId` correcto para el tipo de recurso para el que crea el punto de conexión privado. Cualquier discrepancia producirá un mensaje de respuesta incorrecta.

Como en todas las operaciones asincrónicas de Azure, la llamada a `PUT` devuelve un valor de encabezado `Azure-AsyncOperation` similar al siguiente:

`"Azure-AsyncOperation": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe/operationStatuses/08586060559526078782?api-version=2020-08-01"`

Este identificador URI se puede sondear periódicamente para obtener el estado de la operación.

Si usa Azure Portal para crear el recurso de vínculo privado compartido, el portal realizará automáticamente este sondeo (con el estado de aprovisionamiento de recursos marcado como "Actualizando").

![Captura de pantalla del panel "Add Shared Private Access" (Agregar acceso privado compartido), en la que se muestra la creación de recursos en curso. ](media\search-indexer-howto-secure-access\new-shared-private-link-resource-progress.png)

Una vez creado correctamente el recurso, recibirá una notificación del portal, y el estado de aprovisionamiento del recurso cambiará a "Correcto".

![Captura de pantalla del panel "Add Shared Private Access" (Agregar acceso privado compartido), en la que se muestra la creación de recursos completada. ](media\search-indexer-howto-secure-access\new-shared-private-link-resource-success.png)

Si usa la CLI, puede sondear el estado realizando una consulta manual del valor `Azure-AsyncOperationHeader`.

```dotnetcli
az rest --method get --uri https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe/operationStatuses/08586060559526078782?api-version=2020-08-01
```

Espere hasta que el estado de aprovisionamiento del recurso cambie a "Correcto" antes de continuar con los pasos siguientes.

### <a name="step-2a-approve-the-private-endpoint-connection-for-the-storage-account"></a>Paso 2a: Aprobar la conexión del punto de conexión privado para la cuenta de almacenamiento

> [!NOTE]
> En esta sección, se usa Azure Portal para recorrer el flujo de aprobación de un punto de conexión privado al almacenamiento. Como alternativa, puede usar la [API REST](/rest/api/storagerp/privateendpointconnections) que está disponible a través del proveedor de recursos de almacenamiento.
>
> Otros proveedores, como Azure Cosmos DB o Azure SQL Server, ofrecen API de proveedor de recursos de almacenamiento similares para administrar conexiones de punto de conexión privado.

1. En Azure Portal, seleccione la pestaña **Redes** de la cuenta de almacenamiento y vaya a **Conexiones de punto de conexión privado**. Una vez que la operación asincrónica se ha realizado correctamente, debe haber una solicitud de una conexión de punto de conexión privado con el mensaje de solicitud de la llamada API anterior.

   ![Captura de pantalla de Azure Portal en la que se muestra el panel "Conexiones de punto de conexión privado".](media\search-indexer-howto-secure-access\storage-privateendpoint-approval.png)

1. Seleccione el punto de conexión privado que Azure Cognitive Search ha creado. En la columna **Punto de conexión privado**, identifique la conexión del punto de conexión privado con el nombre que se especifica en la API anterior, seleccione **Aprobar** y escriba un mensaje adecuado. El contenido del mensaje no es significativo.

   Asegúrese de que la conexión del punto de conexión privado aparece como se muestra en la siguiente captura de pantalla. El estado puede tardar entre uno y dos minutos en actualizarse en el portal.

   ![Captura de pantalla de Azure Portal en la que se muestra un estado "Approved" el panel "Conexiones de punto de conexión privado".](media\search-indexer-howto-secure-access\storage-privateendpoint-after-approval.png)

Cuando se aprueba la solicitud de conexión de punto de conexión privado, el tráfico *puede* fluir a través del punto de conexión privado. Después de que se aprueba el punto de conexión privado, Azure Cognitive Search crea las asignaciones de zona DNS necesarias en la zona DNS que se crea para él.

### <a name="step-2b-query-the-status-of-the-shared-private-link-resource"></a>Paso 2b: Consultar el estado del recurso de vínculo privado compartido

Para confirmar que el recurso de vínculo privado compartido se ha actualizado después de la aprobación, vuelva a visitar la hoja "Shared Private Access" (Acceso privado compartido) del servicio Search en Azure Portal y compruebe el "Estado de conexión".

   ![Captura de pantalla de Azure Portal, que muestra un recurso de vínculo privado compartido como "Aprobado".](media\search-indexer-howto-secure-access\new-shared-private-link-resource-approved.png)

Como alternativa, también puede obtener el "Estado de conexión" mediante la [API GET](/rest/api/searchmanagement/sharedprivatelinkresources/get).

```dotnetcli
az rest --method get --uri https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe?api-version=2020-08-01
```

Esto devolvería un elemento JSON, donde el estado de conexión se mostraría como "estado" debajo de la sección "propiedades".

```json
{
      "name": "blob-pe",
      "properties": {
        "privateLinkResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Storage/storageAccounts/contoso-storage",
        "groupId": "blob",
        "requestMessage": "please approve",
        "status": "Approved",
        "resourceRegion": null,
        "provisioningState": "Succeeded"
      }
}

```

Si la opción "Estado de aprovisionamiento" (`properties.provisioningState`) del recurso es `Succeeded` y la opción "Estado de conexión" (`properties.status`) es `Approved`, significa que el recurso de vínculo privado compartido es funcional y el indexador se puede configurar para que se comunique a través del punto de conexión privado.

### <a name="step-3-configure-the-indexer-to-run-in-the-private-environment"></a>Paso 3: Configurar el indexador para que se ejecute en el entorno privado

> [!NOTE]
> Este paso se puede realizar antes de que se apruebe la conexión del punto de conexión privado. Hasta ese momento, cualquier indexador que intente comunicarse con un recurso seguro (como la cuenta de almacenamiento) terminará con un estado de error transitorio. Los nuevos indexadores no se crearán. En cuanto se apruebe la conexión del punto de conexión privado, los indexadores pueden acceder a la cuenta de almacenamiento privada.

1. [Cree un origen de datos](/rest/api/searchservice/create-data-source) que apunte a la cuenta de almacenamiento segura y a un contenedor adecuado dentro de la cuenta de almacenamiento. En la captura de pantalla siguiente se muestra esta solicitud en Postman.

   ![Captura de pantalla que muestra la creación de un origen de datos en la interfaz de usuario de Postman.](media\search-indexer-howto-secure-access\create-ds.png )

1. De forma similar [cree un índice](/rest/api/searchservice/create-index) y, opcionalmente, [cree un conjunto de aptitudes](/rest/api/searchservice/create-skillset) mediante la API REST.

1. [Cree un indexador](/rest/api/searchservice/create-indexer) que apunte al origen de datos, el índice y el conjunto de aptitudes que creó en el paso anterior. Además, fuerce a que el indexador se ejecute en el entorno de ejecución privado; para ello, establezca la propiedad de configuración `executionEnvironment` del indexador en `private`.

   ![Captura de pantalla que muestra la creación de un indexador en la interfaz de usuario de Postman.](media\search-indexer-howto-secure-access\create-idr.png)

   Después de que el indexador se cree correctamente debería empezar la indexación del contenido de la cuenta de almacenamiento a través de la conexión del punto de conexión privado. Puede supervisar el estado del indexador. Para ello, se utiliza [Indexer Status API](/rest/api/searchservice/get-indexer-status).

> [!NOTE]
> Si ya tiene indexadores, puede actualizarlos a través de [PUT API](/rest/api/searchservice/create-indexer). Para ello, debe establecer el valor de `executionEnvironment` en `private`.

## <a name="troubleshooting"></a>Solución de problemas

+ Si se produce un error al crear un indexador con el mensaje "Las credenciales del origen de datos no son válidas", significa que el estado de la conexión del punto de conexión privado aún no es *Approved* o que la conexión no es funcional. Para solucionar el problema: 
  + Obtenga el estado del recurso de vínculo privado compartido mediante [GET API](/rest/api/searchmanagement/sharedprivatelinkresources/get). Si el estado es *Approved*, compruebe el valor `properties.provisioningState` del recurso. Si el estado es `Incomplete`, significa que no se pudieron configurar algunas de las dependencias subyacentes del recurso. Una nueva emisión de la solicitud de `PUT` para volver a crear el recurso de vínculo privado compartido debería corregir el problema. Es posible que haya que volver a aprobarla. Vuelva a comprobar el estado del recurso para asegurarse de que se ha corregido el problema.

+ Si crea el indexador sin establecer su propiedad `executionEnvironment`, el proceso podría finalizar correctamente, pero su historial mostrará que sus ejecuciones no son correctas. Para solucionar el problema:
  + [Actualice el indexador](/rest/api/searchservice/update-indexer) para especificar el entorno de ejecución.

+ Si ha creado el indexador sin establecer la propiedad `executionEnvironment` y se ejecuta correctamente, significa que Azure Cognitive Search ha decidido que su entorno de ejecución es el entorno *privado* específico del servicio de búsqueda. Esto puede cambiar en función de los recursos que haya consumido el indexador y la carga del servicio de búsqueda, entre otros factores, y puede producir un error más adelante. Para solucionar el problema:
  + Se recomienda encarecidamente establecer la propiedad `executionEnvironment` en `private` para asegurarse de que no aparecerán errores en el futuro.

+ Si está viendo la página de redes del origen de datos en Azure Portal y selecciona un punto de conexión privado que creó para que el servicio Azure Cognitive Search acceda a este origen de datos, es posible que reciba un error *Sin acceso*. Se espera que esto sea así. Puede cambiar el estado de la solicitud de conexión en la página del portal del servicio de destino, pero para administrar aún más el recurso de vínculo privado compartido, debe ver el recurso de vínculo privado compartido en la página de red del servicio Search en Azure Portal.

[Cuotas y límites](search-limits-quotas-capacity.md) determinan el número de recursos de vínculo privado compartido que se pueden crear y que pueden depender de la SKU del servicio de búsqueda.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los puntos de conexión privados:

+ [Solución de problemas relacionados con los recursos compartidos de Private Link](troubleshoot-shared-private-link-resources.md)
+ [¿Qué son los puntos de conexión privados?](../private-link/private-endpoint-overview.md)
+ [Configuraciones de DNS necesarias para los puntos de conexión privados](../private-link/private-endpoint-dns.md)