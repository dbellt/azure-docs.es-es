---
title: Creación de parámetros para flujos de trabajo en Azure Logic Apps de un solo inquilino
description: Defina parámetros para los valores que difieren en los flujos de trabajo entre entornos de implementación en entornos de Azure Logic Apps de un solo inquilino.
services: logic-apps
ms.suite: integration
ms.reviewer: azla
ms.topic: how-to
ms.date: 06/08/2021
ms.openlocfilehash: 7de89605f86e47b3062ec07160288ab14584f42e
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747426"
---
# <a name="create-parameters-for-values-that-change-in-workflows-across-environments-for-single-tenant-azure-logic-apps"></a>Creación de parámetros para los valores que cambian en los flujos de trabajo entre entornos de Azure Logic Apps de un solo inquilino

En Azure Logic Apps, puede usar parámetros para abstraer los valores que pueden cambiar entre entornos. Al definir los parámetros que se usarán en los flujos de trabajo, primero puede centrarse en el diseño de los flujos de trabajo y, a continuación, insertar las variables específicas del entorno más adelante.

En Azure Logic Apps *multiinquilino*, puede crear parámetros y hacer referencia a ellos en el diseñador de flujos de trabajo y, a continuación, establecer las variables en los archivos de parámetros y plantilla de Azure Resource Manager (ARM). Los parámetros se definen y establecen en la implementación. Por lo tanto, incluso si solo necesita cambiar una variable, tiene que volver a implementar la plantilla de ARM de la aplicación lógica.

En Azure Logic Apps de *un solo inquilino*, puede trabajar con variables de entorno tanto en tiempo de ejecución como en tiempo de implementación mediante los parámetros y la configuración de la aplicación. En este artículo, se muestra cómo editar variables de entorno, llamarlas y hacer referencia a ellas con la nueva experiencia de parámetros de un solo inquilino.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- Un [flujo de trabajo de aplicación lógica en Azure Logic Apps de un solo inquilino](single-tenant-overview-compare.md).

  Si no tiene una aplicación lógica, [créela (Estándar) en Azure Portal](create-single-tenant-workflows-azure-portal.md) o en [Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md).

## <a name="parameters-versus-app-settings"></a>Parámetros frente a configuración de la aplicación

Antes de decidir dónde almacenar las variables de entorno, revise la siguiente información.

Si ya usa Azure Functions o Azure Web Apps, es posible que esté familiarizado con la configuración de la aplicación. En Azure Logic Apps, la configuración de la aplicación se integra con Azure Key Vault. Puede [hacer referencia directamente a cadenas seguras](../app-service/app-service-key-vault-references.md), como claves y cadenas de conexión. De forma similar a las plantillas de ARM, donde puede definir variables de entorno en el momento de la implementación, puede definir la configuración de la aplicación dentro de la [definición de flujo de trabajo de la aplicación lógica](/azure/templates/microsoft.logic/workflows). A continuación, puede capturar valores de infraestructura generados dinámicamente, como puntos de conexión, cadenas de almacenamiento, etc. Sin embargo, la configuración de la aplicación tiene limitaciones de tamaño y no se puede hacer referencia a ella desde determinadas áreas de Azure Logic Apps.

Si está familiarizado con los flujos de trabajo en Azure Logic Apps multiinquilino, es posible que también esté familiarizado con los parámetros. Puede usar parámetros en una gama más amplia de casos de uso que la configuración de la aplicación, como admitir objetos complejos y valores de tamaños grandes. Si usa Visual Studio Code como herramienta de desarrollo local, también puede hacer referencia a los parámetros de los archivos **workflow.json** y **connections.json** del proyecto de la aplicación lógica. Si desea usar ambas opciones en la solución, también puede hacer referencia a la configuración de la aplicación mediante parámetros.

> [!NOTE]
> Para el desarrollo, si parametriza el archivo **connections.json**, la experiencia del diseñador se restringe, tanto localmente como en Azure Portal. Si necesita usar el diseñador para desarrollo, use un archivo **connections.json** sin parámetros. A continuación, en las canalizaciones de implementación, reemplace por el archivo con parámetros. El tiempo de ejecución sigue funcionando con parametrización. Las mejoras del diseñador están en desarrollo.

Considere la recomendación de usar parámetros como mecanismo predeterminado para la parametrización. De este modo, cuando necesite almacenar claves o cadenas seguras, puede seguir la recomendación para hacer referencia a la configuración de la aplicación a partir de los parámetros.

## <a name="what-is-parameterization"></a>¿Qué es la parametrización?

Si usa Visual Studio Code, en el proyecto de la aplicación lógica, puede definir parámetros en el archivo **parameters.json**. Puede hacer referencia a cualquier parámetro del archivo **parameters.json** desde cualquier flujo de trabajo u objeto de conexión de la aplicación lógica. La parametrización de las entradas de flujo de trabajo en Azure Logic Apps de un solo inquilino funciona de forma similar a Azure Logic Apps multiinquilino.

Para hacer referencia a parámetros en las entradas de desencadenador o acción, use la expresión `@parameters('<parameter-name>')`.

> [!IMPORTANT]
> Asegúrese de incluir también todos los parámetros a los que haga referencia en el archivo **parameters.json**.

En Azure Logic Apps de *un solo inquilino*, puede parametrizar distintas partes del archivo **connections.json**. A continuación, puede comprobar el archivo **connections.json** en el control de código fuente y, después, administrar todas las conexiones mediante el archivo **parameters.json**. Para parametrizar el archivo **connections.json**, reemplace los valores de los literales, como `ConnectionRuntimeUrl`, por una expresión `parameters()` única, como por ejemplo, `@parameters('api-runtimeUrl')`.

También puede parametrizar objetos complejos, como el objeto JSON `authentication`. Por ejemplo, reemplace el valor del objeto `authentication` por una cadena que contiene una expresión de parámetros única, como `@parameters('api-auth')`. 

> [!NOTE]
> Los únicos tipos de expresión válidos en el archivo **connections.json** son `@parameters` y `@appsetting`.

## <a name="define-parameters"></a>Definición de parámetros

En los flujos de trabajo basados en un solo inquilino, debe colocar todos los valores de parámetros en un archivo JSON de nivel raíz denominado **parameters.json**. Este archivo incluye un objeto que contiene pares clave-valor. Las claves son los nombres de cada parámetro, y los valores son las estructuras de cada parámetro. Cada estructura debe incluir una declaración `type` y `value`.

> [!NOTE]
> El único tipo de expresión válido en el archivo **parameters.json** es `@appsetting`.

En el ejemplo siguiente se muestra un archivo de parámetros básicos:

```json
{ 
    "responseString": { 
        "type": "string", 
        "value": "hello" 
    }, 
    "functionAuth": { 
        "type": "object", 
        "value": { 
            "type": "QueryString", 
            "name": "Code", 
            "value": "@appsetting('<AzureFunctionsOperation-FunctionAppKey')" 
        } 
    } 
} 
```

Normalmente, debe administrar varias versiones de archivos de parámetros. Es posible que tenga valores de destino para distintos entornos de implementación, como desarrollo, pruebas y producción. La administración de estos archivos de parámetros suele funcionar como la administración de archivos de parámetros de plantilla de ARM. Al realizar la implementación en un entorno específico, se promueve el archivo de parámetros correspondiente, por lo general a través de una canalización para DevOps.

Para reemplazar los archivos de parámetros de forma dinámica mediante la CLI de Azure, ejecute el siguiente comando:

```azurecli
az functionapp deploy --resource-group MyResourceGroup --name MyLogicApp --src-path C:\parameters.json --type static --target-path parameters.json
```

Si tiene un proyecto de Logic Apps basado en NuGet, tendrá que actualizar el archivo de proyecto ( **&lt;nombre de aplicación lógica&gt;.csproj**) para incluir el archivo de parámetros en la salida de compilación, por ejemplo:
  
```csproj
<ItemGroup>
  <None Update="parameters.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </None>
</ItemGroup>
```

> [!NOTE]
> Actualmente, la capacidad de reemplazar de forma dinámica los archivos de parámetros aún no está disponible en Azure Portal ni en el diseñador de flujos de trabajo.

Para obtener más información sobre cómo configurar las aplicaciones lógicas para implementaciones de DevOps, revise la siguiente documentación:

- [Introducción a la implementación de DevOps para aplicaciones lógicas basadas en un solo inquilino](devops-deployment-single-tenant-azure-logic-apps.md)
- [Configuración de la implementación de DevOps para aplicaciones lógicas basadas en un solo inquilino](set-up-devops-deployment-single-tenant-azure-logic-apps.md)

## <a name="manage-app-settings"></a>Administración de la configuración de la aplicación

En Azure Logic Apps de un solo inquilino, la configuración de la aplicación contiene opciones de configuración globales para *todos los flujos de trabajo* de la misma aplicación lógica. Al ejecutar flujos de trabajo localmente en Visual Studio Code, se puede acceder a esta configuración como variables de entorno local en el archivo **local.settings.json**. A continuación, puede hacer referencia a esta configuración de la aplicación en los parámetros.

Para agregar, actualizar o eliminar la configuración de la aplicación, seleccione y revise las secciones siguientes para Visual Studio Code, Azure Portal, la CLI de Azure o la plantilla de ARM (Bicep).

### <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

Para revisar la configuración de la aplicación de la aplicación lógica en Azure Portal, siga estos pasos:

1. En el cuadro de búsqueda de [Azure Portal](https://portal.azure.com/), busque y abra la aplicación lógica basada en un solo inquilino.
1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración**.
1. En la página **Configuración**, en la pestaña **Configuración de la aplicación**, revise la configuración de la aplicación de la aplicación lógica.
1. Para ver todos los valores, seleccione **Mostrar valores**. O bien, para ver un valor único, seleccione ese valor.

Para agregar una configuración nueva, siga estos pasos:

1. En la pestaña **Configuración de la aplicación**, en **Configuración de la aplicación**, seleccione **Nueva configuración la de aplicación**.
1. En **Nombre**, escriba la *clave* o el nombre de la nueva configuración.
1. En **Valor**, escriba el valor de la nueva configuración.
1. Cuando esté listo para crear el nuevo par *clave-valor*, seleccione **Aceptar**.

:::image type="content" source="./media/parameterize-workflow-app/portal-app-settings-values.png" alt-text="Captura de pantalla que muestra Azure Portal y el panel de configuración con los valores y la configuración de la aplicación de una aplicación lógica basada en un solo inquilino" lightbox="./media/parameterize-workflow-app/portal-app-settings-values.png":::.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para revisar la configuración actual de la aplicación mediante la CLI de Azure, ejecute el comando `az logicapp config appsettings list`. Asegúrese de que el comando incluye los parámetros `--name -n` y `--resource-group -g`, por ejemplo:

```azurecli
az logicapp config appsettings list --name MyLogicApp --resource-group MyResourceGroup
```

Para agregar o actualizar una configuración de la aplicación mediante la CLI de Azure, ejecute el comando `az logicapp config appsettings set`. Asegúrese de que el comando incluye los parámetros `--name n` y `--resource-group -g`. Por ejemplo, el comando siguiente crea una configuración con una clave denominada `CUSTOM_LOGIC_APP_SETTING` con un valor de `12345`:

```azurecli
az logicapp config appsettings set --name MyLogicApp --resource-group MyResourceGroup --settings CUSTOM_LOGIC_APP_SETTING=12345 
```

### <a name="resource-manager-or-bicep-template"></a>[Plantilla de Resource Manager o de Bicep](#tab/azure-resource-manager)

Para revisar y definir la configuración de la aplicación en una plantilla de ARM o en una plantilla de Bicep, busque la definición de recursos de la aplicación lógica y actualice el objeto JSON `appSettings`. Para obtener la definición de recursos completa, consulte la [referencia sobre la plantilla de ARM](/azure/templates/microsoft.web/sites).

En este ejemplo se muestra la configuración de archivos para las plantillas de ARM o las plantillas de Bicep:

```json
"appSettings": [
    {
        "name": "string",
        "value": "string"
    },
    {
        "name": "string",
        "value": "string"
    },
    <...>
], 
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Comparación de las opciones de un solo inquilino, multiinquilino y entorno del servicio de integración para Azure Logic Apps](single-tenant-overview-compare.md)
