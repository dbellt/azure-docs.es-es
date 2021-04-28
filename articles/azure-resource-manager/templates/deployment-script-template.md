---
title: Uso de scripts de implementación en plantillas | Microsoft Docs
description: Use scripts de implementación en plantillas de Azure Resource Manager.
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 04/15/2021
ms.author: jgao
ms.openlocfilehash: c39b332e9ee62a8502d5e2fdf155819194a30e34
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762198"
---
# <a name="use-deployment-scripts-in-arm-templates"></a>Uso de scripts de implementación en plantillas de Resource Manager

Aprenda a usar scripts de implementación en plantillas de Azure Resource Manager (plantillas de ARM). Con un nuevo tipo de recurso denominado `Microsoft.Resources/deploymentScripts`, los usuarios pueden ejecutar scripts en implementaciones de plantilla y revisar los resultados de la ejecución. Estos scripts se pueden usar para realizar pasos personalizados tales como:

- Adición de usuarios a un directorio
- Realización de operaciones de plano de datos como, por ejemplo, copiar blobs o una base de datos de inicialización
- Búsqueda y validación de una clave de licencia
- Creación de un certificado autofirmado.
- Creación de un objeto en Azure AD
- Búsqueda de bloques de direcciones IP en el sistema personalizado

Ventajas del script de implementación:

- Fácil de programar, usar y depurar. Puede desarrollar scripts de implementación en sus entornos de desarrollo favoritos. Los scripts se pueden insertar en plantillas o en archivos de script externos.
- Puede especificar el lenguaje y la plataforma del script. Actualmente, se admiten scripts de implementación de Azure PowerShell y la CLI de Azure en el entorno de Linux.
- Permite pasar cuatro argumentos de la línea de comandos al script.
- Puede especificar salidas de script y pasarlas de nuevo a la implementación.

El recurso de script de implementación solo está disponible en las regiones donde Azure Container Instances está disponible.  Consulte [Disponibilidad de recursos para Azure Container Instances en las regiones de Azure](../../container-instances/container-instances-region-availability.md).

> [!IMPORTANT]
> Se necesita una cuenta de almacenamiento y una instancia de contenedor para la ejecución de scripts y la solución de problemas. Tiene las opciones para especificar una cuenta de almacenamiento existente; de lo contrario, el servicio de script crea automáticamente la cuenta de almacenamiento, junto con la instancia del contenedor. Normalmente, el servicio de script elimina los dos recursos creados automáticamente cuando la ejecución del script de implementación llega a un estado terminal. Los recursos se le facturarán hasta que se eliminen. Para más información, consulte [Limpieza de los recursos del script de implementación](#clean-up-deployment-script-resources).

> [!IMPORTANT]
> La versión 2020-10-01 de la API de recursos de deploymentScripts admite [OnBehalfofTokens(OBO)](../../active-directory/develop/v2-oauth2-on-behalf-of-flow.md). Mediante OBO, el servicio de scripts de implementación utiliza el token de la entidad de seguridad de la implementación para crear los recursos subyacentes para la ejecución de scripts de implementación, entre los que se incluyen Azure Container Instances, las cuentas de Azure Storage y las asignaciones de roles de la identidad administrada. En la versión anterior de la API, la identidad administrada se usa para crear estos recursos.
> La lógica de reintento de inicio de sesión de Azure está ahora integrada en el script contenedor. Si concede permisos en la misma plantilla en la que se ejecutan los scripts de implementación, el servicio del script de implementación volverá a intentar el inicio de sesión durante 10 minutos a intervalos de 10 segundos hasta que se replique la asignación de roles de la identidad administrada.

## <a name="configure-the-minimum-permissions"></a>Configuración de los permisos mínimos

Con la API del script de implementación versión 2020-10-01 o posterior, la entidad de seguridad de implementación se usa para crear los recursos subyacentes necesarios para que el recurso del script de implementación se ejecute: una cuenta de almacenamiento y una instancia de contenedor de Azure. Si el script tiene que autenticarse en Azure y realizar acciones específicas de Azure, se recomienda proporcionar el script con una identidad administrada asignada por el usuario. La identidad administrada debe tener el acceso necesario para completar la operación del script.

Para configurar los permisos con privilegios mínimos, necesita:

- Asignar un rol personalizado con las siguientes propiedades a la entidad de seguridad de implementación:

  ```json
  {
    "roleName": "deployment-script-minimum-privilege-for-deployment-principal",
    "description": "Configure least privilege for the deployment principal in deployment script",
    "type": "customRole",
    "IsCustom": true,
    "permissions": [
      {
        "actions": [
          "Microsoft.Storage/storageAccounts/*",
          "Microsoft.ContainerInstance/containerGroups/*",
          "Microsoft.Resources/deployments/*",
          "Microsoft.Resources/deploymentScripts/*"
        ],
      }
    ],
    "assignableScopes": [
      "[subscription().id]"
    ]
  }
  ```

  Si los proveedores de recursos de Azure Storage y de Instancia de Azure Container no se han registrado, también debe agregar `Microsoft.Storage/register/action` y `Microsoft.ContainerInstance/register/action`.

- Si se usa una identidad administrada, la entidad de seguridad de implementación necesita que el rol **Operador de identidades administradas** (un rol integrado) esté asignado al recurso de identidad administrada.

## <a name="sample-templates"></a>Plantillas de ejemplo

El siguiente código JSON es un ejemplo: Para más información, consulte el [esquema de plantilla](/azure/templates/microsoft.resources/deploymentscripts) más reciente.

```json
{
  "type": "Microsoft.Resources/deploymentScripts",
  "apiVersion": "2020-10-01",
  "name": "runPowerShellInline",
  "location": "[resourceGroup().location]",
  "kind": "AzurePowerShell", // or "AzureCLI"
  "identity": {
    "type": "userAssigned",
    "userAssignedIdentities": {
      "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myID": {}
    }
  },
  "properties": {
    "forceUpdateTag": "1",
    "containerSettings": {
      "containerGroupName": "mycustomaci"
    },
    "storageAccountSettings": {
      "storageAccountName": "myStorageAccount",
      "storageAccountKey": "myKey"
    },
    "azPowerShellVersion": "3.0",  // or "azCliVersion": "2.0.80",
    "arguments": "-name \\\"John Dole\\\"",
    "environmentVariables": [
      {
        "name": "UserName",
        "value": "jdole"
      },
      {
        "name": "Password",
        "secureValue": "jDolePassword"
      }
    ],
    "scriptContent": "
      param([string] $name)
      $output = 'Hello {0}. The username is {1}, the password is {2}.' -f $name,${Env:UserName},${Env:Password}
      Write-Output $output
      $DeploymentScriptOutputs = @{}
      $DeploymentScriptOutputs['text'] = $output
    ", // or "primaryScriptUri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.ps1",
    "supportingScriptUris":[],
    "timeout": "PT30M",
    "cleanupPreference": "OnSuccess",
    "retentionInterval": "P1D"
  }
}
```

> [!NOTE]
> Este ejemplo se ofrece con fines de demostración. Las propiedades `scriptContent` y `primaryScriptUri` no pueden coexistir en una plantilla.

> [!NOTE]
> El valor de _scriptContent_ muestra un script con varias líneas.  Azure Portal y la canalización de Azure DevOps no pueden analizar un script de implementación con varias líneas. Puede encadenar los comandos de PowerShell (mediante puntos y comas o _\\r\\n_ o _\\n_) en una sola línea, o bien usar la propiedad `primaryScriptUri` con un archivo de script externo. Hay muchas herramientas gratuitas disponibles de los valores "escape" o "unescape" de la cadena JSON. Por ejemplo, [https://www.freeformatter.com/json-escape.html](https://www.freeformatter.com/json-escape.html).

Detalles de los valores de propiedad:

- `identity`: En la versión 2020-10-01, u otra posterior, de la API del script de implementación, una identidad administrada asignada por el usuario es opcional a menos que necesite realizar cualquier acción específica de Azure en el script.  En la versión 2019-10-01-preview de la API se requiere una identidad administrada, ya que el servicio del script de implementación la usa para ejecutar los scripts. Cuando se especifica la propiedad de identidad, el servicio de script llama a `Connect-AzAccount -Identity` antes de invocar el script de usuario. Actualmente, solo se admiten identidades asignadas por el usuario. Para iniciar sesión con otra identidad, puede llamar a [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount) en el script.
- `kind`: especifica el tipo de script. Actualmente, se admiten los scripts de Azure PowerShell y de la CLI de Azure. Los valores son **AzurePowerShell** y **AzureCLI**.
- `forceUpdateTag`: el cambio de este valor entre implementaciones de plantilla obliga a que se vuelva a ejecutar el script de implementación. Si usa la función `newGuid()` o `utcNow()`, ambas funciones solo se pueden usar en el valor predeterminado de un parámetro. Para más información, consulte la sección [Ejecución de un script varias veces](#run-script-more-than-once).
- `containerSettings`: permite especificar la configuración para personalizar la instancia de contenedor de Azure. El script de implementación requiere una nueva instancia de Azure Container Instance. No se puede especificar una instancia de Azure Container Instance existente. Sin embargo, puede personalizar el nombre del grupo de contenedores mediante `containerGroupName`. Si no se especifica, el nombre de grupo se genera automáticamente.
- `storageAccountSettings`: permite especificar la configuración para usar una cuenta de almacenamiento existente. Si no se especifica `storageAccountName`, se crea una cuenta de almacenamiento automáticamente. Consulte [Uso de una cuenta de almacenamiento existente](#use-existing-storage-account).
- `azPowerShellVersion`/`azCliVersion`: Especifique la versión del módulo que se va a usar. Consulte una lista de [versiones de Azure PowerShell compatibles](https://mcr.microsoft.com/v2/azuredeploymentscripts-powershell/tags/list). Consulte una lista de [versiones de la CLI de Azure compatibles](https://mcr.microsoft.com/v2/azure-cli/tags/list).

  >[!IMPORTANT]
  > El script de implementación usa las imágenes de la CLI disponibles de Microsoft Container Registry (MCR). Se tarda aproximadamente un mes en certificar una imagen de la CLI para el script de implementación. No utilice las versiones de la CLI que se publicaron en un plazo de 30 días. Para buscar las fechas de publicación de las imágenes, consulte las [notas de la versión de la CLI de Azure](/cli/azure/release-notes-azure-cli). Si se usa una versión no compatible, el mensaje de error muestra las versiones admitidas.

- `arguments`: Especifique los valores de los parámetros. Los valores se separan con espacios.

  Los scripts de implementación dividen los argumentos en una matriz de cadenas mediante la invocación de la llamada del sistema [CommandLineToArgvW](/windows/win32/api/shellapi/nf-shellapi-commandlinetoargvw). Este paso es necesario porque los argumentos se pasan como una [propiedad del comando](/rest/api/container-instances/containergroups/createorupdate#containerexec) a Azure Container Instances, y la propiedad del comando es una matriz de cadena.

  Si los argumentos contienen caracteres de escape, use [JsonEscaper](https://www.jsonescaper.com/) para que estos caracteres sean de escape doble. Pegue la cadena de escape original en la herramienta y, a continuación, seleccione **Escape**.  La herramienta genera una cadena de escape doble. Por ejemplo, en la plantilla de ejemplo anterior, el argumento es `-name \"John Dole\"`. La cadena de escape es `-name \\\"John Dole\\\"`.

  Para pasar un parámetro de plantilla de Resource Manager de tipo objeto como argumento, convierta el objeto en una cadena mediante la función [string()](./template-functions-string.md#string) y, luego, use la función [replace()](./template-functions-string.md#replace) para reemplazar cualquier `\"` por `\\\"`. Por ejemplo:

  ```json
  replace(string(parameters('tables')), '\"', '\\\"')
  ```

  Para más información, consulte la [plantilla de ejemplo](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-jsonEscape.json).

- `environmentVariables`: especifique las variables de entorno que se van a pasar al script. Para más información, consulte [Desarrollo de scripts de implementación](#develop-deployment-scripts).
- `scriptContent`: especifica el contenido del script. Para ejecutar un script externo, use `primaryScriptUri` en su lugar. Para ver ejemplos, consulte [Uso scripts en línea](#use-inline-scripts) y [Uso de scripts externos](#use-external-scripts).
- `primaryScriptUri`: especifique una dirección URL de acceso público al script de implementación principal con las extensiones de archivo compatibles. Para obtener más información, consulte [Uso de scripts externos](#use-external-scripts).
- `primaryScriptUri`: especifique una matriz de direcciones URL de acceso público a los archivos auxiliares a los que se llame en `supportingScriptUris` o `scriptContent`. Para obtener más información, consulte [Uso de scripts externos](#use-external-scripts).
- `timeout`: especifique el tiempo máximo de ejecución de scripts permitido en [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). El valor predeterminado es **P1D**.
- `cleanupPreference`. especifique la preferencia de limpieza de recursos de implementación cuando la ejecución del script llegue a un estado terminal. La configuración predeterminada es **Siempre**, lo que significa que se eliminan los recursos a pesar del estado terminal (correcto, error, cancelado). Para obtener más información, vea el artículo sobre [limpieza de los recursos del script de implementación](#clean-up-deployment-script-resources).
- `retentionInterval`: especifique el intervalo durante el que el servicio conserva los recursos del script de implementación cuando este llega a un estado terminal. Los recursos del script de implementación se eliminarán cuando expire este periodo. La duración se basa en el [patrón ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). El intervalo de retención se sitúa entre 1 y 26 horas (PT26H). Esta propiedad se usa cuando `cleanupPreference` se establece en **OnExpiration**. Para más información, consulte [Limpieza de los recursos del script de implementación](#clean-up-deployment-script-resources).

### <a name="additional-samples"></a>Ejemplos adicionales

- [Ejemplo 1](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault.json): cree un almacén de claves y use el script de implementación para asignar un certificado al almacén de claves.
- [Ejemplo 2](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault-subscription.json): cree un grupo de recursos en el nivel de suscripción, cree un almacén de claves en el grupo de recursos y, a continuación, use el script de implementación para asignar un certificado al almacén de claves.
- [Ejemplo 3](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault-mi.json): cree una identidad administrada asignada por el usuario, asigne el rol de colaborador a la identidad en el nivel de grupo de recursos, cree un almacén de claves y, a continuación, use el script de implementación para asignar un certificado al almacén de claves.

## <a name="use-inline-scripts"></a>Uso de scripts en línea

La plantilla siguiente tiene un recurso definido con el tipo `Microsoft.Resources/deploymentScripts`. La parte resaltada es el script insertado.

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-helloworld.json" range="1-44" highlight="24-30":::

> [!NOTE]
> Como los scripts de implementación insertados se escriben entre comillas dobles, las cadenas dentro de los scripts de implementación deben incluirse entre comillas simples o entre caracteres de escape ( **&#92;** ). También puede plantearse la posibilidad de usar la sustitución de cadenas tal y como se muestra en el ejemplo anterior de JSON.

El script toma un parámetro y genera el valor del parámetro. `DeploymentScriptOutputs` se usa para almacenar las salidas. En la sección de salidas, la línea `value` muestra cómo acceder a los valores almacenados. `Write-Output` se utiliza con fines de depuración. Para obtener información sobre cómo acceder al archivo de salida, consulte la sección sobre cómo [supervisar y solucionar problemas de los scripts de implementación](#monitor-and-troubleshoot-deployment-scripts). Para ver las descripciones de las propiedades, consulte [Plantillas de ejemplo](#sample-templates).

Para ejecutar el script, seleccione **Pruébelo** para abrir Cloud Shell y luego pegue el código siguiente en el panel de Shell.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to be created"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.json"

Write-Host "Press [ENTER] to continue ..."
```

El resultado tendrá una apariencia similar a la siguiente:

![Salida del script Hola mundo de implementación de la plantilla de Resource Manager](./media/deployment-script-template/resource-manager-template-deployment-script-helloworld-output.png)

## <a name="use-external-scripts"></a>Uso de scripts externos

Además de los scripts en línea, también puede usar archivos de script externos. Solo se admiten scripts de PowerShell principales con la extensión de archivo _ps1_. En el caso de los scripts de la CLI, los scripts principales pueden tener extensiones (o no), siempre que los scripts sean scripts de Bash válidos. Para usar archivos de script externos, reemplace `scriptContent` por `primaryScriptUri`. Por ejemplo:

```json
"primaryScriptUri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.ps1",
```

Para más información, consulte la [plantilla de ejemplo](https://github.com/Azure/azure-docs-json-samples/blob/master/deployment-script/deploymentscript-helloworld-primaryscripturi.json).

Los archivos de script externos deben ser accesibles. Para proteger los archivos de script que se almacenan en cuentas de almacenamiento de Azure, genere un token de SAS e inclúyalo en el URI de la plantilla. Establezca el tiempo de expiración con un margen suficiente para completar la implementación. Para obtener más información, consulte [Implementación de una plantilla de Resource Manager privada con el token de SAS](./secure-template-with-sas-token.md).

Tiene la responsabilidad de garantizar la integridad de los scripts a los que hace referencia el script de implementación, ya sea `primaryScriptUri` o `supportingScriptUris`. Haga referencia solo a scripts en los que confíe.

## <a name="use-supporting-scripts"></a>Uso de scripts auxiliares

Puede separar las lógicas complicadas en uno o varios archivos de script auxiliar. En caso necesario, la propiedad `supportingScriptUris` le permite especificar una matriz de identificadores URI para los archivos de script auxiliar:

```json
"scriptContent": "
    ...
    ./Create-Cert.ps1
    ...
"

"supportingScriptUris": [
  "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/create-cert.ps1"
],
```

Se puede llamar a archivos de script auxiliar desde scripts en línea y archivos de script principal. Los archivos de scripts auxiliares no tienen ninguna restricción relativa a la extensión de archivo.

Los archivos auxiliares se copian en `azscripts/azscriptinput` en el entorno de ejecución. Use la ruta de acceso relativa para hacer referencia a los archivos auxiliares desde scripts en línea y archivos de script principal.

## <a name="work-with-outputs-from-powershell-script"></a>Tareas con las salidas del script de PowerShell

En la plantilla siguiente se muestra cómo pasar valores entre dos recursos `deploymentScripts`:

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-basic.json" range="1-68" highlight="30-31,50":::

En el primer recurso, defina una variable denominada `$DeploymentScriptOutputs` y úsela para almacenar los valores de salida. Para acceder al valor de salida de otro recurso dentro de la plantilla, use:

```json
reference('<ResourceName>').outputs.text
```

## <a name="work-with-outputs-from-cli-script"></a>Tareas con las salidas del script de la CLI

A diferencia del script de implementación de PowerShell, la compatibilidad con la CLI/Bash no expone una variable común para almacenar las salidas del script, sino que hay una variable de entorno denominada `AZ_SCRIPTS_OUTPUT_PATH` que almacena la ubicación donde reside el archivo de salida del script. Si se ejecuta un script de implementación desde una plantilla de Resource Manager, esta variable de entorno se establece automáticamente en el shell de Bash. El varlo de `AZ_SCRIPTS_OUTPUT_PATH` es */mnt/azscripts/azscriptoutput/scriptoutputs.json*.

Las salidas del script de implementación se deben guardar en la ubicación `AZ_SCRIPTS_OUTPUT_PATH`, y las salidas deben ser un objeto de cadena JSON válido. El contenido del archivo se debe guardar como par clave-valor. Por ejemplo, una matriz de cadenas se almacena como `{ "MyResult": [ "foo", "bar"] }`.  No es válido almacenar solo los resultados de la matriz, por ejemplo `[ "foo", "bar" ]`.

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-basic-cli.json" range="1-44" highlight="32":::

[jq](https://stedolan.github.io/jq/) se usa en el ejemplo anterior. Se incluye con las imágenes de contenedor. Consulte [Configuración del entorno de desarrollo](#configure-development-environment).

## <a name="use-existing-storage-account"></a>Uso de la cuenta de almacenamiento existente

Se necesita una cuenta de almacenamiento y una instancia de contenedor para la ejecución de scripts y la solución de problemas. Tiene las opciones para especificar una cuenta de almacenamiento existente; de lo contrario, el servicio de script crea automáticamente la cuenta de almacenamiento, junto con la instancia de contenedor. Requisitos para usar una cuenta de almacenamiento existente:

- Tipos de cuenta de almacenamiento admitidos:

    | SKU             | Tipo admitido     |
    |-----------------|--------------------|
    | Premium_LRS     | FileStorage        |
    | Premium_ZRS     | FileStorage        |
    | Standard_GRS    | Storage, StorageV2 |
    | Standard_GZRS   | StorageV2          |
    | Standard_LRS    | Storage, StorageV2 |
    | Standard_RAGRS  | Storage, StorageV2 |
    | Standard_RAGZRS | StorageV2          |
    | Standard_ZRS    | StorageV2          |

    Estas combinaciones admiten recursos compartidos de archivos. Para más información, vea [Creación de un recurso compartido de archivos de Azure](../../storage/files/storage-how-to-create-file-share.md) e [Introducción a las cuentas de almacenamiento](../../storage/common/storage-account-overview.md).

- Todavía no se admiten las reglas de firewall de la cuenta de almacenamiento. Para más información, vea [Configuración de Firewalls y redes virtuales de Azure Storage](../../storage/common/storage-network-security.md).
- La entidad de seguridad de la implementación debe tener permisos para administrar la cuenta de almacenamiento, lo que incluye los recursos compartidos de archivos de lectura, creación y eliminación.

Para especificar una cuenta de almacenamiento existente, agregue el siguiente código JSON al elemento de propiedad de `Microsoft.Resources/deploymentScripts`:

```json
"storageAccountSettings": {
  "storageAccountName": "myStorageAccount",
  "storageAccountKey": "myKey"
},
```

- `storageAccountName`: especifica el nombre de la cuenta de almacenamiento.
- `storageAccountKey`: especifica una de las claves de cuenta de almacenamiento. Se puede usar la función [listKeys()](./template-functions-resource.md#listkeys) para recuperar la clave. Por ejemplo:

    ```json
    "storageAccountSettings": {
        "storageAccountName": "[variables('storageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
    }
    ```

Consulte [Plantillas de ejemplo](#sample-templates) para obtener una muestra de definición de `Microsoft.Resources/deploymentScripts` completa.

Cuando se usa una cuenta de almacenamiento existente, el servicio de script crea un recurso compartido de archivos con un nombre único. Consulte [Limpieza de los recursos del script de implementación](#clean-up-deployment-script-resources) para obtener información sobre cómo el servicio de script limpia el recurso compartido de archivos.

## <a name="develop-deployment-scripts"></a>Desarrollo de scripts de implementación

### <a name="handle-non-terminating-errors"></a>Control de errores de no terminación

Puede controlar el modo en que PowerShell responde a los errores de no terminación mediante la variable `$ErrorActionPreference` en el script de implementación. Si la variable no se establece en el script de implementación, el servicio de script utiliza el valor predeterminado **Continuar**.

El servicio de script establece el estado de aprovisionamiento de los recursos en **Error** cuando el script encuentra un error a pesar de la configuración de `$ErrorActionPreference`.

### <a name="use-environment-variables"></a>Uso de variables de entorno

El script de implementación usa estas variables de entorno:

|Variable de entorno|Valor predeterminado|Sistema reservado|
|--------------------|-------------|---------------|
|AZ_SCRIPTS_AZURE_ENVIRONMENT|AzureCloud|No|
|AZ_SCRIPTS_CLEANUP_PREFERENCE|OnExpiration|No|
|AZ_SCRIPTS_OUTPUT_PATH|<AZ_SCRIPTS_PATH_OUTPUT_DIRECTORY>/<AZ_SCRIPTS_PATH_SCRIPT_OUTPUT_FILE_NAME>|Y|
|AZ_SCRIPTS_PATH_INPUT_DIRECTORY|/mnt/azscripts/azscriptinput|Y|
|AZ_SCRIPTS_PATH_OUTPUT_DIRECTORY|/mnt/azscripts/azscriptoutput|Y|
|AZ_SCRIPTS_PATH_USER_SCRIPT_FILE_NAME|Azure PowerShell: userscript.ps1; Azure CLI: userscript.sh|Y|
|AZ_SCRIPTS_PATH_PRIMARY_SCRIPT_URI_FILE_NAME|primaryscripturi.config|Y|
|AZ_SCRIPTS_PATH_SUPPORTING_SCRIPT_URI_FILE_NAME|supportingscripturi.config|Y|
|AZ_SCRIPTS_PATH_SCRIPT_OUTPUT_FILE_NAME|scriptoutputs.json|Y|
|AZ_SCRIPTS_PATH_EXECUTION_RESULTS_FILE_NAME|executionresult.json|Y|
|AZ_SCRIPTS_USER_ASSIGNED_IDENTITY|/subscriptions/|No|

Para más información sobre el uso de `AZ_SCRIPTS_OUTPUT_PATH`, consulte [Trabajo con salidas de los scripts de implementación](#work-with-outputs-from-cli-script).

### <a name="pass-secured-strings-to-deployment-script"></a>Paso de cadenas protegidas al script de implementación

El establecimiento de variables de entorno (EnvironmentVariable) en las instancias de contenedor le permite proporcionar configuración dinámica de la aplicación o el script ejecutados por el contenedor. El script de implementación controla las variables de entorno no protegidas y protegidas de la misma manera que Azure Container Instance. Para más información, consulte [Establecimiento de variables de entorno en instancias de contenedor](../../container-instances/container-instances-environment-variables.md#secure-values). Por ejemplo, consulte [Plantillas de ejemplo](#sample-templates).

El tamaño máximo permitido para las variables de entorno es de 64 KB.

## <a name="monitor-and-troubleshoot-deployment-scripts"></a>Supervisar y solucionar problemas de los scripts de implementación

El servicio de script crea una [cuenta de almacenamiento](../../storage/common/storage-account-overview.md) (a menos que especifique una cuenta de almacenamiento existente) y una [instancia de contenedor](../../container-instances/container-instances-overview.md) para la ejecución de scripts. Si el servicio de script crea estos recursos automáticamente, ambos recursos tienen el sufijo `azscripts` en los nombres de los recursos.

![Nombres de recursos del script de implementación de la plantilla de Resource Manager](./media/deployment-script-template/resource-manager-template-deployment-script-resources.png)

El script de usuario, los resultados de la ejecución y el archivo stdout se almacenan en los recursos compartidos de archivos de la cuenta de almacenamiento. Hay una carpeta llamada `azscripts`. En la carpeta hay otras dos carpetas para los archivos de entrada y salida: `azscriptinput` y `azscriptoutput`.

La carpeta de salida contiene un archivo _executionresult.json_ y el archivo de salida del script. Puede ver el mensaje de error de ejecución del script en _executionresult.json_. El archivo de salida solo se crea cuando el script se ejecuta correctamente. La carpeta de entrada contiene un archivo de script de PowerShell del sistema y los archivos de script de implementación del usuario. Puede reemplazar el archivo de script de implementación de usuario por uno revisado y volver a ejecutar el script de implementación en la instancia de Azure Container.

### <a name="use-the-azure-portal"></a>Uso de Azure Portal

Después de implementar un recurso de script de implementación, el recurso se muestra en el grupo de recursos de Azure Portal. En la captura de pantalla siguiente se muestra la página **Información general** de un recurso de script de implementación:

![Información general del portal de script de implementación de plantilla de Resource Manager](./media/deployment-script-template/resource-manager-deployment-script-portal.png)

En la página de información general se muestra información importante del recurso, como **Estado de aprovisionamiento**, **Cuenta de almacenamiento**, **Instancia de contenedor** y **Registros**.

En el menú de la izquierda, puede ver el contenido del script de implementación, los argumentos pasados al script y la salida. También puede exportar una plantilla para el script de implementación, incluido el script de implementación.

### <a name="use-powershell"></a>Uso de PowerShell

Al usar Azure PowerShell, puede administrar scripts de implementación en el ámbito del grupo de recursos o la suscripción:

- [Get-AzDeploymentScript](/powershell/module/az.resources/get-azdeploymentscript): obtiene o enumera scripts de implementación.
- [Get-AzDeploymentScriptLog](/powershell/module/az.resources/get-azdeploymentscriptlog): obtiene el registro de la ejecución de un script de implementación.
- [Remove-AzDeploymentScript](/powershell/module/az.resources/remove-azdeploymentscript): quita un script de implementación y sus recursos asociados.
- [Save-AzDeploymentScriptLog](/powershell/module/az.resources/save-azdeploymentscriptlog): guarda el registro de la ejecución de un script de implementación en el disco.

La salida `Get-AzDeploymentScript` es parecida a esta:

```output
Name                : runPowerShellInlineWithOutput
Id                  : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput
ResourceGroupName   : myds0618rg
Location            : centralus
SubscriptionId      : 01234567-89AB-CDEF-0123-456789ABCDEF
ProvisioningState   : Succeeded
Identity            : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/mydentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami
ScriptKind          : AzurePowerShell
AzPowerShellVersion : 3.0
StartTime           : 6/18/2020 7:46:45 PM
EndTime             : 6/18/2020 7:49:45 PM
ExpirationDate      : 6/19/2020 7:49:45 PM
CleanupPreference   : OnSuccess
StorageAccountId    : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.Storage/storageAccounts/ftnlvo6rlrvo2azscripts
ContainerInstanceId : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.ContainerInstance/containerGroups/ftnlvo6rlrvo2azscripts
Outputs             :
                      Key                 Value
                      ==================  ==================
                      text                Hello John Dole

RetentionInterval   : P1D
Timeout             : PT1H
```

### <a name="use-azure-cli"></a>Uso de CLI de Azure

Al usar la CLI de Azure, puede administrar scripts de implementación en el ámbito del grupo de recursos o la suscripción:

- [az deployment-scripts delete](/cli/azure/deployment-scripts#az_deployment_scripts_delete): elimina un script de implementación.
- [az deployment-scripts list](/cli/azure/deployment-scripts#az_deployment_scripts_list): muestra todos los scripts de implementación.
- [az deployment-scripts show](/cli/azure/deployment-scripts#az_deployment_scripts_show): recupera un script de implementación.
- [az deployment-scripts show-log](/cli/azure/deployment-scripts#az_deployment_scripts_show_log): muestra registros de scripts de implementación.

La salida del comando de lista es similar a:

```json
[
  {
    "arguments": "-name \\\"John Dole\\\"",
    "azPowerShellVersion": "3.0",
    "cleanupPreference": "OnSuccess",
    "containerSettings": {
      "containerGroupName": null
    },
    "environmentVariables": null,
    "forceUpdateTag": "20200625T025902Z",
    "id": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput",
    "identity": {
      "tenantId": "01234567-89AB-CDEF-0123-456789ABCDEF",
      "type": "userAssigned",
      "userAssignedIdentities": {
        "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myidentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami": {
          "clientId": "01234567-89AB-CDEF-0123-456789ABCDEF",
          "principalId": "01234567-89AB-CDEF-0123-456789ABCDEF"
        }
      }
    },
    "kind": "AzurePowerShell",
    "location": "centralus",
    "name": "runPowerShellInlineWithOutput",
    "outputs": {
      "text": "Hello John Dole"
    },
    "primaryScriptUri": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "myds0624rg",
    "retentionInterval": "1 day, 0:00:00",
    "scriptContent": "\r\n          param([string] $name)\r\n          $output = \"Hello {0}\" -f $name\r\n          Write-Output $output\r\n          $DeploymentScriptOutputs = @{}\r\n          $DeploymentScriptOutputs['text'] = $output\r\n        ",
    "status": {
      "containerInstanceId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.ContainerInstance/containerGroups/64lxews2qfa5uazscripts",
      "endTime": "2020-06-25T03:00:16.796923+00:00",
      "error": null,
      "expirationTime": "2020-06-26T03:00:16.796923+00:00",
      "startTime": "2020-06-25T02:59:07.595140+00:00",
      "storageAccountId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Storage/storageAccounts/64lxews2qfa5uazscripts"
    },
    "storageAccountSettings": null,
    "supportingScriptUris": null,
    "systemData": {
      "createdAt": "2020-06-25T02:59:04.750195+00:00",
      "createdBy": "someone@contoso.com",
      "createdByType": "User",
      "lastModifiedAt": "2020-06-25T02:59:04.750195+00:00",
      "lastModifiedBy": "someone@contoso.com",
      "lastModifiedByType": "User"
    },
    "tags": null,
    "timeout": "1:00:00",
    "type": "Microsoft.Resources/deploymentScripts"
  }
]
```

### <a name="use-rest-api"></a>Uso de la API de REST

Puede obtener la información de implementación de recursos del script de implementación en el nivel de grupo de recursos y en el nivel de suscripción mediante API REST:

```rest
/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/microsoft.resources/deploymentScripts/<DeploymentScriptResourceName>?api-version=2020-10-01
```

```rest
/subscriptions/<SubscriptionID>/providers/microsoft.resources/deploymentScripts?api-version=2020-10-01
```

En el ejemplo siguiente se usa [ARMClient](https://github.com/projectkudu/ARMClient):

```azurepowershell
armclient login
armclient get /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourcegroups/myrg/providers/microsoft.resources/deploymentScripts/myDeployementScript?api-version=2020-10-01
```

La salida es parecida a esta:

```json
{
  "kind": "AzurePowerShell",
  "identity": {
    "type": "userAssigned",
    "tenantId": "01234567-89AB-CDEF-0123-456789ABCDEF",
    "userAssignedIdentities": {
      "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myidentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami": {
        "principalId": "01234567-89AB-CDEF-0123-456789ABCDEF",
        "clientId": "01234567-89AB-CDEF-0123-456789ABCDEF"
      }
    }
  },
  "location": "centralus",
  "systemData": {
    "createdBy": "someone@contoso.com",
    "createdByType": "User",
    "createdAt": "2020-06-25T02:59:04.7501955Z",
    "lastModifiedBy": "someone@contoso.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2020-06-25T02:59:04.7501955Z"
  },
  "properties": {
    "provisioningState": "Succeeded",
    "forceUpdateTag": "20200625T025902Z",
    "azPowerShellVersion": "3.0",
    "scriptContent": "\r\n          param([string] $name)\r\n          $output = \"Hello {0}\" -f $name\r\n          Write-Output $output\r\n          $DeploymentScriptOutputs = @{}\r\n          $DeploymentScriptOutputs['text'] = $output\r\n        ",
    "arguments": "-name \\\"John Dole\\\"",
    "retentionInterval": "P1D",
    "timeout": "PT1H",
    "containerSettings": {},
    "status": {
      "containerInstanceId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.ContainerInstance/containerGroups/64lxews2qfa5uazscripts",
      "storageAccountId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Storage/storageAccounts/64lxews2qfa5uazscripts",
      "startTime": "2020-06-25T02:59:07.5951401Z",
      "endTime": "2020-06-25T03:00:16.7969234Z",
      "expirationTime": "2020-06-26T03:00:16.7969234Z"
    },
    "outputs": {
      "text": "Hello John Dole"
    },
    "cleanupPreference": "OnSuccess"
  },
  "id": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput",
  "type": "Microsoft.Resources/deploymentScripts",
  "name": "runPowerShellInlineWithOutput"
}

```

La siguiente API REST devuelve el registro:

```rest
/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/microsoft.resources/deploymentScripts/<DeploymentScriptResourceName>/logs?api-version=2020-10-01
```

Solo funciona antes de que se eliminen los recursos del script de implementación.

Para ver el recurso deploymentScripts en el portal, seleccione **Mostrar tipos ocultos**:

![Script de implementación de la plantilla de Resource Manager, mostrar tipos ocultos, portal](./media/deployment-script-template/resource-manager-deployment-script-portal-show-hidden-types.png)

## <a name="clean-up-deployment-script-resources"></a>Limpieza de los recursos del script de implementación

Se necesita una cuenta de almacenamiento y una instancia de contenedor para la ejecución de scripts y la solución de problemas. Tiene opciones para especificar una cuenta de almacenamiento existente; de lo contrario, el servicio de script crea automáticamente una cuenta de almacenamiento, junto con una instancia de contenedor. El servicio de script elimina los dos recursos creados automáticamente cuando la ejecución del script de implementación llega a un estado terminal. Los recursos se le facturarán hasta que se eliminen. Para obtener información sobre los precios, consulte los [precios de Container Instances](https://azure.microsoft.com/pricing/details/container-instances/) y los [precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

El ciclo de vida de estos recursos se controla mediante las siguientes propiedades de la plantilla:

- `cleanupPreference`: Preferencia de limpieza cuando la ejecución del script llega a un estado terminal. Los valores admitidos son:

  - **Always**: Elimine los recursos creados automáticamente cuando la ejecución del script llegue a un estado terminal. Si se usa una cuenta de almacenamiento existente, el servicio de script elimina el recurso compartido de archivos creado en la cuenta de almacenamiento. Puesto que el recurso `deploymentScripts` todavía puede existir después de que se limpien los recursos, el servicio de script conserva los resultados de ejecución del script (por ejemplo, los valores stdout, las salidas y los valores devueltos) antes de que se eliminen los recursos.
  - **OnSuccess**: Elimine los recursos creados automáticamente solo cuando la ejecución del script sea correcta. Si se usa una cuenta de almacenamiento existente, el servicio de script quita el recurso compartido de archivos solo cuando la ejecución del script se completa correctamente. Todavía puede acceder a los recursos para encontrar la información de depuración.
  - **OnExpiration**: Elimine los recursos creados automáticamente solo cuando la configuración de `retentionInterval` haya expirado. Si se usa una cuenta de almacenamiento existente, el servicio de script quita el recurso compartido de archivos, pero conserva la cuenta de almacenamiento.

- `retentionInterval`: especifique el intervalo de tiempo que se conservará un recurso de script y tras el que expirará y se eliminará.

> [!NOTE]
> No se recomienda usar la cuenta de almacenamiento ni la instancia de contenedor que genera el servicio de script para otros fines. Los dos recursos podrían quitarse en función del ciclo de vida del script.

La instancia de contenedor y la cuenta de almacenamiento se eliminan según lo establecido en `cleanupPreference`. Sin embargo, si se produce un error en el script y `cleanupPreference` no está establecido en **Siempre**, el proceso de implementación mantiene automáticamente el contenedor en ejecución durante una hora. Puede usar esta hora para solucionar problemas del script. Si desea mantener el contenedor en ejecución después de implementaciones correctas, agregue un paso de suspensión al script. Por ejemplo, agregue [Start-Sleep](/powershell/module/microsoft.powershell.utility/start-sleep) (Iniciar-Suspender) al final del script. Si no agrega el paso de suspensión, el contenedor se establece en un estado terminal y no se puede acceder a él aunque aún no se haya eliminado.

## <a name="run-script-more-than-once"></a>Ejecución del script más de una vez

La ejecución del script de implementación es una operación idempotente. Si no se cambia ninguna de las propiedades del recurso `deploymentScripts` (incluido el script en línea), el script no se ejecutará al volver a implementar la plantilla. El servicio de scripts de implementación compara los nombres de recursos de la plantilla con los recursos existentes en el mismo grupo de recursos. Hay dos opciones si quiere ejecutar el mismo script de implementación varias veces:

- Cambie el nombre del recurso `deploymentScripts`. Por ejemplo, use la función de plantilla [utcNow](./template-functions-date.md#utcnow) como nombre del recurso o como parte del nombre del recurso. Al cambiar el nombre del recurso, se crea un recurso `deploymentScripts`. Esto es conveniente para mantener un historial de ejecución del script.

    > [!NOTE]
    > La función `utcNow` solo puede utilizarse en el valor predeterminado de un parámetro.

- Especifique otro valor en la propiedad de plantilla `forceUpdateTag`. Por ejemplo, use `utcNow` como valor.

> [!NOTE]
> Escriba los scripts de implementación que son idempotentes. Esto garantiza que si se ejecutan de nuevo accidentalmente, no van a provocar cambios en el sistema. Por ejemplo, si el script de implementación se usa para crear un recurso de Azure, compruebe que el recurso no exista antes de crearlo, para que el script se realice correctamente o no se vuelva a crear el recurso.

## <a name="configure-development-environment"></a>Configuración del entorno de desarrollo

Puede usar una imagen de contenedor preconfigurada como entorno de desarrollo del script de implementación. Para más información, vea [Configuración del entorno de desarrollo para scripts de implementación en plantillas](./deployment-script-template-configure-dev.md).

Después de que el script se pruebe correctamente, puede usarlo como script de implementación en sus plantillas.

## <a name="deployment-script-error-codes"></a>Códigos de error del script de implementación

| Código de error | Descripción |
|------------|-------------|
| DeploymentScriptInvalidOperation | La definición de recursos del script de implementación de la plantilla contiene nombres de propiedad no válidos. |
| DeploymentScriptResourceConflict | No se puede eliminar un recurso del script de implementación que se encuentra en estado no terminal cuando la ejecución no ha superado 1 hora. O bien, no puede volver a ejecutar el mismo script de implementación con el mismo identificador de recurso (la misma suscripción, el mismo nombre de grupo de recursos y el mismo nombre de recurso) pero un contenido de cuerpo del script diferente al mismo tiempo. |
| DeploymentScriptOperationFailed | Error interno en la operación del script de implementación. Póngase en contacto con el soporte técnico de Microsoft. |
| DeploymentScriptStorageAccountAccessKeyNotSpecified | No se ha especificado la clave de acceso para la cuenta de almacenamiento existente.|
| DeploymentScriptContainerGroupContainsInvalidContainers | Un grupo de contenedores creado por el servicio de script de implementación se modificó externamente y se agregaron contenedores no válidos. |
| DeploymentScriptContainerGroupInNonterminalState | Dos o más recursos del script de implementación usan el mismo nombre de instancia de contenedor de Azure en el mismo grupo de recursos, y uno de ellos todavía no ha terminado de ejecutarse. |
| DeploymentScriptStorageAccountInvalidKind | La cuenta de almacenamiento existente del tipo BlobBlobStorage o BlobStorage no admite recursos compartidos de archivos y, por tanto, no se puede usar. |
| DeploymentScriptStorageAccountInvalidKindAndSku | La cuenta de almacenamiento existente no admite recursos compartidos de archivos. Para obtener una lista de los tipos de cuenta de almacenamiento admitidos, vea [Uso de la cuenta de almacenamiento existente](#use-existing-storage-account). |
| DeploymentScriptStorageAccountNotFound | La cuenta de almacenamiento no existe, o bien una herramienta o proceso externo la ha eliminado. |
| DeploymentScriptStorageAccountWithServiceEndpointEnabled | La cuenta de almacenamiento especificada tiene un punto de conexión de servicio. No se admite una cuenta de almacenamiento con un punto de conexión de servicio. |
| DeploymentScriptStorageAccountInvalidAccessKey | Se ha especificado una clave de acceso no válida para la cuenta de almacenamiento existente. |
| DeploymentScriptStorageAccountInvalidAccessKeyFormat | Formato no válido de la clave de la cuenta de almacenamiento. Vea [Administración de las claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md). |
| DeploymentScriptExceededMaxAllowedTime | El tiempo de ejecución del script de implementación superó el valor de tiempo de espera especificado en la definición de recursos del script de implementación. |
| DeploymentScriptInvalidOutputs | La salida del script de implementación no es un objeto JSON válido. |
| DeploymentScriptContainerInstancesServiceLoginFailure | La identidad administrada asignada por el usuario no pudo iniciar sesión después de 10 intentos con un intervalo de 1 minuto. |
| DeploymentScriptContainerGroupNotFound | Un grupo de contenedores creado por el servicio del script de implementación se eliminó mediante una herramienta o un proceso externo. |
| DeploymentScriptDownloadFailure | Error al descargar un script auxiliar. Vea [Uso de scripts auxiliares](#use-supporting-scripts).|
| DeploymentScriptError | El script de usuario generó un error. |
| DeploymentScriptBootstrapScriptExecutionFailed | El script de arranque produjo un error. Script de arranque es el script del sistema que organiza la ejecución del script de implementación. |
| DeploymentScriptExecutionFailed | Error desconocido durante la ejecución del script de implementación. |
| DeploymentScriptContainerInstancesServiceUnavailable | Al crear la instancia de contenedor de Azure (ACI), ACI generó un error de servicio no disponible. |
| DeploymentScriptContainerGroupInNonterminalState | Al crear la instancia de contenedor de Azure (ACI), otro script de implementación usa el mismo nombre de ACI en el mismo ámbito (la misma suscripción, el mismo nombre de grupo de recursos y el mismo nombre de recurso). |
| DeploymentScriptContainerGroupNameInvalid | El nombre de la instancia de contenedor de Azure (ACI) especificado no cumple los requisitos de ACI. Vea [Solución de problemas habituales de Azure Container Instances](../../container-instances/container-instances-troubleshooting.md#issues-during-container-group-deployment).|

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a usar scripts de implementación. Para seguir un tutorial sobre scripts de implementación:

> [!div class="nextstepaction"]
> [Tutorial: Uso de scripts de implementación en plantillas de Azure Resource Manager](./template-tutorial-deployment-script.md).

> [!div class="nextstepaction"]
> [Módulo de aprendizaje: Extensión de las plantillas de Resource Manager mediante scripts de implementación](/learn/modules/extend-resource-manager-template-deployment-scripts/)
