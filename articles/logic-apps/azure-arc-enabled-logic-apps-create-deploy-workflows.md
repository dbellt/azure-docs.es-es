---
title: Creación e implementación de flujos de trabajo con una instancia de Logic Apps habilitada para Azure Arc
description: Cree e implemente flujos de trabajo de aplicaciones lógicas basadas en un solo inquilino que se ejecutan en cualquier lugar donde se pueda ejecutar Kubernetes.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, ladolan, reylons, archidda, sopai, azla
ms.topic: how-to
ms.date: 05/25/2021
ms.openlocfilehash: 2eabd6462edd609d70fc302ce2d0d64cb99dbdc3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475368"
---
# <a name="create-and-deploy-single-tenant-based-logic-app-workflows-with-azure-arc-enabled-logic-apps-preview"></a>Creación e implementación de flujos de trabajo de aplicaciones lógicas basadas en un solo inquilino con instancias de Logic Apps habilitadas para Azure Arc (versión preliminar)

> [!NOTE]
> Esta funcionalidad está en versión preliminar y está sujeta a las [Condiciones de uso complementarias para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Con las instancias de Logic Apps habilitadas para Azure Arc y Azure Portal, puede crear e implementar flujos de trabajo de aplicaciones lógicas basadas en un solo inquilino en una infraestructura de Kubernetes que puede operar y administrar. Las aplicaciones lógicas se ejecutan en una *ubicación personalizada* que se asigna a un clúster de Kubernetes habilitado para Azure Arc donde haya instalado y habilitado el conjunto de extensiones de plataforma de Azure App Service.

Por ejemplo, este clúster puede ser una instancia de Azure Kubernetes Service, una instancia de Kubernetes sin sistema operativo u otra configuración. El conjunto de extensiones permite ejecutar servicios de plataforma como Azure Logic Apps, Azure App Service y Azure Functions en el clúster de Kubernetes. 

Para más información, revise la siguiente documentación:

- [¿Qué es Logic Apps habilitado para Azure Arc?](azure-arc-enabled-logic-apps-overview.md)
- [Entorno de servicio de integración de un solo inquilino o de tipo multiinquilino](../logic-apps/single-tenant-overview-compare.md)
- [Introducción a Azure Arc](../azure-arc/overview.md)
- [Información general de Azure Kubernetes Service](../aks/intro-kubernetes.md)
- [¿Qué es Kubernetes habilitado para Azure Arc?](../azure-arc/kubernetes/overview.md)
- [Ubicaciones personalizadas en Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/conceptual-custom-locations.md)
- [App Service, Functions y Logic Apps en Azure Arc (versión preliminar)](../app-service/overview-arc-integration.md)
- [Configuración de un clúster de Kubernetes habilitado para Azure Arc para ejecutar App Service, Functions y Logic Apps (versión preliminar)](../app-service/manage-create-arc-environment.md)

## <a name="prerequisites"></a>Requisitos previos

En esta sección se describen los requisitos previos comunes en todos los enfoques y herramientas que puede usar para crear e implementar los flujos de trabajo de la aplicación lógica. Los requisitos previos específicos de la herramienta aparecen junto a los pasos correspondientes.

- Una cuenta de Azure con una suscripción activa. Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- Un entorno de Kubernetes con un clúster de Kubernetes habilitado para Azure Arc y una *ubicación personalizada* donde puede hospedar y ejecutar Azure Logic Apps, Azure App Service y Azure Functions. Asegúrese de usar la misma ubicación para el entorno de Kubernetes, la ubicación personalizada y el recurso de aplicación lógica.

  Por ejemplo, para implementar y ejecutar el entorno en el Oeste de Europa, use la opción "Oeste de Europa" como ubicación para los tres recursos.

  Asimismo, al crear la extensión de conjunto de App Service en el clúster de Kubernetes, puede [cambiar el comportamiento de escalado predeterminado](#change-scaling) para ejecutar los flujos de trabajo de la aplicación lógica. Al crear la extensión mediante el comando de la CLI de Azure, [ **`az k8s-extension create`**](/cli/azure/k8s-extension), asegúrese de incluir la opción de configuración `keda.enabled=true`:

  `az k8s-extension create {other-command-options} --configuration-settings "keda.enabled=true"`

  Para más información, revise la siguiente documentación:

  * [App Service, Functions y Logic Apps en Azure Arc (versión preliminar)](../app-service/overview-arc-integration.md)
  * [Extensiones de clústeres de Kubernetes con Azure Arc habilitado](../azure-arc/kubernetes/conceptual-extensions.md)
  * [Configuración de un clúster de Kubernetes habilitado para Azure Arc para ejecutar App Service, Functions y Logic Apps (versión preliminar)](../app-service/manage-create-arc-environment.md)
  * [Cambio del comportamiento de escalado predeterminado](#change-scaling)

- Su propia identidad de Azure Active Directory (Azure AD)

  Si los flujos de trabajo deben usar conexiones hospedadas en Azure, como Office 365 Outlook o Azure Storage, la aplicación lógica debe usar una identidad de AD para realizar la autenticación. Logic Apps habilitado para Azure Arc se puede ejecutar en cualquier infraestructura, pero requiere una identidad que tenga permisos para usar conexiones hospedadas en Azure. Para configurar esta identidad, cree un registro de aplicaciones en Azure AD que la aplicación lógica usará como la identidad necesaria.

  > [!NOTE]
  > Tenga en cuenta que la compatibilidad con identidades administradas no está disponible actualmente en Logic Apps habilitado para Azure Arc.

  Para crear un registro de aplicaciones de Azure Active Directory (Azure AD) mediante la CLI de Azure, siga estos pasos:
    1. Cree un registro de aplicaciones mediante el comando [`az ad sp create`](/cli/azure/ad/sp#az_ad_sp_create).
    1. Para revisar todos los detalles, ejecute el comando [`az ad sp show`](/cli/azure/ad/sp#az_ad_sp_show).
    1. En los resultados de ambos comandos, busque y guarde los valores de id. de cliente, id. de objeto, id. de inquilino y de secreto de cliente, que debe conservar para su uso posterior.

  Para crear un registro de aplicaciones de Azure Active Directory (Azure AD) mediante Azure Portal, siga estos pasos:
    1. Cree un nuevo registro de aplicaciones de Azure AD mediante [Azure Portal](../active-directory/develop/quickstart-register-app.md).
    1. Una vez que finalice la operación de creación, busque el nuevo registro de aplicaciones en el portal.
    1. En el menú de registro, seleccione **Información general** y guarde los valores de id. de cliente, id. de inquilino y del secreto de cliente.
    1. Para buscar el id. de objeto, junto al campo **Aplicación administrada en el directorio local**, seleccione el nombre del registro de la aplicación. En la vista de propiedades, copie el id. de objeto.

## <a name="create-and-deploy-logic-apps"></a>Creación e implementación de aplicaciones lógicas

En función de si quiere usar la CLI de Azure, Visual Studio Code o Azure Portal, seleccione la pestaña correspondiente para revisar los requisitos previos y los pasos específicos.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

#### <a name="prerequisites"></a>Requisitos previos

- La [CLI de Azure instalada](/cli/azure/install-azure-cli) en un equipo local.
- [Grupo de recursos de Azure](#create-resource-group) en el que se crea la aplicación lógica.

Compruebe el entorno antes de empezar:

1. Inicie sesión en Azure Portal. Para comprobar que la suscripción esté activa, ejecute el siguiente comando:

   ```azurecli-interactive
   az login
   ```

1. Compruebe la versión de la CLI de Azure en una ventana de terminal o de comandos, para lo que debe ejecutar el siguiente comando:

   ```azurecli-interactive
   az --version
   ```

   Para saber cuál es la versión más reciente, consulte las [notas de la versión más reciente](/cli/azure/release-notes-azure-cli?tabs=azure-cli).

1. Si no tiene la versión más reciente, actualice la instalación, para lo que debe seguir las instrucciones que encontrará en la [guía de instalación del sistema operativo o de la plataforma](/cli/azure/install-azure-cli).

#### <a name="install-logic-apps-extension"></a>Instalación de una extensión de Logic Apps

Instale la versión preliminar de la extensión de Logic Apps para la CLI de Azure:

```azurecli
az extension add --yes --source "https://aka.ms/logicapp-latest-py2.py3-none-any.whl"
```

#### <a name="create-resource-group"></a>Creación de un grupo de recursos

Si aún no tiene un grupo de recursos para la aplicación lógica, créelo con el comando `az group create`. Asegúrese de usar el parámetro `--subscription` con el nombre o identificador de la suscripción. Por ejemplo, el siguiente comando crea un grupo de recursos denominado `MyResourceGroupName` en la ubicación `eastus`:

```azurecli
az group create --name MyResourceGroupName --location eastus --subscription MySubscription
```

> [!TIP]
> Tenga en cuenta que no tiene que usar el parámetro `--subscription` si ya ha establecido una suscripción predeterminada para la cuenta de Azure.
> Para establecer una suscripción predeterminada, ejecute el siguiente comando y reemplace `MySubscription` por el nombre o identificador de la suscripción.
> `az account set --subscription MySubscription`

La salida muestra `provisioningState` como `Succeeded` cuando el grupo de recursos se crea correctamente:

```output
<...>
   "name": "testResourceGroup",
   "properties": {
      "provisioningState": "Succeeded"
    },
<...>
```

#### <a name="create-logic-app"></a>Creación de la aplicación lógica

Para crear una aplicación lógica habilitada para Azure Arc mediante la CLI de Azure, ejecute el comando `az logicapp create`, tal como se muestra a continuación:

```azurecli
az logicapp create --resource-group MyResourceGroupName --name MyLogicAppName 
   --storage-account MyStorageAccount --custom-location MyCustomLocation 
   --subscription MySubscription
```

> [!IMPORTANT]
> Asegúrese de usar la misma ubicación del recurso (región de Azure) que la ubicación personalizada y el entorno de Kubernetes. Las ubicaciones del recurso de la aplicación lógica, la ubicación personalizada y el entorno de Kubernetes deben tener los mismos valores. Este valor *no es el mismo* que el *nombre* de la ubicación personalizada.

Asegúrese de proporcionar los parámetros necesarios siguientes en el comando:

| Parámetros | Descripción |
|------------|-------------|
| `--name -n` | Nombre único de la aplicación lógica |
| `--resource-group -g` | Nombre del [grupo de recursos](../azure-resource-manager/management/manage-resource-groups-cli.md) en el que quiere crear la aplicación lógica. Si no dispone de ninguno para usarlo, [cree un grupo de recursos](#create-resource-group). |
| `--storage-account -s` | La [cuenta de almacenamiento](/cli/azure/storage/account) que quiera usar con la aplicación lógica. En cuanto a las cuentas de almacenamiento del mismo grupo de recursos, use un valor de cadena. Para las cuentas de almacenamiento de un grupo de recursos diferente, use un id. de recurso. |
|||

Para crear una aplicación lógica en Azure Arc mediante una imagen privada de Azure Container Registry, ejecute `az logicapp create` de la siguiente manera:

```azurecli
az logicapp create --resource-group MyResourceGroupName --name MyLogicAppName 
   --storage-account MyStorageAccount --subscription MySubscription
   --custom-location MyCustomLocation 
   --deployment-container-image-name myacr.azurecr.io/myimage:tag
   --docker-registry-server-password passw0rd 
   --docker-registry-server-user MyUser
```

#### <a name="show-logic-app-details"></a>Muestra de los detalles de la aplicación lógica

Para mostrar los detalles sobre la aplicación lógica habilitada en Azure Arc, ejecute el comando `az logicapp show`, tal como se muestra a continuación:

```azurecli
az logicapp show --name MyLogicAppName 
   --resource-group MyResourceGroupName --subscription MySubscription
```

#### <a name="deploy-logic-app"></a>Implementación de la aplicación lógica

Para implementar la aplicación lógica mediante la implementación zip de Kudu, ejecute el comando `az logicapp deployment source config-zip`. Por ejemplo:

```azurecli
az logicapp deployment source config-zip --name MyLogicAppName 
   --resource-group MyResourceGroupName 
   --src C:\uploads\v22.zip 
   --subscription MySubscription
```

#### <a name="start-logic-app"></a>Inicio de la aplicación lógica

Para iniciar la aplicación lógica habilitada para Azure Arc, ejecute el comando `az logicapp start` con los siguientes parámetros necesarios:

```azurecli
az logicapp start --name MyLogicAppName 
   --resource-group MyResourceGroupName --subscription MySubscription
```

#### <a name="stop-logic-app"></a>Detención de la aplicación lógica

Para detener la aplicación lógica habilitada para Azure Arc, ejecute el comando `az logicapp stop` con los siguientes parámetros necesarios:

```azurecli
az logicapp stop --name MyLogicAppName 
   --resource-group MyResourceGroupName --subscription MySubscription
```

#### <a name="restart-logic-app"></a>Reinicio de la aplicación lógica

Para reiniciar la aplicación lógica habilitada para Azure Arc, ejecute el comando `az logicapp restart` con los siguientes parámetros necesarios:

```azurecli
az logicapp restart --name MyLogicAppName 
   --resource-group MyResourceGroupName --subscription MySubscription
```

#### <a name="delete-logic-app"></a>Eliminación de la aplicación lógica

Para eliminar la aplicación lógica habilitada para Azure Arc, ejecute el comando `az logicapp delete` con los siguientes parámetros necesarios:

Por ejemplo: 

```azurecli
az logicapp delete --name MyLogicAppName --resource-group MyResourceGroupName --subscription MySubscription
```

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Puede crear, implementar y supervisar los flujos de trabajo de la aplicación lógica de un extremo a otro mediante Visual Studio Code. No hay ningún cambio o diferencia en la experiencia del diseñador que se usa para el desarrollo de flujos de trabajo de aplicaciones lógicas que se ejecutan en un solo inquilino de Azure Logic Apps en comparación con Logic Apps habilitado para Azure Arc.

#### <a name="create-and-deploy-logic-app-workflows"></a>Creación e implementación de flujos de trabajo de aplicación lógica

1. Para crear un proyecto de aplicación lógica, siga los requisitos previos y los pasos de la documentación [Creación de flujos de trabajo de integración en una instancia de Azure Logic Apps de un solo inquilino con Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md).

1. Cuando esté listo para realizar la implementación en Azure, use la experiencia **Implementar en la aplicación lógica** para crear el recurso de aplicación lógica en la ubicación personalizada que creó anteriormente e implementar los flujos de trabajo en la misma ubicación.

   1. En un área en blanco en la ventana del proyecto de aplicación lógica, abra el menú contextual y seleccione **Implementar en la aplicación lógica**.

   1. Seleccione la suscripción de Azure asociada a la ubicación personalizada.

   1. Para crear un nuevo recurso de Logic Apps habilitado para Azure Arc, seleccione **Creación de una nueva aplicación lógica en Azure (avanzado)** . Si lo desea, también puede seleccionar un recurso de aplicación lógica existente de la lista y omitir los pasos siguientes.

   1. Proporcione un nombre único global para la aplicación lógica.

   1. Seleccione la ubicación personalizada para el entorno de Kubernetes habilitado para Azure Arc en el lugar donde quiera implementarlo. Si selecciona una región genérica de Azure, debe crear un recurso de aplicación lógica que no esté habilitado en Azure Arc y que se ejecute en una instancia de Azure Logic Apps de un solo inquilino.

   1. Seleccione o cree un nuevo grupo de recursos en el que quiera implementar la aplicación lógica.

   1. Seleccione o cree una nueva cuenta de almacenamiento para guardar el estado y los metadatos de la aplicación lógica.

   1. Seleccione o cree un nuevo recurso de Application Insights para almacenar los registros de aplicación de la aplicación lógica.

   1. Si aún no lo ha hecho, configure la identidad de Azure Active Directory (Azure AD) para que la aplicación lógica pueda autenticar las conexiones de API administradas. Para obtener más información, consulte los [Requisitos previos](#prerequisites) de nivel superior.

   1. Escriba el id. de cliente, el id. de inquilino, el id. de objeto y el secreto de cliente para la identidad de Azure AD.

      > [!NOTE]
      > Solo tiene que completar este paso una vez. Visual Studio Code actualiza las directivas de acceso del archivo connections.json y de las conexiones de API administradas.

1. Cuando haya terminado, la aplicación lógica estará activa y en ejecución en el clúster de Kubernetes habilitado para Azure Arc, lista para que la pruebe.

### <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

#### <a name="create-and-deploy-logic-app-workflows"></a>Creación e implementación de flujos de trabajo de aplicación lógica

La funcionalidad de edición del diseñador basado en el portal está actualmente en desarrollo para la instancia de Logic Apps habilitada para Azure Arc. Puede crear, implementar y ver las aplicaciones lógicas mediante el diseñador basado en el portal, pero tenga en cuenta que no puede editarlas en el portal después de la implementación. Por ahora, puede crear y editar un proyecto de aplicaciones lógicas de forma local en Visual Studio Code y, a continuación, implementar mediante Visual Studio Code, la CLI de Azure o mediante implementaciones automatizadas.

1. [En el portal, cree un recurso de **aplicación lógica (estándar)**](create-single-tenant-workflows-azure-portal.md), pero asegúrese de usar la ubicación personalizada que creó anteriormente como ubicación de la aplicación.

   > [!IMPORTANT]
   > Las ubicaciones del recurso de la aplicación lógica, la ubicación personalizada y el entorno de Kubernetes deben tener los mismos valores.

   De forma predeterminada, el recurso de **aplicación lógica (estándar)** se ejecuta en una instancia de Azure Logic Apps de un solo inquilino. Sin embargo, en el caso de Logic Apps habilitado para Azure Arc, el recurso de aplicación lógica se ejecuta en la ubicación personalizada que creó para el entorno de Kubernetes. Asimismo, no es necesario crear un plan de App Service, ya que se crea automáticamente.

1. [Edite e implemente la aplicación lógica mediante Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md).

1. Después de compilar e implementar la aplicación lógica, puede supervisar y ver los flujos de trabajo como de costumbre mediante el portal o Application Insights.

   La experiencia del portal para las aplicaciones lógicas implementadas está disponible actualmente en modo de solo lectura, lo que significa que no se puede cambiar la configuración de los flujos de trabajo ni de la aplicación. Sin embargo, todavía puede ver el historial de ejecución, el historial de desencadenadores y otra información sobre las aplicaciones. Por ahora, para actualizar las aplicaciones lógicas, puede usar la CLI de Azure, Visual Studio Code o las implementaciones automatizadas.

---

## <a name="set-up-connection-authentication"></a>Configuración de la autenticación de la conexión

Actualmente, los clústeres de Kubernetes habilitados para Azure Arc no admiten el uso de la identidad administrada de una aplicación lógica para autenticar las conexiones de API administradas. Estas conexiones administradas y hospedadas en Azure se crean cuando se usan conectores administrados en los flujos de trabajo.

En su lugar, tiene que crear su propio registro de aplicación en Azure Active Directory (Azure AD). A continuación, puede usar este registro de aplicación como identidad para las aplicaciones lógicas implementadas y ejecutándose en Logic Apps habilitado para Azure Arc. Para obtener más información, revise los [Requisitos previos de nivel superior](#prerequisites).

En el registro de la aplicación, encontrará el id. de cliente, el id. de objeto, el id. de inquilino y el secreto de cliente. Si usa Visual Studio Code para realizar implementaciones, tiene una experiencia integrada para configurar la aplicación lógica con una identidad de Azure AD. Para obtener más información, consulte [Creación e implementación de flujos de trabajo de aplicación lógica: Visual Studio Code](#create-and-deploy-logic-apps).

Sin embargo, si usa Visual Studio Code para el desarrollo, pero usa la CLI de Azure o canalizaciones automatizadas para realizar las implementaciones, siga estos pasos.

### <a name="configure-connection-and-app-settings-in-your-project"></a>Configuración de la conexión y la aplicación en el proyecto

1. En el archivo **connections.json** del proyecto de aplicación lógica, busque el objeto `authentication` de la conexión administrada. Reemplace el contenido de este objeto por la información de registro de la aplicación que generó anteriormente en los [requisitos previos de nivel superior](#prerequisites):

   ```json
   "authentication": {
      "type": "ActiveDirectoryOAuth",
      "audience": "https://management.core.windows.net/",
      "credentialType": "Secret",
      "clientId": "@appsetting('WORKFLOWAPP_AAD_CLIENTID')",
      "tenant": "@appsetting('WORKFLOWAPP_AAD_TENANTID')",
      "secret": "@appsetting('WORKFLOWAPP_AAD_CLIENTSECRET')"
   } 
   ```

1. En el archivo **local.settings.json** del proyecto de aplicación lógica, agregue el id. de cliente, el id. de objeto, el id. de inquilino y el secreto de cliente. Después de realizar la implementación, esta configuración se convierte en la configuración de la aplicación lógica.

   ```json
   {
      "IsEncrypted": false,
      "Values": {
         <...>
         "WORKFLOWAPP_AAD_CLIENTID":"<my-client-ID>",
         "WORKFLOWAPP_AAD_OBJECTID":"<my-object-ID",
         "WORKFLOWAPP_AAD_TENANTID":"<my-tenant-ID>",
         "WORKFLOWAPP_AAD_CLIENTSECRET":"<my-client-secret>"
      }
   }
   ```

> [!IMPORTANT]
> En escenarios o entornos de producción, asegúrese de proteger estos secretos e información confidencial, por ejemplo, mediante el uso de un [almacén de claves](../app-service/app-service-key-vault-references.md).

### <a name="add-access-policies"></a>Adición de directivas de acceso

En las instancias de Azure Logic Apps de un solo inquilino, cada aplicación lógica tiene una identidad a la que las directivas de acceso conceden permisos para usar conexiones administradas y hospedadas en Azure. Puede configurar estas directivas de acceso mediante las implementaciones de Azure Portal o de infraestructura.

#### <a name="arm-template"></a>Plantilla ARM

En la plantilla de Azure Resource Manager (plantilla de ARM), incluya la siguiente definición de recurso para *cada* conexión de API administrada y proporcione la siguiente información:

| Parámetro | Descripción |
|-----------|-------------|
| <*connection-name*> | El nombre de la conexión de la API administrada, por ejemplo, `office365`. |
| <*object-ID*> | Es el id. de objeto de la identidad de Azure AD, guardada previamente desde el registro de la aplicación. |
| <*tenant-ID*> | Es el id. de inquilino de la identidad de Azure AD, guardada previamente desde el registro de la aplicación. |
|||

```json
{
   "type": "Microsoft.Web/connections/accessPolicies",
   "apiVersion": "2016-06-01",
   "name": "[concat('<connection-name>'),'/','<object-ID>')]",
   "location": "<location>",
   "dependsOn": [
      "[resourceId('Microsoft.Web/connections', parameters('connection_name'))]"
   ],
   "properties": {
      "principal": {
         "type": "ActiveDirectory",
         "identity": {
            "objectId": "<object-ID>",
            "tenantId": "<tenant-ID>"
         }
      }
   }
}
```

Para obtener más información, revise la documentación [Microsoft.Web/connections/accesspolicies (plantilla de ARM)](/templates/microsoft.web/connections?tabs=json). 

#### <a name="azure-portal"></a>Azure Portal

Para esta tarea, use el id. de cliente que guardó anteriormente como *id. de aplicación*.

1. En Azure Portal, busque y abra la aplicación lógica. En el menú de la aplicación lógica, en la opción **Flujos de trabajo**, seleccione **Conexiones**, que enumera todas las conexiones de los flujos de trabajo de la aplicación lógica.

1. En **Conexiones de API**, seleccione una conexión; en este ejemplo es `office365`.

1. En el menú de la conexión, en la opción **Configuración**, seleccione **Directivas de acceso** > **Agregar**.
 
1. En el panel **Agregar directiva de acceso**, en el cuadro de búsqueda, busque y seleccione el id. de cliente que guardó anteriormente.

1. Cuando finalice, seleccione **Agregar**.

1. Repita estos pasos para cada conexión hospedada en Azure en la aplicación lógica.

## <a name="automate-devops-deployment"></a>Automatización de la implementación de DevOps

Para compilar e implementar las aplicaciones lógicas habilitadas para Azure Arc, puede usar las mismas canalizaciones y procesos de las [aplicaciones lógicas basadas en un solo inquilino](single-tenant-overview-compare.md). Para automatizar las implementaciones de infraestructura mediante las canalizaciones de DevOps, realice los siguientes cambios en el nivel de infraestructura para las implementaciones que no sean de contenedor y las de contenedor.

### <a name="standard-deployment-non-container"></a>Implementación estándar (sin contenedor)

Si usa la implementación de tipo ZIP para realizar la implementación de aplicaciones lógicas, no es necesario que configure un registro de Docker para hospedar imágenes de contenedor. Aunque las aplicaciones lógicas en Kubernetes se ejecutan técnicamente en contenedores, Logic Apps habilitado para Azure Arc puede administrar estos contenedores automáticamente. En este escenario, realice las siguientes tareas al configurar la infraestructura:

- Notifique al proveedor de recursos que va a crear una aplicación lógica en Kubernetes.
- Incluya un plan de App Service con la implementación. Para obtener más información, consulte [Incluir un plan de App Service con una implementación](#include-app-service-plan).

En la [plantilla de Azure Resource Manager (plantilla de ARM)](../azure-resource-manager/templates/overview.md), incluya los valores siguientes:

| Elemento | Propiedad JSON | Descripción |
|------|---------------|-------------|
| Location | `location` | Asegúrese de usar la misma ubicación del recurso (región de Azure) que la ubicación personalizada y el entorno de Kubernetes. La ubicación del recurso de la aplicación lógica, la ubicación personalizada y el entorno de Kubernetes deben tener los mismos valores. <p><p>**Nota**: Este valor no es el mismo que el *nombre* de la ubicación personalizada. |
| Tipo de aplicación | `kind` | El tipo de aplicación que va a implementar para que la plataforma de Azure pueda identificar la aplicación. En Azure Logic Apps, esta información es similar al ejemplo siguiente: `kubernetes,functionapp,workflowapp,linux`. |
| Ubicación extendida | `extendedLocation` | Este objeto requiere el valor `"name"` de la *ubicación personalizada* para el entorno de Kubernetes y debe tener el valor `"type"` establecido en `"CustomLocation"`. |
| Id. de recurso del plan de hospedaje | `serverFarmId` | Id. de recurso del plan de App Service asociado, con el formato siguiente: <p><p>`"/subscriptions/{subscriptionID}/resourceGroups/{groupName}/providers/Microsoft.Web/serverfarms/{appServicePlanName}"` |
| Cadena de conexión de almacenamiento | `AzureWebJobsStorage` | Cadena de conexión para la cuenta de almacenamiento. <p><p>**Importante**: debe proporcionar la cadena de conexión para la cuenta de almacenamiento en la plantilla de ARM. En escenarios o entornos de producción, asegúrese de proteger estos secretos e información confidencial, por ejemplo, mediante un almacén de claves. |
||||

#### <a name="arm-template"></a>Plantilla ARM

En el ejemplo siguiente se describe un ejemplo de la definición de recursos de Logic Apps habilitado para Azure Arc que puede usar en la plantilla de ARM. Para obtener más información, revise la documentación sobre el [formato de plantilla Microsoft.Web/sites (JSON)](/templates/microsoft.web/sites?tabs=json).

```json
{
   "type": "Microsoft.Web/sites",
   "apiVersion": "2020-12-01",
   "name": "[parameters('name')]",
   "location": "[parameters('location')]",
   "kind": "kubernetes,functionapp,workflowapp,linux",
   "extendedLocation": {
      "name": "[parameters('customLocationId')]",
      "type": "CustomLocation"
    },
   "properties": {
      "clientAffinityEnabled": false,
      "name": "[parameters('name')]",
      "serverFarmId": "<hosting-plan-ID>",
      "siteConfig": {
         "appSettings": [
            {
               "name": "FUNCTIONS_EXTENSION_VERSION",
               "value": "~3"
            },
            {
               "name": "FUNCTIONS_WORKER_RUNTIME",
               "value": "node"
            },
            {
               "name": "AzureWebJobsStorage",
               "value": "<storage-connection-string>"
            },
            {
               "name": "AzureFunctionsJobHost__extensionBundle__id",
               "value": "Microsoft.Azure.Functions.ExtensionBundle.Workflows"
            },
            {
               "name": "AzureFunctionsJobHost__extensionBundle__version",
               "value": "[1.*, 2.0.0)"
            },
            {
               "name": "APP_KIND",
               "value": "workflowapp"
            }
         ],
         "use32BitWorkerProcess": "[parameters('use32BitWorkerProcess')]",
         "linuxFxVersion": "Node|12"
      }
   }
}
```

### <a name="container-deployment"></a>Implementación de contenedores

Si prefiere usar herramientas de contenedor y procesos de implementación, puede crear en contenedores las aplicaciones lógicas e implementarlas en Logic Apps habilitado para Azure Arc. En este escenario, realice las siguientes tareas de alto nivel al configurar la infraestructura:

- Configure un registro de Docker para hospedar las imágenes de contenedor.
- Notifique al proveedor de recursos que va a crear una aplicación lógica en Kubernetes.
- En la plantilla de implementación, seleccione el registro de Docker y la imagen de contenedor donde planea implementarlo. La instancia de Azure Logic Apps de un solo inquilino usa esta información para obtener la imagen de contenedor del registro de Docker.
- Incluya un plan de App Service con la implementación. Para obtener más información, consulte [Incluir un plan de App Service con una implementación](#include-app-service-plan).

En la [plantilla de Azure Resource Manager (plantilla de ARM)](../azure-resource-manager/templates/overview.md), incluya los valores siguientes:

| Elemento | Propiedad JSON | Descripción |
|------|---------------|-------------|
| Location | `location` | Asegúrese de usar la misma ubicación del recurso (región de Azure) que la ubicación personalizada y el entorno de Kubernetes. Las ubicaciones del recurso de la aplicación lógica, la ubicación personalizada y el entorno de Kubernetes deben tener los mismos valores. <p><p>**Nota**: Este valor *no es el mismo* que el *nombre* de la ubicación personalizada. |
| Tipo de aplicación | `kind` | El tipo de aplicación que va a implementar para que la plataforma de Azure pueda identificar la aplicación. En Azure Logic Apps, esta información es similar al ejemplo siguiente: `kubernetes,functionapp,workflowapp,container`. |
| Ubicación extendida | `extendedLocation` | Este objeto requiere el valor `"name"` de la *ubicación personalizada* para el entorno de Kubernetes y debe tener el valor `"type"` establecido en `"CustomLocation"`. |
| Nombre del contenedor | `linuxFxVersion` | El nombre del contenedor cuenta con el formato siguiente: `DOCKER\|<container-name>`. |
| Id. de recurso del plan de hospedaje | `serverFarmId` | Id. de recurso del plan de App Service asociado, con el formato siguiente: <p><p>`"/subscriptions/{subscriptionID}/resourceGroups/{groupName}/providers/Microsoft.Web/serverfarms/{appServicePlanName}"` |
| Cadena de conexión de almacenamiento | `AzureWebJobsStorage` | Cadena de conexión para la cuenta de almacenamiento. <p><p>**Importante:** al implementar contenido en un contenedor de Docker, debe proporcionar la cadena de conexión de la cuenta de almacenamiento en la plantilla de ARM. En escenarios o entornos de producción, asegúrese de proteger estos secretos e información confidencial, por ejemplo, mediante un almacén de claves. |
||||

Para hacer referencia a la imagen de contenedor y del registro de Docker, incluya estos valores en la plantilla:

| Elemento | Propiedad JSON | Descripción |
|------|---------------|-------------|
| Dirección URL del servidor de registro de Docker | `DOCKER_REGISTRY_SERVER_URL` | Es la dirección URL del servidor del registro de Docker. |
| Servidor de registro de Docker | `DOCKER_REGISTRY_SERVER_USERNAME` | Es el nombre de usuario para obtener acceso al servidor del registro de Docker. |
| Contraseña del servidor de registro de Docker | `DOCKER_REGISTRY_SERVER_PASSWORD` | Es la contraseña para obtener acceso al servidor del registro de Docker. |
||||

#### <a name="arm-template"></a>Plantilla ARM

En el ejemplo siguiente se describe un ejemplo de la definición de recursos de Logic Apps habilitado para Azure Arc que puede usar en la plantilla de ARM. Para obtener más información, revise la documentación sobre el [formato de plantilla Microsoft.Web/sites (plantilla de ARM)](/templates/microsoft.web/sites?tabs=json).

```json
{
   "type": "Microsoft.Web/sites",
   "apiVersion": "2020-12-01",
   "name": "[parameters('name')]",
   "location": "[parameters('location')]",
   "kind": " kubernetes,functionapp,workflowapp,container",
   "extendedLocation": {
      "name": "[parameters('customLocationId')]",
      "type": "CustomLocation"
    },
   "properties": {
      "name": "[parameters('name')]",
      "clientAffinityEnabled": false,
      "serverFarmId": "<hosting-plan-ID>",
      "siteConfig": {
         "appSettings": [
            {
               "name": "FUNCTIONS_EXTENSION_VERSION",
               "value": "~3"
            },
            {
               "name": "FUNCTIONS_WORKER_RUNTIME",
               "value": "node"
            },
            {
               "name": "AzureWebJobsStorage",
               "value": "<storage-connection-string>"
            },
            {
               "name": "AzureFunctionsJobHost__extensionBundle__id",
               "value": "Microsoft.Azure.Functions.ExtensionBundle.Workflows"
            },
            {
               "name": "AzureFunctionsJobHost__extensionBundle__version",
               "value": "[1.*, 2.0.0)"
            },
            {
               "name": "APP_KIND",
               "value": "workflowapp"
            }, 
            {
               "name": "DOCKER_REGISTRY_SERVER_URL",
               "value": "<docker-registry-server-URL>"
            },
            { 
               "name": "DOCKER_REGISTRY_SERVER_USERNAME",
               "value": "<docker-registry-server-username>"
            },
            {
               "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
               "value": "<docker-registry-server-password>"
            }
         ],
         "use32BitWorkerProcess": "[parameters('use32BitWorkerProcess')]",
         "linuxFxVersion": "DOCKER|<container-name>"
      }
   }
}
```

<a name="include-app-service-plan"></a>

### <a name="include-app-service-plan-with-deployment"></a>Inclusión del plan de App Service con la implementación

Tanto si tiene una implementación estándar como de contenedor, debe incluir un plan de App Service con la implementación. Aunque este plan se vuelve menos relevante con un entorno de Kubernetes, recuerde que las implementaciones estándar y de contenedor seguirán necesitando un plan de App Service.

Aunque otras opciones de creación normalmente controlan el aprovisionamiento del recurso de Azure para este plan, si las implementaciones usan plantillas de tipo "infraestructura como código", tendrá que crear explícitamente el recurso de Azure para el plan. El recurso del plan de hospedaje no cambia, solo la información `sku`.

En la [plantilla de Azure Resource Manager (plantilla de ARM)](../azure-resource-manager/templates/overview.md), incluya los valores siguientes:

| Elemento | Propiedad JSON | Descripción |
|------|---------------|-------------|
| Location | `location` | Asegúrese de usar la misma ubicación del recurso (región de Azure) que la ubicación personalizada y el entorno de Kubernetes. La ubicación del recurso de la aplicación lógica, la ubicación personalizada y el entorno de Kubernetes deben tener los mismos valores. <p><p>**Nota**: Este valor no es el mismo que el *nombre* de la ubicación personalizada. |
| Tipo | `kind` | El tipo de plan de App Service implementado que debe ser `kubernetes,linux` |
| Ubicación extendida | `extendedLocation` | Este objeto requiere el valor `"name"` de la *ubicación personalizada* para el entorno de Kubernetes y debe tener el valor `"type"` establecido en `"CustomLocation"`. |
| Nombre del plan de hospedaje | `name` | El nombre del plan de App Service. |
| Nivel de plan | `sku: tier` | Es el nivel del plan de App Service, que es `K1`. |
| Nombre del plan | `sku: name` | Es el nombre del plan de App Service, que es `Kubernetes`. |
||||

#### <a name="arm-template"></a>Plantilla ARM

En el ejemplo siguiente se describe una definición de recursos del plan de App Service que puede usar con la implementación de la aplicación. Para obtener más información, revise la documentación sobre el [formato de plantilla Microsoft.Web/serverfarms (plantilla de ARM)](/templates/microsoft.web/serverfarms?tabs=json).

```json
{
   "type": "Microsoft.Web/serverfarms",
   "apiVersion": "2020-12-01",
   "location": "<location>",
   "name": "<hosting-plan-name>",
   "kind": "kubernetes,linux",
   "extendedLocation": {
      "name": "[parameters('customLocationId')]",
      "type": "CustomLocation"
   },
   "sku": {
      "tier": "K1",
      "name": "Kubernetes", 
      "capacity": 1
   },
   "properties": {
      "kubeEnvironmentProfile": {
         "id": "[parameters('kubeEnvironmentId')]"
      }
   }
}
```

<a name="change-scaling"></a>

## <a name="change-default-scaling-behavior"></a>Cambio del comportamiento de escalado predeterminado

Logic Apps habilitado para Azure Arc administra automáticamente el escalado de las aplicaciones lógicas en función del número de *trabajos* de la cola de almacenamiento de back-end. Sin embargo, puede cambiar el comportamiento de escalado predeterminado.

En una aplicación lógica, la definición del flujo de trabajo especifica la secuencia de acciones que se ejecutarán. Cada vez que se desencadena una ejecución de flujo de trabajo, el tiempo de ejecución de Azure Logic Apps crea un *trabajo* para cada tipo de acción en la definición del flujo de trabajo. A continuación, el tiempo de ejecución organiza estos trabajos en un *secuenciador de trabajos*. Este secuenciador orquesta la ejecución de los trabajos para la definición del flujo de trabajo, pero el motor de orquestación del trabajo subyacente de Azure Logic Apps ejecuta cada trabajo.

En cuanto a los flujos de trabajo con estado, el motor de orquestación de trabajos usa mensajes de cola de almacenamiento para programar trabajos en los secuenciadores de trabajos. En segundo plano, los *distribuidores de trabajos* (o las *instancias de trabajo del distribuidor*) supervisan estas colas de trabajos. El motor de orquestación usa un número mínimo y máximo predeterminado de instancias de trabajo para supervisar las colas de trabajos. En el caso de los flujos de trabajo sin estado, el motor de orquestación mantiene completamente los estados de acción en la memoria.

Para cambiar el comportamiento de escalado predeterminado, especifique diferentes números mínimos y máximos de instancias de trabajo que supervisan las colas de trabajos.

### <a name="prerequisites-to-change-scaling"></a>Requisitos previos para cambiar el escalado

En el clúster de Kubernetes habilitado para Arc, la extensión de agrupación de App Service que se creó anteriormente debe tener la propiedad `keda.enabled` establecida en `true`. Para obtener más información, revise los [Requisitos previos de nivel superior](#prerequisites).

### <a name="change-scaling-threshold"></a>Cambio del umbral de escalado

En Logic Apps habilitado para Azure Arc, la longitud de la cola de trabajos desencadena un evento de escalado y establece un umbral para indicar la frecuencia con la que se produce el escalado de la aplicación lógica. Puede cambiar la longitud de la cola, que tiene el valor predeterminado establecido en `20` trabajos. Para escalar con menos frecuencia, aumente la longitud de la cola. Para escalar con mayor frecuencia, aumente la longitud de la cola. Este proceso puede requerir algunas operaciones de prueba y error.

Para cambiar la longitud de la cola, en el archivo de nivel raíz **host.json** del proyecto de aplicación lógica, configure la propiedad `Runtime.ScaleMonitor.KEDA.TargetQueueLength`, por ejemplo:

```json
"extensions": {
   "workflow": {
      "settings": {
         "Runtime.ScaleMonitor.KEDA.TargetQueueLength": "10"
      }
   }
}
```

### <a name="change-throughput-maximum"></a>Cambio del valor máximo del rendimiento

En un recurso de aplicación lógica existente, puede cambiar el número máximo de instancias de trabajo, que tiene el valor predeterminado establecido en `2`. Este valor controla el límite superior en el número de instancias de trabajo que pueden supervisar las colas de trabajos.

Para cambiar este máximo, use la CLI de Azure (solo para la creación de la aplicación lógica) y Azure Portal.

#### <a name="azure-cli"></a>Azure CLI

Para una nueva aplicación lógica, ejecute el comando de la CLI de Azure `az logicapp create`, por ejemplo:

```azurecli
az logicapp create --resource-group MyResourceGroupName 
   --name MyLogicAppName --storage-account MyStorageAccount 
   --custom-location --subscription MySubscription  MyCustomLocation 
   [--plan MyHostingPlan] [--min-worker-count 1] [--max-worker-count 4]
```

Para configurar el número máximo de instancias, use el parámetro `--settings`:

```azurecli
az logicapp config appsettings set --name MyLogicAppName 
   --resource-group MyResourceGroupName 
   --settings "K8SE_APP_MAX_INSTANCE_COUNT=10" 
   --subscription MySubscription
```

#### <a name="azure-portal"></a>Azure Portal

En la configuración de la aplicación lógica basada en un solo inquilino, agregue o edite el valor de configuración `K8SE_APP_MAX_INSTANCE_COUNT` siguiendo estos pasos:

1. En Azure Portal, busque y abra la aplicación lógica basada en un solo inquilino.
1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración**.
1. En el **panel Configuración,** en la opción **Configuración de la aplicación**, agregue una nueva configuración de aplicación o edite el valor existente, si ya se ha agregado.

   1. Seleccione **Nueva configuración de aplicación** y agregue la configuración `K8SE_APP_MAX_INSTANCE_COUNT` con el valor máximo que quiera.

   1. Edite el valor existente para la configuración `K8SE_APP_MAX_INSTANCE_COUNT`.

1. Cuando haya terminado, guarde los cambios.

### <a name="change-throughput-minimum"></a>Cambio del valor mínimo del rendimiento

En un recurso de aplicación lógica existente, puede cambiar el número mínimo de instancias de trabajo, que tiene el valor predeterminado establecido en `1`. Este valor controla el límite inferior en el número de instancias de trabajo que pueden supervisar las colas de trabajos. Para obtener una alta disponibilidad o rendimiento, aumente este valor.

Para cambiar este mínimo, use la CLI de Azure o Azure Portal.

#### <a name="azure-cli"></a>Azure CLI

Para un recurso de aplicación lógica existente, ejecute el comando de la CLI de Azure `az logicapp scale`, por ejemplo:

```azurecli
az logicapp scale --name MyLogicAppName --resource-group MyResourceGroupName 
   --instance-count 5 --subscription MySubscription
```

Para una nueva aplicación lógica, ejecute el comando de la CLI de Azure `az logicapp create`, por ejemplo:

```azurecli
az logicapp create --resource-group MyResourceGroupName --name MyLogicAppName 
   --storage-account MyStorageAccount --custom-location 
   --subscription MySubscription MyCustomLocation 
   [--plan MyHostingPlan] [--min-worker-count 2] [--max-worker-count 4]
```

#### <a name="azure-portal"></a>Azure Portal

En la configuración de la aplicación lógica basada en un solo inquilino, cambie el valor de la propiedad **Escalar horizontalmente** siguiendo estos pasos:

1. En Azure Portal, busque y abra la aplicación lógica basada en un solo inquilino.
1. En la opción **Configuración** del menú de la aplicación lógica, seleccione **Escalar horizontalmente**.
1. En el panel **Escalar horizontalmente**, arrastre el control deslizante de instancias mínimas al valor que quiera.
1. Cuando haya terminado, guarde los cambios.

## <a name="troubleshoot-problems"></a>Solucionar problemas

Para obtener más información sobre las aplicaciones lógicas implementadas, pruebe las siguientes opciones:

### <a name="access-app-settings-and-configuration"></a>Acceso a la configuración y la configuración de la aplicación

Para acceder a la configuración de la aplicación, ejecute el siguiente comando de la CLI de Azure:

```azurecli
az logicapp config appsettings list --name MyLogicAppName 
   --resource-group MyResourceGroupName --subscription MySubscription
```

Para configurar una aplicación, ejecute el comando `az logicapp config appsettings set` de la siguiente manera. Asegúrese de usar el parámetro `--settings` con el nombre y el valor de su configuración.

```azurecli
az logicapp config appsettings set --name MyLogicAppName 
   --resource-group MyResourceGroupName 
   --settings "MySetting=1" 
   --subscription MySubscription
```

Para eliminar la configuración de una aplicación, ejecute el comando `az logicapp config appsettings delete` de la siguiente manera. Asegúrese de usar el parámetro `--setting-names` con el nombre de la configuración que quiere eliminar.

```azurecli
az logicapp config appsettings delete --name MyLogicAppName 
   --resource-group MyResourceGroupName 
   --setting-names MySetting 
   --subscription MySubscription
```

### <a name="view-logic-app-properties"></a>Visualización de las propiedades de la aplicación lógica

Para ver la información y las propiedades de la aplicación, ejecute el siguiente comando de la CLI de Azure: 

```azurecli
az logicapp show --name MyLogicAppName --resource-group MyResourceGroupName --subscription MySubscription
```

### <a name="monitor-workflow-activity"></a>Supervisión de la actividad del flujo de trabajo

Para ver la actividad de un flujo de trabajo en la aplicación lógica, siga estos pasos: 

1. En Azure Portal, busque la aplicación lógica implementada.

1. En el menú de la aplicación lógica, seleccione **Flujos de trabajo** y, a continuación, seleccione el flujo de trabajo. 

1. En el menú del flujo de trabajo, seleccione  **Supervisar**.

### <a name="collect-logs"></a>Recopilación de registros

Para obtener los datos registrados sobre la aplicación lógica, habilite Application Insights en la aplicación lógica si aún no está habilitada.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre [Logic Apps habilitado para Azure Arc](azure-arc-enabled-logic-apps-overview.md).
