---
title: 'Cambio del tipo de autenticación de subdominio mediante PowerShell y Graph: Azure Active Directory | Microsoft Docs'
description: Cambie la configuración de autenticación de subdominio predeterminada heredada de la configuración de dominio raíz en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 04/18/2021
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: f4acf01a6672d17e62b6ebf5c6f43c8d6145f95a
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739321"
---
# <a name="change-subdomain-authentication-type-in-azure-active-directory"></a>Cambio del tipo de autenticación de subdominio en Azure Active Directory

Después de agregar un dominio raíz a Azure Active Directory (Azure AD), todos los subdominios siguientes agregados a esa raíz en la organización de Azure AD heredan automáticamente la configuración de autenticación del dominio raíz. Pero si quiere administrar la configuración de autenticación de dominio independientemente de la configuración de dominio raíz, puede hacerlo con Microsoft Graph API. Por ejemplo, si tiene un dominio raíz federado como contoso.com, este artículo puede ayudarle a comprobar un subdominio como child.contoso.com como administrado en lugar de federado.

En el portal de Azure AD, si el dominio primario está federado y el administrador intenta comprobar un subdominio administrado en la página **Nombres de dominio personalizados**, se obtiene un error "Error al agregar dominio" con el motivo "Una o más propiedades contienen valores no válidos". Si intenta agregar este subdominio desde el centro de administración de Microsoft 365, recibe un error similar. Para obtener más información sobre el error, vea [Un dominio secundario no hereda los cambios de dominio primario en Office 365, Azure o Intune](/office365/troubleshoot/administration/child-domain-fails-inherit-parent-domain-changes).

## <a name="how-to-verify-a-custom-subdomain"></a>Comprobación de un subdominio personalizado

Dado que los subdominios heredan el tipo de autenticación del dominio raíz de manera predeterminada, debe promover el subdominio a un dominio raíz en Azure AD mediante Microsoft Graph para poder establecer el tipo de autenticación en el deseado.

### <a name="add-the-subdomain-and-view-its-authentication-type"></a>Incorporación del subdominio y visualización de su tipo de autenticación

1. Use PowerShell para agregar el nuevo subdominio, que tiene el tipo de autenticación predeterminado del dominio raíz. Los centros de administración de Azure AD y Microsoft 365 todavía no admiten esta operación.

   ```powershell
   New-MsolDomain -Name "child.mydomain.com" -Authentication Federated
   ```

1. Use [Azure AD Graph Explorer](https://graphexplorer.azurewebsites.net) para obtener el dominio. Dado que el dominio no es un dominio raíz, hereda el tipo de autenticación del dominio raíz. El comando y los resultados podrían tener el aspecto siguiente, con su propio identificador de inquilino:

   ```http
   GET https://graph.windows.net/{tenant_id}/domains?api-version=1.6
   
   Return:
     {
         "authenticationType": "Federated",
         "availabilityStatus": null,
         "isAdminManaged": true,
         "isDefault": false,
         "isDefaultForCloudRedirections": false,
         "isInitial": false,
         "isRoot": false,          <---------------- Not a root domain, so it inherits parent domain's authentication type (federated)
         "isVerified": true,
         "name": "child.mydomain.com",
         "supportedServices": [],
         "forceDeleteState": null,
         "state": null,
         "passwordValidityPeriodInDays": null,
         "passwordNotificationWindowInDays": null
     },
   ```

### <a name="use-azure-ad-graph-explorer-api-to-make-this-a-root-domain"></a>Uso de la API de Azure AD Graph Explorer para convertirlo en un dominio raíz

Use el siguiente comando para promover el subdominio:

```http
POST https://graph.windows.net/{tenant_id}/domains/child.mydomain.com/promote?api-version=1.6
```

### <a name="change-the-subdomain-authentication-type"></a>Cambio del tipo de autenticación del subdominio

1. Use el siguiente comando para cambiar el tipo de autenticación del subdominio:

   ```powershell
   Set-MsolDomainAuthentication -DomainName child.mydomain.com -Authentication Managed
   ```

1. Compruebe mediante GET en Azure AD Graph Explorer que el tipo de autenticación del subdominio ahora es administrado:

   ```http
   GET https://graph.windows.net/{{tenant_id} }/domains?api-version=1.6
   
   Return:
     {
         "authenticationType": "Managed",   <---------- Now this domain is successfully added as Managed and not inheriting Federated status
         "availabilityStatus": null,
         "isAdminManaged": true,
         "isDefault": false,
         "isDefaultForCloudRedirections": false,
         "isInitial": false,
         "isRoot": true,   <------------------------------ Also a root domain, so not inheriting from parent domain any longer
         "isVerified": true,
         "name": "child.mydomain.com",
         "supportedServices": [
             "Email",
             "OfficeCommunicationsOnline",
             "Intune"
         ],
         "forceDeleteState": null,
         "state": null,
         "passwordValidityPeriodInDays": null,
         "passwordNotificationWindowInDays": null }
   ```

## <a name="next-steps"></a>Pasos siguientes

- [Agregar nombres de dominio personalizados](../fundamentals/add-custom-domain.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)
- [Administración de nombres de dominio](domains-manage.md)
- [ForceDelete a custom domain name with Microsoft Graph API](/graph/api/domain-forcedelete?view=graph-rest-beta&preserve-view=true) (Uso de la operación ForceDelete en un nombre de dominio personalizado con Microsoft Graph API)