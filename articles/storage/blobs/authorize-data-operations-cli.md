---
title: Distintas formas de autorizar el acceso a datos en blobs con la CLI de Azure
titleSuffix: Azure Storage
description: Especifique cómo se autorizan las operaciones de datos en datos de blobs con la CLI de Azure. Puede autorizar las operaciones de datos con las credenciales de Azure AD, con la clave de acceso de la cuenta o con un token de firma de acceso compartido (SAS).
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/10/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 53ad6dd922c1ccebd79aebcd2966b23b38de00e7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776904"
---
# <a name="choose-how-to-authorize-access-to-blob-data-with-azure-cli"></a>Distintas formas de autorizar el acceso a datos en blobs con la CLI de Azure

Azure Storage proporciona extensiones para la CLI de Azure que permiten especificar el modo de autorizar las operaciones en los datos de blobs. Puede autorizar las operaciones de datos de las siguientes maneras:

- Con una entidad de seguridad de Azure Active Directory (Azure AD). Microsoft recomienda usar las credenciales de Azure AD para mayor seguridad y facilidad de uso.
- Con la clave de acceso de la cuenta o un token de firma de acceso compartido (SAS).

## <a name="specify-how-data-operations-are-authorized"></a>Cómo se autorizan las operaciones de datos

Los comandos de la CLI de Azure para leer y escribir datos de blobs incluyen el parámetro `--auth-mode` opcional. Especifique ese parámetro para indicar cómo se va a autorizar una operación de datos:

- Establezca el parámetro `--auth-mode` en `login` para iniciar sesión con una entidad de seguridad de Azure AD (recomendado).
- Establezca el parámetro `--auth-mode` en el valor `key` heredado para intentar recuperar la clave de acceso de la cuenta que se va a usar para la autorización. Si omite el parámetro `--auth-mode`, la CLI de Azure también intentará recuperar la clave de acceso.

Para usar el parámetro `--auth-mode`, asegúrese de que ha instalado la CLI de Azure, versión 2.0.46 o posterior. Ejecute `az --version` para comprobar la versión instalada.

> [!NOTE]
> Cuando una cuenta de almacenamiento está bloqueada con un bloqueo **ReadOnly** de Azure Resource Manager, no se permite la operación [Crear lista de claves](/rest/api/storagerp/storageaccounts/listkeys) para esa cuenta de almacenamiento. **Crear lista de claves** es una operación POST y todas las operaciones POST se impiden cuando se configura un bloqueo **ReadOnly** para la cuenta. Por esta razón, cuando la cuenta se bloquea con un bloqueo **ReadOnly**, los usuarios que aún no disponen de las claves de cuenta deben usar credenciales de Azure AD para tener acceso a los datos de blob.

> [!IMPORTANT]
> Si omite el parámetro `--auth-mode` o lo establece en `key`, la CLI de Azure intentará usar la clave de acceso de la cuenta para la autorización. En este caso, Microsoft recomienda que proporcione la clave de acceso en el comando o en la variable de entorno **AZURE_STORAGE_KEY**. Para obtener más información sobre las variables de entorno, vea la sección titulada [Establecimiento de variables de entorno para parámetros de autorización](#set-environment-variables-for-authorization-parameters).
>
> Si no proporciona la clave de acceso, la CLI de Azure intentará llamar al proveedor de recursos de Azure Storage para recuperarla con cada operación. La realización de numerosas operaciones de datos que requieren una llamada al proveedor de recursos puede generar una limitación. Para obtener más información sobre los límites del proveedor de recursos, consulte [Destinos de escalabilidad y rendimiento para el proveedor de recursos de Azure Storage](../common/scalability-targets-resource-provider.md).

## <a name="authorize-with-azure-ad-credentials"></a>Autorización con credenciales de Azure AD

Al iniciar sesión en la CLI de Azure con credenciales de Azure AD, se devuelve un token de acceso OAuth 2.0. La CLI de Azure usa automáticamente ese token para autorizar las operaciones de datos posteriores en Blob Storage o Queue Storage. Para las operaciones admitidas, ya no necesita pasar una clave de cuenta o token SAS con el comando.

Puede asignar permisos en los datos de blob a una entidad de seguridad de Azure AD mediante el control de acceso basado en roles de Azure (Azure RBAC). Para más información sobre los roles de Azure en Azure Storage, consulte [cómo administrar los derechos de acceso a los datos de Azure Storage mediante Azure RBAC](../common/storage-auth-aad-rbac-portal.md).

### <a name="permissions-for-calling-data-operations"></a>Permisos para llamar a operaciones de datos

Las extensiones de Azure Storage se admiten para las operaciones en datos de blob. Las operaciones a las que podrá llamar dependerán de los permisos que se concedan a la entidad de seguridad de Azure AD con la que inicie sesión en la CLI de Azure. Los permisos para contenedores de Azure Storage se asignan mediante Azure RBAC. Por ejemplo, si se le asigna el rol **Lector de datos de Storage Blob**, puede ejecutar comandos de scripting que lean datos de un contenedor. Si se le asigna el rol **Colaborador de datos de Storage Blob**, podrá ejecutar comandos de scripting que lean, escriban o eliminen un contenedor, o los datos que contiene.

Para más información sobre los permisos requeridos para cada operación de Azure Storage en un contenedor, consulte la sección [Llamadas a operaciones de almacenamiento con tokens de OAuth](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens).  

### <a name="example-authorize-an-operation-to-create-a-container-with-azure-ad-credentials"></a>Ejemplo: Autorización de una operación para crear un contenedor con credenciales de Azure AD

En el ejemplo siguiente se muestra cómo crear un contenedor a partir de la CLI de Azure con las credenciales de Azure AD. Para crear el contenedor, debe iniciar sesión en la CLI de Azure. Necesitará un grupo de recursos y una cuenta de almacenamiento. Para obtener información sobre cómo crear esos recursos, consulte [Inicio rápido: Creación, descarga y enumeración de blobs mediante la CLI de Azure](../blobs/storage-quickstart-blobs-cli.md).

1. Antes de crear el contenedor, asígnese a sí mismo el rol [Colaborador de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Aunque sea el propietario de la cuenta, necesita permisos explícitos para realizar operaciones de datos en la cuenta de almacenamiento. Para más información sobre la asignación de roles de Azure, consulte [Uso de Azure Portal para asignar un rol de Azure para el acceso a datos de blobs y colas](../common/storage-auth-aad-rbac-portal.md).

    > [!IMPORTANT]
    > La propagación de las asignaciones de roles de Azure pueden tardar unos minutos.

1. Llame al comando [az storage container create](/cli/azure/storage/container#az_storage_container_create) con el parámetro `--auth-mode` establecido en `login` para crear el contenedor con sus credenciales de Azure AD. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

    ```azurecli
    az storage container create \
        --account-name <storage-account> \
        --name sample-container \
        --auth-mode login
    ```

## <a name="authorize-with-the-account-access-key"></a>Autorización con la clave de acceso de la cuenta

Si posee la clave de cuenta, puede llamar a cualquier operación de datos Azure Storage. En general, usar la clave de cuenta es menos seguro. Si la clave de cuenta se ha puesto en riesgo, es posible que también estén en peligro todos los datos de dicha cuenta.

En el ejemplo siguiente se muestra cómo crear un contenedor con la clave de acceso de la cuenta. Especifique la clave de cuenta y proporcione el parámetro `--auth-mode` con el valor `key`:

```azurecli
az storage container create \
    --account-name <storage-account> \
    --name sample-container \
    --account-key <key>
    --auth-mode key
```

> [!IMPORTANT]
> Cuando una cuenta de almacenamiento se bloquea con un bloqueo **ReadOnly** de Azure Resource Manager, no se permite la operación [Enumerar claves](/rest/api/storagerp/storageaccounts/listkeys) para esa cuenta de almacenamiento. **Enumerar claves** es una operación POST y todas las operaciones POST se evitan cuando se configura un bloqueo **ReadOnly** para la cuenta. Por esta razón, cuando la cuenta se bloquea con un bloqueo **ReadOnly**, los usuarios deben tener acceso a los datos con credenciales de Azure AD.

## <a name="authorize-with-a-sas-token"></a>Autorización con un token de SAS

Si posee un token de SAS, puede llamar a las operaciones de datos permitidas por la firma de acceso compartido. En el ejemplo siguiente se muestra cómo crear un contenedor con un token de SAS:

```azurecli
az storage container create \
    --account-name <storage-account> \
    --name sample-container \
    --sas-token <token>
```

## <a name="set-environment-variables-for-authorization-parameters"></a>Establecimiento de variables de entorno para parámetros de autorización

Puede especificar parámetros de autorización en variables de entorno para evitar incluirlos en cada llamada a una operación de datos de Azure Storage. En la tabla siguiente se describen las variables de entorno disponibles.

| Variable de entorno                  | Descripción                                                                                                                                                                                                                                                                                                                                                                     |
|---------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    AZURE_STORAGE_ACCOUNT              |    El nombre de la cuenta de almacenamiento. Esta variable se debe usar junto con la clave de cuenta de almacenamiento o un token de SAS. Si ninguno de ellos está presente, la CLI de Azure intentará recuperar la clave de acceso de la cuenta de almacenamiento con la cuenta de Azure AD autenticada. Si se ejecutan un gran número de comandos al mismo tiempo, se puede alcanzar la limitación del proveedor de recursos de Azure Storage. Para obtener más información sobre los límites del proveedor de recursos, consulte [Destinos de escalabilidad y rendimiento para el proveedor de recursos de Azure Storage](../common/scalability-targets-resource-provider.md).             |
|    AZURE_STORAGE_KEY                  |    La clave de la cuenta de almacenamiento. Esta variable debe usarse junto con el nombre de la cuenta de almacenamiento.                                                                                                                                                                                                                                                                          |
|    AZURE_STORAGE_CONNECTION_STRING    |    Una cadena de conexión que incluye la clave de cuenta de almacenamiento o un token de SAS. Esta variable debe usarse junto con el nombre de la cuenta de almacenamiento.                                                                                                                                                                                                                       |
|    AZURE_STORAGE_SAS_TOKEN            |    Un token de firma de acceso compartido (SAS). Esta variable debe usarse junto con el nombre de la cuenta de almacenamiento.                                                                                                                                                                                                                                                            |
|    AZURE_STORAGE_AUTH_MODE            |    El modo de autorización con el que se ejecuta el comando. Los valores permitidos son `login` (recomendado) o `key`. Si especifica `login`, la CLI de Azure utilizará sus credenciales de Azure AD para autorizar la operación de datos. Si especifica el modo `key` heredado, la CLI de Azure intentará consultar la clave de acceso de la cuenta y autorizar el comando con la clave.    |

## <a name="next-steps"></a>Pasos siguientes

- [Uso de la CLI de Azure para asignar un rol de Azure para obtener acceso a datos de blobs y colas](../common/storage-auth-aad-rbac-cli.md)
- [Autorización del acceso a datos de blobs y colas con identidades administradas para los recursos de Azure](../common/storage-auth-aad-msi.md)