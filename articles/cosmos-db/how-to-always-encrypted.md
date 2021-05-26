---
title: Use el cifrado del lado cliente con Always Encrypted para Azure Cosmos DB
description: Aprenda a utilizar el cifrado del lado cliente con Always Encrypted para Azure Cosmos DB
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/25/2021
ms.author: thweiss
ms.openlocfilehash: d8aaa34fe2e81515866c047c6a171b1bdb58ff3f
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388448"
---
# <a name="use-client-side-encryption-with-always-encrypted-for-azure-cosmos-db-preview"></a>Use el cifrado del lado cliente con Always Encrypted para Azure Cosmos DB (versión preliminar)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Always Encrypted es una característica diseñada para proteger la información confidencial, como números de tarjeta de crédito o números de identificación nacional (por ejemplo, los números de seguridad social de EE. UU.), almacenada en Azure Cosmos DB. Always Encrypted permite a los clientes cifrar información confidencial en aplicaciones cliente y nunca revela las claves de cifrado a la base de datos.

Always Encrypted ofrece las funcionalidades de cifrado del lado cliente a Azure Cosmos DB. Es posible que sea necesario cifrar los datos del lado del cliente en los siguientes escenarios:

- **Protección de datos confidenciales que tienen características de confidencialidad específicas**: Always Encrypted permite a los clientes cifrar datos confidenciales dentro de sus aplicaciones y nunca revelar los datos de texto sin formato o las claves de cifrado al servicio Azure Cosmos DB.
- **Implementar el control de acceso por propiedad**: dado que el cifrado se controla con claves que posee y administra el usuario desde Azure Key Vault, este puede aplicar directivas de acceso para controlar a qué propiedades confidenciales tienen acceso los clientes.

> [!IMPORTANT]
> Always Encrypted para Azure Cosmos DB está actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para obtener más información, consulte [Términos de uso complementarios de las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

Para empezar a usar la versión preliminar de Always Encrypted para Azure Cosmos DB, puede hacer lo siguiente:

- Use la versión 2.11.13.0 o posterior del [emulador local de Azure Cosmos DB](local-emulator.md).
- Rellene [este formulario](https://ncv.microsoft.com/poTcF52I6N) y solicite que la versión preliminar se habilite en su cuenta de Azure Cosmos DB.

> [!TIP]
> ¿Quiere hacer algún comentario con respecto a la versión preliminar de Always Encrypted para Azure Cosmos DB? Póngase en contacto con [azurecosmosdbcse@service.microsoft.com](mailto:azurecosmosdbcse@service.microsoft.com).

## <a name="concepts"></a>Conceptos

Always Encrypted para Azure Cosmos DB presenta nuevos conceptos que intervienen en la configuración del cifrado del lado cliente.

### <a name="encryption-keys"></a>Claves de cifrado

#### <a name="data-encryption-keys"></a>Claves de cifrado de datos

Con Always Encrypted, los datos se cifran con claves de cifrado de datos (DEK) que deben crearse con antelación. Estas claves de cifrado de datos se almacenan en el servicio Azure Cosmos DB y se definen en el nivel de base de datos, por lo que se pueden compartir entre varios contenedores. La creación de las claves de cifrado de datos se realiza en el lado cliente mediante el SDK de Azure Cosmos DB.

Puede hacer lo siguiente:

- Cree una clave de cifrado de datos por propiedad para cifrar, o
- utilice la misma clave para cifrar múltiples propiedades.

#### <a name="customer-managed-keys"></a>Claves administradas por el cliente

Antes de que se almacenen en Azure Cosmos DB, las claves de cifrado de datos se encapsulan mediante una clave administrada por el cliente (CMK). Al controlar el encapsulado y desencapsulado de las claves de cifrado de datos, las claves administradas por el cliente controlan eficazmente el acceso a los datos cifrados con sus claves de cifrado de datos correspondientes. El almacenamiento de las claves administradas por el cliente está diseñado como un modelo extensible o de complemento, con una implementación predeterminada que espera que se almacenen en Azure Key Vault.

:::image type="content" source="./media/how-to-always-encrypted/encryption-keys.png" alt-text="Claves de cifrado" border="true":::

### <a name="encryption-policy"></a>Directiva de cifrado

Similar a una [directiva de indexación](index-policy.md), una directiva de cifrado es una especificación de nivel de contenedor que describe cómo deben cifrarse las propiedades JSON. Esta directiva, que es inmutable, debe proporcionarse cuando se crea el contenedor. En la versión actual, no se puede actualizar la directiva de cifrado.

Para cada propiedad que quiera cifrar, la directiva de cifrado define lo siguiente:

- La ruta de acceso de la propiedad en forma de `/property`. Actualmente solo se admiten rutas de acceso de nivel superior; no se admiten rutas de acceso anidadas como `/path/to/property`.
- El identificador de la [clave de cifrado de datos](#data-encryption-keys) que se usará al cifrar y descifrar la propiedad.
- Un tipo de cifrado. Puede ser aleatorio o determinista.
- El algoritmo de cifrado que se usará al cifrar la propiedad. El algoritmo especificado puede invalidar el algoritmo definido al crear la clave si son compatibles.

> [!NOTE]
> Las siguientes propiedades no se pueden cifrar:
> - ID
> - La clave de partición del contenedor

#### <a name="randomized-vs-deterministic-encryption"></a>Cifrado aleatorio frente a cifrado determinista

El servicio de Azure Cosmos DB nunca ve el texto sin formato de las propiedades cifradas con Always Encrypted. Sin embargo, sigue siendo compatible con algunas funcionalidades de consulta sobre los datos cifrados, en función del tipo de cifrado que se utiliza para una propiedad. Always Encrypted es compatible con los dos tipos de cifrado siguientes:

- El **cifrado determinista** genera siempre el mismo valor cifrado para cualquier valor de texto sin formato determinado y configuración de cifrado. El uso del cifrado determinista permite que las consultas realicen filtros de igualdad en propiedades cifradas. Sin embargo, puede permitir a los atacantes averiguar información sobre los valores cifrados mediante el análisis de patrones en la propiedad cifrada. Esto sucede especialmente cuando hay un conjunto pequeño de posibles valores cifrados, como verdadero o falso, o región norte, sur, este u oeste.

- El **cifrado aleatorio** usa un método que cifra los datos de una manera menos predecible. El cifrado aleatorio es más seguro, pero evita que las consultas se filtren por propiedades cifradas.

## <a name="setup-azure-key-vault"></a>Configuración de Azure Key Vault

Para empezar a trabajar con Always Encrypted, lo primero que debe hacer es crear sus CMK en Azure Key Vault:

1. Cree una instancia nueva de Azure Key Vault o busque una que ya exista.
1. Cree una clave nueva en la sección **Claves**.
1. Una vez creada la clave, busque la versión actual y copie el identificador de clave completo:<br>`https://<my-key-vault>.vault.azure.net/keys/<key>/<version>`. Si omite la versión de la clave al final del identificador, se utiliza la versión más reciente de la clave.

A continuación, debe configurar el acceso del SDK de Azure Cosmos DB a su instancia de Azure Key Vault. Esta autenticación se realiza a través de una identidad de Azure Active Directory (AD). Aunque se podría usar cualquier tipo de identidad, lo más probable es que utilice la identidad de una aplicación de Azure AD o una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) como proxy entre el código de cliente y su instancia de Azure Key Vault. Siga estos pasos para usar una aplicación de Azure AD como proxy:

1. Cree una aplicación nueva y agregue un secreto de cliente tal y como se describe en [este inicio rápido](../active-directory/develop/quickstart-register-app.md).

1. Vuelva a su instancia de Azure Key Vault, vaya a la sección **Directivas de acceso** y agregue una nueva directiva:

   1. En **Permisos de clave**, seleccione **Obtener**, **Enumerar**, **Desencapsular clave**, **Encapsular clave**, **Comprobar** y **Firmar**.
   1. En **Seleccionar entidad de seguridad**, busque la aplicación de AAD que ha creado antes.

### <a name="protect-your-cmk-from-accidental-deletion"></a>Protección de la CMK frente a eliminaciones accidentales

Para asegurarse de que no pierde acceso a los datos cifrados después de una eliminación accidental de la CMK, se recomienda establecer dos propiedades en la instancia de Azure Key Vault: **Eliminación temporal** y **Protección de purgas**.

Si crea una nueva instancia de Azure Key Vault, habilite estas propiedades durante la creación:

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-prop.png" alt-text="Habilitación de la eliminación temporal y la protección de purgas para una nueva instancia de Azure Key Vault":::

Si usa una instancia de Azure Key Vault existente y desea verificar si estas propiedades estén habilitadas, puede consultar la sección **Propiedades** en Azure Portal. Si alguna de estas propiedades no está habilitada, consulte las secciones "Habilitar la eliminación temporal" y "Habilitación de la protección de purgas" en cualquiera de los siguientes artículos:

- [Uso de la eliminación temporal con PowerShell](../key-vault/general/key-vault-recovery.md)
- [Uso de la eliminación temporal con la CLI de Azure](../key-vault/general/key-vault-recovery.md)

## <a name="initialize-the-sdk"></a>Inicializar el SDK

> [!NOTE]
> Always Encrypted para Azure Cosmos DB es actualmente compatible con
> - el [paquete Microsoft.Azure.Cosmos.Encryption](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Encryption) de **.NET** y
> - el [paquete azure.cosmos.encryption](https://mvnrepository.com/artifact/com.azure/azure-cosmos-encryption) de **Java**.

Para utilizar Always Encrypted, debe adjuntar una instancia de `EncryptionKeyStoreProvider` a su instancia de SDK de Azure Cosmos DB. Este objeto se usa para interactuar con el almacén de claves que hospeda sus CMK. El proveedor de almacén de claves predeterminado para Azure Key Vault se denomina `AzureKeyVaultKeyStoreProvider`.

Los fragmentos de código siguientes muestran cómo usar la identidad de una aplicación de Azure AD con un secreto de cliente. Puede encontrar ejemplos de creación de diferentes tipos de clases `TokenCredential`

- [En .NET](/dotnet/api/overview/azure/identity-readme#credential-classes)
- [En Java](/java/api/overview/azure/identity-readme#credential-classes)

# <a name="net"></a>[.NET](#tab/dotnet)

> [!NOTE]
> En .NET, necesitará el [paquete adicional Microsoft.Data.Encryption.AzureKeyVaultProvider](https://www.nuget.org/packages/Microsoft.Data.Encryption.AzureKeyVaultProvider) para acceder a la clase `AzureKeyVaultKeyStoreProvider`.

```csharp
var tokenCredential = new ClientSecretCredential(
    "<aad-app-tenant-id>", "<aad-app-client-id>", "<aad-app-secret>");
var keyStoreProvider = new AzureKeyVaultKeyStoreProvider(tokenCredential);
var client = new CosmosClient("<connection-string>")
    .WithEncryption(keyStoreProvider);
```

# <a name="java"></a>[Java](#tab/java)

```java
TokenCredential tokenCredential = new ClientSecretCredentialBuilder()
    .authorityHost("https://login.microsoftonline.com")
    .tenantId("<aad-app-tenant-id>")
    .clientId("<aad-app-client-id>")
    .clientSecret("<aad-app-secret>")
    .build();
AzureKeyVaultKeyStoreProvider encryptionKeyStoreProvider =
    new AzureKeyVaultKeyStoreProvider(tokenCredential);
CosmosAsyncClient client = new CosmosClientBuilder()
    .endpoint("<endpoint>")
    .key("<primary-key>")
    .buildAsyncClient();
EncryptionAsyncCosmosClient encryptionClient =
    EncryptionAsyncCosmosClient.buildEncryptionAsyncClient(client, encryptionKeyStoreProvider);
```
---

## <a name="create-a-data-encryption-key"></a>Cree una clave de cifrado de datos

Para poder cifrar los datos en un contenedor, debe crearse una [clave de cifrado de datos](#data-encryption-keys) en la base de datos primaria. Esta operación se realiza mediante una llamada al método `CreateClientEncryptionKeyAsync`, se pasa

- un identificador de cadena que identificará de forma única la clave en la base de datos y
- el algoritmo de cifrado previsto para utilizar con la clave. Actualmente, solo se admite un algoritmo.
- El identificador de clave de la [CMK](#customer-managed-keys) almacenada en Azure Key Vault. Este parámetro pasa en un objeto genérico `EncryptionKeyWrapMetadata` donde `name` puede ser cualquier nombre descriptivo que quiera y el `value` debe ser el identificador de clave.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
var database = client.GetDatabase("my-database");
await database.CreateClientEncryptionKeyAsync(
    "my-key",
    DataEncryptionKeyAlgorithm.AEAD_AES_256_CBC_HMAC_SHA256,
    new EncryptionKeyWrapMetadata(
        keyStoreProvider.ProviderName,
        "akvKey",
        "https://<my-key-vault>.vault.azure.net/keys/<key>/<version>"));
```

# <a name="java"></a>[Java](#tab/java)

```java
EncryptionCosmosAsyncDatabase database =
    client.getEncryptedCosmosAsyncDatabase("my-database");
database.createClientEncryptionKey(
    "my-key",
    CosmosEncryptionAlgorithm.AEAES_256_CBC_HMAC_SHA_256,
    new EncryptionKeyWrapMetadata(
        "akvKey",
        "https://<my-key-vault>.vault.azure.net/keys/<key>/<version>"));
```
---

## <a name="create-a-container-with-encryption-policy"></a>Creación de un contenedor con directiva de cifrado

Especifique la directiva de cifrado de nivel de contenedor al crear el contenedor.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
var path1 = new ClientEncryptionIncludedPath
{
    Path = "/property1",
    ClientEncryptionKeyId = "my-key",
    EncryptionType = EncryptionType.Deterministic.ToString(),
    EncryptionAlgorithm = DataEncryptionKeyAlgorithm.AEAD_AES_256_CBC_HMAC_SHA256.ToString()
};
var path2 = new ClientEncryptionIncludedPath
{
    Path = "/property2",
    ClientEncryptionKeyId = "my-key",
    EncryptionType = EncryptionType.Randomized.ToString(),
    EncryptionAlgorithm = DataEncryptionKeyAlgorithm.AEAD_AES_256_CBC_HMAC_SHA256.ToString()
};
await database.DefineContainer("my-container", "/partition-key")
    .WithClientEncryptionPolicy()
    .WithIncludedPath(path1)
    .WithIncludedPath(path2)
    .Attach()
    .CreateAsync();
```

# <a name="java"></a>[Java](#tab/java)

```java
ClientEncryptionIncludedPath path1 = new ClientEncryptionIncludedPath();
path1.clientEncryptionKeyId = "my-key":
path1.path = "/property1";
path1.encryptionType = CosmosEncryptionType.DETERMINISTIC;
path1.encryptionAlgorithm = CosmosEncryptionAlgorithm.AEAES_256_CBC_HMAC_SHA_256;

ClientEncryptionIncludedPath path2 = new ClientEncryptionIncludedPath();
path2.clientEncryptionKeyId = "my-key":
path2.path = "/property2";
path2.encryptionType = CosmosEncryptionType.RANDOMIZED;
path2.encryptionAlgorithm = CosmosEncryptionAlgorithm.AEAES_256_CBC_HMAC_SHA_256;

List<ClientEncryptionIncludedPath> paths = new ArrayList<>();
paths.add(path1);
paths.add(path2);

CosmosContainerProperties containerProperties =
    new CosmosContainerProperties("my-container", "/id");
containerProperties.setClientEncryptionPolicy(new ClientEncryptionPolicy(paths));
database.createEncryptionContainerAsync(containerProperties);
```
---

## <a name="read-and-write-encrypted-data"></a>Lectura y escritura de datos cifrados

### <a name="how-data-gets-encrypted"></a>¿Cómo se cifran los datos?

Cada vez que se escribe un documento en Azure Cosmos DB, el SDK busca la directiva de cifrado para averiguar qué propiedades se deben cifrar y de qué manera. El resultado del cifrado es una cadena de base 64.

**Cifrado de tipos complejos**:

- Si la propiedad que se va a cifrar es una matriz JSON, se cifran todas las entradas de la matriz.

- Si la propiedad que se va a cifrar es un objeto JSON, solo se cifran los valores hoja del objeto. Los nombres de subpropiedades intermedias se quedan como texto sin formato.

### <a name="read-encrypted-items"></a>Lectura de elementos cifrados

No se requiere ninguna acción explícita para descifrar las propiedades cifradas al emitir lecturas puntuales (captura de un solo elemento a través de su identificador y clave de partición), consultas o al leer la fuente de cambios. El motivo es el siguiente:

- El SDK busca la directiva de cifrado para averiguar qué propiedades deben descifrarse.
- El resultado del cifrado inserta el tipo JSON original del valor.

Tenga en cuenta que la resolución de las propiedades cifradas y su posterior descifrado se basan solo en los resultados devueltos por las solicitudes. Por ejemplo, si `property1` está cifrado pero se proyecta en `property2` (`SELECT property1 AS property2 FROM c`), no se identificará como una propiedad cifrada cuando lo reciba el SDK.

### <a name="filter-queries-on-encrypted-properties"></a>Consultas de filtro por propiedades cifradas

Al escribir consultas que filtran por propiedades cifradas, debe utilizarse el método `AddParameterAsync` para pasar el valor del parámetro de consulta. Este método toma los argumentos siguientes:

- Nombre del parámetro de consulta.
- Valor que se usará en la consulta.
- Ruta de acceso de la propiedad cifrada (como se define en la directiva de cifrado).

> [!IMPORTANT]
> Las propiedades cifradas solo se pueden usar en filtros de igualdad (`WHERE c.property = @Value`). Cualquier otro uso devolverá resultados de consulta impredecibles e incorrectos. Esta restricción se aplicará mejor en las próximas versiones del SDK.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
var queryDefinition = container.CreateQueryDefinition(
    "SELECT * FROM c where c.property1 = @Property1");
await queryDefinition.AddParameterAsync(
    "@Property1",
    1234,
    "/property1");
```

# <a name="java"></a>[Java](#tab/java)

```java
EncryptionSqlQuerySpec encryptionSqlQuerySpec = new EncryptionSqlQuerySpec(
    new SqlQuerySpec("SELECT * FROM c where c.property1 = @Property1"), container);
encryptionSqlQuerySpec.addEncryptionParameterAsync(
    new SqlParameter("@Property1", 1234), "/property1")
```
---

### <a name="reading-documents-when-only-a-subset-of-properties-can-be-decrypted"></a>Lectura de documentos cuando solo se puede descifrar un subconjunto de propiedades

En situaciones en las que el cliente no tiene acceso a todas las CMK utilizadas para cifrar las propiedades, solo se puede descifrar un subconjunto de propiedades cuando se leen los datos. Por ejemplo, si `property1` se cifra con key1 y `property2` se cifra con key2, una aplicación cliente que solo tiene acceso a key1 puede seguir leyendo datos, pero `property2` no. En este caso, el usuario debe leer sus datos a través de consultas SQL y proyectar las propiedades que el cliente no puede descifrar: `SELECT c.property1, c.property3 FROM c`.

## <a name="cmk-rotation"></a>Rotación de CMK

Es posible que quiera "girar" la CMK (es decir, usar una CMK nueva en lugar de la actual) si sospecha que la CMK actual se ha visto comprometida. Girar la CMK con regularidad es una práctica de seguridad común. Para realizar esta rotación, solo tiene que proporcionar el identificador de clave de la nueva CMK que se debe usar para encapsular una DEK específica. Tenga en cuenta que esta operación no afecta al cifrado de los datos, sino a la protección de la DEK. El acceso a la CMK anterior no debe revocarse hasta que se complete la rotación.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
await database.RewrapClientEncryptionKeyAsync(
    "my-key",
    new EncryptionKeyWrapMetadata(
        keyStoreProvider.ProviderName,
        "akvKey",
        " https://<my-key-vault>.vault.azure.net/keys/<new-key>/<version>"));
```

# <a name="java"></a>[Java](#tab/java)

```java
database. rewrapClientEncryptionKey(
    "my-key",
    new EncryptionKeyWrapMetadata(
        "akvKey", " https://<my-key-vault>.vault.azure.net/keys/<new-key>/<version>"));
```
---

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información general sobre el [acceso seguro a los datos en Cosmos DB](secure-access-to-data.md).
- Más información sobre las [claves administradas por el cliente](how-to-setup-cmk.md).