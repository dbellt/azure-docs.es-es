---
title: Configuración de identidades administradas con Azure App Configuration
description: Obtenga información sobre cómo funcionan las identidades administradas en Azure App Configuration y cómo configurar una identidad administrada.
author: barbkess
ms.topic: article
ms.date: 02/25/2020
ms.author: barbkess
ms.reviewer: lcozzens
ms.service: azure-app-configuration
ms.openlocfilehash: e4fdff2515dde941b2e9037a21ad931ac27b6fef
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764232"
---
# <a name="how-to-use-managed-identities-for-azure-app-configuration"></a>Uso de identidades administradas para Azure App Configuration

En este tema se muestra cómo crear una identidad administrada para Azure App Configuration. Una identidad administrada de Azure Active Directory (AAD) permite a Azure App Configuration acceder fácilmente a otros recursos protegidos por AAD, como Azure Key Vault. La plataforma de Azure administra la identidad. No es necesario que aprovisione ni rote ningún secreto. Para más información sobre las identidades administradas en AAD, consulte [Identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md).

La aplicación puede tener dos tipos de identidades:

- Una **identidad asignada por el sistema** está asociada al almacén de configuración. Se elimina si se elimina el almacén de configuración. Un almacén de configuración solo puede tener una identidad asignada por el sistema.
- Una **identidad asignada por el usuario** es un recurso de Azure independiente que se puede asignar al almacén de configuración. Un almacén de configuración puede tener varias identidades asignadas por el usuario.

## <a name="adding-a-system-assigned-identity"></a>Adición de una identidad asignada por el sistema

Para crear un almacén de App Configuration con una identidad asignada por el sistema se requiere establecer una propiedad adicional en el almacén.

### <a name="using-the-azure-cli"></a>Uso de la CLI de Azure

Para configurar una identidad administrada mediante la CLI de Azure, use el comando [az appconfig identity assign] en un almacén de configuración existente. Tiene tres opciones para ejecutar los ejemplos de esta sección:

- Usar [Azure Cloud Shell](../cloud-shell/overview.md) desde Azure Portal.
- Use Azure Cloud Shell integrado mediante el botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
- [Instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli) (2.1 o posterior) si prefiere usar una consola de la CLI local.

Los siguientes pasos le guiarán en la creación de un almacén de App Configuration y la asignación al mismo de una identidad mediante la CLI:

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az]. Utilice una cuenta asociada a su suscripción de Azure:

    ```azurecli-interactive
    az login
    ```

1. Cree un almacén de App Configuration mediante la CLI. Para más ejemplos sobre cómo usar la CLI con Azure App Configuration, consulte [Ejemplos de la CLI de App Configuration](scripts/cli-create-service.md):

    ```azurecli-interactive
    az group create --name myResourceGroup --location eastus
    az appconfig create --name myTestAppConfigStore --location eastus --resource-group myResourceGroup --sku Free
    ```

1. Ejecute el comando [az appconfig identity assign] para crear la identidad asignada por el sistema para este almacén de configuración:

    ```azurecli-interactive
    az appconfig identity assign --name myTestAppConfigStore --resource-group myResourceGroup
    ```

## <a name="adding-a-user-assigned-identity"></a>Adición de una identidad asignada por el usuario

La creación de un almacén de App Configuration con una identidad asignada por el usuario requiere que se cree la identidad y luego se asigne su identificador de recurso al almacén.

### <a name="using-the-azure-cli"></a>Uso de la CLI de Azure

Para configurar una identidad administrada mediante la CLI de Azure, use el comando [az appconfig identity assign] en un almacén de configuración existente. Tiene tres opciones para ejecutar los ejemplos de esta sección:

- Usar [Azure Cloud Shell](../cloud-shell/overview.md) desde Azure Portal.
- Use Azure Cloud Shell integrado mediante el botón "Pruébelo", situado en la esquina superior derecha de cada bloque de código.
- [Instale la versión más reciente de la CLI de Azure](/cli/azure/install-azure-cli) (2.0.31 o posterior) si prefiere usar una consola de CLI local.

Los pasos siguientes le guiarán en la creación de una identidad asignada por el usuario y un almacén de App Configuration y la asignación de la identidad al almacén mediante la CLI:

1. Si usa la CLI de Azure en una consola local, lo primero que debe hacer es iniciar sesión en Azure mediante el [inicio de sesión de az]. Utilice una cuenta asociada a su suscripción de Azure:

    ```azurecli-interactive
    az login
    ```

1. Cree un almacén de App Configuration mediante la CLI. Para más ejemplos sobre cómo usar la CLI con Azure App Configuration, consulte [Ejemplos de la CLI de App Configuration](scripts/cli-create-service.md):

    ```azurecli-interactive
    az group create --name myResourceGroup --location eastus
    az appconfig create --name myTestAppConfigStore --location eastus --resource-group myResourceGroup --sku Free
    ```

1. Cree una identidad asignada por el usuario llamada `myUserAssignedIdentity` mediante la CLI.

    ```azurecli-interactive
    az identity create -resource-group myResourceGroup --name myUserAssignedIdentity
    ```

    Anote el valor de la propiedad `id` de la salida de este comando.

1. Ejecute el comando [az appconfig identity assign] para asignar la nueva identidad asignada por el usuario a este almacén de configuración. Use el valor de la propiedad `id` que anotó en el paso anterior.

    ```azurecli-interactive
    az appconfig identity assign --name myTestAppConfigStore --resource-group myResourceGroup --identities /subscriptions/[subscription id]/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
    ```

## <a name="removing-an-identity"></a>Eliminación una identidad

Una identidad asignada por el sistema se puede eliminar; para ello, deshabilite la característica mediante el comando [az appconfig identity remove](/cli/azure/appconfig/identity#az_appconfig_identity_remove) de la CLI de Azure. Las identidades asignadas por el usuario se pueden quitar individualmente. Al quitar una identidad asignada por el sistema de esta manera también se eliminará de AAD. Las identidades asignadas por el sistema también se quitan automáticamente de AAD cuando se elimina el recurso de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación ASP.NET Core con Azure App Configuration](quickstart-aspnet-core-app.md)

[az appconfig identity assign]: /cli/azure/appconfig/identity#az_appconfig_identity_assign
[inicio de sesión de az]: /cli/azure/reference-index#az_login
