---
title: Uso de JavaScript (Node.js) para administrar ACL en Azure Data Lake Storage Gen2
description: Use la biblioteca cliente de Azure Storage Data Lake para JavaScript con el fin de administrar listas de control de acceso en cuentas de almacenamiento que tengan habilitado el espacio de nombres jerárquico (HNS).
author: normesta
ms.service: storage
ms.date: 03/19/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-js
ms.openlocfilehash: 98656f6751ec7ed8d18c4cd4c25715df5d59d011
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633634"
---
# <a name="use-javascript-sdk-in-nodejs-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Uso de JavaScript SDK en Node.js para administrar ACL en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar Node.js para obtener, establecer y actualizar las listas de control de acceso de directorios y archivos. 

[Paquete (Administrador de paquetes de Node)](https://www.npmjs.com/package/@azure/storage-file-datalake) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples) | [Enviar comentarios](https://github.com/Azure/azure-sdk-for-java/issues)

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- Una cuenta de almacenamiento que tenga habilitado el espacio de nombres jerárquico (HNS). Siga [estas](create-data-lake-storage-account.md) instrucciones para crear uno.

- CLI de Azure versión `2.6.0` o posterior.

- Uno de los siguientes permisos de seguridad:

  - Una [entidad de seguridad](../../role-based-access-control/overview.md#security-principal) aprovisionada de Azure Active Directory a la que se ha asignado el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito del contenedor de destino, el grupo de recursos primario o la suscripción.  

  - El usuario propietario del contenedor o directorio de destino al que va a aplicar la configuración de ACL. Para establecer listas de control de acceso de forma recursiva, se incluyen todos los elementos secundarios en el contenedor o el directorio de destino.
  
  - Clave de la cuenta de almacenamiento.

## <a name="set-up-your-project"></a>Configurar su proyecto

Instale la biblioteca cliente de Data Lake para JavaScript; para ello, abra una ventana de terminal y, a continuación, escriba el siguiente comando.

```javascript
npm install @azure/storage-file-datalake
```

Importe el paquete `storage-file-datalake` colocando esta instrucción en la parte superior del archivo de código. 

```javascript
const {
AzureStorageDataLake,
DataLakeServiceClient,
StorageSharedKeyCredential
} = require("@azure/storage-file-datalake");
```

## <a name="connect-to-the-account"></a>Conexión con la cuenta

Para usar los fragmentos de código de este artículo, tiene que crear una instancia de **DataLakeServiceClient** que represente la cuenta de almacenamiento. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Conexión con Azure Active Directory (AD)

> [!NOTE]
> Si usa Azure Active Directory (Azure AD) para autorizar el acceso, asegúrese de que la entidad de seguridad tenga asignado el [rol Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Para obtener más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, consulte [Modelo de control de acceso de Azure Data Lake Storage Gen2](./data-lake-storage-access-control-model.md).

Puede usar la [biblioteca cliente de Azure Identity para JS](https://www.npmjs.com/package/@azure/identity) para autenticar la aplicación con Azure AD.

Obtenga un identificador de cliente, un secreto de cliente y un identificador de inquilino. Para ello, consulte [Adquisición de un token de Azure AD para la autorización de solicitudes desde una aplicación cliente](../common/storage-auth-aad-app.md). Como parte de ese proceso, tendrá que asignar uno de los siguientes roles de [control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md) a la entidad de seguridad. 

|Role|Capacidad de configuración de ACL|
|--|--|
|[Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)|Todos los directorios y archivos de la cuenta.|
|[Colaborador de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor)|Solo los directorios y archivos que pertenecen a la entidad de seguridad.|

En este ejemplo se crea una instancia de **DataLakeServiceClient** con un identificador de cliente, un secreto de cliente y un identificador de inquilino.  

```javascript
function GetDataLakeServiceClientAD(accountName, clientID, clientSecret, tenantID) {

  const credential = new ClientSecretCredential(tenantID, clientID, clientSecret);
  
  const datalakeServiceClient = new DataLakeServiceClient(
      `https://${accountName}.dfs.core.windows.net`, credential);

  return datalakeServiceClient;             
}
```

> [!NOTE]
> Para ver más ejemplos, consulte la documentación de [Biblioteca cliente de Azure Identity para JS](https://www.npmjs.com/package/@azure/identity).

### <a name="connect-by-using-an-account-key"></a>Conexión con una clave de cuenta

Es la manera más sencilla de conectarse a una cuenta. 

En este ejemplo se crea una instancia de **DataLakeServiceClient** con una clave de cuenta.

```javascript

function GetDataLakeServiceClient(accountName, accountKey) {

  const sharedKeyCredential = 
     new StorageSharedKeyCredential(accountName, accountKey);
  
  const datalakeServiceClient = new DataLakeServiceClient(
      `https://${accountName}.dfs.core.windows.net`, sharedKeyCredential);

  return datalakeServiceClient;             
}      

```

> [!NOTE]
> Este método de autorización solo funciona para aplicaciones de Node.js. Si tiene previsto ejecutar el código en un explorador, se puede autorizar mediante Azure Active Directory (AD).

## <a name="get-and-set-a-directory-acl"></a>Obtener y establecer una ACL de directorio

En este ejemplo se obtiene y después se establece la ACL de un directorio denominado `my-directory`. En este ejemplo se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

> [!NOTE]
> Si la aplicación autoriza el acceso mediante Azure Active Directory (Azure AD), asegúrese de que la entidad de seguridad que la aplicación usa para autorizar el acceso tiene asignado el [rol de propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Para más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, vea [Control de acceso en Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md).

```javascript
async function ManageDirectoryACLs(fileSystemClient) {

    const directoryClient = fileSystemClient.getDirectoryClient("my-directory"); 
    const permissions = await directoryClient.getAccessControl();

    console.log(permissions.acl);

    const acl = [
    {
      accessControlType: "user",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: true,
        execute: true
      }
    },
    {
      accessControlType: "group",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: false,
        execute: true
      }
    },
    {
      accessControlType: "other",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: true,
        execute: false
      }

    }

  ];

  await directoryClient.setAccessControl(acl);
}
```

También puede obtener y establecer la ACL del directorio raíz de un contenedor. Para obtener el directorio raíz, pase una cadena vacía (`/`) en el método **DataLakeFileSystemClient.getDirectoryClient**.

### <a name="get-and-set-a-file-acl"></a>Obtener y establecer una ACL de archivo

En este ejemplo se obtiene y después se establece la ACL de un archivo denominado `upload-file.txt`. En este ejemplo se concede al usuario propietario permisos de lectura, escritura y ejecución, permisos de solo lectura y ejecución al grupo propietario, y permisos de lectura y escritura al resto.

> [!NOTE]
> Si la aplicación autoriza el acceso mediante Azure Active Directory (Azure AD), asegúrese de que la entidad de seguridad que la aplicación usa para autorizar el acceso tiene asignado el [rol de propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Para más información sobre cómo se aplican los permisos de ACL y las consecuencias de cambiarlos, vea [Control de acceso en Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md).

```javascript
async function ManageFileACLs(fileSystemClient) {

  const fileClient = fileSystemClient.getFileClient("my-directory/uploaded-file.txt"); 
  const permissions = await fileClient.getAccessControl();

  console.log(permissions.acl);

  const acl = [
  {
    accessControlType: "user",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: true,
      execute: true
    }
  },
  {
    accessControlType: "group",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: false,
      execute: true
    }
  },
  {
    accessControlType: "other",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: true,
      execute: false
    }

  }

];

await fileClient.setAccessControl(acl);        
}
```

## <a name="see-also"></a>Consulte también

- [Paquete (Administrador de paquetes de Node)](https://www.npmjs.com/package/@azure/storage-file-datalake)
- [Muestras](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples)
- [Envíenos sus comentarios](https://github.com/Azure/azure-sdk-for-java/issues)
- [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)
- [Listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md)