---
title: Acceso al almacenamiento con una identidad administrada de Media Services
description: Si quiere acceder a una cuenta de almacenamiento cuando está configurada para bloquear las solicitudes de direcciones IP desconocidas, se debe conceder a la cuenta de Media Services acceso a la cuenta de almacenamiento. Siga los pasos que se indican a continuación para crear una identidad administrada para la cuenta de Media Services y conceder a esta identidad acceso al almacenamiento mediante la CLI de Media Services.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: tutorial
ms.date: 05/17/2021
ms.author: inhenkel
ms.openlocfilehash: 77eb737a48213701e424bbc8b1a49ead0c95e88c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110464303"
---
# <a name="tutorial-access-storage-with-a-media-services-managed-identity"></a>Tutorial: Acceso al almacenamiento con una identidad administrada de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Si quiere acceder a una cuenta de almacenamiento cuando está configurada para bloquear las solicitudes de direcciones IP desconocidas, se debe conceder a la cuenta de Media Services acceso a la cuenta de almacenamiento. Siga los pasos que se indican a continuación para crear una identidad administrada para la cuenta de Media Services y conceder a esta identidad acceso al almacenamiento mediante la CLI de Media Services.

:::image type="content" source="media/diagrams/managed-identities-scenario-storage-permissions-media-services-account.svg" alt-text="La cuenta de Media Services usa una identidad administrada para acceder al almacenamiento":::

En este tutorial se usa la API de Media Services 2020-05-01.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para usar cualquiera de los comandos de este artículo, primero debe haber iniciado sesión en la suscripción que quiere usar.

 [!INCLUDE [Sign in to Azure with the CLI](./includes/task-sign-in-azure-cli.md)]

### <a name="set-subscription"></a>Definir la suscripción

Use este comando para establecer la suscripción con la que quiere trabajar.

[!INCLUDE [Sign in to Azure with the CLI](./includes/task-set-azure-subscription-cli.md)]

## <a name="resource-names"></a>Nombres de recurso

Antes de empezar, decida los nombres de los recursos que va a crear.  Estos deben identificarse fácilmente como un conjunto, especialmente si no planea usarlos una vez que haya terminado las pruebas. Las reglas de nomenclatura son diferentes para muchos tipos de recursos, por lo que es mejor mantener las minúsculas. Por ejemplo, "mediatest1rg" como nombre del grupo de recursos y "mediatest1stor" como nombre de la cuenta de almacenamiento. Use los mismos nombres en cada paso de este artículo.

Verá que se hace referencia a ellos en los comandos siguientes.  Los nombres de los recursos que necesitará son:

- your-resource-group-name
- your-storage-account-name
- your-media-services-account-name
- your-region

> [!NOTE]
> Los guiones anteriores solo se usan para separar las palabras de guía. Debido a la incoherencia de asignar nombres a los recursos en los servicios de Azure, no use guiones al asignar un nombre a los recursos.
> Además, no cree el nombre de la región.  Azure es quien lo determina.

### <a name="list-azure-regions"></a>Enumeración de regiones de Azure

Si no está seguro del nombre real de la región que se va a usar, utilice este comando para obtener una lista:

[!INCLUDE [List Azure regions with the CLI](./includes/task-list-azure-regions-cli.md)]

## <a name="sequence"></a>Secuencia

Cada uno de los pasos siguientes se realiza en un orden determinado porque uno o varios valores de las respuestas JSON se usan en el paso siguiente de la secuencia.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Los recursos que va a crear deben pertenecer a un grupo de recursos. Cree primero el grupo de recursos. Usará `your-resource-group-name` para la creación de la cuenta de Media Services y en los pasos posteriores.

[!INCLUDE [Create a resource group with the CLI](./includes/task-create-resource-group-cli.md)]

## <a name="create-a-storage-account"></a>Creación de una cuenta de Storage

La cuenta de Media Services que va a crear debe tener una cuenta de almacenamiento asociada. Cree primero la cuenta de almacenamiento para la cuenta de Media Services. Usará `your-storage-account-name` en los pasos posteriores.

[!INCLUDE [Create a Storage account with the CLI](./includes/task-create-storage-account-cli.md)]

## <a name="create-a-media-services-account-with-a-service-principal-managed-identity"></a>Creación de una cuenta de Media Services con una entidad de servicio (identidad administrada)

Ahora cree la cuenta de Media Services con una entidad de servicio, también conocida como identidad administrada.

> [!IMPORTANT]
> Es importante que recuerde usar la marca --mi en el comando.  De lo contrario, no podrá encontrar el elemento `principalId` en un paso posterior.

[!INCLUDE [Create a Media Services account with the CLI](./includes/task-create-media-services-account-managed-identity-cli.md)]

## <a name="grant-the-media-services-managed-identity-access-to-the-storage-account"></a>Concesión a la identidad administrada de Media Services acceso a la cuenta de almacenamiento

Conceda a la identidad administrada de Media Services acceso a la cuenta de almacenamiento. Hay tres comandos:

### <a name="get-show-the-managed-identity-of-the-media-services-account"></a>Obtener (mostrar) la identidad administrada de la cuenta de Media Services

El primer comando muestra la identidad administrada de la cuenta de Media Services, que es el valor de `principalId` enumerado en el código JSON devuelto por el comando.

[!INCLUDE [Show the Managed Identity of a Media Services account with the CLI](./includes/task-show-account-managed-identity-cli.md)]

### <a name="create-the-storage-blob-contributor-role-assignment"></a>Creación de la asignación del rol Colaborador de Storage Blob

[!INCLUDE [Create the Storage Blob Contributor role assignment](./includes/task-create-storage-blob-contributor-role-cli.md)]

### <a name="create-the-reader-role-assignment"></a>Creación de la asignación del rol Lector

[!INCLUDE [Create the Reader role assignment](./includes/task-create-reader-role-cli.md)]

## <a name="use-the-managed-identity-to-access-the-storage-account"></a>Uso de la identidad administrada para acceder a la cuenta de almacenamiento

[!INCLUDE [Use the Managed Identity to access the Storage account](./includes/task-set-storage-managed-identity-cli.md)]

## <a name="validation"></a>Validación

Para comprobar que la cuenta está cifrada mediante una clave administrada por el cliente, vea las propiedades de cifrado de la cuenta:

[!INCLUDE [Set Media Services to use the key from Key Vault](./includes/task-show-account-managed-identity-cli.md)]

La propiedad `storageAuthentication` debe mostrar "ManagedIdentity".

Para realizar una validación adicional, puede comprobar los registros de Azure Storage para ver qué método de autenticación se usa para cada solicitud.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no tiene previsto usar los recursos que ha creado, elimine el grupo de recursos.

[!INCLUDE [Create a Media Services account with the CLI](./includes/clean-up-resources-cli.md)]
