---
title: Gobernanza de las cuentas de servicio de Azure Active Directory
description: Principios y procedimientos para administrar el ciclo de vida de las cuentas de servicio en Azure Active Directory.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 3/1/2021
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: aec870e6027d9a6d2e6c3b66363f93c73fad5e78
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108064150"
---
# <a name="governing-azure-ad-service-accounts"></a>Gobernanza de las cuentas de servicio de Azure AD

Hay tres tipos de cuentas de servicio en Azure Active Directory (Azure AD): [identidades administradas](service-accounts-managed-identities.md), [entidades de servicio](service-accounts-principal.md) y cuentas de usuario empleadas como cuentas de servicio. Cuando se crean estas cuentas de servicio para su uso automatizado, se les conceden permisos para acceder a los recursos de Azure y Azure AD. Los recursos pueden incluir servicios de Microsoft 365, aplicaciones de software como servicio (SaaS), aplicaciones personalizadas, bases de datos, sistemas de recursos humanos, etc. La gobernanza de cuentas de servicio de Azure AD implica que el usuario está a cargo de su creación, permisos y ciclo de vida para garantizar la seguridad y la continuidad.

> [!IMPORTANT] 
> No se recomienda el uso de cuentas de usuario como cuentas de servicio, ya que son intrínsecamente menos seguras. Esto incluye las cuentas de servicio locales que se sincronizan con Azure AD, ya que no se convierten en entidades de servicio. En su lugar, se recomienda el uso de identidades administradas o entidades de servicio. Tenga en cuenta que en este momento no es posible el uso de directivas de acceso condicional con entidades de servicio, pero la funcionalidad estará disponible próximamente.


## <a name="plan-your-service-account"></a>Planeamiento de su cuenta de servicio

Antes de crear una cuenta de servicio o de registrar una aplicación, documente la información de la clave de la cuenta de servicio. La información documentada facilita la supervisión y la gobernanza efectivas de la cuenta. Se recomienda recopilar los datos siguientes y realizar su seguimiento en la base de datos de administración de configuración (CMDB) centralizada.

| data| Descripción| Detalles |
| - | - | - |
| Propietario| Usuario o grupo responsable de la administración y la supervisión de la cuenta de servicio.| Aprovisione al propietario con los permisos necesarios para supervisar la cuenta e implementar una manera de mitigar los problemas. La mitigación de problemas puede realizarla el propietario o bien solicitarse al departamento de TI. |
| Propósito| Cómo se utilizará la cuenta.| Asigne la cuenta de servicio a un servicio, aplicación o script específico. Evite la creación de cuentas de servicio multiuso. |
| Permisos (ámbitos)| Conjunto previsto de permisos.| Documente los recursos a los que tendrá acceso y los permisos para esos recursos. |
| Vínculo a CMDB| Vínculo a los recursos a los que se va a tener acceso y scripts en los que se usa la cuenta de servicio.| Asegúrese de documentar los propietarios de los recursos y scripts para que pueda comunicar los efectos ascendentes y descendentes necesarios de los cambios. |
| Evaluación de riesgos| Riesgo e impacto para la empresa si la cuenta estuviera en peligro.| Utilice esta información para restringir el ámbito de los permisos y determinar quién debe tener acceso a la información de la cuenta. |
| Período de revisión| El plazo en que el propietario debe revisar la cuenta de servicio.| Utilícelo para programar las comunicaciones y las revisiones. Documente qué debe ocurrir si una revisión no se realiza antes de un momento determinado después del período de revisión programado. |
| Período de duración| Vigencia máxima prevista de la cuenta.| Úselo para programar las comunicaciones con el propietario y, en última instancia, deshabilitar y eliminar las cuentas. Siempre que sea posible, establezca una fecha de expiración para las credenciales, donde estas no puedan renovarse automáticamente. |
| Nombre| Nombre normalizado de la cuenta| Cree un esquema de nomenclatura para todas las cuentas de servicio, de modo que pueda buscar, ordenar y filtrar fácilmente las cuentas de servicio. |


## <a name="use-the-principle-of-least-privileges"></a>Uso del principio de privilegios mínimos
Conceda a la cuenta de servicio solo los permisos necesarios para realizar sus tareas, y ninguno más. Si una cuenta de servicio necesita permisos de alto nivel, como un nivel de privilegio de administrador global, evalúe los motivos y trate de reducir los permisos necesarios.

Se recomiendan los siguientes procedimientos para los privilegios de la cuenta de servicio.

**Permisos**

* No asigne roles integrados a las cuentas de servicio. En su lugar, use el [modelo de concesión de permisos OAuth2 para Microsoft Graph](/graph/api/resources/oauth2permissiongrant).

* Si a la entidad de servicio se le debe asignar un rol con privilegios, considere la posibilidad de asignar un [rol personalizado](../roles/custom-create.md) con privilegios específicos y necesarios en un modo tiempo limitado.

* No incluya cuentas de servicio como miembros de un grupo con permisos elevados. 

* [Use PowerShell para enumerar los miembros de roles con privilegios](/powershell/module/azuread/get-azureaddirectoryrolemember), como   
‎`Get-AzureADDirectoryRoleMember`, y filtre por el tipo de objeto "entidad de servicio".

   o bien, use  
‎   `Get-AzureADServicePrincipal | % { Get-AzureADServiceAppRoleAssignment -ObjectId $_ }`

* [Use los ámbitos de OAuth 2.0](../develop/v2-permissions-and-consent.md) para limitar la funcionalidad a la que una cuenta de servicio puede tener acceso en un recurso.
* Las entidades de servicio y las identidades administradas pueden usar ámbitos de OAuth 2.0 en un contexto delegado que suplanta a un usuario que ha iniciado sesión o como cuenta de servicio en el contexto de la aplicación. En el contexto de la aplicación no se inicia sesión.

* Compruebe la solicitud de cuentas de servicio de ámbito para recursos para asegurarse de que son adecuadas. Por ejemplo, si una cuenta solicita Files.ReadWrite.All, valore si realmente solo necesita File.Read.All. Para obtener más información sobre los permisos, vea para la [referencia de permisos de Microsoft Graph](/graph/permissions-reference).

* Asegúrese de que confía en el desarrollador de la aplicación o la API con el acceso solicitado a sus recursos.

**Duración**

* Limite las credenciales de la cuenta de servicio (secreto de cliente, certificado) a un período de uso previsto.

* Programe revisiones periódicas del uso y la finalidad de las cuentas de servicio. Asegúrese de que se realizan revisiones antes de la expiración de la cuenta.

Una vez que tenga claro el propósito, el alcance y los permisos necesarios, cree su cuenta de servicio. 

[Creación y uso de identidades administradas](../../app-service/overview-managed-identity.md?tabs=dotnet)

[Creación y uso de entidades de servicio](../develop/howto-create-service-principal-portal.md)

Use una identidad administrada siempre que sea posible. Si no puede usar una identidad administrada, use una entidad de servicio. Si no puede utilizar una entidad de servicio, entonces y solo entonces utilice una cuenta de usuario de Azure AD.

 

## <a name="build-a-lifecycle-process"></a>Creación de un proceso de ciclo de vida

La administración del ciclo de vida de una cuenta de servicio comienza con el planeamiento y finaliza con su eliminación permanente. 

En este artículo se ha tratado anteriormente la parte del planeamiento y la creación. También debe supervisarla, revisar los permisos, determinar el uso continuado de una cuenta y, en última instancia, desaprovisionarla.

### <a name="monitor-service-accounts"></a>Supervisión de cuentas de servicio

Supervise de forma proactiva las cuentas de servicio para asegurarse de que los patrones de uso de la cuenta de servicio reflejan los patrones previstos y de que la cuenta de servicio sigue utilizándose activamente.

**Recopile y supervise los inicios de sesión de la cuenta de servicio mediante uno de los métodos siguientes:**

* Con los registros de inicio de sesión de Azure AD en el portal de Azure AD.

* Mediante la exportación de los registros de inicio de sesión de Azure AD a [Azure Storage](../../storage/index.yml), [Azure Event Hubs](../../event-hubs/index.yml) o [Azure Monitor](../../azure-monitor/logs/data-platform-logs.md).


![Captura de pantalla que muestra la pantalla de inicio de sesión de la entidad de servicio.](./media/securing-service-accounts/service-accounts-govern-azure-1.png)

**La información que debe buscar en los registros de inicio de sesión incluye lo siguiente:**

* ¿Hay cuentas de servicio que ya no inician sesión en el inquilino?

* ¿Cambian los patrones de inicio de sesión de las cuentas de servicio?

Le recomendamos que exporte los registros de inicio de sesión de Azure AD y los importe en sus herramientas existentes de Administración de eventos e información de seguridad (SIEM), como Azure Sentinel. Use SIEM para crear alertas y paneles.

### <a name="review-service-account-permissions"></a>Revisión de los permisos de la cuenta de servicio

Revise periódicamente los permisos concedidos y los ámbitos a los que tienen acceso las cuentas de servicio para ver si se pueden reducir.

* Use [PowerShell](/powershell/module/azuread/get-azureadserviceprincipaloauth2permissiongrant) para [compilar la automatización con el fin de comprobar y documentar](https://gist.github.com/psignoret/41793f8c6211d2df5051d77ca3728c09) los ámbitos para los que se concede el consentimiento a una cuenta de servicio.

* Use PowerShell para [revisar las credenciales de las entidades de servicio existentes](https://github.com/AzureAD/AzureADAssessment) y comprobar su validez.

* No establezca las credenciales de la entidad de servicio en "No expira nunca".

* Use certificados o credenciales almacenados en Azure KeyVault siempre que sea posible.

En el ejemplo gratuito de PowerShell de Microsoft se recopilan las concesiones OAuth2 y la información de credenciales de la entidad de servicio, se registran en un archivo de valores separados por comas (CSV) y se usa un panel de ejemplo de Power BI para interpretar y usar los datos. Para obtener más información, consulte [AzureAD/AzureADAssessment: Herramientas para la evaluación de un estado y configuración de inquilino de Azure AD (github.com)](https://github.com/AzureAD/AzureADAssessment)

### <a name="recertify-service-account-use"></a>Recertificación del uso de la cuenta de servicio

Establezca un proceso de revisión para asegurarse de que las cuentas de servicio se revisan por sus propietarios y el equipo de seguridad o de TI a intervalos regulares. 

**El proceso debe incluir:**

* Cómo determinar el ciclo de revisión de cada cuenta de servicio (debe documentarse en su CMDB).

* Las comunicaciones con el propietario y los equipos de seguridad o de TI antes de que se inicien las revisiones.

* La temporización y el contenido de las comunicaciones de advertencia si la revisión no se ha cumplido.

* Instrucciones sobre qué hacer si los propietarios no pueden revisar o responder. Por ejemplo, puede que desee deshabilitar (pero no eliminar) la cuenta hasta que se complete la revisión.

* Instrucciones para determinar las dependencias ascendentes y descendentes y para notificar los efectos a otros propietarios de recursos.

**La revisión debe incluir que el propietario y su socio de TI certifiquen que:**

* La cuenta sigue siendo necesaria.

* Los permisos concedidos a la cuenta son adecuados y necesarios, o se solicita un cambio.

* Se controla el acceso a la cuenta y sus credenciales.

* Las credenciales que utiliza la cuenta son adecuadas, con respecto al riesgo con el que se evaluó la cuenta (tanto el tipo de credencial como su duración).

* La puntuación del riesgo de la cuenta no ha cambiado desde la última recertificación.

* Una actualización de la vigencia prevista de la cuenta y la próxima fecha de recertificación.

### <a name="deprovision-service-accounts"></a>Desaprovisionamiento de cuentas de servicio

** Desaprovisionamiento de cuentas de servicio en las siguientes circunstancias:****

* El script o la aplicación para los que se creó la cuenta de servicio se ha retirado.

* La función del script o la aplicación para el que se usa la cuenta de servicio (por ejemplo, para acceder a un recurso específico) se ha retirado.


* La cuenta de servicio se ha reemplazado por una cuenta de servicio diferente.

* Las credenciales han caducado, o la cuenta no funciona por algún otro motivo, y no hay ninguna queja.

**Los procesos de desaprovisionamiento deben incluir las siguientes tareas.**

1. Después de desaprovisionar la aplicación o el script asociado, [supervise los inicios de sesión](../reports-monitoring/concept-sign-ins.md) y el acceso a los recursos de la cuenta de servicio.

   * Si la cuenta aún está activa, determine cómo se está usando antes de realizar los pasos siguientes.
 
2. Si se trata de una identidad de servicio administrada, deshabilite la cuenta de servicio para el inicio de sesión, pero no la quite del directorio.

3. Revoque las asignaciones de roles y las concesiones de consentimiento de OAuth2 para la cuenta de servicio.

4. Después de un período definido, y una advertencia exhaustiva a los propietarios, elimine la cuenta de servicio del directorio.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre la protección de las cuentas de servicio de Azure, consulte:

[Introducción a las cuentas de servicio de Azure](service-accounts-introduction-azure.md)

[Protección de identidades administradas](service-accounts-managed-identities.md)

[Protección de entidades de servicio](service-accounts-principal.md)




 

