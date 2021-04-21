---
title: Autenticación con identidades administradas
description: Acceda a recursos protegidos por Azure Active Directory sin iniciar sesión con credenciales ni secretos mediante una identidad administrada
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, azla
ms.topic: article
ms.date: 03/30/2021
ms.openlocfilehash: 8e081257d70c9bc9c9f75df18b30f8dcf119e48e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763350"
---
# <a name="authenticate-access-to-azure-resources-by-using-managed-identities-in-azure-logic-apps"></a>Autenticar el acceso a los recursos de Azure mediante identidades administradas en Azure Logic Apps

Para acceder fácilmente a otros recursos protegidos por Azure Active Directory (Azure AD) y autenticar la identidad, la aplicación lógica puede usar una [identidad administrada](../active-directory/managed-identities-azure-resources/overview.md) (anteriormente conocida como Managed Service Identity o MSI), en lugar de credenciales, secretos o tokens de Azure AD. Azure administra esta identidad y le ayuda a proteger las credenciales porque, de esta forma, no tiene que administrar secretos ni usar directamente tokens de Azure AD.

Azure Logic Apps admite identidades administradas tanto [*asignadas por el sistema*](../active-directory/managed-identities-azure-resources/overview.md) como [*asignadas por el usuario*](../active-directory/managed-identities-azure-resources/overview.md). Su aplicación lógica o sus conexiones individuales pueden usar la identidad asignada por el sistema o una *sola* identidad asignada por el usuario, que puede compartir en un grupo de aplicaciones lógicas, pero no ambas.

<a name="triggers-actions-managed-identity"></a>

## <a name="where-can-logic-apps-use-managed-identities"></a>¿Dónde pueden las aplicaciones lógicas usar identidades administradas?

Actualmente, solo las [acciones y desencadenadores integrados específicos](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions) y los [conectores administrados específicos](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions) que admiten OAuth de Azure AD pueden usar una identidad administrada para la autenticación. Por ejemplo, a continuación se muestra una selección:

<a name="built-in-managed-identity"></a>

**Acciones y desencadenadores integrados**

* Azure API Management
* Azure App Services
* Azure Functions
* HTTP
* HTTP + Webhook

> [!NOTE]
> Aunque la acción y el desencadenador HTTP pueden autenticar conexiones en las cuentas de Azure Storage detrás de firewalls de Azure mediante la identidad administrada asignada por el sistema, no pueden usar la identidad administrada asignada por el usuario para autenticar las mismas conexiones.

<a name="managed-connectors-managed-identity"></a>

**Conectores administrados**

* Azure Automation
* Azure Event Grid
* Azure Key Vault
* Azure Resource Manager
* HTTP with Azure AD

La compatibilidad con conectores administrados está actualmente en versión preliminar. Para ver la lista actual, consulte [Tipos de autenticación para desencadenadores y acciones que admiten la autenticación](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

En este artículo se muestra cómo configurar ambos tipos de identidades administradas para la aplicación lógica. Para más información, consulte los temas siguientes:

* [Desencadenadores y acciones que admiten identidades administradas](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions)
* [Límites sobre las identidades administradas para aplicaciones lógicas](../logic-apps/logic-apps-limits-and-config.md#managed-identity)
* [Servicios de Azure que admiten la autenticación de Azure AD con las identidades administradas](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication)

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/). La identidad administrada y el recurso de Azure de destino al que necesita acceder deben usar la misma suscripción a Azure.

* Para conceder a una identidad administrada acceso a un recurso de Azure, debe agregar un rol al recurso de destino para esa identidad. Para agregar roles, necesita [permisos de administrador de Azure AD](../active-directory/roles/permissions-reference.md) que puedan asignar roles a identidades en el inquilino de Azure AD correspondiente.

* El recurso de Azure de destino al que quiere acceder. En este recurso, agregará un rol para la identidad administrada, lo que ayuda a la aplicación lógica a autenticar el acceso al recurso de destino.

* La aplicación lógica en la que desea usar el [desencadenador o las acciones que admiten identidades administradas](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

## <a name="enable-managed-identity"></a>Habilitación de una entidad administrada

Para configurar la identidad administrada que desea usar, siga el vínculo de esa identidad:

* [Identidad asignada por el sistema](#system-assigned)
* [Identidad asignada por el usuario](#user-assigned)

<a name="system-assigned"></a>

### <a name="enable-system-assigned-identity"></a>Activar una identidad asignada por el sistema

A diferencia de las identidades asignadas por el usuario, no tiene que crear manualmente la identidad asignada por el sistema. Para configurar la identidad asignada por el sistema de la aplicación lógica, estas son las opciones que puede usar:

* [Azure Portal](#azure-portal-system-logic-app)
* [Plantillas del Administrador de recursos de Azure](#template-system-logic-app)

<a name="azure-portal-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-portal"></a>Activar una identidad asignada por el sistema en Azure Portal

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Identidad**. Seleccione **Asignado por el sistema** > **Activado** > **Guardar**. Cuando Azure le pida confirmación, seleccione **Sí**.

   ![Activar la identidad asignada por el sistema](./media/create-managed-service-identity/enable-system-assigned-identity.png)

   > [!NOTE]
   > Si recibe un error que le indica que solo puede tener una identidad administrada, la aplicación lógica ya está asociada a la identidad asignada por el usuario. Para poder agregar la identidad asignada por el sistema, primero debe *quitar* la identidad asignada por el usuario de la aplicación lógica.

   Ahora, la aplicación lógica puede usar la identidad asignada por el sistema, que se registra con Azure AD y se representa mediante un identificador de objeto.

   ![Identificador de objeto para la identidad asignada por el sistema](./media/create-managed-service-identity/object-id-system-assigned-identity.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Id. de objeto** | <*identity-resource-ID*> | Identificador único global (GUID) que representa la identidad asignada por el sistema de la aplicación lógica en un inquilino de Azure AD. |
   ||||

1. Ahora siga los [pasos que proporcionan a la identidad acceso al recurso](#access-other-resources), especificados más adelante en este tema.

<a name="template-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-resource-manager-template"></a>Habilitar la identidad asignada por el sistema en la plantilla de Azure Resource Manager

Para automatizar la creación e implementación de los recursos de Azure, como las aplicaciones lógicas, puede usar [plantillas de Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Para habilitar la identidad administrada asignada por el sistema para la aplicación lógica en la plantilla, agregue el objeto `identity` y la propiedad secundaria `type` a la definición de recursos de la aplicación lógica en la plantilla, por ejemplo:

```json
{
   "apiVersion": "2016-06-01",
   "type": "Microsoft.logic/workflows",
   "name": "[variables('logicappName')]",
   "location": "[resourceGroup().location]",
   "identity": {
      "type": "SystemAssigned"
   },
   "properties": {
      "definition": {
         "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
         "actions": {},
         "parameters": {},
         "triggers": {},
         "contentVersion": "1.0.0.0",
         "outputs": {}
   },
   "parameters": {},
   "dependsOn": []
}
```

Cuando Azure crea la definición de recurso de la aplicación lógica, el objeto `identity` incluye estas propiedades adicionales:

```json
"identity": {
   "type": "SystemAssigned",
   "principalId": "<principal-ID>",
   "tenantId": "<Azure-AD-tenant-ID>"
}
```

| Propiedad (JSON) | Value | Descripción |
|-----------------|-------|-------------|
| `principalId` | <*principal-ID*> | El identificador único global (GUID) del objeto de entidad de servicio para la identidad administrada que representa la aplicación lógica en el inquilino de Azure AD. Este GUID aparece a veces como "Id. de objeto" o `objectID`. |
| `tenantId` | <*Azure-AD-tenant-ID*> | El identificador único global (GUID) que representa un inquilino de Azure AD del que la aplicación lógica es ahora miembro. En el inquilino de Azure AD, la entidad de servicio tiene el mismo nombre que la instancia de aplicación lógica. |
||||

<a name="user-assigned"></a>

### <a name="enable-user-assigned-identity"></a>Habilitar la identidad asignada por el usuario

Para configurar una identidad administrada asignada por el usuario para su aplicación lógica, primero debe crear esa identidad como un recurso de Azure independiente distinto. Estas son las opciones que puede usar:

* [Azure Portal](#azure-portal-user-identity)
* [Plantillas del Administrador de recursos de Azure](#template-user-identity)
* Azure PowerShell
  * [Crear una identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
  * [Agregar asignación de roles](../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md)
* Azure CLI
  * [Crear una identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
  * [Agregar asignación de roles](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md)
* API REST de Azure
  * [Crear una identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)
  * [Agregar asignación de roles](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-user-identity"></a>

#### <a name="create-user-assigned-identity-in-the-azure-portal"></a>Crear una identidad asignada por el usuario en Azure Portal

1. En [Azure Portal](https://portal.azure.com), en el cuadro de búsqueda de cualquier página, escriba `managed identities` y seleccione **Identidades administradas**.

   ![Buscar y seleccionar "Identidades administradas"](./media/create-managed-service-identity/find-select-managed-identities.png)

1. En **Identidades administradas**, seleccione **Agregar**.

   ![Agregar una identidad administrada nueva](./media/create-managed-service-identity/add-user-assigned-identity.png)

1. Proporcione información sobre su identidad administrada y, a continuación, seleccione **Revisar y crear**; por ejemplo:

   ![Creación de una identidad administrada asignada por el usuario](./media/create-managed-service-identity/create-user-assigned-identity.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Suscripción** | Sí | <*Azure-subscription-name*> | Nombre de la suscripción a Azure que se va a usar |
   | **Grupos de recursos** | Sí | <*nombre del grupo de recursos de Azure*> | Nombre del grupo de recursos que se va a utilizar. Cree un nuevo grupo o seleccione uno existente. En este ejemplo se crea un grupo denominado `fabrikam-managed-identities-RG`. |
   | **Región** | Sí | <*Azure-region*> | Región de Azure en la que se va a almacenar la información sobre el recurso. En este ejemplo se utiliza "Oeste de EE. UU.". |
   | **Nombre** | Sí | <*user-assigned-identity-name*> | Nombre que se va a asignar a la identidad asignada por el usuario. En este ejemplo se usa `Fabrikam-user-assigned-identity`. |
   |||||

   Después de validar estos detalles, Azure crea la identidad administrada. Ahora puede agregar la identidad asignada por el usuario a la aplicación lógica. Puede agregar más de una identidad asignada por el usuario a la aplicación lógica.

1. En Azure Portal, busque y abra la aplicación lógica en el Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Identidad** y, a continuación, **Usuario asignado** > **Agregar**.

   ![Agregar una identidad administrada asignada por el usuario](./media/create-managed-service-identity/add-user-assigned-identity-logic-app.png)

1. En el panel **Agregar identidad administrada asignada por el usuario**, en la lista **Suscripción**, seleccione su suscripción a Azure si aún no está seleccionada. En la lista que muestra *todas* las identidades administradas de esa suscripción, busque y seleccione la identidad asignada por el usuario que desee. Para filtrar la lista, en el cuadro de búsqueda **Identidades administradas asignadas por el usuario**, escriba el nombre de la identidad o el grupo de recursos. Cuando finalice, seleccione **Agregar**.

   ![Seleccionar la identidad asignada por el usuario que se va a usar](./media/create-managed-service-identity/select-user-assigned-identity.png)

   > [!NOTE]
   > Si recibe un error que le indica que solo puede tener una identidad administrada, la aplicación lógica ya está asociada a la identidad asignada por el sistema. Para poder agregar la identidad asignada por el usuario, primero debe deshabilitar la identidad asignada por el sistema en la aplicación lógica.

   La aplicación lógica ahora está asociada a la identidad administrada asignada por el usuario.

   ![Asociación con una identidad asignada por el usuario](./media/create-managed-service-identity/added-user-assigned-identity.png)

1. Ahora siga los [pasos que proporcionan a la identidad acceso al recurso](#access-other-resources), especificados más adelante en este tema.

<a name="template-user-identity"></a>

#### <a name="create-user-assigned-identity-in-an-azure-resource-manager-template"></a>Crear una identidad asignada por el usuario en la plantilla de Azure Resource Manager

Para automatizar la creación e implementación de los recursos de Azure, tales como las aplicaciones lógicas, puede usar [plantillas de Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md) que admiten [identidades asignadas por el usuario para la autenticación](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md). En la sección `resources` de la plantilla, la definición de recursos de la aplicación lógica requiere estos elementos:

* Un objeto `identity` con la propiedad `type` establecida en `UserAssigned`.

* Objeto secundario `userAssignedIdentities` que especifica el nombre y el recurso asignados por el usuario.

En este ejemplo se muestra una definición de recurso de aplicación lógica para una solicitud HTTP PUT y se incluye un objeto `identity` no parametrizado. La respuesta a la solicitud PUT y la operación GET subsiguiente también tienen este objeto `identity`:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicappName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user-assigned-identity-name>": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": []
      },
   ],
   "outputs": {}
}
```

Si la plantilla también incluye la definición de recursos de la identidad administrada, puede parametrizar el objeto `identity`. En este ejemplo se muestra cómo el objeto secundario `userAssignedIdentities` hace referencia a una variable `userAssignedIdentity` que se define en la sección `variables` de la plantilla. Esta variable hace referencia al identificador de recurso de la identidad asignada por el usuario.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "Template_LogicAppName": {
         "type": "string"
      },
      "Template_UserAssignedIdentityName": {
         "type": "securestring"
      }
   },
   "variables": {
      "logicAppName": "[parameters(`Template_LogicAppName')]",
      "userAssignedIdentityName": "[parameters('Template_UserAssignedIdentityName')]"
   },
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicAppName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": [
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]"
         ]
      },
      {
         "apiVersion": "2018-11-30",
         "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
         "name": "[parameters('Template_UserAssignedIdentityName')]",
         "location": "[resourceGroup().location]",
         "properties": {}
      }
  ]
}
```

<a name="access-other-resources"></a>

## <a name="give-identity-access-to-resources"></a>Conceder acceso de identidad a los recursos

Para poder usar la identidad administrada de la aplicación lógica para la autenticación, configure el acceso para esa identidad en el recurso de Azure en el que planea usar la identidad. Para completar esta tarea, asigne el rol adecuado a esa identidad en el recurso de Azure de destino. Estas son las opciones que puede usar:

* [Azure Portal](#azure-portal-assign-access)
* [Plantilla de Azure Resource Manager](../role-based-access-control/role-assignments-template.md)
* Azure PowerShell ([New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment)): para más información, vea [Incorporación o eliminación de asignaciones de roles con RBAC de Azure y Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).
* CLI de Azure ([az role assignment create](/cli/azure/role/assignment#az_role_assignment_create)): para más información, vea [Incorporación o eliminación de asignaciones de roles con RBAC de Azure y la CLI de Azure](../role-based-access-control/role-assignments-cli.md).
* [API de REST de Azure](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-assign-access"></a>

### <a name="assign-access-in-the-azure-portal"></a>Asignación del acceso en Azure Portal

En el recurso de Azure de destino al que quiere que la identidad administrada tenga acceso, asigne a esa identidad acceso basado en rol al recurso de destino.

1. En el [Azure Portal](https://portal.azure.com), vaya al recurso de Azure en el que quiere asignar acceso para la identidad administrada.

1. En el menú del recurso, seleccione **Control de acceso (IAM)**  > **asignaciones de roles** donde pueda revisar las asignaciones de roles actuales para ese recurso. En la barra de herramientas, seleccione **Agregar** > **Agregar asignación de roles**.

   ![Seleccione "Agregar" > "Agregar asignación de roles"](./media/create-managed-service-identity/add-role-to-resource.png)

   > [!TIP]
   > Si la opción **Agregar asignación de roles** está deshabilitada, lo más probable es que no tenga permisos. Para más información acerca de los permisos que le permiten administrar roles para los recursos, consulte [Permisos de roles de administrador en Azure Active Directory](../active-directory/roles/permissions-reference.md).

1. En **Agregar asignación de roles**, seleccione un **Rol** que proporcione a su identidad el acceso necesario al recurso de destino.

   En el ejemplo de este tema, su identidad necesita un [rol que pueda obtener acceso al blob en un contenedor de Azure Storage](../storage/common/storage-auth-aad.md#assign-azure-roles-for-access-rights), por lo que debe seleccionar el rol de **Colaborador de datos del blob de almacenamiento** para la identidad administrada.

   ![Seleccione rol "Colaborador de datos de blobs de almacenamiento"](./media/create-managed-service-identity/select-role-for-identity.png)

1. Siga estos pasos para su identidad administrada:

   * **Identidad asignada por el sistema**

     1. En el cuadro **Asignar acceso a**, seleccione **Aplicación lógica**. Cuando aparezca la propiedad **Suscripción**, seleccione la suscripción a Azure asociada a su identidad.

        ![Seleccione el acceso para la identidad asignada por el sistema](./media/create-managed-service-identity/assign-access-system.png)

     1. En el cuadro **Seleccionar**, elija la aplicación lógica en la lista. Si la lista es demasiado larga, use el cuadro **Seleccionar** para filtrarla.

        ![Seleccione la aplicación lógica para la identidad asignada por el sistema](./media/create-managed-service-identity/add-permissions-select-logic-app.png)

   * **Identidad asignada por el usuario**

     1. En el cuadro **Asignar acceso a**, seleccione **Identidad administrada asignada por el usuario**. Cuando aparezca la propiedad **Suscripción**, seleccione la suscripción a Azure asociada a su identidad.

        ![Seleccionar el acceso para la identidad asignada por el usuario](./media/create-managed-service-identity/assign-access-user.png)

     1. En el cuadro **Seleccionar**, elija la identidad en la lista. Si la lista es demasiado larga, use el cuadro **Seleccionar** para filtrarla.

        ![Seleccionar la identidad asignada por el usuario](./media/create-managed-service-identity/add-permissions-select-user-assigned-identity.png)

1. Cuando finalice, seleccione **Guardar**.

   La lista de asignaciones de roles del recurso de destino muestra ahora la identidad administrada y el rol administrados. En este ejemplo se muestra cómo usar la identidad asignada por el sistema para una aplicación lógica y una identidad asignada por el usuario para un grupo de aplicaciones lógicas diferentes.

   ![Se han agregado las identidades y roles administrados al recurso de destino](./media/create-managed-service-identity/added-roles-for-identities.png)

   Para más información, consulte [Asignación de acceso de una identidad administrada a un recurso mediante Azure Portal](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md).

1. Ahora siga los [pasos para autenticar el acceso con la identidad](#authenticate-access-with-identity) en un desencadenador o una acción que admita identidades administradas.

<a name="authenticate-access-with-identity"></a>

## <a name="authenticate-access-with-managed-identity"></a>Autenticación del acceso con una identidad administrada

Después de [habilitar la identidad administrada para la aplicación lógica](#azure-portal-system-logic-app) y [conceder a esa identidad acceso al recurso o a la entidad de destino](#access-other-resources), puede usar esa identidad en [desencadenadores y acciones que admiten identidades administradas](logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

> [!IMPORTANT]
> Antes de que una función de Azure pueda usar la identidad asignada por el sistema, primero [habilite la autenticación de Azure Functions](../logic-apps/logic-apps-azure-functions.md#enable-authentication-for-functions).

En estos pasos se muestra cómo usar la identidad administrada con un desencadenador o una acción a través del Azure Portal. Para especificar la identidad administrada en la definición de JSON subyacente de un desencadenador o una acción, consulte [Autenticación de identidad administrada](../logic-apps/logic-apps-securing-a-logic-app.md#managed-identity-authentication).

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. Si todavía no lo ha hecho, agregue el [desencadenador o la acción que admita identidades administradas](logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

   > [!NOTE]
   > No todos los desencadenadores y las acciones permiten agregar un tipo de autenticación. Para más información, consulte [Tipos de autenticación para desencadenadores y acciones que admiten la autenticación](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

1. Siga estos pasos en el desencadenador o en la acción que agregó:

   * **Acciones y desencadenadores integrados que admiten el uso de una identidad administrada**

     1. Agregue la propiedad **Autenticación** si todavía no aparece la propiedad.

     1. En **Tipo de autenticación**, seleccione **Identidad administrada**.

     Para más información, vea [Ejemplo: Autentique una acción o un desencadenador integrado con una identidad administrada](#authenticate-built-in-managed-identity).
 
   * **Acciones y desencadenadores de conector administrado que admiten el uso de una identidad administrada**

     1. En la página de selección de inquilinos, seleccione **Conexión con identidad administrada**.

     1. En la página siguiente, proporcione un nombre de conexión.

        De manera predeterminada, la lista de identidades administradas muestra solo la identidad administrada actualmente habilitada porque una aplicación lógica solo admite la habilitación de una identidad administrada a la vez, por ejemplo:

        ![Captura de pantalla que muestra la página de nombre de conexión y la identidad administrada seleccionada.](./media/create-managed-service-identity/system-assigned-managed-identity.png)

     Para más información, vea [Ejemplo: Autentique una acción o un desencadenador de conector administrado con una identidad administrada](#authenticate-managed-connector-managed-identity).

<a name="authenticate-built-in-managed-identity"></a>

#### <a name="example-authenticate-built-in-trigger-or-action-with-a-managed-identity"></a>Ejemplo: Autentique una acción o un desencadenador integrado con una identidad administrada

El desencadenador o la acción HTTP pueden usar la identidad asignada por el sistema que habilitó para la aplicación lógica. En general, el desencadenador o la acción HTTP utiliza estas propiedades para especificar el recurso o la entidad a los que desea obtener acceso:

| Propiedad | Obligatorio | Descripción |
|----------|----------|-------------|
| **Método** | Sí | El método HTTP usado por la operación que desea ejecutar |
| **URI** | Sí | La dirección URL del punto de conexión para acceder a la entidad o recurso de Azure de destino. La sintaxis del URI normalmente incluye el [Id. de recurso](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) para el recurso o servicio de Azure. |
| **Encabezados** | No | Los valores de encabezado que necesita o desea incluir en la solicitud saliente, como el tipo de contenido |
| **Consultas** | No | Los parámetros de consulta que necesita o desea incluir en la solicitud, como el parámetro para una operación específica o la versión de la API para la operación que desea ejecutar |
| **Autenticación** | Sí | El tipo de autenticación que se va a usar para autenticar el acceso al recurso o entidad de destino |
||||

Como ejemplo específico, supongamos que desea ejecutar la [Operación instantánea de blob](/rest/api/storageservices/snapshot-blob) en un BLOB de la cuenta Azure Storage en la que previamente configuró el acceso para su identidad. Sin embargo, el [conector de Azure Blob Storage](/connectors/azureblob/) no ofrece actualmente esta operación. En su lugar, puede ejecutar esta operación mediante la [acción HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action) u otra [operación de la API REST de servicios blob](/rest/api/storageservices/operations-on-blobs).

> [!IMPORTANT]
> Para acceder a cuentas de Azure Storage detrás de firewalls mediante solicitudes HTTP e identidades administradas, asegúrese de que también configura la cuenta de almacenamiento con la [excepción que permite el acceso a los servicios de Microsoft de confianza](../connectors/connectors-create-api-azureblobstorage.md#access-trusted-service).

Para ejecutar la [operación blob de instantáneas](/rest/api/storageservices/snapshot-blob), la acción HTTP especifica estas propiedades:

| Propiedad | Obligatorio | Valor de ejemplo | Descripción |
|----------|----------|---------------|-------------|
| **Método** | Sí | `PUT`| El método HTTP que utiliza la operación blob de instantáneas |
| **URI** | Sí | `https://{storage-account-name}.blob.core.windows.net/{blob-container-name}/{folder-name-if-any}/{blob-file-name-with-extension}` | El identificador de recurso de un archivo Azure Blob Storage en el entorno de Azure Global (público), que usa esta sintaxis |
| **Encabezados** | Para Azure Storage | `x-ms-blob-type` = `BlockBlob` <p>`x-ms-version` = `2019-02-02` <p>`x-ms-date` = `@{formatDateTime(utcNow(),'r')}` | Los valores de encabezado `x-ms-blob-type`, `x-ms-version` y `x-ms-date` son necesarios para las operaciones de Azure Storage. <p><p>**Importante**: En el desencadenador HTTP saliente y en las solicitudes de acción de Azure Storage, el encabezado requiere la propiedad `x-ms-version` y la versión de la API para la operación que se desea ejecutar. El valor `x-ms-date` debe ser la fecha actual. En caso contrario, se produce un error de tipo `403 FORBIDDEN` en la aplicación lógica. Para obtener la fecha actual en el formato requerido, puede usar la expresión del valor de ejemplo. <p>Para más información, consulte los temas siguientes: <p><p>- [Encabezados de solicitud - Blob de instantáneas](/rest/api/storageservices/snapshot-blob#request) <br>- [Control de versiones para servicios de Azure Storage](/rest/api/storageservices/versioning-for-the-azure-storage-services#specifying-service-versions-in-requests) |
| **Consultas** | Solo para la operación del blob de instantáneas | `comp` = `snapshot` | El nombre y el valor del parámetro de consulta para la operación. |
|||||

Este es un ejemplo de la acción HTTP que muestra todos estos valores de propiedad:

![Incorporación de una acción HTTP para acceder a un recurso de Azure](./media/create-managed-service-identity/http-action-example.png)

1. Después de agregar la acción HTTP, agregue la propiedad **Autenticación** a la acción HTTP. En la lista **Agregar nuevo parámetro**, seleccione **Autenticación**.

   ![Agregar la propiedad "Autenticación" a la acción HTTP](./media/create-managed-service-identity/add-authentication-property.png)

   > [!NOTE]
   > No todos los desencadenadores y las acciones permiten agregar un tipo de autenticación. Para más información, consulte [Tipos de autenticación para desencadenadores y acciones que admiten la autenticación](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

1. En la lista **Tipo de autenticación**, seleccione **Identidad administrada**.

   ![En "Autenticación", seleccionar "Identidad administrada"](./media/create-managed-service-identity/select-managed-identity.png)

1. En la lista identidades administradas, seleccione entre las opciones disponibles en función de su escenario.

   * Si configura la identidad asignada por el sistema, seleccione **Identidad administrada asignada por el sistema** si aún no está seleccionada.

     ![Seleccionar "Identidad administrada asignada por el sistema"](./media/create-managed-service-identity/select-system-assigned-identity-for-action.png)

   * Si configura una identidad asignada por el sistema, seleccione esa identidad si aún no lo está.

     ![Seleccionar la identidad asignada por el usuario](./media/create-managed-service-identity/select-user-assigned-identity-for-action.png)

   Este ejemplo continúa con la **Identidad administrada asignada por el sistema**.

1. En algunos desencadenadores y acciones, la propiedad **Audiencia** también aparece para que pueda establecer el identificador de recurso de destino. Establezca la propiedad **Audiencia** en el [identificador de recurso para el recurso o servicio de destino](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication). De lo contrario, de forma predeterminada, la propiedad **Audiencia** usa el identificador de recurso `https://management.azure.com/`, que es el identificador de recurso para Azure Resource Manager.

   > [!IMPORTANT]
   > Asegúrese de que el identificador de este recurso de destino *coincide exactamente* con el valor esperado en Azure Active Directory (AD), incluida toda barra diagonal necesaria al final. Por ejemplo, el Id. de recurso para todas las cuentas de Azure Blob Storage requiere una barra diagonal final. Sin embargo, el Id. de recurso de una cuenta de almacenamiento específica no requiere una barra diagonal final. Verifique los [identificadores de recursos para los servicios de Azure que admiten Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

   En este ejemplo se establece la propiedad **Audiencia** en `https://storage.azure.com/`, de modo que los tokens de acceso que se usan para la autenticación sean válidos para todas las cuentas de almacenamiento. Sin embargo, también puede especificar una dirección URL de servicio raíz, `https://fabrikamstorageaccount.blob.core.windows.net`, para una cuenta de almacenamiento específica.

   ![Especificar la propiedad "Audiencia" en el identificador de recurso de destino](./media/create-managed-service-identity/specify-audience-url-target-resource.png)

   Para obtener más información sobre cómo autorizar el acceso con Azure AD para Azure Storage, consulte estos temas:

   * [Autorización del acceso a blobs y colas de Azure con Azure Active Directory](../storage/common/storage-auth-aad.md)
   * [Autorización del acceso a Azure Storage con Azure Active Directory](/rest/api/storageservices/authorize-with-azure-active-directory#use-oauth-access-tokens-for-authentication)

1. Siga creando la aplicación lógica de la forma que desee.

<a name="authenticate-managed-connector-managed-identity"></a>

#### <a name="example-authenticate-managed-connector-trigger-or-action-with-a-managed-identity"></a>Ejemplo: Autentique una acción o un desencadenador de conector administrado con una identidad administrada

La acción de Azure Resource Manager, **Leer un recurso**, puede usar la identidad administrada que habilitó para la aplicación lógica. En este ejemplo se muestra cómo usar la identidad administrada asignada por el sistema.

1. Después de agregar la acción al flujo de trabajo, en la página selección de inquilinos, seleccione **Conexión con identidad administrada**.

   ![Captura de pantalla que muestra la acción de Azure Resource Manager y la opción "Conexión con identidad administrada" seleccionada.](./media/create-managed-service-identity/select-connect-managed-identity.png)

   La acción ahora muestra la página de nombre de conexión con la lista de identidades administradas, que incluye el tipo de identidad administrada que está habilitado actualmente en la aplicación lógica.

1. En la página de nombre de la conexión, escriba un nombre para la conexión. En la lista de identidades administradas, seleccione la identidad administrada, que en este ejemplo es la **Identidad administrada asignada por el sistema**, y seleccione **Crear**. Por el contrario, si habilitó una identidad administrada asignada por el usuario, seleccione esa.

   ![Captura de pantalla que muestra la acción de Azure Resource Manager con el nombre de conexión especificado y la opción "Identidad administrada asignada por el sistema" seleccionada.](./media/create-managed-service-identity/system-assigned-managed-identity.png)

   Si la identidad administrada no está habilitada, aparece el error siguiente al intentar crear la conexión:

   *You must enable managed identity for your logic app and then grant required access to the identity in the target resource* (Debe habilitar la identidad administrada para la aplicación lógica y luego conceder el acceso necesario a la identidad en el recurso de destino).

   ![Captura de pantalla que muestra la acción de Azure Resource Manager con error cuando no hay habilitada ninguna identidad administrada.](./media/create-managed-service-identity/system-assigned-managed-identity-disabled.png)

1. Después de crear correctamente la conexión, el diseñador puede usar la autenticación de identidad administrada para capturar cualquier esquema, contenido o valores dinámicos.

1. Siga creando la aplicación lógica de la forma que desee.

<a name="logic-app-resource-definition-connection-managed-identity"></a>

### <a name="logic-app-resource-definition-and-connections-that-use-a-managed-identity"></a>Definición de recursos de la aplicación lógica y conexiones que usan una identidad administrada

Una conexión que permite y usa una identidad administrada es un tipo de conexión especial que solo funciona con una identidad administrada. En tiempo de ejecución, la conexión usa la identidad administrada que está habilitada en la aplicación lógica. Esta configuración se guarda en el objeto `parameters` de la definición de recursos de la aplicación lógica, que contiene el objeto `$connections` que incluye punteros al identificador de recurso de la conexión junto con el identificador de recurso de la identidad, si la identidad asignada por el usuario está habilitada.

En este ejemplo se muestra el aspecto de la configuración cuando la aplicación lógica habilita la identidad administrada asignada por el sistema:

```json
"parameters": {
   "$connections": {
      "value": {
         "<action-name>": {
            "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/connections/{connection-name}",
            "connectionName": "{connection-name}",
            "connectionProperties": {
               "authentication": {
                  "type": "ManagedServiceIdentity"
               }
            },
            "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{managed-connector-type}"
         }
      }
   }
}
```

En este ejemplo se muestra el aspecto de la configuración cuando la aplicación lógica habilita una identidad administrada asignada por el usuario:

```json
"parameters": {
   "$connections": {
      "value": {
         "<action-name>": {
            "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/connections/{connection-name}",
            "connectionName": "{connection-name}",
            "connectionProperties": {
               "authentication": {
                  "identity": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/microsoft.managedidentity/userassignedidentities/{managed-identity-name}",
                  "type": "ManagedServiceIdentity"
               }
            },
            "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{managed-connector-type}"
         }
      }
   }
}
```

Durante el tiempo de ejecución, el servicio Logic Apps comprueba si las acciones o los desencadenadores de conector administrado de la aplicación lógica están configurados para usar la identidad administrada y si todos los permisos necesarios están configurados para usar la identidad administrada a fin de acceder a los recursos de destino especificados por el desencadenador y las acciones. Si la comprobación se completa correctamente, el servicio Logic Apps recupera el token de Azure AD que está asociado a la identidad administrada y usa dicha identidad para autenticar el acceso al recurso de destino y realizar la operación configurada en desencadenadores y acciones.

<a name="arm-templates-connection-resource-managed-identity"></a>

## <a name="arm-template-for-managed-connections-and-managed-identities"></a>Plantilla de ARM para conexiones e identidades administradas

Si automatiza la implementación con una plantilla de ARM y la aplicación lógica incluye un desencadenador o una acción de conector administrado que usa una identidad administrada, confirme que la definición de recurso de conexión subyacente incluye la propiedad `parameterValueType` con `Alternative` como valor de propiedad. De lo contrario, la implementación de ARM no configurará la conexión de uso de la identidad administrada para la autenticación y la conexión no funcionará en el flujo de trabajo de la aplicación lógica. Este requisito solo se aplica a [desencadenadores y acciones de conectores administrados específicos](#managed-connectors-managed-identity) en los que se ha seleccionado la [opción **Conectar con identidad administrada**](#authenticate-managed-connector-managed-identity).

Por ejemplo, esta es la definición de recurso de conexión subyacente para una acción Azure Automation que usa una identidad administrada en la que la definición incluye la propiedad `parameterValueType`, que se establece en `Alternative` como el valor de la propiedad:

```json
{
    "type": "Microsoft.Web/connections",
    "name": "[variables('automationAccountApiConnectionName')]",
    "apiVersion": "2016-06-01",
    "location": "[parameters('location')]",
    "kind": "V1",
    "properties": {
        "api": {
            "id": "[subscriptionResourceId('Microsoft.Web/locations/managedApis', parameters('location'), 'azureautomation')]"
        },
        "customParameterValues": {},
        "displayName": "[variables('automationAccountApiConnectionName')]",
        "parameterValueType": "Alternative"
    }
},
```

<a name="remove-identity"></a>

## <a name="disable-managed-identity"></a>Deshabilitar la identidad administrada

Para dejar de usar una identidad administrada para la aplicación lógica, tiene estas opciones:

* [Azure Portal](#azure-portal-disable)
* [Plantillas del Administrador de recursos de Azure](#template-disable)
* Azure PowerShell
  * [Eliminar asignación de roles](../role-based-access-control/role-assignments-powershell.md)
  * [Eliminar la identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
* Azure CLI
  * [Eliminar asignación de roles](../role-based-access-control/role-assignments-cli.md)
  * [Eliminar la identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
* API REST de Azure
  * [Eliminar asignación de roles](../role-based-access-control/role-assignments-rest.md)
  * [Eliminar la identidad asignada por el usuario](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)

Si se elimina la aplicación lógica, Azure quita automáticamente de Azure AD la identidad administrada.

<a name="azure-portal-disable"></a>

### <a name="disable-managed-identity-in-the-azure-portal"></a>Deshabilitar la identidad administrada en Azure Portal

En Azure Portal, quite primero el acceso de la identidad a [su recurso de destino](#disable-identity-target-resource). A continuación, desactive la identidad asignada por el sistema o quite la identidad asignada por el usuario de [su aplicación lógica](#disable-identity-logic-app).

<a name="disable-identity-target-resource"></a>

#### <a name="remove-identity-access-from-resources"></a>Quitar acceso de identidad a los recursos

1. En [Azure Portal](https://portal.azure.com), vaya al recurso de Azure de destino donde quiere quitar el acceso de la identidad administrada.

1. En el menú del recurso de destino, seleccione **Control de acceso (IAM)** . En la barra de herramientas, seleccione **Asignación de roles**.

1. En la lista de roles, seleccione las identidades administradas que desea quitar. En la barra de herramientas, seleccione **Eliminar**.

   > [!TIP]
   > Si está deshabilitada la opción **Eliminar**, lo más probable es que no tenga los permisos correspondientes. Para más información acerca de los permisos que le permiten administrar roles para los recursos, consulte [Permisos de roles de administrador en Azure Active Directory](../active-directory/roles/permissions-reference.md).

La identidad administrada se ha quitado y ya no tiene acceso al recurso de destino.

<a name="disable-identity-logic-app"></a>

#### <a name="disable-managed-identity-on-logic-app"></a>Deshabilitar la identidad administrada en la aplicación lógica

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Identidad** y, a continuación, siga los pasos adecuados para su identidad:

   * Seleccione **Asignado por el sistema** > **Activado** > **Guardar**. Cuando Azure le pida confirmación, seleccione **Sí**.

     ![Deshabilitar la identidad asignada por el sistema](./media/create-managed-service-identity/disable-system-assigned-identity.png)

   * Seleccione **Usuario asignado** y la identidad administrada y, después, elija **Quitar**. Cuando Azure le pida confirmación, seleccione **Sí**.

     ![Quitar la identidad asignada por el usuario](./media/create-managed-service-identity/remove-user-assigned-identity.png)

La identidad administrada ahora está deshabilitada en la aplicación lógica.

<a name="template-disable"></a>

### <a name="disable-managed-identity-in-azure-resource-manager-template"></a>Deshabilitar la identidad administrada con una plantilla de Azure Resource Manager

Si ha creado la identidad administrada de la aplicación lógica mediante una plantilla de Azure Resource Manager, establezca la propiedad `type` del objeto `identity` en `None`.

```json
"identity": {
   "type": "None"
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Proteger el acceso y los datos en Azure Logic Apps](../logic-apps/logic-apps-securing-a-logic-app.md)
