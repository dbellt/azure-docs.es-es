---
title: Protección de entidades de servicio en Azure Active Directory
description: Busque, evalúe y proteja las entidades de servicio.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 2/15/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: d8a4b97ef095f5860ddf99751186ca04926e1698
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108761748"
---
# <a name="securing-service-principals"></a>Protección de entidades de servicio

Una [entidad de servicio](../develop/app-objects-and-service-principals.md) de Azure Active Directory (Azure AD) es la representación local de un objeto de aplicación en un único inquilino o directorio.  Funciona como la identidad de la instancia de la aplicación. Las entidades de servicio definen quién puede tener acceso a la aplicación y a qué recursos puede tener acceso la aplicación. La entidad de servicio se crea en cada inquilino donde se usa la aplicación y hace referencia al objeto de aplicación único globalmente. El inquilino protege el inicio de sesión y el acceso a los recursos de la entidad de servicio.  

### <a name="tenant-service-principal-relationships"></a>Relaciones de entidad de servicio del inquilino
Una aplicación de un solo inquilino solamente tiene una entidad de servicio en su inquilino principal. Una API o aplicación web multiinquilino requiere una entidad de servicio en cada inquilino. Una entidad de servicio se crea cuando un usuario de ese inquilino ha dado su consentimiento al uso de la aplicación o de la API. Este consentimiento crea una relación de uno a varios entre la aplicación multiinquilino y sus entidades de servicio asociadas.

Una aplicación multiinquilino se hospeda en un solo inquilino y está diseñada para tener instancias en otros inquilinos. La mayoría de las aplicaciones de software como servicio (SaaS) están diseñadas para usar varios inquilinos. Use entidades de servicio para garantizar la postura de seguridad adecuada para la aplicación y sus usuarios, tanto en casos de uso de un solo inquilino como de varios inquilinos.

## <a name="applicationid-and-objectid"></a>ApplicationID y ObjectID

Una instancia de aplicación determinada tiene dos propiedades distintas: ApplicationID (también conocida como ClientID) y ObjectID.

> [!NOTE] 
> Es posible que los términos aplicación y entidad de servicio se utilicen indistintamente cuando se refieran de manera general a una aplicación en el contexto de las tareas relacionadas con la autenticación. Sin embargo, son dos representaciones diferentes de aplicaciones en Azure AD.
 

El valor ApplicationID representa la aplicación global y es igual para todas las instancias de la aplicación en todos los inquilinos. El valor ObjectID es único para un objeto de aplicación. Al igual que con usuarios, grupos y otros recursos, el valor ObjectID ayuda a identificar de forma exclusiva una instancia de aplicación en Azure AD.

Para obtener información más detallada sobre este tema, consulte [Relación entre los objetos de aplicación y las entidades de servicio](../develop/app-objects-and-service-principals.md).

También puede crear una aplicación y su objeto de entidad de servicio (ObjectID) en un inquilino mediante Azure PowerShell, la CLI de Azure, Microsoft Graph, Azure Portal y otras herramientas. 

![Captura de pantalla que muestra un nuevo registro de aplicación, con los campos del identificador de aplicación y del identificador de objeto resaltados.](./media/securing-service-accounts/secure-principal-image-1.png)

## <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Existen dos mecanismos para la autenticación con entidades de servicio: certificados de cliente y secretos de cliente. 

![ Captura de pantalla de la página de nueva aplicación que muestra las áreas de certificados y de secretos de cliente resaltadas.](./media/securing-service-accounts/secure-principal-certificates.png)

Los certificados son más seguros: use certificados de cliente si es posible. A diferencia de los secretos de cliente, los certificados de cliente no se pueden insertar accidentalmente en el código. Use Azure Key Vault para la administración de certificados y secretos cuando sea posible para cifrar los siguientes recursos mediante claves protegidas por módulos de seguridad de hardware:

* claves de autenticación

* claves de cuenta de almacenamiento

* claves de cifrado de datos

* Archivos .pfx

* contraseñas 

Para obtener más información acerca de Azure Key Vault y cómo usarlo para la administración de certificados y secretos, consulte [Acerca de Azure Key Vault](../../key-vault/general/overview.md) y [Asignación de una directiva de acceso de Key Vault mediante Azure Portal](../../key-vault/general/assign-access-policy-portal.md). 

 ### <a name="challenges-and-mitigations"></a>Desafíos y mitigaciones
En la tabla siguiente se presentan las mitigaciones a los desafíos que puede encontrar al usar entidades de seguridad de servicio.


| Desafíos| Mitigaciones |
| - | - |
| Revisiones de acceso para las entidades de servicio asignadas a roles con privilegios.| Esta funcionalidad está en versión preliminar y aún no está ampliamente disponible. |
| Acceso de las entidades de servicio de las revisiones| Comprobación manual de la lista de control de acceso de recursos mediante Azure Portal. |
| Entidades de servicio con más permisos de los previstos| Cuando cree entidades de servicio o cuentas de servicio de automatización, proporcione solo los permisos necesarios para la tarea. Evalúe las entidades de servicio existentes para ver si puede reducir los privilegios. |
|Identificación de las modificaciones de las credenciales o los métodos de autenticación de las entidades de servicio |Utilice el libro del informe de operaciones confidenciales, que puede ayudar a mitigar este problema. [Vea la explicación en esta entrada de blog](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/azure-ad-workbook-to-help-you-assess-solorigate-risk/ba-p/2010718).|

## <a name="find-accounts-using-service-principals"></a>Búsqueda de cuentas con entidades de servicio
Ejecute los siguientes comandos para buscar cuentas con entidades de servicio.

Uso de la CLI de Azure


`az ad sp list`

Usar PowerShell

`Get-AzureADServicePrincipal -All:$true` 


Para obtener más información, consulte [Get-AzureADServicePrincipal](/powershell/module/azuread/get-azureadserviceprincipal).

## <a name="assess-service-principal-security"></a>Evaluación de la seguridad de la entidad de servicio

Para evaluar la seguridad de las entidades de servicio, asegúrese de evaluar los privilegios y el almacenamiento de credenciales.

Mitigue los posibles desafíos con la siguiente información.

|Desafíos | Mitigaciones|
| - | - |
| Detección del usuario que ha dado su consentimiento a una aplicación multiinquilino y detección de las concesiones de consentimiento ilícitas a una aplicación multiinquilino | Ejecute el siguiente PowerShell para buscar aplicaciones multiinquilino.<br>`Get-AzureADServicePrincipal -All:$true ? {$_.Tags -eq WindowsAzureActiveDirectoryIntegratedApp"}`<br>Deshabilite el consentimiento del usuario. <br>Permiso para el consentimiento del usuario de los editores verificados, para los permisos seleccionados (recomendado) <br> Use el acceso condicional para bloquear las entidades de servicio de ubicaciones que no son de confianza. Configúrelas en el contexto del usuario, y sus tokens deben usarse para activar la entidad de servicio.|
|Use un secreto compartido codificado de forma rígida en un script mediante una entidad de servicio.|Use un certificado o un almacén de Azure Key Vault.|
|Seguimiento de quién usa el certificado o el secreto| Supervise los inicios de sesión de la entidad de servicio mediante los registros de inicio de sesión de Azure AD.|
No se puede administrar el inicio de sesión de entidades de servicio con acceso condicional.| Supervisión de los inicios de sesión mediante los registros de inicio de sesión de Azure AD
| El rol de RBAC de Azure predeterminado es Colaborador. |Evalúe las necesidades y aplique el rol con los mínimos permisos posibles para satisfacer esa necesidad.|

## <a name="move-from-a-user-account-to-a-service-principal"></a>Traslado de una cuenta de usuario a una entidad de servicio  
Si utiliza una cuenta de usuario de Azure como entidad de servicio, evalúe si puede pasar a una [identidad administrada](../../app-service/overview-managed-identity.md?tabs=dotnet) o a una entidad de servicio. Si no puede usar una identidad administrada, aprovisione una entidad de servicio que tenga solo los permisos y el ámbito suficientes para ejecutar las tareas necesarias. Puede crear una entidad de servicio mediante el [registro de una aplicación](../develop/howto-create-service-principal-portal.md) o con [PowerShell](../develop/howto-authenticate-service-principal-powershell.md).

Al usar Microsoft Graph, consulte la documentación de la API específica, [como en este ejemplo](/powershell/azure/create-azure-service-principal-azureps), y asegúrese de que el tipo de permiso para la aplicación se muestra como compatible.

## <a name="next-steps"></a>Pasos siguientes

**Para obtener más información sobre las entidades de servicio:**

[Creación de una entidad de servicio](../develop/howto-create-service-principal-portal.md)

 [Supervisión de inicios de sesión de entidad de servicio](../reports-monitoring/concept-sign-ins.md)

**Para obtener más información acerca de la protección de cuentas de servicio:**

[Introducción a las cuentas de servicio de Azure](service-accounts-introduction-azure.md)

[Protección de identidades administradas](service-accounts-managed-identities.md)

[Gobernanza de las cuentas de servicio de Azure](service-accounts-governing-azure.md)

[Introducción a las cuentas de servicio locales](service-accounts-on-premises.md)
