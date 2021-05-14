---
title: 'Inicio rápido: Su primera consulta de Go'
description: En este inicio rápido, dará los pasos necesarios para habilitar el paquete de Resource Graph para Go y ejecutará la primera consulta.
ms.date: 05/01/2021
ms.topic: quickstart
ms.openlocfilehash: 42e2ee9b5fc5c34fab2785d32b8b2de55dee0d71
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108751794"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-go"></a>Inicio rápido: Ejecución de la primera consulta de Resource Graph con Go

El primer paso para usar Azure Resource Graph es comprobar que están instalados los paquetes necesarios para Go. Este inicio rápido lo guiará a través del proceso de agregar los paquetes a la instalación de Go.

Cuando finalice, habrá agregado los paquetes a la instalación de Go y ejecutado la primera consulta de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="add-the-resource-graph-package"></a>Adición del paquete de Resource Graph

Para que Go pueda consultar Azure Resource Graph, se debe agregar el paquete. Este paquete funciona siempre que se pueda usar Go, lo que incluye [bash en Windows 10](/windows/wsl/install-win10) o instalado localmente.

1. Compruebe que está instalada la versión más reciente de Go (al menos la **1.14**). Si aún no está instalada, descárguela en [Golang.org](https://golang.org/dl/).

1. Compruebe que está instalada la versión más reciente de la CLI de Azure (al menos la **2.5.1**). Si aún no está instalada, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Se requiere la CLI de Azure para permitir que Go utilice el método `auth.NewAuthorizerFromCLI()` en el ejemplo siguiente. Para más información sobre otras opciones, consulte [Azure SDK para Go: Detalles de autenticación](https://github.com/Azure/azure-sdk-for-go#more-authentication-details).

1. Autentíquese mediante la CLI de Azure.

   ```azurecli
   az login
   ```

1. En el entorno de Go que elija, instale los paquetes necesarios para Azure Resource Graph:

   ```bash
   # Add the Resource Graph package for Go
   go get -u github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph

   # Add the Azure auth package for Go
   go get -u github.com/Azure/go-autorest/autorest/azure/auth
   ```

## <a name="run-your-first-resource-graph-query"></a>Ejecutar la primera consulta de Resource Graph

Una vez agregados los paquetes de Go al entorno de su elección, es el momento de probar una consulta simple de Resource Graph. La consulta devolverá los cinco primeros recursos de Azure con el **nombre** y el **tipo de recurso** de cada recurso.

1. Cree la aplicación Go y guarde el siguiente origen como `argQuery.go`:

   ```go
   package main

   import (
      "fmt"
      "os"
      "context"
      "strconv"
      arg "github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph"
      "github.com/Azure/go-autorest/autorest/azure/auth"
   )

   func main() {
       // Get variables from command line arguments
       var query = os.Args[1]
       var subList = os.Args[2:]

       // Create and authorize a ResourceGraph client
       argClient := arg.New()
       authorizer, err := auth.NewAuthorizerFromCLI()
       if err == nil {
           argClient.Authorizer = authorizer
       } else {
           fmt.Printf(err.Error())
       }

       // Set options
       RequestOptions := arg.QueryRequestOptions {
           ResultFormat: "objectArray",
       }

       // Create the query request
       Request := arg.QueryRequest {
           Subscriptions: &subList,
           Query: &query,
           Options: &RequestOptions,
       }

       // Run the query and get the results
       var results, queryErr = argClient.Resources(context.Background(), Request)
       if queryErr == nil {
           fmt.Printf("Resources found: " + strconv.FormatInt(*results.TotalRecords, 10) + "\n")
           fmt.Printf("Results: " + fmt.Sprint(results.Data) + "\n")
       } else {
           fmt.Printf(queryErr.Error())
       }
   }
   ```

1. Compile la aplicación Go:

   ```bash
   go build argQuery.go
   ```

1. Ejecute la primera consulta de Azure Resource Graph mediante la aplicación Go compilada. Reemplace `<SubID>` por el identificador de suscripción:

   ```bash
   argQuery "Resources | project name, type | limit 5" "<SubID>"
   ```

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Cambie el primer parámetro a `argQuery` y cambie la consulta para realizar la operación `order by` en la propiedad **Nombre**. Reemplace `<SubID>` por el identificador de suscripción:

   ```bash
   argQuery "Resources | project name, type | limit 5 | order by name asc" "<SubID>"
   ```

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Cambie el primer parámetro a `argQuery` y cambie la consulta para realizar la operación `order by` primero en la propiedad **Nombre** y, luego, para realizar la operación `limit` a los cinco primeros resultados. Reemplace `<SubID>` por el identificador de suscripción:

   ```bash
   argQuery "Resources | project name, type | order by name asc | limit 5" "<SubID>"
   ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea quitar los paquetes instalados del entorno de Go, puede hacerlo mediante el comando siguiente:

```bash
# Remove the installed packages from the Go environment
go clean -i github.com/Azure/azure-sdk-for-go/services/resourcegraph/mgmt/2019-04-01/resourcegraph
go clean -i github.com/Azure/go-autorest/autorest/azure/auth
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha agregado los paquetes de Resource Graph al entorno de Go y ha ejecutado la primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)
