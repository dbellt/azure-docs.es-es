---
title: Cómo seleccionar un destino para versiones en tiempo de ejecución de Azure Functions
description: Azure Functions admite varias versiones del runtime. Obtenga información sobre cómo especificar la versión en tiempo de ejecución de una aplicación de función hospedada en Azure.
ms.topic: conceptual
ms.date: 07/22/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ca107ec2f0ce04bf7b1eae3a98087217c267d33d
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107830923"
---
# <a name="how-to-target-azure-functions-runtime-versions"></a>Cómo seleccionar un destino para versiones en tiempo de ejecución de Azure Functions

Una aplicación de función se ejecuta en una versión específica del runtime de Azure Functions. Hay tres versiones principales: [3.x, 2.x y 1.x](functions-versions.md). De forma predeterminada, las aplicaciones de función se crean en la versión 3.x del entorno de ejecución. En este artículo se explica cómo configurar una aplicación de función en Azure para que se ejecute en la versión que elija. Para obtener información sobre cómo configurar un entorno de desarrollo local para una versión específica, consulte [Codificación y comprobación de las funciones de Azure Functions en un entorno local](functions-run-local.md).

La manera de dirigirse manualmente a una versión específica depende de si se ejecuta Windows o Linux.

## <a name="automatic-and-manual-version-updates"></a>Actualizaciones de versiones automáticas y manuales

_Esta sección no se aplica al ejecutar la aplicación de funciones [en Linux](#manual-version-updates-on-linux)._

Azure Functions permite dirigir una versión específica del entorno de ejecución de Windows mediante el uso de la configuración de aplicación `FUNCTIONS_EXTENSION_VERSION` en la aplicación de funciones. La aplicación de función se mantiene en la versión principal especificada hasta que elija explícitamente moverla a una nueva versión. Si especifica solo la versión principal, la aplicación de funciones se actualiza automáticamente a las nuevas versiones secundarias del entorno de ejecución cuando están disponibles. Las nuevas versiones secundarias no deberían introducir cambios importantes. 

Si especifica una versión secundaria (por ejemplo, "2.0.12345"), la aplicación de función se mantiene en esa versión específica hasta que la modifique explícitamente. Las versiones secundarias anteriores se quitan del entorno de producción de manera periódica. Si se quita la versión secundaria, la aplicación de funciones vuelve a ejecutarse con la versión más reciente en lugar de la versión establecida en `FUNCTIONS_EXTENSION_VERSION`. Por este motivo, debe resolver rápidamente cualquier problema con la aplicación de funciones que requiera una versión secundaria específica. Luego, puede volver a dirigirse a la versión principal. Las eliminaciones de versiones secundarias se anuncian en los [anuncios de App Service](https://github.com/Azure/app-service-announcements/issues).

> [!NOTE]
> Si ancla una versión principal específica de Azure Functions y, a continuación, intenta publicar en Azure mediante Visual Studio, aparecerá un cuadro de diálogo que le solicitará que actualice a la versión más reciente o que cancele la publicación. Para evitarlo, agregue la propiedad `<DisableFunctionExtensionVersionUpdate>true</DisableFunctionExtensionVersionUpdate>` en su archivo `.csproj`.

Cuando una nueva versión está disponible públicamente, un aviso en el portal le da la oportunidad de pasar a esa versión. Después de pasar a una nueva versión, siempre puede usar la configuración de la aplicación `FUNCTIONS_EXTENSION_VERSION` para volver a una versión anterior.

En la tabla siguiente se muestran los valores de `FUNCTIONS_EXTENSION_VERSION` de cada versión principal, para habilitar las actualizaciones automáticas:

| Versión principal | Valor de `FUNCTIONS_EXTENSION_VERSION` |
| ------------- | ----------------------------------- |
| 3.x  | `~3` |
| 2.x  | `~2` |
| 1.x  | `~1` |

Un cambio en la versión del runtime hace que la aplicación de función se reinicie.

>[!NOTE]
>Las aplicaciones de funciones .NET ancladas a `~2.0` no reciben la actualización automática a .Net Core 3.1. Para más información, consulte [Consideraciones sobre Functions v2.x](functions-dotnet-class-library.md#functions-v2x-considerations).  

## <a name="view-and-update-the-current-runtime-version"></a>Visualización y actualización de la versión actual del entorno de ejecución

_Esta sección no se aplica al ejecutar la aplicación de funciones [en Linux](#manual-version-updates-on-linux)._

Puede cambiar la versión del entorno en tiempo de ejecución que la aplicación de función usa. Debido a la posibilidad de cambios importantes, solo puede cambiar la versión del entorno de ejecución antes de haber creado cualquier función en la aplicación de funciones. 

> [!IMPORTANT]
> Si bien la versión del entorno de ejecución viene determinada por la opción `FUNCTIONS_EXTENSION_VERSION`, este cambio se debe hacer únicamente en Azure Portal y no cambiar directamente la opción. Esto se debe a que el portal valida los cambios y hace cambios relacionados adicionales según sea necesario.

# <a name="portal"></a>[Portal](#tab/portal)

[!INCLUDE [Set the runtime version in the portal](../../includes/functions-view-update-version-portal.md)]

> [!NOTE]
> Azure Portal no permite cambiar la versión del entorno en tiempo de ejecución para una aplicación de función que ya contiene funciones.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

`FUNCTIONS_EXTENSION_VERSION` también se puede visualizar y establecer desde la CLI de Azure.  

Mediante la CLI de Azure, puede ver la versión actual del entorno de ejecución con el comando [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings).

```azurecli-interactive
az functionapp config appsettings list --name <function_app> \
--resource-group <my_resource_group>
```

En este código, reemplace `<function_app>` por el nombre de la aplicación de función. Reemplace también `<my_resource_group>` por el nombre del grupo de recursos para la aplicación de función. 

Puede ver el elemento `FUNCTIONS_EXTENSION_VERSION` en la salida siguiente, que se ha truncado para mayor claridad:

```output
[
  {
    "name": "FUNCTIONS_EXTENSION_VERSION",
    "slotSetting": false,
    "value": "~2"
  },
  {
    "name": "FUNCTIONS_WORKER_RUNTIME",
    "slotSetting": false,
    "value": "dotnet"
  },
  
  ...
  
  {
    "name": "WEBSITE_NODE_DEFAULT_VERSION",
    "slotSetting": false,
    "value": "8.11.1"
  }
]
```

Puede actualizar el valor de `FUNCTIONS_EXTENSION_VERSION` en la aplicación de función con el comando [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings).

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP> \
--resource-group <RESOURCE_GROUP> \
--settings FUNCTIONS_EXTENSION_VERSION=<VERSION>
```

Reemplace `<FUNCTION_APP>` por el nombre de la aplicación de función. Reemplace también `<RESOURCE_GROUP>` por el nombre del grupo de recursos para la aplicación de función. Además, reemplace `<VERSION>` por una versión específica: `~3`, `~2` o `~1`.

Elija **Probar** en el ejemplo de código anterior para ejecutar el comando en [Azure Cloud Shell](../cloud-shell/overview.md). También puede usar la [CLI de Azure localmente](/cli/azure/install-azure-cli) para ejecutar este comando. Cuando se use localmente, primero debe ejecutar [az login](/cli/azure/reference-index#az_login) para iniciar sesión.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el runtime de Azure Functions, use el siguiente cmdlet: 

```powershell
Get-AzFunctionAppSetting -Name "<FUNCTION_APP>" -ResourceGroupName "<RESOURCE_GROUP>"
```

Reemplace `<FUNCTION_APP>` por el nombre de la aplicación de funciones y `<RESOURCE_GROUP>`. El valor actual de la configuración `FUNCTIONS_EXTENSION_VERSION` se devuelve en la tabla hash.

Ejecute el siguiente script para cambiar el runtime de Functions:

```powershell
Update-AzFunctionAppSetting -Name "<FUNCTION_APP>" -ResourceGroupName "<RESOURCE_GROUP>" -AppSetting @{"FUNCTIONS_EXTENSION_VERSION" = "<VERSION>"} -Force
```

Como antes, reemplace `<FUNCTION_APP>` por el nombre de su aplicación de funciones y `<RESOURCE_GROUP>` por el nombre del grupo de recursos. Además, reemplace `<VERSION>` por la versión específica o la versión principal, como `~2` o `~3`. Puede comprobar el valor actualizado de la configuración `FUNCTIONS_EXTENSION_VERSION` en la tabla hash devuelta. 

---

La aplicación de funciones se reinicia después de realizar el cambio en la configuración de la aplicación.

## <a name="manual-version-updates-on-linux"></a>Actualizaciones manuales de versiones en Linux

Para anclar una aplicación de funciones de Linux a una versión de host específica, especifique la dirección URL de la imagen en el campo "LinuxFxVersion" en la configuración del sitio. Por ejemplo, si quiere anclar una aplicación de funciones de 10 nodos, digamos la versión del host 3.0.13142:

Para el **servicio de aplicaciones de Linux o las aplicaciones Elastic Premium**, establezca `LinuxFxVersion` en `DOCKER|mcr.microsoft.com/azure-functions/node:3.0.13142-node10-appservice`.

Para las **aplicaciones de consumo de Linux**, establezca `LinuxFxVersion` en `DOCKER|mcr.microsoft.com/azure-functions/mesh:3.0.13142-node10`.

# <a name="portal"></a>[Portal](#tab/portal)

La visualización y modificación de los valores de configuración del sitio en las aplicaciones de funciones no se admiten en Azure Portal. Use mejor la CLI de Azure.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azurecli)

Puede ver y establecer el valor de `LinuxFxVersion` con la CLI de Azure.  

Para ver la versión actual del entorno de ejecución, use el comando [az functionapp config show](/cli/azure/functionapp/config).

```azurecli-interactive
az functionapp config show --name <function_app> \
--resource-group <my_resource_group> --query 'linuxFxVersion' -o tsv
```

En este código, reemplace `<function_app>` por el nombre de la aplicación de función. Reemplace también `<my_resource_group>` por el nombre del grupo de recursos para la aplicación de función. Se devuelve el valor actual de `linuxFxVersion`.

Para actualizar el valor de `linuxFxVersion` en la aplicación de funciones, use el comando [az functionapp config set](/cli/azure/functionapp/config).

```azurecli-interactive
az functionapp config set --name <FUNCTION_APP> \
--resource-group <RESOURCE_GROUP> \
--linux-fx-version <LINUX_FX_VERSION>
```

Reemplace `<FUNCTION_APP>` por el nombre de la aplicación de función. Reemplace también `<RESOURCE_GROUP>` por el nombre del grupo de recursos para la aplicación de función. Además, reemplace `<LINUX_FX_VERSION>` por el valor de una imagen específica, tal y como se ha descrito anteriormente.

Este comando se puede ejecutar desde [Azure Cloud Shell](../cloud-shell/overview.md), para lo que es preciso hacer clic en **Pruébelo** en el código de ejemplo anterior. También puede usar la [CLI de Azure localmente](/cli/azure/install-azure-cli) para ejecutar este comando después de ejecutar [az login](/cli/azure/reference-index#az_login) para iniciar sesión.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Por el momento, no se puede usar Azure PowerShell para establecer `linuxFxVersion`. Use mejor la CLI de Azure.

---

La aplicación de funciones se reinicia después de realizar el cambio en la configuración del sitio.

> [!NOTE]
> En el caso de las aplicaciones que se ejecutan en un plan de consumo, el establecimiento de `LinuxFxVersion` en una imagen específica puede aumentar las horas de inicio en frío. Esto se debe a que el anclaje a una imagen específica evita que las funciones usen algunas optimizaciones de inicio en frío. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Selección del destino de la versión 2.0 del runtime en el entorno de desarrollo local](functions-run-local.md)

> [!div class="nextstepaction"]
> [Vea las notas de versión para las versiones del runtime](https://github.com/Azure/azure-webjobs-sdk-script/releases)
