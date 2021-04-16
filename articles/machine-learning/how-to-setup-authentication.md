---
title: Configuración de la autenticación
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo instalar y configurar la autenticación para varios recursos y flujos de trabajo en Azure Machine Learning.
services: machine-learning
author: cjgronlund
ms.author: cgronlun
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 04/02/2021
ms.topic: conceptual
ms.custom: how-to, has-adal-ref, devx-track-js, devx-track-azurecli, contperf-fy21q2
ms.openlocfilehash: ee767cbeafcce604ecb4a79ce0fdf21c70bbb47c
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106219644"
---
# <a name="set-up-authentication-for-azure-machine-learning-resources-and-workflows"></a>Configuración de la autenticación para recursos y flujos de trabajo de Azure Machine Learning


Obtenga información sobre cómo configurar la autenticación en el área de trabajo de Azure Machine Learning. La autenticación en el área de trabajo de Azure Machine Learning se basa en __Azure Active Directory__ (Azure AD) en la mayoría de los casos. En general, hay tres flujos de trabajo de autenticación que se pueden usar al conectarse al área de trabajo:

* __Interactiva:__ el usuario utiliza la cuenta en Azure Active Directory para autenticarse directamente o para obtener un token que se usa para la autenticación. La autenticación interactiva se usa durante la _experimentación y el desarrollo iterativo_. La autenticación interactiva permite controlar el acceso a los recursos (por ejemplo, un servicio web) por usuario.

* __Entidad de servicio__: el usuario crea una cuenta de entidad de servicio en Azure Active Directory y la usa para autenticarse u obtener un token. Una entidad de servicio se usa cuando se necesita un _proceso automatizado para autenticarse_ en el servicio sin necesidad de interacción del usuario. Por ejemplo, un script de implementación e integración continua que entrena y prueba un modelo cada vez que cambia el código de entrenamiento.

* __Identidad administrada__: Si utiliza el SDK de Azure Machine Learning _en una máquina virtual de Azure_, puede usar una identidad administrada para Azure. Este flujo de trabajo permite a la máquina virtual conectarse al área de trabajo mediante la identidad administrada, sin almacenar las credenciales en el código de Python ni pedir al usuario que se autentique. Los clústeres de proceso de Azure Machine Learning también se pueden configurar para usar una identidad administrada para acceder al área de trabajo cuando se _entrenen modelos_.

> [!IMPORTANT]
> Independientemente del flujo de trabajo de autenticación que se use, el control de acceso basado en roles de Azure (Azure RBAC) se usa para limitar el nivel de acceso (autorización) permitido a los recursos. Por ejemplo, un proceso de administración o de automatización podría acceder para crear una instancia de proceso, pero no usarla, mientras que un científico de datos podría usarla, pero no eliminarla ni crearla. Para obtener más información, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md).

## <a name="prerequisites"></a>Requisitos previos

* Cree un [área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).
* [Configure un entorno de desarrollo](how-to-configure-environment.md) para instalar el SDK de Azure Machine Learning o use una [instancia de proceso de Azure Machine Learning](concept-azure-machine-learning-architecture.md#compute-instance) con el SDK ya instalado.

## <a name="azure-active-directory"></a>Azure Active Directory

Todos los flujos de trabajo de autenticación del área de trabajo dependen de Azure Active Directory. Si desea que los usuarios se autentiquen mediante cuentas individuales, deben tener cuentas en su instancia de Azure AD. Si desea usar entidades de servicio, deben existir en su instancia de Azure AD. Las identidades administradas también son una característica de Azure AD. 

Para obtener más información sobre Azure AD, consulte [¿Qué es la autenticación de Azure Active Directory?](..//active-directory/authentication/overview-authentication.md).

Una vez que haya creado las cuentas de Azure AD, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md) para obtener información sobre cómo concederles acceso al área de trabajo y a otras operaciones en Azure Machine Learning.

## <a name="configure-a-service-principal"></a>Configuración de una entidad de servicio

Para usar una entidad de servicio, primero debe crear la entidad de servicio y concederle acceso al área de trabajo. Como se mencionó anteriormente, se usa el control de acceso basado en rol (RBAC) de Azure para controlar el acceso, por lo que también debe decidir qué acceso conceder a la entidad de servicio.

> [!IMPORTANT]
> Cuando use una entidad de servicio, concédale el __acceso mínimo necesario para la tarea__ para la que se usa. Por ejemplo, no debería conceder acceso a un propietario de la entidad de servicio o a un colaborador si todo para lo que se usa es para leer el token de acceso para una implementación web.
>
> La razón para conceder el menor acceso es que una entidad de servicio usa una contraseña para autenticarse y la contraseña se puede almacenar como parte de un script de automatización. Si se pierde la contraseña, tener el acceso mínimo necesario para una tarea específica minimiza el uso malintencionado de la entidad de servicio.

La forma más fácil de crear una entidad de servicio y de conceder acceso al área de trabajo es mediante la [CLI de Azure](/cli/azure/install-azure-cli). Para crear una entidad de servicio y concederle acceso al área de trabajo, siga estos pasos:

> [!NOTE]
> Debe ser un administrador de la suscripción para realizar todos estos pasos.

1. Autentíquese en la suscripción de Azure:

    ```azurecli-interactive
    az login
    ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión. De lo contrario, tendrá que abrir un explorador y seguir las instrucciones de la línea de comandos. Las instrucciones implican navegar a [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escribir un código de autorización.

    Si tiene varias suscripciones de Azure, puede usar el comando `az account set -s <subscription name or ID>` para establecer la suscripción. Para más información, consulte [Use multiple Azure subscriptions](/cli/azure/manage-azure-subscriptions-azure-cli) (Uso de varias suscripciones de Azure).

    Para obtener otros métodos de autenticación, consulte [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

1. Cree la entidad de servicio. En el ejemplo siguiente, se crea una entidad de servicio denominada **ml-auth**:

    ```azurecli-interactive
    az ad sp create-for-rbac --sdk-auth --name ml-auth
    ```

    La salida será un código JSON similar al siguiente. Tome nota de los campos `clientId`, `clientSecret` y `tenantId`, ya que los necesitará para otros pasos de este artículo.

    ```json
    {
        "clientId": "your-client-id",
        "clientSecret": "your-client-secret",
        "subscriptionId": "your-sub-id",
        "tenantId": "your-tenant-id",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com",
        "activeDirectoryGraphResourceId": "https://graph.windows.net",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:5555",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net"
    }
    ```

1. Recupere los detalles de la entidad de servicio mediante el valor `clientId` devuelto en el paso anterior:

    ```azurecli-interactive
    az ad sp show --id your-client-id
    ```

    El siguiente JSON es un ejemplo simplificado de la salida del comando. Tome nota del campo `objectId`, ya que necesitará su valor para el paso siguiente.

    ```json
    {
        "accountEnabled": "True",
        "addIns": [],
        "appDisplayName": "ml-auth",
        ...
        ...
        ...
        "objectId": "your-sp-object-id",
        "objectType": "ServicePrincipal"
    }
    ```

1. Permita que la entidad de servicio tenga acceso al área de trabajo de Azure Machine Learning. Necesitará el nombre del área de trabajo y el nombre de su grupo de recursos para los parámetros `-w` y `-g`, respectivamente. Para el parámetro `--user`, use el valor `objectId` del paso anterior. El parámetro `--role` permite establecer el rol de acceso para la entidad de servicio. En el ejemplo siguiente, la entidad de servicio se asigna al rol de **propietario**. 

    > [!IMPORTANT]
    > El acceso de propietario permite que la entidad de servicio realice prácticamente cualquier operación en el área de trabajo. Se usa en este documento para demostrar cómo conceder acceso. En un entorno de producción, Microsoft recomienda conceder a la entidad de servicio el acceso mínimo necesario para realizar el rol para el que se pretende usar. Para obtener información sobre cómo crear un rol personalizado con el acceso necesario para su escenario, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md).

    ```azurecli-interactive
    az ml workspace share -w your-workspace-name -g your-resource-group-name --user your-sp-object-id --role owner
    ```

    Esta llamada no produce ningún resultado en caso de éxito.

## <a name="configure-a-managed-identity"></a>Configuración de una identidad administrada

> [!IMPORTANT]
> La identidad administrada solo se admite cuando se usa el SDK de Azure Machine Learning desde una máquina virtual de Azure o con un clúster de proceso de Azure Machine Learning. La capacidad de usar una identidad administrada con un clúster de proceso se encuentra actualmente en versión preliminar.

### <a name="managed-identity-with-a-vm"></a>Identidad administrada con una máquina virtual

1. Habilite una [identidad administrada asignada por el sistema para los recursos de Azure en la máquina virtual](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

1. En [Azure Portal](https://portal.azure.com), seleccione el área de trabajo y, a continuación, __Control de acceso (IAM)__ , __Agregar asignación de rol__ y __Máquina virtual__ en la lista desplegable __Asignar acceso a__. Por último, seleccione la identidad de la máquina virtual.

1. Seleccione el rol que se asignará a esta identidad. Por ejemplo, el de colaborador o un rol personalizado. Para más información, consulte [Control de acceso a los recursos](how-to-assign-roles.md).

### <a name="managed-identity-with-compute-cluster"></a>Identidad administrada con un clúster de proceso

Para obtener más información, vea [Configuración de identidades administradas para el clúster de proceso](how-to-create-attach-compute-cluster.md#managed-identity).

<a id="interactive-authentication"></a>

## <a name="use-interactive-authentication"></a>Uso de la autenticación interactiva

> [!IMPORTANT]
> En la autenticación interactiva se usa el explorador y se necesitan cookies (por ejemplo, cookies de terceros). Si ha deshabilitado las cookies, es posible que reciba un error del tipo "no se pudo iniciar sesión". Este error también puede producirse si ha habilitado [Multi-Factor Authentication de Azure AD](../active-directory/authentication/concept-mfa-howitworks.md).

La mayoría de los ejemplos de la documentación usan la autenticación interactiva. Por ejemplo, al usar el SDK hay dos llamadas de función que muestran automáticamente un flujo de autenticación basada en la interfaz de usuario:

* La llamada a la función `from_config()` emitirá el mensaje.

    ```python
    from azureml.core import Workspace
    ws = Workspace.from_config()
    ```

    La función `from_config()` busca un archivo JSON que contenga la información de conexión del área de trabajo.

* Con el constructor `Workspace` para proporcionar información sobre la suscripción, el grupo de recursos y el área de trabajo, también se solicitará la autenticación interactiva.

    ```python
    ws = Workspace(subscription_id="your-sub-id",
                  resource_group="your-resource-group-id",
                  workspace_name="your-workspace-name"
                  )
    ```

> [!TIP]
> Si tiene acceso a varios inquilinos, es posible que tenga que importar la clase y definir explícitamente el inquilino que tiene como destino. La llamada al constructor de `InteractiveLoginAuthentication` también le pedirá que inicie sesión de forma similar a las llamadas anteriores.
>
> ```python
> from azureml.core.authentication import InteractiveLoginAuthentication
> interactive_auth = InteractiveLoginAuthentication(tenant_id="your-tenant-id")
> ```

Al usar la CLI de Azure, se usa el comando `az login` para autenticar la sesión de la CLI. Para más información, consulte [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli).

> [!TIP]
> Si usa el SDK desde un entorno en el que ya se ha autenticado de forma interactiva mediante la CLI de Azure, puede usar la clase `AzureCliAuthentication` para autenticarse en el área de trabajo mediante las credenciales almacenadas en caché por la CLI:
>
> ```python
> from azureml.core.authentication import AzureCliAuthentication
> cli_auth = AzureCliAuthentication()
> ws = Workspace(subscription_id="your-sub-id",
>                resource_group="your-resource-group-id",
>                workspace_name="your-workspace-name",
>                auth=cli_auth
>                )
> ```

<a id="service-principal-authentication"></a>

## <a name="use-service-principal-authentication"></a>Uso de la autenticación de entidad de servicio

Para autenticarse en el área de trabajo desde el SDK mediante la entidad de servicio, use el constructor de clase `ServicePrincipalAuthentication`. Utilice los valores que recibió al crear el proveedor de servicios como parámetros. El parámetro `tenant_id` se asigna al campo `tenantId` anterior, `service_principal_id` se asigna a `clientId`y `service_principal_password` se asigna a `clientSecret`.

```python
from azureml.core.authentication import ServicePrincipalAuthentication

sp = ServicePrincipalAuthentication(tenant_id="your-tenant-id", # tenantID
                                    service_principal_id="your-client-id", # clientId
                                    service_principal_password="your-client-secret") # clientSecret
```

La variable `sp` ahora contiene un objeto de autenticación que se usa directamente en el SDK. En general, se recomienda almacenar los identificadores y secretos usados anteriormente en las variables de entorno, tal como se muestra en el código siguiente. El almacenamiento en variables de entorno impide que la información se compruebe accidentalmente en un repositorio de GitHub.

```python
import os

sp = ServicePrincipalAuthentication(tenant_id=os.environ['AML_TENANT_ID'],
                                    service_principal_id=os.environ['AML_PRINCIPAL_ID'],
                                    service_principal_password=os.environ['AML_PRINCIPAL_PASS'])
```

En el caso de los flujos de trabajo automatizados que se ejecutan en Python y usan el SDK principalmente, puede usar este objeto tal cual en la mayoría de los casos para la autenticación. El código siguiente se autentica en el área de trabajo mediante el objeto de autenticación que acaba de crear.

```python
from azureml.core import Workspace

ws = Workspace.get(name="ml-example",
                   auth=sp,
                   subscription_id="your-sub-id")
ws.get_details()
```

### <a name="use-a-service-principal-from-the-azure-cli"></a>Uso de una entidad de servicio desde la CLI de Azure

Puede usar una entidad de servicio para los comandos de la CLI de Azure. Para más información, vea [Inicio de sesión mediante una entidad de servicio](/cli/azure/create-an-azure-service-principal-azure-cli#sign-in-using-a-service-principal).

### <a name="use-a-service-principal-with-the-rest-api-preview"></a>Uso de una entidad de servicio con la API REST (versión preliminar)

La entidad de servicio también se puede usar para autenticar en la [API REST](/rest/api/azureml/) de Azure Machine Learning (versión preliminar). Use el [flujo de concesión de credenciales de cliente](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) de Azure Active Directory, que permiten llamadas de servicio a servicio para la autenticación sin periféricos en flujos de trabajo automatizados. Los ejemplos se implementan con la [biblioteca de ADAL](../active-directory/azuread-dev/active-directory-authentication-libraries.md) tanto en Python como en Node.js, pero también puede usar cualquier biblioteca de código abierto que admita OpenID Connect 1.0.

> [!NOTE]
> MSAL.js es una biblioteca más reciente que ADAL, pero no puede realizar la autenticación de servicio a servicio mediante las credenciales de cliente con MSAL.js, ya que es principalmente una biblioteca del lado cliente diseñada para la autenticación interactiva o de interfaz de usuario asociada a un usuario específico. Se recomienda usar ADAL como se muestra a continuación para crear flujos de trabajo automatizados con la API REST.

#### <a name="nodejs"></a>Node.js

Siga estos pasos para generar un token de autenticación mediante Node.js. En su entorno, ejecute `npm install adal-node`. A continuación, use su `tenantId`, `clientId` y `clientSecret` de la entidad de servicio que creó en los pasos anteriores como valores para las variables coincidentes en el siguiente script.

```javascript
const adal = require('adal-node').AuthenticationContext;

const authorityHostUrl = 'https://login.microsoftonline.com/';
const tenantId = 'your-tenant-id';
const authorityUrl = authorityHostUrl + tenantId;
const clientId = 'your-client-id';
const clientSecret = 'your-client-secret';
const resource = 'https://management.azure.com/';

const context = new adal(authorityUrl);

context.acquireTokenWithClientCredentials(
  resource,
  clientId,
  clientSecret,
  (err, tokenResponse) => {
    if (err) {
      console.log(`Token generation failed due to ${err}`);
    } else {
      console.dir(tokenResponse, { depth: null, colors: true });
    }
  }
);
```

La variable `tokenResponse` es un objeto que incluye el token y los metadatos asociados, como la fecha de expiración. Los tokens son válidos durante una hora y se pueden actualizar ejecutando la misma llamada de nuevo para recuperar un nuevo token. El siguiente fragmento es una respuesta de ejemplo.

```javascript
{
    tokenType: 'Bearer',
    expiresIn: 3599,
    expiresOn: 2019-12-17T19:15:56.326Z,
    resource: 'https://management.azure.com/',
    accessToken: "random-oauth-token",
    isMRRT: true,
    _clientId: 'your-client-id',
    _authority: 'https://login.microsoftonline.com/your-tenant-id'
}
```

Use la propiedad `accessToken` para capturar el token de autenticación. Consulte la [documentación de la API REST](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API) para ejemplos sobre cómo usar el token para realizar llamadas API.

#### <a name="python"></a>Python

Siga estos pasos para generar un token de autenticación mediante Python. En su entorno, ejecute `pip install adal`. A continuación, use su `tenantId`, `clientId` y `clientSecret` de la entidad de servicio que creó en los pasos anteriores como valores para las variables apropiadas en el siguiente script.

```python
from adal import AuthenticationContext

client_id = "your-client-id"
client_secret = "your-client-secret"
resource_url = "https://login.microsoftonline.com"
tenant_id = "your-tenant-id"
authority = "{}/{}".format(resource_url, tenant_id)

auth_context = AuthenticationContext(authority)
token_response = auth_context.acquire_token_with_client_credentials("https://management.azure.com/", client_id, client_secret)
print(token_response)
```

La variable `token_response` es un diccionario que incluye el token y los metadatos asociados, como la fecha de expiración. Los tokens son válidos durante una hora y se pueden actualizar ejecutando la misma llamada de nuevo para recuperar un nuevo token. El siguiente fragmento es una respuesta de ejemplo.

```python
{
    'tokenType': 'Bearer',
    'expiresIn': 3599,
    'expiresOn': '2019-12-17 19:47:15.150205',
    'resource': 'https://management.azure.com/',
    'accessToken': 'random-oauth-token',
    'isMRRT': True,
    '_clientId': 'your-client-id',
    '_authority': 'https://login.microsoftonline.com/your-tenant-id'
}
```

Use `token_response["accessToken"]` para capturar el token de autenticación. Consulte la [documentación de la API REST](https://github.com/microsoft/MLOps/tree/master/examples/AzureML-REST-API) para ejemplos sobre cómo usar el token para realizar llamadas API.

#### <a name="java"></a>Java

En Java, recupere el token de portador mediante una llamada REST estándar:

```java
String tenantId = "your-tenant-id";
String clientId = "your-client-id";
String clientSecret = "your-client-secret";
String resourceManagerUrl = "https://management.azure.com";

HttpRequest tokenAuthenticationRequest = tokenAuthenticationRequest(tenantId, clientId, clientSecret, resourceManagerUrl);

HttpClient client = HttpClient.newBuilder().build();
Gson gson = new Gson();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
if (response.statusCode == 200)
{
     body = gson.fromJson(body, AuthenticationBody.class);

    // ... etc ... 
}
// ... etc ...

static HttpRequest tokenAuthenticationRequest(String tenantId, String clientId, String clientSecret, String resourceManagerUrl){
    String authUrl = String.format("https://login.microsoftonline.com/%s/oauth2/token", tenantId);
    String clientIdParam = String.format("client_id=%s", clientId);
    String resourceParam = String.format("resource=%s", resourceManagerUrl);
    String clientSecretParam = String.format("client_secret=%s", clientSecret);

    String bodyString = String.format("grant_type=client_credentials&%s&%s&%s", clientIdParam, resourceParam, clientSecretParam);

    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(authUrl))
            .POST(HttpRequest.BodyPublishers.ofString(bodyString))
            .build();
    return request;
}

class AuthenticationBody {
    String access_token;
    String token_type;
    int expires_in;
    String scope;
    String refresh_token;
    String id_token;
    
    AuthenticationBody() {}
}
```

El código anterior debería controlar las excepciones y los códigos de estado distintos de `200 OK`, pero muestra el siguiente patrón: 

- Uso del identificador y secreto de cliente para validar que el programa debe tener acceso
- Uso del identificador de inquilino para especificar dónde debe realizar la búsqueda `login.microsoftonline.com`
- Uso de Azure Resource Manager como origen del token de autorización

## <a name="use-managed-identity-authentication"></a>Uso de la autenticación de identidad administrada

Para autenticarse en el área de trabajo desde una máquina virtual o un clúster de proceso que está configurado con una identidad administrada, utilice la clase `MsiAuthentication`. En el ejemplo siguiente se muestra cómo utilizar esta clase para autenticarse en un área de trabajo:

```python
from azureml.core.authentication import MsiAuthentication

msi_auth = MsiAuthentication()

ws = Workspace(subscription_id="your-sub-id",
                resource_group="your-resource-group-id",
                workspace_name="your-workspace-name",
                auth=msi_auth
                )
```

## <a name="next-steps"></a>Pasos siguientes

* [Uso de secretos en ejecuciones de entrenamiento](how-to-use-secrets-in-runs.md).
* [Configuración de la autenticación para los modelos implementados como servicios web](how-to-authenticate-web-service.md)
* [Consumir un modelo de Azure Machine Learning que está implementado como un servicio web](how-to-consume-web-service.md).
