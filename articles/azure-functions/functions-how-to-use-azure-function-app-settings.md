---
title: Configuración de la aplicación de funciones en Azure Functions
description: Obtenga información sobre cómo configurar las opciones de la aplicación de funciones en Azure Functions.
ms.assetid: 81eb04f8-9a27-45bb-bf24-9ab6c30d205c
ms.topic: conceptual
ms.date: 04/13/2020
ms.custom: cc996988-fb4f-47, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 6775fdf8d5174600344f3c7177a3130ef63e8f76
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107832687"
---
# <a name="manage-your-function-app"></a>Administración de la aplicación de funciones 

En Azure Functions, una Function App ofrece el contexto de ejecución de funciones individuales. Los comportamientos de Function App se aplican a todas las funciones hospedadas en una Function App determinada. Todas las funciones de una aplicación de funciones deben compartir el mismo [lenguaje](supported-languages.md). 

Las funciones individuales de una aplicación de funciones se implementan y escalan juntas. Todas las funciones de la misma aplicación de funciones comparten recursos (por instancia) a medida que se escala la aplicación de funciones. 

Las cadenas de conexión, las variables de entorno y otras opciones de configuración de la aplicación se definen por separado para cada aplicación de funciones. Los datos que se deben compartir entre las aplicaciones de funciones deben almacenarse externamente en un almacén persistente.

## <a name="get-started-in-the-azure-portal"></a>Introducción a Azure Portal

1. Para comenzar, vaya a [Azure Portal] e inicie sesión en la cuenta de Azure. En la barra de búsqueda en la parte superior del portal, escriba el nombre de la aplicación de funciones y selecciónela en la lista. 

2. En **Configuración** en el panel izquierdo, seleccione **Configuración**.

    :::image type="content" source="./media/functions-how-to-use-azure-function-app-settings/azure-function-app-main.png" alt-text="Información general sobre las aplicaciones de funciones en Azure Portal":::

Puede navegar a todo lo que necesita para administrar Function App desde la página de información general, en particular la **[Configuración de la aplicación](#settings)** y las **[Características de la plataforma](#platform-features)** .

## <a name="work-with-application-settings"></a><a name="settings"></a>Trabajo con la configuración de la aplicación

La configuración de la aplicación se puede administrar no solo tanto desde [Azure Portal](functions-how-to-use-azure-function-app-settings.md?tabs=portal#settings), como desde la [CLI de Azure](functions-how-to-use-azure-function-app-settings.md?tabs=azurecli#settings) y [Azure PowerShell](functions-how-to-use-azure-function-app-settings.md?tabs=powershell#settings). También se puede administrar desde [Visual Studio Code](functions-develop-vs-code.md#application-settings-in-azure) y desde [Visual Studio](functions-develop-vs.md#function-app-settings). 

Esta configuración se almacena cifrada. Para más información, consulte [Seguridad de la configuración de la aplicación](security-concepts.md#application-settings).

# <a name="portal"></a>[Portal](#tab/portal)

Para buscar la configuración de la aplicación, consulte [Introducción a Azure Portal](#get-started-in-the-azure-portal). 

La pestaña **Configuración de la aplicación** mantiene la configuración de la aplicación que la aplicación de funciones usa. Debe seleccionar **Mostrar valores** para ver los valores en el portal. Para agregar una configuración en el portal, seleccione **Nueva configuración de la aplicación** y agregue el nuevo par clave-valor.

![Configuración de Function App en Azure Portal.](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-settings-tab.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

El comando [`az functionapp config appsettings list`](/cli/azure/functionapp/config/appsettings#az_functionapp_config_appsettings_list) devuelve la configuración de la aplicación existente, como en el ejemplo siguiente:

```azurecli-interactive
az functionapp config appsettings list --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME>
```

El comando [`az functionapp config appsettings set`](/cli/azure/functionapp/config/appsettings#az_functionapp_config_appsettings_set) agrega o actualiza una configuración de la aplicación. En el ejemplo siguiente se crea una configuración con una clave denominada `CUSTOM_FUNCTION_APP_SETTING` y un valor de `12345`:


```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings CUSTOM_FUNCTION_APP_SETTING=12345
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

El cmdlet [`Get-AzFunctionAppSetting`](/powershell/module/az.functions/get-azfunctionappsetting) devuelve la configuración de la aplicación existente, como en el ejemplo siguiente: 

```azurepowershell-interactive
Get-AzFunctionAppSetting -Name <FUNCTION_APP_NAME> -ResourceGroupName <RESOURCE_GROUP_NAME>
```

El comando [`Update-AzFunctionAppSetting`](/powershell/module/az.functions/update-azfunctionappsetting) agrega o actualiza una configuración de la aplicación. En el ejemplo siguiente se crea una configuración con una clave denominada `CUSTOM_FUNCTION_APP_SETTING` y un valor de `12345`:

```azurepowershell-interactive
Update-AzFunctionAppSetting -Name <FUNCTION_APP_NAME> -ResourceGroupName <RESOURCE_GROUP_NAME> -AppSetting @{"CUSTOM_FUNCTION_APP_SETTING" = "12345"}
```

---

### <a name="use-application-settings"></a>Uso de la configuración de la aplicación

[!INCLUDE [functions-environment-variables](../../includes/functions-environment-variables.md)]

Al desarrollar una aplicación de funciones localmente, debe mantener copias locales de estos valores en el archivo del proyecto local.settings.json. Para más información, consulte el [archivo de configuración local](functions-run-local.md#local-settings-file).

## <a name="hosting-plan-type"></a>Tipo de plan de hospedaje

Al crear una aplicación de funciones, también se crea un plan de hospedaje en el que se ejecuta la aplicación. Un plan puede tener una o varias aplicaciones de funciones. La funcionalidad, el escalado y el precio de las funciones dependen del tipo de plan. Para más información, consulte las [opciones de hospedaje de Azure Functions](functions-scale.md).

Puede determinar el tipo de plan que usa la aplicación de funciones desde Azure Portal o mediante el uso de las API de la CLI de Azure o Azure PowerShell. 

Los valores siguientes indican el tipo de plan:

| Tipo de plan | Portal | Azure CLI/PowerShell |
| --- | --- | --- |
| [Consumo](consumption-plan.md) | **Consumo** | `Dynamic` |
| [Premium](functions-premium-plan.md) | **ElasticPremium** | `ElasticPremium` |
| [Dedicado (App Service)](dedicated-plan.md) | Varios | Varios |

# <a name="portal"></a>[Portal](#tab/portal)

Para determinar el tipo de plan que usa la aplicación de funciones, consulte **Plan de App Service** en la pestaña **Introducción** de la aplicación de funciones en [Azure Portal](https://portal.azure.com). Para ver el plan de tarifa, seleccione el nombre del **Plan de App Service** y, a continuación, seleccione **Propiedades** en el panel izquierdo.

![Vista del plan de escalado en el portal](./media/functions-scale/function-app-overview-portal.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

Ejecute el siguiente comando de CLI de Azure para obtener el tipo de plan de hospedaje:

```azurecli-interactive
functionApp=<FUNCTION_APP_NAME>
resourceGroup=FunctionMonitoringExamples
appServicePlanId=$(az functionapp show --name $functionApp --resource-group $resourceGroup --query appServicePlanId --output tsv)
az appservice plan list --query "[?id=='$appServicePlanId'].sku.tier" --output tsv

```  

En el ejemplo anterior, reemplace `<RESOURCE_GROUP>` y `<FUNCTION_APP_NAME>` por los nombres del grupo de recursos y de la aplicación de funciones, respectivamente. 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Ejecute el siguiente comando de Azure PowerShell para obtener el tipo de plan de hospedaje:

```azurepowershell-interactive
$FunctionApp = '<FUNCTION_APP_NAME>'
$ResourceGroup = '<RESOURCE_GROUP>'

$PlanID = (Get-AzFunctionApp -ResourceGroupName $ResourceGroup -Name $FunctionApp).AppServicePlan
(Get-AzFunctionAppPlan -Name $PlanID -ResourceGroupName $ResourceGroup).SkuTier
```
En el ejemplo anterior, reemplace `<RESOURCE_GROUP>` y `<FUNCTION_APP_NAME>` por los nombres del grupo de recursos y de la aplicación de funciones, respectivamente. 

---

## <a name="plan-migration"></a>Planear la migración

Puede usar los comandos de la CLI de Azure para migrar una aplicación de funciones de un plan de consumo y un plan Prémium en Windows. Los comandos específicos dependen de la dirección de la migración. Actualmente no se admite la migración directa a un plan dedicado (App Service).

Esta migración no se admite en Linux.

### <a name="consumption-to-premium"></a>De consumo a Prémium

Use el siguiente procedimiento para migrar de un plan de consumo a un plan Premium en Windows:

1. Ejecute el siguiente comando para crear un plan de App Service (Prémium elástico) en la misma región y grupo de recursos en que se encuentra la aplicación de funciones existente.  

    ```azurecli-interactive
    az functionapp plan create --name <NEW_PREMIUM_PLAN_NAME> --resource-group <MY_RESOURCE_GROUP> --location <REGION> --sku EP1
    ```

1. Ejecute el siguiente comando para migrar la aplicación de funciones existente al nuevo plan Prémium.

    ```azurecli-interactive
    az functionapp update --name <MY_APP_NAME> --resource-group <MY_RESOURCE_GROUP> --plan <NEW_PREMIUM_PLAN>
    ```

1. Si ya no necesita el plan de consumo de la aplicación de funciones anterior, elimínelo después de confirmar que ha migrado correctamente al nuevo. Ejecute el siguiente comando para obtener una lista de todos los planes de consumo del grupo de recursos.

    ```azurecli-interactive
    az functionapp plan list --resource-group <MY_RESOURCE_GROUP> --query "[?sku.family=='Y'].{PlanName:name,Sites:numberOfSites}" -o table
    ```

    Puede eliminar el plan con seguridad sin ningún sitio, que es desde el que migró.

1. Ejecute el siguiente comando para eliminar el plan de consumo del que ha migrado.

    ```azurecli-interactive
    az functionapp plan delete --name <CONSUMPTION_PLAN_NAME> --resource-group <MY_RESOURCE_GROUP>
    ```

### <a name="premium-to-consumption"></a>De Prémium a consumo

Use el siguiente procedimiento para migrar de un plan Prémium a un plan de consumo en Windows:

1. Ejecute el siguiente comando para crear una aplicación de funciones (consumo) en la misma región y grupo de recursos en que se encuentra la aplicación de funciones existente. Este comando también crea un plan de consumo en el que se ejecuta la aplicación de funciones.

    ```azurecli-interactive
    az functionapp create --resource-group <MY_RESOURCE_GROUP> --name <NEW_CONSUMPTION_APP_NAME> --consumption-plan-location <REGION> --runtime dotnet --functions-version 3 --storage-account <STORAGE_NAME>
    ```

1. Ejecute el siguiente comando para migrar la aplicación de funciones existente al nuevo plan de consumo.

    ```azurecli-interactive
    az functionapp update --name <MY_APP_NAME> --resource-group <MY_RESOURCE_GROUP> --plan <NEW_CONSUMPTION_PLAN>
    ```

1. Elimine la aplicación de funciones que creó en el paso 1, ya que solo necesita el plan que se creó para ejecutar la aplicación de funciones existente.

    ```azurecli-interactive
    az functionapp delete --name <NEW_CONSUMPTION_APP_NAME> --resource-group <MY_RESOURCE_GROUP>
    ```

1. Si ya no necesita el plan Prémium de la aplicación de funciones anterior, elimínelo después de confirmar que ha migrado correctamente al nuevo. Tenga en cuenta que si no se elimina el plan, se le seguirá cobrando por él. Ejecute el siguiente comando para obtener una lista de todos los planes Prémium del grupo de recursos.

    ```azurecli-interactive
    az functionapp plan list --resource-group <MY_RESOURCE_GROUP> --query "[?sku.family=='EP'].{PlanName:name,Sites:numberOfSites}" -o table
    ```

1. Ejecute el siguiente comando para eliminar el plan Prémium del que ha migrado.

    ```azurecli-interactive
    az functionapp plan delete --name <PREMIUM_PLAN> --resource-group <MY_RESOURCE_GROUP>
    ```

## <a name="platform-features"></a>Características de la plataforma

Las aplicaciones de funciones se ejecutan en la plataforma de Azure App Service, donde también se realiza su mantenimiento. Por tanto, Function App tiene acceso a la mayoría de las características de la plataforma de hospedaje web principal de Azure. En el panel de la izquierda puede acceder a muchas características de la plataforma de App Service que puede usar en las aplicaciones de funciones. 

> [!NOTE]
> No todas las características de App Service están disponibles cuando una Function App se ejecuta con el plan de hospedaje de consumo.

El resto de este artículo se centra en las siguientes características de App Service en Azure Portal que resultan útiles para Functions:

+ [Editor de App Service](#editor)
+ [Console](#console)
+ [Herramientas avanzadas (Kudu)](#kudu)
+ [Opciones de implementación](#deployment)
+ [CORS](#cors)
+ [Autenticación](#auth)

Para más información sobre cómo trabajar con la configuración de App Service, vea [Configuración de Azure App Service](../app-service/configure-common.md).

### <a name="app-service-editor"></a><a name="editor"></a>Editor de App Service

![Editor de App Service](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-appservice-editor.png)

El Editor de App Service es un editor en portal avanzado que puede usar para modificar archivos de configuración JSON y archivos de código similares. Al seleccionar esta opción se inicia una pestaña de explorador independiente con un editor básico. Esto le permite realizar la integración con el repositorio Git, ejecutar y depurar código y modificar la configuración de Function App. Este editor proporciona un entorno de desarrollo mejorado para las funciones en comparación con el editor de funciones integrado.  

Se recomienda que considere la posibilidad de desarrollar las funciones en el equipo local. Al desarrollar localmente y publicar en Azure, los archivos del proyecto son de solo lectura en el portal. Para más información, consulte [Codificación y comprobación de las funciones de Azure Functions en un entorno local](functions-develop-local.md).

### <a name="console"></a><a name="console"></a>Consola

![Consola de Function App](./media/functions-how-to-use-azure-function-app-settings/configure-function-console.png)

La consola del portal es una herramienta ideal para desarrolladores si prefiere interactuar con Function App desde la línea de comandos. Los comandos comunes incluyen creación de archivos y directorios y navegación por los mismos, así como la ejecución de archivos y scripts por lotes. 

Al desarrollar localmente, se recomienda usar [Azure Functions Core Tools](functions-run-local.md) y la [CLI de Azure].

### <a name="advanced-tools-kudu"></a><a name="kudu"></a>Herramientas avanzadas (Kudu)

![Configurar Kudu](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-kudu.png)

Las herramientas avanzadas para App Service (también conocidas como Kudu) proporcionan acceso a las características administrativas avanzadas de la Function App. Con Kudu, puede administrar la información del sistema, la configuración de las aplicaciones, las variables del entorno, las extensiones del sitio, los encabezados HTTP y las variables del servidor. También puede iniciar **Kudu** si examina el punto de conexión de SCM de la Function App, como `https://<myfunctionapp>.scm.azurewebsites.net/`. 


### <a name="deployment-center"></a><a name="deployment"></a>Centro de implementación

Cuando se usa una solución de control de código fuente para desarrollar y mantener el código de las funciones, el Centro de implementación le permite compilar e implementar desde el control de código fuente. El proyecto se compila e implementa en Azure cuando se realizan actualizaciones. Para más información, vea [Tecnologías de implementación en Azure Functions](functions-deployment-technologies.md).

### <a name="cross-origin-resource-sharing"></a><a name="cors"></a>Uso compartido de recursos entre orígenes

Para evitar la ejecución de código malintencionado en el cliente, los exploradores modernos bloquean las solicitudes de las aplicaciones web en los recursos que se ejecutan en un dominio independiente. El [Intercambio de Recursos de Origen Cruzado (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS) permite a un encabezado `Access-Control-Allow-Origin` declarar qué orígenes pueden llamar a los puntos de conexión en la aplicación de funciones.

#### <a name="portal"></a>Portal

Al configurar la lista de **orígenes permitidos** para la aplicación de funciones, el encabezado `Access-Control-Allow-Origin` se agrega automáticamente a todas las respuestas de los puntos de conexión HTTP en la aplicación de funciones. 

![Configuración de la lista de CORS de la aplicación de funciones](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-cors.png)

Cuando se usa el carácter comodín (`*`), se omiten todos los demás dominios. 

Use el comando [`az functionapp cors add`](/cli/azure/functionapp/cors#az_functionapp_cors_add) para agregar un dominio a la lista de orígenes permitidos. En el ejemplo siguiente se agrega el dominio contoso.com:

```azurecli-interactive
az functionapp cors add --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--allowed-origins https://contoso.com
```

Use el comando [`az functionapp cors show`](/cli/azure/functionapp/cors#az_functionapp_cors_show) para enumerar los orígenes permitidos actuales.

### <a name="authentication"></a><a name="auth"></a>Autenticación

![Configuración de la autenticación de una Function App](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-authentication.png)

Si las funciones usan un desencadenador HTTP, puede requerir que las llamadas se autentiquen primero. App Service admite la autenticación de Azure Active Directory y el inicio de sesión en proveedores locales, como Facebook, Microsoft y Twitter. Para más información sobre cómo configurar los proveedores de autenticación específicos, consulte [Autenticación y autorización en Azure App Service](../app-service/overview-authentication-authorization.md). 


## <a name="next-steps"></a>Pasos siguientes

+ [Configuración de Azure App Service](../app-service/configure-common.md)
+ [Implementación continua para Azure Functions](functions-continuous-deployment.md)

[CLI de Azure]: /cli/azure/
[Azure Portal]: https://portal.azure.com
