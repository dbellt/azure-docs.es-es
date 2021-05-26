---
title: Información sobre la protección del acceso a los datos de Azure Cosmos DB
description: Obtenga información sobre los conceptos de control de acceso en Azure Cosmos DB, incluidas las claves principales, las claves de solo lectura, los usuarios y los permisos.
author: thomasweiss
ms.author: thweiss
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/25/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 7a3d7a3892e5d90e4ee8cc4e5b80df768fc138d7
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378811"
---
# <a name="secure-access-to-data-in-azure-cosmos-db"></a>Protección del acceso a los datos de Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En este artículo se proporciona información general sobre el control de acceso de Azure Cosmos DB.

Azure Cosmos DB proporciona tres maneras de controlar el acceso a los datos.

| Tipo de control de acceso | Características |
|---|---|
| [Claves principales](#primary-keys) | Secreto compartido que permite cualquier operación de administración o de datos. Se incluye en las variantes de lectura y escritura y de solo lectura. |
| [Control de acceso basado en rol](#rbac) (versión preliminar) | Modelo de permisos específico basado en roles mediante identidades de Azure Active Directory (AAD) para la autenticación. |
| [Tokens de recursos](#resource-tokens)| Modelo de permisos específico basado en permisos y usuarios nativos de Azure Cosmos DB. |

## <a name="primary-keys"></a>Claves principales <a id="primary-keys"></a>

Las claves principales proporcionan acceso a todos los recursos administrativos de la cuenta de base de datos. Cada cuenta consta de dos claves principales: una principal y una secundaria. El fin de las claves dobles es que se pueda regenerar, o distribuir claves, lo que proporciona un acceso continuo a su cuenta y datos. Para obtener más información sobre las claves principales, consulte el artículo [Seguridad de bases de datos](database-security.md#primary-keys).

### <a name="key-rotation"></a>Rotación de claves <a id="key-rotation"></a>

El proceso de rotación de la clave principal es simple. 

1. Vaya a Azure Portal para recuperar la clave secundaria.
2. Reemplace la clave principal por su clave secundaria en la aplicación. Asegúrese de que todos los clientes de Cosmos DB en todas las implementaciones se hayan reiniciado rápidamente y de que comenzarán a usar la clave actualizada.
3. Rote la clave principal en Azure Portal.
4. Valide que la nueva clave principal funcione en todos los recursos. El proceso de rotación de claves puede tardar entre menos de un minuto y unas horas, en función del tamaño de la cuenta de Cosmos DB.
5. Reemplace la clave secundaria por la nueva clave principal.

:::image type="content" source="./media/secure-access-to-data/nosql-database-security-master-key-rotate-workflow.png" alt-text="Rotación de la clave principal en Azure Portal: demostración de la seguridad de bases de datos NoSQL" border="false":::

### <a name="code-sample-to-use-a-primary-key"></a>Ejemplo de código para usar una clave principal

El ejemplo de código siguiente ilustra cómo usar un punto de conexión y la clave principal de la cuenta de Cosmos DB para crear una instancia de DocumentClient y una base de datos:

```csharp
//Read the Azure Cosmos DB endpointUrl and authorization keys from config.
//These values are available from the Azure portal on the Azure Cosmos DB account blade under "Keys".
//Keep these values in a safe and secure location. Together they provide Administrative access to your Azure Cosmos DB account.

private static readonly string endpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
private static readonly string authorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];

CosmosClient client = new CosmosClient(endpointUrl, authorizationKey);
```

El ejemplo de código siguiente ilustra cómo usar un punto de conexión y la clave principal de la cuenta de Azure Cosmos DB para crear una instancia de un objeto `CosmosClient`:

:::code language="python" source="~/cosmosdb-python-sdk/sdk/cosmos/azure-cosmos/samples/access_cosmos_with_resource_token.py" id="configureConnectivity":::

## <a name="role-based-access-control"></a><a id="rbac"></a> Control de acceso basado en rol

Azure Cosmos DB expone un sistema de control de acceso basado en roles integrado que le permite:

- Autenticar las solicitudes de datos con una identidad de Azure Active Directory (Azure AD).
- Autorizar las solicitudes de datos con un modelo de permisos basado en roles específico.

RBAC de Azure Cosmos DB es el método de control de acceso ideal en situaciones en las que:

- No quiere usar un secreto compartido como la clave principal y prefiere depender de un mecanismo de autenticación basado en tokens.
- Quiere usar identidades de Azure AD para autenticar las solicitudes.
- Necesita un modelo de permisos específico para restringir estrechamente qué operaciones de base de datos pueden realizar las identidades.
- Desea materializar las directivas de control de acceso como "roles" que puede asignar a varias identidades.

Consulte [Configuración del control de acceso basado en rol para la cuenta de Azure Cosmos DB](how-to-setup-rbac.md) para obtener más información sobre RBAC de Azure Cosmos DB.

## <a name="resource-tokens"></a>Tokens de recursos <a id="resource-tokens"></a>

Los tokens de recursos proporcionan acceso a los recursos de la aplicación en una base de datos. Los tokens de recursos:

- Proporcionan acceso a contenedores, claves de partición, documentos, datos adjuntos, procedimientos almacenados, desencadenadores y UDF concretos.
- Se crean cuando a un [usuario](#users) se le conceden [permisos](#permissions) para un recurso concreto.
- Se vuelven a crear cuando una llamada POST, GET o PUT aplica una acción a un recurso de permiso.
- Usan un token de recurso de hash construido específicamente para el usuario, el recurso y el permiso.
- Está limitado por un período de validez personalizable. El intervalo de tiempo válido predeterminado es una hora. Sin embargo, la vigencia del token puede especificarse explícitamente hasta un máximo de cinco horas.
- Proporcionan una alternativa segura a proporcionar la clave principal.
- Permiten que los clientes lean, escriban y eliminen recursos de la cuenta de Cosmos DB en función de los permisos que se les haya otorgado.

Si quiere proporcionar acceso a los recursos de su cuenta de Cosmos DB a un cliente que no es de confianza con la clave principal, puede usar un token de recurso (mediante la creación de usuarios y permisos de Cosmos DB).  

Los tokens de recurso de Cosmos DB ofrecen una alternativa segura que permite a los clientes leer, escribir y eliminar recursos de la cuenta de Cosmos DB en función de los permisos que se les hayan concedido y sin necesidad de una clave principal o de solo lectura.

Este es un patrón de diseño típico para solicitar, generar y entregar tokens de recursos a los clientes:

1. Se configura un servicio de nivel intermedio para que una aplicación móvil pueda usarlo para compartir fotos del usuario.
2. Dicho servicio tiene la clave principal de la cuenta de Cosmos DB.
3. La aplicación fotográfica se instala en los dispositivos móviles de los usuarios finales.
4. Al iniciar sesión, dicha aplicación establece la identidad del usuario con el servicio de nivel intermedio. Este mecanismo de establecimiento de identidad depende únicamente de la aplicación.
5. Una vez establecida la identidad, el servicio de nivel intermedio solicita permisos en función de esta.
6. El servicio de nivel intermedio reenvía un token de recurso a la aplicación de teléfono.
7. La aplicación de teléfono puede seguir usando el token de recurso para obtener acceso directo a los recursos de Cosmos DB con los permisos definidos por el token de recurso y para el intervalo permitido por dicho token.
8. Cuando el token de recurso expira, las solicitudes posteriores reciben un mensaje 401 de excepción no autorizada.  En este punto, la aplicación de teléfono restablece la identidad y solicita un nuevo token de recurso.

    :::image type="content" source="./media/secure-access-to-data/resourcekeyworkflow.png" alt-text="Flujo de trabajo de tokens de recursos de Azure Cosmos DB" border="false":::

Las bibliotecas cliente de Cosmos DB nativas controlan la generación y administración de los tokens de recursos; sin embargo, si se usa REST, debe construir los encabezados de solicitud o autenticación. Para más información sobre cómo crear encabezados de autenticación para REST, consulte [Control de acceso en recursos de Cosmos DB](/rest/api/cosmos-db/access-control-on-cosmosdb-resources) o el código fuente de nuestros [.NET SDK](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos/src/Authorization/AuthorizationHelper.cs) o [Node.js SDK](https://github.com/Azure/azure-cosmos-js/blob/master/src/auth.ts).

Para ver un ejemplo de un servicio de nivel intermedio que se usa para generarlo o los tokens de recursos del agente, consulte la [aplicación ResourceTokenBroker](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin/UserItems/ResourceTokenBroker/ResourceTokenBroker/Controllers).

### <a name="users"></a>Usuarios<a id="users"></a>

Los usuarios de Azure Cosmos DB están asociados a una base de datos de Cosmos.  Cada base de datos puede contener cero, o más, usuarios de Cosmos DB. En el siguiente código de ejemplo se muestra cómo crear un usuario de Cosmos DB mediante el [SDK de .NET v3 de Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement).

```csharp
//Create a user.
Database database = benchmark.client.GetDatabase("SalesDatabase");

User user = await database.CreateUserAsync("User 1");
```

> [!NOTE]
> Cada usuario de Cosmos DB tiene un método ReadAsync() que se puede usar para recuperar la lista de [permisos](#permissions) asociados al usuario.

### <a name="permissions"></a>Permisos<a id="permissions"></a>

Un recurso de permiso está asociado a un usuario y está asignado en el contenedor, así como en el nivel de clave de partición. Cada usuario puede contener cero o más permisos. Un recurso de permiso proporciona acceso a un token de seguridad que el usuario necesita al intentar acceder a un contenedor específico o a los datos de una clave de partición específica. Hay dos niveles de acceso disponibles que puede proporcionar un recurso de permiso:

- Todo: el usuario tiene pleno permiso en el recurso.
- Lectura: el usuario solo puede leer el contenido del recurso, pero no realizar operaciones de escritura, actualización o eliminación en este.

> [!NOTE]
> Para ejecutar procedimientos almacenados, el usuario debe tener el permiso "Todo" en el contenedor donde se va a ejecutar el procedimiento almacenado.

Si habilita los [registros de diagnóstico en solicitudes del plano de datos](cosmosdb-monitor-resource-logs.md), se registran las dos propiedades siguientes correspondientes al permiso:

* **resourceTokenPermissionId**: esta propiedad indica el identificador del permiso del token de recurso que ha especificado. 

* **resourceTokenPermissionMode**: esta propiedad indica el modo de permiso que estableció al crear el token de recurso. El modo de permiso puede tener valores como "all" o "read".

#### <a name="code-sample-to-create-permission"></a>Ejemplo de código para crear permisos

El código de ejemplo siguiente muestra cómo crear un recurso de permisos, leer el token de dicho recurso y asociar los permisos al [usuario](#users) creado anteriormente.

```csharp
// Create a permission on a container and specific partition key value
Container container = client.GetContainer("SalesDatabase", "OrdersContainer");
user.CreatePermissionAsync(
    new PermissionProperties(
        id: "permissionUser1Orders",
        permissionMode: PermissionMode.All,
        container: benchmark.container,
        resourcePartitionKey: new PartitionKey("012345")));
```

#### <a name="code-sample-to-read-permission-for-user"></a>Ejemplo de código para los permisos de lectura del usuario

En el siguiente fragmento de código se muestra cómo recuperar el permiso asociado al usuario creado anteriormente y crear una instancia de un nuevo elemento CosmosClient en nombre del usuario, en el ámbito de una única clave de partición.

```csharp
//Read a permission, create user client session.
PermissionProperties permissionProperties = await user.GetPermission("permissionUser1Orders")

CosmosClient client = new CosmosClient(accountEndpoint: "MyEndpoint", authKeyOrResourceToken: permissionProperties.Token);
```

## <a name="differences-between-rbac-and-resource-tokens"></a>Diferencias entre RBAC y los tokens de recursos

| Asunto | RBAC | Tokens de recursos |
|--|--|--|
| Authentication  | Con Azure Active Directory (Azure AD) | Basado en los usuarios de Azure Cosmos DB nativos<br>La integración de tokens de recursos con Azure AD requiere trabajo adicional para unir identidades de Azure AD y usuarios de Azure Cosmos DB. |
| Authorization | Basada en roles: las definiciones de roles asignan acciones permitidas y se pueden asignar a varias identidades. | Basada en permisos: para cada usuario de Azure Cosmos DB se debe asignar permisos de acceso a datos. |
| Ámbito del token | Un token de AAD lleva la identidad del solicitante. Esta identidad se compara con todas las definiciones de roles asignadas para realizar la autorización. | Un token de recurso lleva el permiso concedido a un usuario de Azure Cosmos DB específico en un recurso de Azure Cosmos DB específico. Las solicitudes de autorización en diferentes recursos pueden requerir distintos tokens. |
| Actualización de tokens | ¿Actualizan automáticamente los SDK de Azure Cosmos DB el token de Azure AD cuando este expira? | No se admite la actualización del token del recurso. Cuando expira un token del recurso, es necesario emitir uno nuevo. |

## <a name="add-users-and-assign-roles"></a>Incorporación de usuarios y asignación de roles

Para agregar acceso de lectura en la cuenta de Azure Cosmos DB a su cuenta de usuario, necesita que un propietario de la suscripción realice los pasos siguientes en Azure Portal.

1. Vaya a Azure Portal y seleccione su cuenta de Azure Cosmos DB.
2. Haga clic en la pestaña **Control de acceso (IAM)** y, después, haga clic en **+ Add role assignment** (+ Agregar asignación de roles).
3. En el panel **Add role assignment** (Agregar asignación de roles), en el cuadro **Rol**, seleccione **Rol de lector de la cuenta de Cosmos DB**.
4. En el cuadro **Asignar acceso a**, seleccione **usuario de Azure AD, grupo o aplicación**.
5. Seleccione el usuario, el grupo o la aplicación en el directorio al que desea conceder acceso.  Puede buscar en el directorio con los nombres para mostrar, dirección de correo electrónico o identificadores de objeto.
    El usuario, el grupo o la aplicación seleccionados aparecen en la lista de los miembros seleccionados.
6. Haga clic en **Save**(Guardar).

La entidad ahora puede leer los recursos de Azure Cosmos DB.

## <a name="delete-or-export-user-data"></a>Eliminación o exportación de datos de usuario

Como servicio de base de datos, Azure Cosmos DB le permite buscar, seleccionar, modificar y eliminar los datos que se encuentran en la base de datos o en los contenedores. No obstante, es responsabilidad suya usar las API proporcionadas y definir la lógica necesaria para buscar y borrar datos personales, si es necesario. Cada API multimodelo (SQL, MongoDB, Gremlin, Cassandra, Table) proporciona SDK de diferentes lenguajes que contienen métodos para buscar y eliminar datos basados en predicados personalizados. También puede habilitar la característica [período de vida (TTL)](time-to-live.md) para eliminar datos automáticamente después de un período especificado, sin incurrir en ningún costo adicional.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre la seguridad de las bases de datos de Cosmos, consulte [Cosmos DB: seguridad de bases de datos](database-security.md).
- Para aprender a construir tokens de autorización de Azure Cosmos DB, consulte [Access Control on Azure Cosmos DB Resources](/rest/api/cosmos-db/access-control-on-cosmosdb-resources) (Control de acceso en recursos de Azure Cosmos DB).
- Ejemplos de administración de usuarios con usuarios y permisos, [ejemplos de administración de usuarios del SDK de .NET v3](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement/UserManagementProgram.cs)
