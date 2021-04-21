---
title: Control del acceso a la cuenta de almacenamiento para el grupo de SQL sin servidor
description: Se describe el modo en que el grupo de SQL sin servidor accede a Azure Storage y cómo puede controlar su acceso al almacenamiento en Azure Synapse Analytics.
services: synapse-analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 06/11/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 266a6c27261107b883fdc0c1cdd274e6345de6db
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107483459"
---
# <a name="control-storage-account-access-for-serverless-sql-pool-in-azure-synapse-analytics"></a>Control del acceso a la cuenta de almacenamiento del grupo de SQL sin servidor en Azure Synapse Analytics

Una consulta del grupo de SQL sin servidor lee los archivos directamente desde Azure Storage. Los permisos para tener acceso a los archivos en Azure Storage se controlan en dos niveles:
- **Nivel de almacenamiento**: el usuario debe tener permiso de acceso a los archivos de almacenamiento subyacentes. El administrador de almacenamiento debe permitir que la entidad de seguridad de Azure AD lea y escriba archivos o que genere una clave SAS que se usará para acceder al almacenamiento.
- **Nivel de servicio de SQL**: el usuario debe tener permiso para leer datos mediante una [tabla externa](develop-tables-external-tables.md) o para ejecutar la función `OPENROWSET`. Conozca más información sobre [los permisos necesarios en esta sección](develop-storage-files-overview.md#permissions).

En este artículo se describen los tipos de credenciales que puede usar y cómo se realiza la búsqueda de credenciales para los usuarios de SQL y Azure AD.

## <a name="storage-permissions"></a>Permisos de almacenamiento

Un grupo de SQL sin servidor en un área de trabajo de Synapse Analytics puede leer el contenido de los archivos almacenados en Azure Data Lake Storage. Es necesario configurar permisos en el almacenamiento para que un usuario que ejecute una consulta SQL pueda leer los archivos. Hay tres formas de habilitar el acceso a los archivos:
- El **[control de acceso basado en roles (RBAC)](../../role-based-access-control/overview.md)** permite asignar un rol a usuarios de Azure AD en el inquilino en el que reside el almacenamiento. Los roles RBAC se pueden asignar a usuarios de Azure AD. Un lector debe tener un rol `Storage Blob Data Reader`, `Storage Blob Data Contributor` o `Storage Blob Data Owner`. Un usuario que escribe datos en Azure Storage debe tener el rol `Storage Blob Data Writer` o `Storage Blob Data Owner`. Tenga en cuenta que el rol `Storage Owner` no implica que un usuario sea también `Storage Data Owner`.
- Las **listas de control de acceso (ACL)** permiten definir un modelo de permisos específico en los archivos y directorios de Azure Storage. Las listas ACL se pueden asignar a usuarios de Azure AD. Si los lectores quieren leer un archivo en una ruta de acceso de Azure Storage, deben ejecutar (X) ACL en cada carpeta de la ruta de acceso del y leer (R) ACL en el archivo. [Más información sobre cómo establecer permisos de ACL en la capa de almacenamiento](../../storage/blobs/data-lake-storage-access-control.md#how-to-set-acls)
- La **firma de acceso compartido (SAS)** permite que un lector acceda a los archivos de Azure Data Lake Storage mediante el token de tiempo limitado. El lector ni siquiera necesita autenticarse como usuario de Azure A. El token de SAS contiene los permisos concedidos al lector, así como el período durante el cual es válido el token. El token de SAS es una buena opción para el acceso con restricciones de tiempo en el caso de los usuarios que ni siquiera tienen que estar en el mismo inquilino de Azure AD. El token de SAS se puede definir en la cuenta de almacenamiento o en directorios específicos. Obtenga más información sobre el [acceso limitado a recursos de Azure Storage mediante el uso de firmas de acceso compartido](../../storage/common/storage-sas-overview.md).

## <a name="supported-storage-authorization-types"></a>Tipos de autorización de almacenamiento admitidos

Un usuario que haya iniciado sesión en un grupo de SQL sin servidor debe estar autorizado para acceder y consultar los archivos de Azure Storage si los archivos no están disponibles públicamente. Para acceder al almacenamiento no público puede usar tres tipos de autorización: [Identidad de usuario](?tabs=user-identity), [Firma de acceso compartido](?tabs=shared-access-signature) e [Identidad administrada](?tabs=managed-identity).

> [!NOTE]
> **Paso a través de Azure AD** es el comportamiento predeterminado cuando se crea un área de trabajo.

### <a name="user-identity"></a>[Identidad de usuario](#tab/user-identity)

La **identidad de usuario**, conocida también como "paso a través de Azure AD", es un tipo de autorización en el que se usa la identidad del usuario de Azure AD que inició sesión en el grupo de SQL sin servidor para autorizar el acceso a los datos. Antes de acceder a los datos, el administrador de Azure Storage debe conceder permisos al usuario de Azure AD. Como se indica en la tabla siguiente, no se admite para el tipo de usuario de SQL.

> [!IMPORTANT]
> Las aplicaciones cliente pueden almacenar en caché el token de autenticación de Azure Active Directory. Por ejemplo, PowerBI almacena en caché el token de Azure Active Directory y reutiliza el mismo token durante una hora. Puede producirse un error en las consultas de ejecución prolongada si el token expira en mitad de la ejecución de la consulta. Si experimenta errores de consulta causados por la expiración del token de acceso de Azure Active Directory en mitad de la consulta, considere la posibilidad de cambiar a la [identidad administrada](develop-storage-files-storage-access-control.md?tabs=managed-identity#supported-storage-authorization-types) o a la [firma de acceso compartido](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#supported-storage-authorization-types).

Debe tener un rol de Propietario, Colaborador o Lector de datos de un blob de almacenamiento para usar su identidad para acceder a los datos. Como alternativa, puede establecer reglas de ACL específicas para acceder a archivos y carpetas. Incluso si es propietario de una cuenta de almacenamiento, deberá agregarse a uno de los roles de datos del blob de almacenamiento.
Para más información sobre el control de acceso en Azure Data Lake Store Gen2, revise el artículo [Control de acceso en Azure Data Lake Storage Gen2](../../storage/blobs/data-lake-storage-access-control.md).


### <a name="shared-access-signature"></a>[Firma de acceso compartido](#tab/shared-access-signature)

La **Firma de acceso compartido (SAS)** ofrece acceso delegado a los recursos de una cuenta de almacenamiento. Con SAS, un cliente puede conceder a los clientes acceso a los recursos de una cuenta de almacenamiento sin compartir las claves de la cuenta. SAS le ofrece un control detallado sobre el tipo de acceso que concede a los clientes que tienen una SAS, incluido el intervalo de validez, los permisos concedidos, el intervalo de direcciones IP aceptable y el protocolo aceptable (https/http).

Para obtener un token de SAS, vaya a **Azure portal -> Cuenta de Storage -> Firma de acceso compartido -> Configurar permisos -> Generar la cadena de conexión y SAS.**

> [!IMPORTANT]
> Cuando se genera un token de SAS, este incluye un signo de interrogación ("?") al principio del token. Para usar el token en el grupo de SQL sin servidor, debe quitar el signo de interrogación ("?") al crear una credencial. Por ejemplo:
>
> SAS token: ?sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D

Para habilitar el acceso mediante un token de SAS, debe crear una credencial con ámbito en la base de datos o con ámbito en el servidor. 


> [!IMPORTANT]
> No es posible acceder a las cuentas de almacenamiento privado con el token de SAS. Considere la posibilidad de cambiar a la [identidad administrada](develop-storage-files-storage-access-control.md?tabs=managed-identity#supported-storage-authorization-types) o a la autenticación de [paso a través de Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity#supported-storage-authorization-types) para acceder al almacenamiento protegido.

### <a name="managed-identity"></a>[Identidad administrada](#tab/managed-identity)

La **identidad administrada** también se conoce como MSI. Es una característica de Azure Active Directory (Azure AD) que proporciona servicios de Azure para el grupo de SQL sin servidor. Además, implementa una identidad administrada automáticamente en Azure AD. Esta identidad se puede usar para autorizar la solicitud de acceso a los datos de Azure Storage.

Antes de acceder a los datos, el administrador de Azure Storage debe conceder permisos a la identidad administrada para acceder a los datos. La concesión de permisos a la identidad administrada se realiza de la misma forma que la concesión de permisos a cualquier otro usuario de Azure AD.

### <a name="anonymous-access"></a>[Acceso anónimo](#tab/public-access)

Puede tener acceso a los archivos disponibles públicamente ubicados en cuentas de almacenamiento de Azure que [permitan el acceso anónimo](../../storage/blobs/anonymous-read-access-configure.md).

---

### <a name="supported-authorization-types-for-databases-users"></a>Tipos de autorización admitidos para usuarios de bases de datos

En la tabla siguiente puede encontrar los tipos de autorización disponibles:

| Tipo de autorización                    | *Usuario de SQL*    | *Usuario de Azure AD*     |
| ------------------------------------- | ------------- | -----------    |
| [Identidad de usuario](?tabs=user-identity#supported-storage-authorization-types)       | No compatible | Compatible      |
| [SAS](?tabs=shared-access-signature#supported-storage-authorization-types)       | Compatible     | Compatible      |
| [Identidad administrada](?tabs=managed-identity#supported-storage-authorization-types) | Compatible | Compatible      |

### <a name="supported-storages-and-authorization-types"></a>Tipos de almacenamiento y autorización admitidos

Puede usar las siguientes combinaciones de tipos de autorización y almacenamiento de Azure:

| Tipo de autorización  | Blob Storage   | ADLS Gen1        | ADLS Gen2     |
| ------------------- | ------------   | --------------   | -----------   |
| [SAS](?tabs=shared-access-signature#supported-storage-authorization-types)    | Compatible\*      | No compatible   | Compatible\*     |
| [Identidad administrada](?tabs=managed-identity#supported-storage-authorization-types) | Compatible      | Compatible        | Compatible     |
| [Identidad de usuario](?tabs=user-identity#supported-storage-authorization-types)    | Compatible\*      | Compatible\*        | Compatible\*     |

\* El token de SAS y la identidad de Azure AD se pueden usar para tener acceso a un almacenamiento que no esté protegido con el firewall.


### <a name="querying-firewall-protected-storage"></a>Consulta del almacenamiento protegido por firewall

Al acceder al almacenamiento protegido con firewall, solo se puede usar **Identidad de usuario** o **Identidad administrada**.

> [!NOTE]
> La característica de firewall de Azure Storage está en versión preliminar pública y está disponible en todas las regiones de la nube pública. 

#### <a name="user-identity"></a>Identidad del usuario

Para acceder a una solución de almacenamiento que está protegido por el firewall por medio de una identidad de usuario, puede usar la interfaz de usuario de Azure Portal o el módulo Az.Storage de PowerShell.
#### <a name="configuration-via-azure-portal"></a>Configuración mediante Azure Portal

1. Busque su cuenta de almacenamiento en Azure Portal.
1. Vaya a Redes en la sección Configuración.
1. En la sección "Resource instances" (Instancias de recursos), agregue una excepción para el área de trabajo de Synapse.
1. Seleccione Microsoft.Synapse/workspaces en Tipo de recurso.
1. Seleccione el nombre del área de trabajo como un nombre de instancia.
1. Haga clic en Guardar.

#### <a name="configuration-via-powershell"></a>Configuración mediante PowerShell

Siga estos pasos para configurar el firewall de la cuenta de almacenamiento y agregar una excepción para el área de trabajo de Synapse.

1. Abra o [instale PowerShell](/powershell/scripting/install/installing-powershell-core-on-windows).
2. Instale el módulo Az.Storage 3.4.0 y Az.Synapse 0.7.0: 
    ```powershell
    Install-Module -Name Az.Storage -RequiredVersion 3.4.0
    Install-Module -Name Az.Synapse -RequiredVersion 0.7.0
    ```
    > [!IMPORTANT]
    > Asegúrese de usar la **versión 3.4.0**. Puede comprobar la versión de Az.Storage con este comando:  
    > ```powershell 
    > Get-Module -ListAvailable -Name  Az.Storage | select Version
    > ```
    > 

3. Conéctese a su inquilino de Azure: 
    ```powershell
    Connect-AzAccount
    ```
4. Defina las variables en PowerShell: 
    - Nombre del grupo de recursos: puede encontrarlo en Azure Portal, en la información general de la cuenta de almacenamiento.
    - Nombre de cuenta: nombre de la cuenta de almacenamiento que está protegida por reglas de firewall.
    - Id. de inquilino: puede encontrarlo en Azure Portal, en la información del inquilino de Azure Active Directory.
    - Nombre del área de trabajo: nombre del área de trabajo de Synapse.

    ```powershell
        $resourceGroupName = "<resource group name>"
        $accountName = "<storage account name>"
        $tenantId = "<tenant id>"
        $workspaceName = "<synapse workspace name>"
        
        $workspace = Get-AzSynapseWorkspace -Name $workspaceName
        $resourceId = $workspace.Id
        $index = $resourceId.IndexOf("/resourceGroups/", 0)
        # Replace G with g - /resourceGroups/ to /resourcegroups/
        $resourceId = $resourceId.Substring(0,$index) + "/resourcegroups/" + $resourceId.Substring($index + "/resourceGroups/".Length)
        $resourceId
    ```
    > [!IMPORTANT]
    > Asegúrese de que el identificador de recurso coincide con esta plantilla en la impresión de la variable resourceId.
    >
    > Es importante escribir **resourcegroups** en minúsculas.
    > Ejemplo de un identificador de recurso: 
    > ```
    > /subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Synapse/workspaces/{name-of-workspace}
    > ```
    > 
5. Agregue una regla de red de almacenamiento: 
    ```powershell
        Add-AzStorageAccountNetworkRule -ResourceGroupName $resourceGroupName -Name $accountName -TenantId $tenantId -ResourceId $resourceId
    ```
6. Compruebe que la regla se aplicó en la cuenta de almacenamiento: 
    ```powershell
        $rule = Get-AzStorageAccountNetworkRuleSet -ResourceGroupName $resourceGroupName -Name $accountName
        $rule.ResourceAccessRules | ForEach-Object { 
            if ($_.ResourceId -cmatch "\/subscriptions\/(\w\-*)+\/resourcegroups\/(.)+") { 
                Write-Host "Storage account network rule is successfully configured." -ForegroundColor Green
                $rule.ResourceAccessRules
            } else {
                Write-Host "Storage account network rule is not configured correctly. Remove this rule and follow the steps in detail." -ForegroundColor Red
                $rule.ResourceAccessRules
            }
        }
    ```

#### <a name="managed-identity"></a>Identidad administrada
Debe establecer [Permitir servicios de Microsoft de confianza…](../../storage/common/storage-network-security.md#trusted-microsoft-services) y [asignar un rol de Azure](../../storage/common/storage-auth-aad.md#assign-azure-roles-for-access-rights) de manera explícita a la [identidad administrada asignada por el sistema](../../active-directory/managed-identities-azure-resources/overview.md) para esa instancia del recurso. En ese caso, el ámbito de acceso de la instancia corresponde al rol de Azure que se asigna a la identidad administrada.

## <a name="credentials"></a>Credenciales

Para consultar un archivo ubicado en Azure Storage, el punto de conexión del grupo de SQL sin servidor necesita una credencial que contenga la información de autenticación. Se usan dos tipos de credenciales:
- La CREDENCIAL de nivel de servidor se utiliza para las consultas ad hoc ejecutadas mediante la función `OPENROWSET`. El nombre de la credencial debe coincidir con la dirección URL de almacenamiento.
- LA CREDENCIAL CON ÁMBITO EN LA BASE DE DATOS se usa para las tablas externas. La tabla externa hace referencia a `DATA SOURCE` con la credencial que se debe usar para tener acceso al almacenamiento.

Para permitir a un usuario crear o anular una credencial, el administrador puede CONCEDER O DENEGAR un permiso para MODIFICAR CUALQUIER CREDENCIAL a un usuario:

```sql
GRANT ALTER ANY CREDENTIAL TO [user_name];
```

Los usuarios de bases de datos que tienen acceso a almacenamiento externo deben tener permiso para usar las credenciales.

### <a name="grant-permissions-to-use-credential"></a>Concesión de permisos para usar credenciales

Para usar las credenciales, un usuario debe tener el permiso `REFERENCES` sobre una credencial específica. Para conceder un permiso `REFERENCES` SOBRE una credencial de almacenamiento para un usuario específico, ejecute:

```sql
GRANT REFERENCES ON CREDENTIAL::[storage_credential] TO [specific_user];
```

## <a name="server-scoped-credential"></a>Credencial con ámbito en el servidor

Las credenciales con ámbito en el servidor se usan cuando el inicio de sesión de SQL llama a la función `OPENROWSET` sin `DATA_SOURCE` para leer archivos en alguna cuenta de almacenamiento. El nombre de la credencial con ámbito en el servidor **debe** coincidir con la dirección URL base de Azure Storage (seguida opcionalmente por un nombre de contenedor). Para agregar una credencial, ejecute [CREATE CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true). Deberá proporcionar un argumento CREDENTIAL NAME.

> [!NOTE]
> El argumento `FOR CRYPTOGRAPHIC PROVIDER` no se admite.

El nombre de la CREDENCIAL de nivel de servidor debe coincidir con la ruta de acceso completa de la cuenta de almacenamiento (y opcionalmente, del contenedor) en el formato siguiente: `<prefix>://<storage_account_path>[/<container_name>]`. Las rutas de acceso de la cuenta de almacenamiento se describen en la tabla siguiente:

| Origen de datos externo       | Prefijo | Ruta de acceso a la cuenta de almacenamiento                                |
| -------------------------- | ------ | --------------------------------------------------- |
| Azure Blob Storage         | https  | <cuentaDeAlmacenamiento>.blob.core.windows.net             |
| Azure Data Lake Storage Gen1 | https  | <cuentaDeAlmacenamiento>.azuredatalakestore.net/webhdfs/v1 |
| Azure Data Lake Storage Gen2 | https  | <cuentaDeAlmacenamiento>.dfs.core.windows.net              |

Las credenciales con ámbito en el servidor permiten el acceso a Azure Storage mediante los siguientes tipos de autenticación:

### <a name="user-identity"></a>[Identidad de usuario](#tab/user-identity)

Los usuarios de Azure AD que tengan los roles `Storage Blob Data Owner`, `Storage Blob Data Contributor`o `Storage Blob Data Reader` pueden acceder a cualquier archivo de Azure Storage. Los usuarios de Azure AD no necesitan credenciales para acceder al almacenamiento. 

Los usuarios de SQL no pueden utilizar la autenticación de Azure AD para acceder al almacenamiento.

### <a name="shared-access-signature"></a>[Firma de acceso compartido](#tab/shared-access-signature)

El script siguiente crea una credencial de nivel de servidor que puede usar la función `OPENROWSET` para tener acceso a cualquier archivo de Azure Storage mediante el token de SAS. Cree esta credencial para habilitar la entidad de seguridad de SQL que ejecuta la función `OPENROWSET` para leer archivos protegidos con clave SAS en la instancia de Azure Storage que coincida con la dirección URL del nombre de la credencial.

Reemplace <*mystorageaccountname*> por el nombre de la cuenta de almacenamiento real y <*mystorageaccountcontainername*> por el nombre real del contenedor:

```sql
CREATE CREDENTIAL [https://<mystorageaccountname>.dfs.core.windows.net/<mystorageaccountcontainername>]
WITH IDENTITY='SHARED ACCESS SIGNATURE'
, SECRET = 'sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D';
GO
```

Opcionalmente, puede usar solo la dirección URL base de la cuenta de almacenamiento, sin el nombre de contenedor.

### <a name="managed-identity"></a>[Identidad administrada](#tab/managed-identity)

El script siguiente crea una credencial de nivel de servidor que puede usar la función `OPENROWSET` para tener acceso a cualquier archivo de Azure Storage mediante la identidad administrada del área de trabajo.

```sql
CREATE CREDENTIAL [https://<storage_account>.dfs.core.windows.net/<container>]
WITH IDENTITY='Managed Identity'
```

Opcionalmente, puede usar solo la dirección URL base de la cuenta de almacenamiento, sin el nombre de contenedor.

### <a name="public-access"></a>[Acceso público](#tab/public-access)

La credencial con ámbito en la base de datos no es necesaria para permitir el acceso a archivos disponibles públicamente. Cree un [origen de datos sin credenciales con ámbito en la base de datos](develop-tables-external-tables.md?tabs=sql-ondemand#example-for-create-external-data-source) para tener acceso a los archivos disponibles públicamente en Azure Storage.

---

## <a name="database-scoped-credential"></a>Credencial con ámbito en la base de datos

Las credenciales con ámbito en la base de datos se usan cuando cualquier entidad de seguridad llama a la función `OPENROWSET` con `DATA_SOURCE` o selecciona datos de [tabla externa](develop-tables-external-tables.md) que no tienen acceso a archivos públicos. No es necesario que la credencial con ámbito en la base de datos coincida con el nombre de la cuenta de almacenamiento. Se usará explícitamente en el ORIGEN DE DATOS que define la ubicación del almacenamiento.

Las credenciales con ámbito en la base de datos permiten el acceso a Azure Storage mediante los siguientes tipos de autenticación:

### <a name="azure-ad-identity"></a>[Identidad de Azure AD](#tab/user-identity)

Los usuarios de Azure AD que tengan, como mínimo, los roles `Storage Blob Data Owner`, `Storage Blob Data Contributor`o `Storage Blob Data Reader` pueden acceder a cualquier archivo de Azure Storage. Los usuarios de Azure AD no necesitan credenciales para acceder al almacenamiento.

```sql
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>'
)
```

Los usuarios de SQL no pueden utilizar la autenticación de Azure AD para acceder al almacenamiento.

### <a name="shared-access-signature"></a>[Firma de acceso compartido](#tab/shared-access-signature)

El script siguiente crea una credencial que se usa para tener acceso a los archivos del almacenamiento mediante el token de SAS especificado en la credencial. El script creará un origen de datos externo de ejemplo que usa este token de SAS para acceder al almacenamiento.

```sql
-- Optional: Create MASTER KEY if not exists in database:
-- CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Very Strong Password>'
GO
CREATE DATABASE SCOPED CREDENTIAL [SasToken]
WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
     SECRET = 'sv=2018-03-28&ss=bfqt&srt=sco&sp=rwdlacup&se=2019-04-18T20:42:12Z&st=2019-04-18T12:42:12Z&spr=https&sig=lQHczNvrk1KoYLCpFdSsMANd0ef9BrIPBNJ3VYEIq78%3D';
GO
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>',
          CREDENTIAL = SasToken
)
```

### <a name="managed-identity"></a>[Identidad administrada](#tab/managed-identity)

El script siguiente crea una credencial con ámbito en la base de datos que se puede usar para suplantar al usuario actual de Azure AD como identidad administrada del servicio. El script creará un origen de datos externo de ejemplo que usa la identidad del área de trabajo para acceder al almacenamiento.

```sql
-- Optional: Create MASTER KEY if not exists in database:
-- CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Very Strong Password>
CREATE DATABASE SCOPED CREDENTIAL SynapseIdentity
WITH IDENTITY = 'Managed Identity';
GO
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>',
          CREDENTIAL = SynapseIdentity
)
```

No es necesario que la credencial con ámbito en la base de datos coincida con el nombre de la cuenta de almacenamiento porque se usará explícitamente en el ORIGEN DE DATOS que define la ubicación del almacenamiento.

### <a name="public-access"></a>[Acceso público](#tab/public-access)

La credencial con ámbito en la base de datos no es necesaria para permitir el acceso a archivos disponibles públicamente. Cree un [origen de datos sin credenciales con ámbito en la base de datos](develop-tables-external-tables.md?tabs=sql-ondemand#example-for-create-external-data-source) para tener acceso a los archivos disponibles públicamente en Azure Storage.

```sql
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.blob.core.windows.net/<container>/<path>'
)
```
---

Las credenciales con ámbito en la base de datos se usan en orígenes de datos externos para especificar el método de autenticación que se utilizará para tener acceso a este almacenamiento:

```sql
CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>',
          CREDENTIAL = <name of database scoped credential> 
)
```

## <a name="examples"></a>Ejemplos

### <a name="access-a-publicly-available-data-source"></a>**Acceso a un origen de datos disponible públicamente**

Use el siguiente script para crear una tabla que tenga acceso al origen de datos disponible públicamente.

```sql
CREATE EXTERNAL FILE FORMAT [SynapseParquetFormat]
       WITH ( FORMAT_TYPE = PARQUET)
GO
CREATE EXTERNAL DATA SOURCE publicData
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<public_container>/<path>' )
GO

CREATE EXTERNAL TABLE dbo.userPublicData ( [id] int, [first_name] varchar(8000), [last_name] varchar(8000) )
WITH ( LOCATION = 'parquet/user-data/*.parquet',
       DATA_SOURCE = [publicData],
       FILE_FORMAT = [SynapseParquetFormat] )
```

El usuario de la base de datos puede leer el contenido de los archivos desde el origen de datos mediante la tabla externa o la función [OPENROWSET](develop-openrowset.md) que hace referencia al origen de datos:

```sql
SELECT TOP 10 * FROM dbo.userPublicData;
GO
SELECT TOP 10 * FROM OPENROWSET(BULK 'parquet/user-data/*.parquet',
                                DATA_SOURCE = 'mysample',
                                FORMAT='PARQUET') as rows;
GO
```

### <a name="access-a-data-source-using-credentials"></a>**Acceso a un origen de datos mediante credenciales**

Modifique el script siguiente para crear una tabla externa que tenga acceso a Azure Storage mediante el token de SAS, la identidad de Azure AD del usuario o la identidad administrada del área de trabajo.

```sql
-- Create master key in databases with some password (one-off per database)
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'Y*********0'
GO

-- Create databases scoped credential that use Managed Identity or SAS token. User needs to create only database-scoped credentials that should be used to access data source:

CREATE DATABASE SCOPED CREDENTIAL WorkspaceIdentity
WITH IDENTITY = 'Managed Identity'
GO
CREATE DATABASE SCOPED CREDENTIAL SasCredential
WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = 'sv=2019-10-1********ZVsTOL0ltEGhf54N8KhDCRfLRI%3D'

-- Create data source that one of the credentials above, external file format, and external tables that reference this data source and file format:

CREATE EXTERNAL FILE FORMAT [SynapseParquetFormat] WITH ( FORMAT_TYPE = PARQUET)
GO

CREATE EXTERNAL DATA SOURCE mysample
WITH (    LOCATION   = 'https://<storage_account>.dfs.core.windows.net/<container>/<path>'
-- Uncomment one of these options depending on authentication method that you want to use to access data source:
--,CREDENTIAL = WorkspaceIdentity 
--,CREDENTIAL = SasCredential 
)

CREATE EXTERNAL TABLE dbo.userData ( [id] int, [first_name] varchar(8000), [last_name] varchar(8000) )
WITH ( LOCATION = 'parquet/user-data/*.parquet',
       DATA_SOURCE = [mysample],
       FILE_FORMAT = [SynapseParquetFormat] );

```

El usuario de la base de datos puede leer el contenido de los archivos desde el origen de datos mediante la [tabla externa](develop-tables-external-tables.md) o la función [OPENROWSET](develop-openrowset.md) que hace referencia al origen de datos:

```sql
SELECT TOP 10 * FROM dbo.userdata;
GO
SELECT TOP 10 * FROM OPENROWSET(BULK 'parquet/user-data/*.parquet', DATA_SOURCE = 'mysample', FORMAT='PARQUET') as rows;
GO
```

## <a name="next-steps"></a>Pasos siguientes

Los artículos que se enumeran a continuación le ayudarán a aprender a consultar diferentes tipos de carpeta y de archivo, y a crear y usar vistas:

- [Consulta de archivos .csv](query-single-csv-file.md)
- [Consulta de carpetas y varios archivos .csv](query-folders-multiple-csv-files.md)
- [Consulta de archivos específicos](query-specific-files.md)
- [Consulta de archivos Parquet](query-parquet-files.md)
- [Creación y uso de vistas](create-use-views.md)
- [Consulta de archivos JSON](query-json-files.md)
- [Consulta de tipos anidados de Parquet](query-parquet-nested-types.md)
