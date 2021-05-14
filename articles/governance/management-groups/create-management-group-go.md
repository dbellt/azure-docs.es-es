---
title: 'Inicio rápido: Creación de un grupo de administración con Go'
description: En este inicio rápido se usa Go para crear un grupo de administración para organizar los recursos en una jerarquía.
ms.date: 03/31/2021
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 32665afec15d17b2bc15d61853ef6b1dc58388fa
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108733722"
---
# <a name="quickstart-create-a-management-group-with-go"></a>Inicio rápido: Creación de un grupo de administración con Go

Los grupos de administración son contenedores que ayudan a administran el acceso, las directivas y el cumplimiento de varias suscripciones. Cree estos contenedores para construir una jerarquía eficaz y eficiente que pueda usarse con [Azure Policy](../policy/overview.md) y los [controles de acceso basados en roles de Azure](../../role-based-access-control/overview.md). Para obtener más información sobre los grupos de administración, consulte [Organización de los recursos con grupos de administración de Azure](overview.md).

El primer grupo de administración creado en el directorio puede tardar hasta 15 minutos en completarse. Hay procesos que se ejecutan la primera vez para configurar el servicio de grupos de administración dentro de Azure para su directorio. Recibirá una notificación cuando finalice el proceso. Para obtener más información, vea [Instalación inicial de los grupos de administración](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- Una entidad de servicio de Azure, que incluya _clientId_ y _clientSecret_. Si no tiene una entidad de servicio para usarla con Azure Policy o si desea crear una, consulte [Bibliotecas de administración de Azure para la autenticación de .NET](/dotnet/azure/sdk/authentication#mgmt-auth).
  Omita el paso en el que se instalan los paquetes de .NET Core, ya que lo haremos en los pasos siguientes.

- Si la [protección de jerarquías](./how-to/protect-resource-hierarchy.md#setting---require-authorization) no está habilitada, cualquier usuario de Azure AD del inquilino puede crear un grupo de administración sin tener el permiso de escritura del grupo de administración asignado. Este nuevo grupo de administración pasa a ser un elemento secundario del grupo de administración raíz o del [grupo de administración predeterminado](./how-to/protect-resource-hierarchy.md#setting---default-management-group) y al creador se le asigna el rol de "Propietario". El servicio del grupo de administración ofrece esta funcionalidad para que las asignaciones de roles no sean necesarias en el nivel raíz. Ningún usuario tiene acceso al grupo de administración raíz cuando se crea. Para sortear el obstáculo de buscar administradores globales de Azure AD para empezar a usar grupos de administración, se permite la creación de grupos de administración iniciales en el nivel raíz.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-management-group-package"></a>Incorporación del paquete del grupo de administración

Para habilitar Go para administrar grupos de administración, el paquete debe agregarse. Este paquete funciona siempre que se pueda usar Go, lo que incluye [bash en Windows 10](/windows/wsl/install-win10) o instalado localmente.

1. Compruebe que está instalada la versión más reciente de Go (al menos la **1.15**). Si aún no está instalada, descárguela en [Golang.org](https://golang.org/dl/).

1. Compruebe que está instalada la versión más reciente de la CLI de Azure (al menos la **2.5.1**). Si aún no está instalada, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Se requiere la CLI de Azure para permitir que Go utilice el método `auth.NewAuthorizerFromCLI()` en el ejemplo siguiente. Para más información sobre otras opciones, consulte [Azure SDK para Go: Detalles de autenticación](https://github.com/Azure/azure-sdk-for-go#more-authentication-details).

1. Autentíquese mediante la CLI de Azure.

   ```azurecli
   az login
   ```

1. En el entorno de Go que elija, instale los paquetes necesarios para los grupos de administración:

   ```bash
   # Add the management group package for Go
   go get -u github.com/Azure/azure-sdk-for-go/services/preview/resources/mgmt/2018-03-01-preview/managementgroups

   # Add the Azure auth package for Go
   go get -u github.com/Azure/go-autorest/autorest/azure/auth
   ```

## <a name="application-setup"></a>Configuración de la aplicación

Con los paquetes de Go agregados al entorno elegido, es el momento de configurar la aplicación de Go para crear un grupo de administración.

1. Cree la aplicación Go y guarde el siguiente origen como `mgCreate.go`:

   ```go
   package main

   import (
    "context"
    "fmt"
    "os"

    mg "github.com/Azure/azure-sdk-for-go/services/preview/resources/mgmt/2018-03-01-preview/managementgroups"
    "github.com/Azure/go-autorest/autorest/azure/auth"
   )

   func main() {
    // Get variables from command line arguments
    var mgName = os.Args[1]

    // Create and authorize a client
    mgClient := mg.NewClient()
    authorizer, err := auth.NewAuthorizerFromCLI()
    if err == nil {
        mgClient.Authorizer = authorizer
    } else {
        fmt.Printf(err.Error())
    }

    // Create the request
    Request := mg.CreateManagementGroupRequest{
        Name: &mgName,
    }

    // Run the query and get the results
    var results, queryErr = mgClient.CreateOrUpdate(context.Background(), mgName, Request, "no-cache")
    if queryErr == nil {
        fmt.Printf("Results: " + fmt.Sprint(results) + "\n")
    } else {
        fmt.Printf(queryErr.Error())
    }
   }
   ```

1. Compile la aplicación Go:

   ```bash
   go build mgCreate.go
   ```

1. Cree un grupo de administración mediante la aplicación Go creada. Reemplace `<Name>` por el nombre del grupo de administración nuevo:

   ```bash
   mgCreate "<Name>"
   ```

El resultado es un nuevo grupo de administración en el grupo de administración raíz.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar los paquetes instalados del entorno de Go, puede hacerlo mediante el comando siguiente:

```bash
# Remove the installed packages from the Go environment
go clean -i github.com/Azure/azure-sdk-for-go/services/preview/resources/mgmt/2018-03-01-preview/managementgroups
go clean -i github.com/Azure/go-autorest/autorest/azure/auth
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de administración para organizar una jerarquía de recursos. El grupo de administración puede contener suscripciones u otros grupos de administración.

Para más información acerca de los grupos de administración y cómo administrar la jerarquía de recursos, vaya a:

> [!div class="nextstepaction"]
> [Administración de los recursos con grupos de administración](./manage.md)
