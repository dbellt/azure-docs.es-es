---
title: 'Tutorial: Uso de la identidad administrada para invocar Azure Functions'
description: Uso de la identidad administrada para invocar Azure Functions desde una aplicación de Azure Spring Cloud
author: MarkGardner
ms.author: margard
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 07/10/2020
ms.openlocfilehash: b737ea751d3b3d2132691e04a1a2cd853748db65
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107792514"
---
# <a name="tutorial-use-a-managed-identity-to-invoke-azure-functions-from-an-azure-spring-cloud-app"></a>Tutorial: Uso de una identidad administrada para invocar Azure Functions desde una aplicación de Azure Spring Cloud

En este artículo se describe cómo crear una identidad administrada para una aplicación de Azure Spring Cloud y cómo usarla para invocar funciones desencadenadas por HTTP.

Tanto Azure Functions como App Services tienen compatibilidad integrada para la autenticación de Azure Active Directory (Azure AD). Al aprovechar esta funcionalidad de autenticación integrada junto con las identidades administradas de Azure Spring Cloud, se pueden invocar servicios RESTful con la semántica moderna de OAuth. Este método no requiere almacenar secretos en el código y proporciona controles más pormenorizados del acceso a los recursos externos. 


## <a name="prerequisites"></a>Requisitos previos

* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* [Instalación de la CLI de Azure versión 2.0.67 o superior](/cli/azure/install-azure-cli)
* [Instalación de Maven 3.0, o cualquier versión superior](https://maven.apache.org/download.cgi)
* [Instalación de la versión 3.0.2009 de Azure Functions Core Tools o una posterior](../azure-functions/functions-run-local.md#install-the-azure-functions-core-tools)


## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos que contenga tanto la aplicación de funciones como Spring Cloud mediante el comando [az group create](/cli/azure/group#az_group_create):

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```


## <a name="create-a-function-app"></a>Creación de una aplicación de funciones
Para crear una aplicación de funciones, primero debe crear la cuenta de almacenamiento subyacente; para ello, use el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create):

> [!Important]
> Las aplicaciones de funciones y las cuentas de almacenamiento deben tener nombres únicos. En los ejemplos siguientes, reemplace <nombreDeLaAplicaciónDeFunciones> por el nombre de la aplicación de funciones y <nombreDeLaCuentaDeAlmacenamiento> por el nombre de la cuenta de almacenamiento.

```azurecli-interactive
az storage account create --name <your-storageaccount-name> --resource-group myResourceGroup --location eastus --sku Standard_LRS
```

Una vez creada la cuenta de almacenamiento, podrá crear la aplicación de funciones.

```azurecli-interactive
az functionapp create --name <your-functionapp-name> --resource-group myResourceGroup --consumption-plan-location eastus --os-type windows --runtime node --storage-account <your-storageaccount-name> --functions-version 3
```

Tome nota del valor de **hostNames** devuelto, que tendrá el formato "https://<nombreDeLaAplicaciónDeFunciones>.azurewebsites.net". Se usará en un paso posterior.


## <a name="enable-azure-active-directory-authentication"></a>Habilitación de la autenticación de Azure Active Directory

Acceda a la aplicación de funciones recién creada desde [Azure Portal](https://portal.azure.com) y seleccione "Autenticación/autorización" en el menú de configuración. Habilite la autenticación de App Service y establezca "Acción necesaria cuando la solicitud no está autenticada" en "Iniciar sesión con Azure Active Directory". Esta configuración garantizará que se denieguen todas las solicitudes no autenticadas (respuesta 401).

![Configuración de autenticación que muestra Azure Active Directory como proveedor predeterminado](media/spring-cloud-tutorial-managed-identities-functions/function-auth-config-1.jpg)

En Proveedores de autenticación, seleccione Azure Active Directory para configurar el registro de la aplicación. Al seleccionar Modo de administración Rápido, se creará automáticamente un registro de aplicación en el inquilino de Azure AD con la configuración correcta.

![Proveedor Azure Active Directory establecido en Modo de administración Rápido](media/spring-cloud-tutorial-managed-identities-functions/function-auth-config-2.jpg)

Una vez guardada la configuración, la aplicación de funciones se reiniciará y todas las solicitudes posteriores deberán iniciar sesión mediante Azure AD. Para probar que las solicitudes no autenticadas se están rechazando, vaya a la dirección URL raíz de la aplicación de funciones (que se devuelve en la salida **hostNames** en el paso anterior). Se le redirigirá a la pantalla de inicio de sesión de Azure AD de su organización.


## <a name="create-an-http-triggered-function"></a>Creación de una función desencadenada mediante HTTP

En un directorio local vacío, cree una aplicación de funciones y agregue una función desencadenada mediante HTTP.

```console
func init --worker-runtime node
func new --template HttpTrigger --name HttpTrigger
```

De forma predeterminada, las funciones usan la autenticación basada en claves para proteger los puntos de conexión HTTP. Como vamos a habilitar la autenticación de Azure AD para proteger el acceso a las funciones, queremos [establecer el nivel de autenticación en las funciones como anónimo](../azure-functions/functions-bindings-http-webhook-trigger.md#secure-an-http-endpoint-in-production).

```json function.json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      ...
    }
  ]
}
```

La aplicación ahora se puede publicar en la instancia de [aplicación de Functions](#create-a-function-app) creada en el paso anterior.

```console
func azure functionapp publish <your-functionapp-name>
```

La salida del comando publish debe mostrar la dirección URL de la función recién creada.

```output
Deployment completed successfully.
Syncing triggers...
Functions in <your-functionapp-name>:
    HttpTrigger - [httpTrigger]
        Invoke url: https://<your-functionapp-name>.azurewebsites.net/api/httptrigger
```


## <a name="create-azure-spring-cloud-service-and-app"></a>Creación de una aplicación y un servicio de Azure Spring Cloud
Después de instalar la extensión spring-cloud, cree una instancia de Azure Spring Cloud con el comando de la CLI de Azure `az spring-cloud create`. 

```azurecli-interactive
az extension add --name spring-cloud
az spring-cloud create --name mymsispringcloud --resource-group myResourceGroup --location eastus
```

En el ejemplo siguiente, se crea una aplicación denominada `msiapp` con una identidad administrada asignada por el sistema, según solicitó el parámetro `--assign-identity`.

```azurecli
az spring-cloud app create --name "msiapp" --service "mymsispringcloud" --resource-group "myResourceGroup" --assign-endpoint true --assign-identity
```

## <a name="build-sample-spring-boot-app-to-invoke-the-function"></a>Creación de una aplicación de Spring Boot de ejemplo para invocar la función

En este ejemplo se invocará la función desencadenada por HTTP; para ello, primero se solicita un token de acceso desde el [punto de conexión de MSI](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md#get-a-token-using-http) y, después, ese token se usa para autenticar la solicitud HTTP de la función.

1. Clone el proyecto de ejemplo. 

    ```console
    git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples.git
    ```

2. Especifique el URI de la función y el nombre del desencadenador en las propiedades de la aplicación. 

    ```bash
    cd Azure-Spring-Cloud-Samples/managed-identity-function
    vim src/main/resources/application.properties
    ```

    Para usar la identidad administrada para las aplicaciones de Azure Spring Cloud, agregue propiedades con el siguiente contenido a *src/main/resources/application.properties*.

    ```
    azure.function.uri=https://<your-functionapp-name>.azurewebsites.net
    azure.function.triggerPath=httptrigger
    ```

3. Empaquete la aplicación de ejemplo. 

    ```console
    mvn clean package
    ```

4. Ahora implemente la aplicación en Azure con el comando `az spring-cloud app deploy` de la CLI de Azure. 

    ```azurecli
    az spring-cloud app deploy  --name "msiapp" --service "mymsispringcloud" --resource-group "myResourceGroup" --jar-path target/sc-managed-identity-function-sample-0.1.0.jar
    ```

5. Acceda al punto de conexión público o al punto de conexión de prueba para probar la aplicación. 

    ```console
    curl https://mymsispringcloud-msiapp.azuremicroservices.io/func/springcloud
    ```

    Verá el siguiente mensaje devuelto en el cuerpo de la respuesta.
    ```output
    Function Response: Hello, springcloud. This HTTP triggered function executed successfully.
    ```
    
    Cambie el parámetro path para intentar pasar valores diferentes a la función.

## <a name="next-steps"></a>Pasos siguientes

* [Habilitación de la identidad administrada asignada por el sistema para una aplicación de Azure Spring Cloud](./spring-cloud-howto-enable-system-assigned-managed-identity.md)
* [Más información sobre las identidades administradas para recursos de Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Configuración de las aplicaciones cliente para acceder a la instancia de App Service](../app-service/configure-authentication-provider-aad.md#configure-client-apps-to-access-your-app-service)
