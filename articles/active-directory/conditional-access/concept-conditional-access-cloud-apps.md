---
title: 'Aplicaciones o acciones en la nube en la directiva de acceso condicional: Azure Active Directory'
description: Qué son las aplicaciones o acciones en la nube en una directiva de acceso condicional de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 10/16/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2f179ba02a47617a931906bb2b7575eb2bb3963d
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109750616"
---
# <a name="conditional-access-cloud-apps-or-actions"></a>Acceso condicional: Aplicaciones o acciones en la nube

Las aplicaciones o acciones en la nube son una señal esencial en una directiva de acceso condicional. Las directivas de acceso condicional permiten que los administradores asignen controles a determinadas aplicaciones o acciones.

- Los administradores pueden elegir en la lista de aplicaciones entre aplicaciones integradas de Microsoft y cualquier [aplicación integrada de Azure AD](../manage-apps/what-is-application-management.md), incluidas aplicaciones de la galería, aplicaciones que no son de la galería y aplicaciones publicadas a través de [Application Proxy](../manage-apps/what-is-application-proxy.md).
- Los administradores pueden optar por definir una directiva que no se base en una aplicación en la nube, sino en una acción del usuario. Se admiten dos acciones de usuario.
   - Registrar la información de seguridad (versión preliminar), para aplicar controles en torno a la [experiencia de registro combinado de la información de seguridad](../authentication/howto-registration-mfa-sspr-combined.md). 
   - Registrar o unir dispositivos (versión preliminar), para aplicar controles cuando los usuarios [registran](../devices/concept-azure-ad-register.md) o [unen](../devices/concept-azure-ad-join.md) dispositivos a Azure AD. 

![Definición de una directiva de acceso condicional y especificación de aplicaciones en la nube](./media/concept-conditional-access-cloud-apps/conditional-access-cloud-apps-or-actions.png)

## <a name="microsoft-cloud-applications"></a>Aplicaciones de nube de Microsoft

En la lista de aplicaciones que se pueden seleccionar están muchas de las aplicaciones en la nube de Microsoft existentes. 

Los administradores pueden asignar una directiva de acceso condicional a las siguientes aplicaciones en la nube de Microsoft. Algunas aplicaciones, como Office 365 y la Administración de Microsoft Azure, incluyen varios servicios o aplicaciones secundarios relacionados. Agregamos continuamente más aplicaciones, por lo que la lista siguiente no es exhaustiva y está sujeta a cambios.

- [Office 365](#office-365)
- Azure Analysis Services
- Azure DevOps
- Azure Event Hubs
- Azure Service Bus
- [Azure SQL Database y Azure Synapse Analytics](../../azure-sql/database/conditional-access-configure.md)
- Dynamics CRM Online
- Analytics de Microsoft Application Insights
- [Microsoft Azure Information Protection](/azure/information-protection/faqs#i-see-azure-information-protection-is-listed-as-an-available-cloud-app-for-conditional-accesshow-does-this-work)
- [Administración de Microsoft Azure](#microsoft-azure-management)
- Administración de suscripciones de Microsoft Azure
- Microsoft Cloud App Security
- Portal de Control de acceso de las herramientas de Microsoft Commerce
- Servicio de autenticación de las herramientas de Microsoft Commerce
- Microsoft Flow
- Microsoft Forms
- Microsoft Intune
- [Inscripción en Microsoft Intune](/intune/enrollment/multi-factor-authentication)
- Microsoft Planner
- Microsoft PowerApps
- Microsoft Search en Bing
- Microsoft StaffHub
- Microsoft Stream
- Equipos de Microsoft
- Exchange Online
- SharePoint
- Yammer
- Office Delve
- Office Sway
- Outlook Groups
- Servicio Power BI
- Project Online
- Skype Empresarial Online
- Red privada virtual (VPN)
- ATP de Windows Defender

Las aplicaciones que están disponibles para el acceso condicional han pasado por un proceso de incorporación y validación. Esto no incluye todas las aplicaciones de Microsoft, ya que muchas son servicios back-end y no están diseñadas para que se les aplique la directiva directamente. Si está buscando una aplicación que falta, puede ponerse en contacto con el equipo de la aplicación específica o hacer una solicitud en [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=167259).

### <a name="office-365"></a>Office 365

Microsoft 365 proporciona servicios de colaboración y productividad basados en la nube, como Exchange, SharePoint y Microsoft Teams. Los servicios en la nube de Microsoft 365 están profundamente integrados para garantizar experiencias de colaboración fluidas. Esta integración puede producir confusión a la hora de crear directivas, ya que algunas aplicaciones, como Microsoft Teams, dependen de otras, como SharePoint o Exchange.

La aplicación Office 365 permite la segmentación de todos estos servicios a la vez. Se recomienda usar la nueva aplicación Office 365 en lugar de aplicaciones en la nube individuales para evitar problemas con las [dependencias de servicio](service-dependencies.md). Trabajar con este grupo de aplicaciones ayuda a evitar problemas que pueden surgir debido a directivas y dependencias incoherentes.

Los administradores pueden optar por excluir aplicaciones específicas de la directiva si lo desean, incluyendo la aplicación Office 365 y excluyendo las aplicaciones específicas que decidan.

Aplicaciones clave que se incluyen en la aplicación cliente Office 365:

   - Microsoft Flow
   - Microsoft Forms
   - Microsoft Stream
   - Microsoft To-Do
   - Equipos de Microsoft
   - Exchange Online
   - SharePoint Online
   - Servicio de búsqueda de Microsoft 365
   - Yammer
   - Office Delve
   - Office en línea
   - Office.com
   - OneDrive
   - PowerApps
   - Skype Empresarial Online
   - Sway

### <a name="microsoft-azure-management"></a>Microsoft Azure Management (Administración de Microsoft Azure)

La aplicación de administración de Microsoft Azure incluye varios servicios subyacentes. 

   - Portal de Azure
   - Proveedor de Azure Resource Manager
   - API del modelo de implementación clásica
   - Azure PowerShell
   - Azure CLI
   - Portal de administrador de suscripciones de Visual Studio
   - Azure DevOps
   - Portal de Azure Data Factory

> [!NOTE]
> La aplicación de administración de Microsoft Azure se aplica a Azure PowerShell, que llama a la API de Azure Resource Manager. No se aplica a Azure AD PowerShell, que llama a Microsoft Graph.

## <a name="other-applications"></a>Otras aplicaciones

Además de las aplicaciones de Microsoft, los administradores pueden agregar cualquier aplicación registrada de Azure AD a las directivas de acceso condicional. Estas aplicaciones pueden incluir: 

- Aplicaciones web que se publican a través de[Azure AD Application Proxy](../manage-apps/what-is-application-proxy.md)
- [Aplicaciones agregadas desde la galería](../manage-apps/add-application-portal.md)
- [Aplicaciones personalizadas que no están en la galería](../manage-apps/view-applications-portal.md)
- [Aplicaciones heredadas publicadas a través de redes y controladores de entrega de aplicaciones](../manage-apps/secure-hybrid-access.md)
- Aplicaciones que usan el [inicio de sesión único basado en contraseña](../manage-apps/configure-password-single-sign-on-non-gallery-applications.md)

> [!NOTE]
> Puesto que la directiva de acceso condicional establece los requisitos para obtener acceso a un servicio, no puede aplicarla a una aplicación cliente (pública o nativa). Es decir, la directiva no está establecida directamente en una aplicación cliente (pública o nativa), pero se aplica cuando un cliente llama a un servicio. Por ejemplo, una directiva establecida en el servicio SharePoint se aplica a los clientes que llamen a SharePoint. Así mismo, una directiva establecida en Exchange se aplica al intento de acceder al correo electrónico mediante el cliente de Outlook. Este es el motivo por el que las aplicaciones cliente (públicas o nativas) no están disponibles para su selección en el selector Aplicaciones en la nube y la opción Acceso condicional no está disponible en la configuración de la aplicación para la aplicación cliente (pública o nativa) registrada en el inquilino. 

## <a name="user-actions"></a>Acciones del usuario

Las acciones del usuario son tareas que un usuario puede realizar. Actualmente, el Acceso condicional admite dos acciones del usuario: 

- **Registro de la información de seguridad**: esta acción del usuario permite que la directiva de Acceso condicional se aplique cuando los usuarios que están habilitados para el registro combinado intentan registrar su información de seguridad. Para más información, consulte el artículo [Registro de información de seguridad combinado](../authentication/concept-registration-mfa-sspr-combined.md).

- **Registro o unión de dispositivos (versión preliminar)** : esta acción del usuario permite a los administradores aplicar la directiva de Acceso condicional cuando los usuarios [registran](../devices/concept-azure-ad-register.md) o [unen](../devices/concept-azure-ad-join.md) dispositivos a Azure AD. Proporciona granularidad en la configuración de la autenticación multifactor para registrar o unir dispositivos en lugar de la directiva para todo el inquilino que existe actualmente. Existen tres consideraciones principales con esta acción de usuario: 
   - `Require multi-factor authentication` es el único control de acceso disponible con esta acción del usuario y todos los demás están deshabilitados. Esta restricción evita conflictos con controles de acceso que dependen del registro de dispositivos de Azure AD o que no se pueden aplicar al registro de dispositivos de Azure AD. 
   - Las condiciones `Client apps` y `Device state` no están disponibles con esta acción de usuario, ya que dependen del registro de dispositivos de Azure AD para aplicar directivas de acceso condicional.
   - Cuando se habilita una directiva de Acceso condicional con esta acción del usuario, debe establecer **Azure Active Directory** > **Dispositivos** > **Configuración del dispositivo** - `Devices to be Azure AD joined or Azure AD registered require Multi-Factor Authentication` en **No**. De lo contrario, no se aplica correctamente la directiva de acceso condicional con esta acción de usuario. Puede encontrar más información sobre esta configuración de dispositivo en [Configuración de las opciones de dispositivo](../devices/device-management-azure-portal.md#configure-device-settings). 
   
## <a name="next-steps"></a>Pasos siguientes

- [Acceso condicional: Condiciones](concept-conditional-access-conditions.md)

- [Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)
- [¿Cuáles son las dependencias de servicio del acceso condicional de Azure Active Directory?](service-dependencies.md)
