---
title: Sincronización de atributos en Azure Active Directory para asignación
description: Al configurar el aprovisionamiento de usuarios con Azure Active Directory y aplicaciones SaaS, use la característica de extensión de directorio para agregar atributos de origen que no están sincronizados de manera predeterminada.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 03/31/2021
ms.author: kenwith
ms.openlocfilehash: f7a2429161cebe867d844b4ca7aa08ec3613edcd
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388217"
---
# <a name="syncing-extension-attributes-for-app-provisioning"></a>Sincronización de atributos de extensión para el aprovisionamiento de aplicaciones

Azure Active Directory (Azure AD) debe contener todos los datos (atributos) necesarios para crear un perfil de usuario al aprovisionar cuentas de usuario de Azure AD en una [aplicación SaaS](../saas-apps/tutorial-list.md). Al personalizar las asignaciones de atributos para el aprovisionamiento de usuarios, es posible que el atributo que quiera asignar no aparezca en la lista **Atributo de origen**. En este artículo se muestra cómo agregar el atributo que falta.

Para los usuarios solo en Azure AD, puede [crear extensiones de esquema mediante PowerShell o Microsoft Graph](#create-an-extension-attribute-on-a-cloud-only-user).

En el caso de los usuarios de Active Directory local, debe sincronizar los usuarios con Azure AD. Puede sincronizar usuarios y atributos mediante [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md). Azure AD Connect sincroniza automáticamente determinados atributos en Azure AD, pero no todos. Además, es posible que algunos atributos (por ejemplo, SAMAccountName) que se sincronizan de manera predeterminada no puedan exponerse a través de Azure AD Graph API. En estos casos, puede [usar la característica de extensión de directorio de Azure AD Connect para sincronizar el atributo en Azure AD](#create-an-extension-attribute-using-azure-ad-connect). De este modo, el atributo será visible para Azure AD Graph API y el servicio de aprovisionamiento de Azure AD.

## <a name="create-an-extension-attribute-on-a-cloud-only-user"></a>Creación de un atributo de extensión en un usuario solo en la nube
Puede usar Microsoft Graph y PowerShell a fin de ampliar el esquema de usuario para los usuarios de Azure AD. Estos atributos de extensión se detectan automáticamente en la mayoría de los casos.

Si tiene más de 1000 entidades de servicio, puede encontrar extensiones en la lista atributo de origen. Si un atributo que ha creado no aparece automáticamente, compruebe que se ha creado el atributo y agréguelo manualmente al esquema. Para comprobar que se ha creado, use Microsoft Graph y [Probador de Graph](/graph/graph-explorer/graph-explorer-overview). Para agregarlo manualmente al esquema, consulte [Edición de la lista de atributos admitidos](customize-application-attributes.md#editing-the-list-of-supported-attributes).

### <a name="create-an-extension-attribute-on-a-cloud-only-user-using-microsoft-graph"></a>Creación de un atributo de extensión en un usuario solo en la nube mediante Microsoft Graph
Puede extender el esquema de usuarios de Azure AD mediante [Microsoft Graph](/graph/overview). 

En primer lugar, enumere las aplicaciones de su inquilino para obtener el identificador de la aplicación en el que está trabajando. Para obtener más información, consulte la [lista extensionProperties](/graph/api/application-list-extensionproperty?view=graph-rest-1.0&tabs=http&preserve-view=true).

```json
GET https://graph.microsoft.com/v1.0/applications
```

A continuación, cree el atributo de extensión. Reemplace la propiedad **ID** siguiente por el **ID** recuperado en el paso anterior. Tendrá que usar el atributo **"ID"** en lugar de "appId". Para obtener más información, consulte [Create extensionProperty]/graph/api/application-post-extensionproperty.md?view=graph-rest-1.0&tabs=http&preserve-view=true).

```json
POST https://graph.microsoft.com/v1.0/applications/{id}/extensionProperties
Content-type: application/json

{
    "name": "extensionName",
    "dataType": "string",
    "targetObjects": [
        "User"
    ]
}
```

La solicitud anterior creó un atributo de extensión con el formato `extension_appID_extensionName`. Ahora puede actualizar un usuario con este atributo de extensión. Para obtener más información, consulte [Actualización del clúster](/graph/api/user-update?view=graph-rest-1.0&tabs=http&preserve-view=true).
```json
PATCH https://graph.microsoft.com/v1.0/users/{id}
Content-type: application/json

{
  "extension_inputAppId_extensionName": "extensionValue"
}
```
Por último, compruebe el atributo para el usuario. Para más información, consulte [Obtener un usuario](/graph/api/user-get?view=graph-rest-1.0&tabs=http#example-3-users-request-using-select&preserve-view=true).

```json
GET https://graph.microsoft.com/v1.0/users/{id}?$select=displayName,extension_inputAppId_extensionName
```


### <a name="create-an-extension-attribute-on-a-cloud-only-user-using-powershell"></a>Creación de un atributo de extensión en un usuario solo en la nube mediante PowerShell
Cree una extensión personalizada mediante PowerShell y asigne un valor a un usuario. 

```
#Connect to your Azure AD tenant   
Connect-AzureAD

#Create an application (you can instead use an existing application if you would like)
$App = New-AzureADApplication -DisplayName “test app name” -IdentifierUris https://testapp

#Create a service principal
New-AzureADServicePrincipal -AppId $App.AppId

#Create an extension property
New-AzureADApplicationExtensionProperty -ObjectId $App.ObjectId -Name “TestAttributeName” -DataType “String” -TargetObjects “User”

#List users in your tenant to determine the objectid for your user
Get-AzureADUser

#Set a value for the extension property on the user. Replace the objectid with the ID of the user and the extension name with the value from the previous step
Set-AzureADUserExtension -objectid 0ccf8df6-62f1-4175-9e55-73da9e742690 -ExtensionName “extension_6552753978624005a48638a778921fan3_TestAttributeName”

#Verify that the attribute was added correctly.
Get-AzureADUser -ObjectId 0ccf8df6-62f1-4175-9e55-73da9e742690 | Select -ExpandProperty ExtensionProperty

```

## <a name="create-an-extension-attribute-using-azure-ad-connect"></a>Creación de un atributo de extensión mediante Azure AD Connect

1. Abra el asistente de Azure AD Connect, elija Tareas y, a continuación, elija **Personalizar las opciones de sincronización**.

   ![Página de tareas adicionales del asistente de Azure Active Directory Connect](./media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-customize.png)
 
2. Inicie sesión como administrador global de Azure AD. 

3. En la página **Características opcionales**, seleccione **Sincronización de atributos de las extensiones de directorios**.
 
   ![Página de características opcionales del asistente de Azure Active Directory Connect](./media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-directory-extension-attribute-sync.png)

4. Seleccione los atributos que quiere extender a Azure AD.
   > [!NOTE]
   > La búsqueda en **Atributos disponibles** distingue mayúsculas de minúsculas.

   ![Captura de pantalla en la que se muestra la página de selección "Extensiones de directorio".](./media/user-provisioning-sync-attributes-for-mapping/active-directory-connect-directory-extensions.png)

5. Finalice el asistente de Azure AD Connect y permita que se ejecute un ciclo de sincronización completo. Cuando se complete el ciclo, se extiende el esquema y los nuevos valores se sincronizan entre AD local y Azure AD.
 
6. En Azure Portal, mientras [edita las asignaciones de atributos de usuario](customize-application-attributes.md), la lista **Atributo de origen** ahora contendrá el atributo agregado en el formato `<attributename> (extension_<appID>_<attributename>)`. Seleccione el atributo y asígnelo a la aplicación de destino para el aprovisionamiento.

   ![Página de selección de las extensiones de directorios del asistente de Azure Active Directory Connect](./media/user-provisioning-sync-attributes-for-mapping/attribute-mapping-extensions.png)

> [!NOTE]
> La capacidad de aprovisionar atributos de referencia desde AD local, como **managedby** o **DN/DistinguishedName**, no se admite actualmente. Puede solicitar esta característica en [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory). 


## <a name="next-steps"></a>Pasos siguientes

* [Definir quién está en el ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)
