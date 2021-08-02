---
title: 'Aplicaciones, acciones y contexto de autenticación en la nube en la directiva de acceso condicional: Azure Active Directory'
description: Aplicaciones, acciones y contexto de autenticación en la nube en una directiva de acceso condicional de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 05/20/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99da9afc9afb3c6eb19caf696c6b9802aed6a2dd
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111953659"
---
# <a name="conditional-access-cloud-apps-actions-and-authentication--context"></a>Acceso condicional: aplicaciones, acciones y contexto de autenticación en la nube

Las aplicaciones, acciones y el contexto de autenticación en la nube son señales clave en una directiva de acceso condicional. Las directivas de acceso condicional permiten que los administradores asignen controles a determinadas aplicaciones, acciones o contextos de autenticación.

- Los administradores pueden elegir en la lista de aplicaciones entre aplicaciones integradas de Microsoft y cualquier [aplicación integrada de Azure AD](../manage-apps/what-is-application-management.md), incluidas aplicaciones de la galería, aplicaciones que no son de la galería y aplicaciones publicadas a través de [Application Proxy](../app-proxy/what-is-application-proxy.md).
- Los administradores pueden optar por definir una directiva no basada en una aplicación en la nube, sino en una [acción del usuario](#user-actions) como **Registro de la información de seguridad** o **Registrar o unir dispositivos (versión preliminar)** , lo que permite que el acceso condicional aplique controles en torno a esas acciones.
- Los administradores pueden usar el [contexto de autenticación](#authentication-context-preview) para proporcionar una capa adicional de seguridad dentro de las aplicaciones. 

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

Las aplicaciones que están disponibles para el acceso condicional han pasado por un proceso de incorporación y validación. Esta lista no incluye todas las aplicaciones de Microsoft, ya que muchas son servicios back-end y no están diseñadas para que se les aplique la directiva directamente. Si está buscando una aplicación que falta, puede ponerse en contacto con el equipo de la aplicación específica o hacer una solicitud en [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=167259).

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

   - Azure portal
   - Proveedor de Azure Resource Manager
   - API del modelo de implementación clásica
   - Azure PowerShell
   - Azure CLI
   - Portal de administrador de suscripciones de Visual Studio
   - Azure DevOps
   - Portal de Azure Data Factory

> [!NOTE]
> La aplicación de administración de Microsoft Azure se aplica a Azure PowerShell, que llama a la API de Azure Resource Manager. No se aplica a Azure AD PowerShell, que llama a Microsoft Graph.

### <a name="other-applications"></a>Otras aplicaciones

Además de las aplicaciones de Microsoft, los administradores pueden agregar cualquier aplicación registrada de Azure AD a las directivas de acceso condicional. Estas aplicaciones pueden incluir: 

- Aplicaciones web que se publican a través de[Azure AD Application Proxy](../app-proxy/what-is-application-proxy.md)
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

## <a name="authentication-context-preview"></a>Contexto de autenticación (versión preliminar)

El contexto de autenticación se puede usar para proteger aún más los datos y acciones de las aplicaciones. Estas aplicaciones pueden ser sus propias aplicaciones personalizadas, aplicaciones de línea de negocio (LOB) personalizadas, aplicaciones como SharePoint o aplicaciones protegidas por Microsoft Cloud App Security (MCAS). 

Por ejemplo, una organización puede conservar archivos en sitios de SharePoint como, por ejemplo, el menú de comidas o la receta secreta de su salsa barbacoa. Todos los usuarios pueden acceder al sitio del menú de comidas, pero es posible que para acceder al sitio de la receta secreta de la salsa barbacoa tengan que utilizar un dispositivo administrado y aceptar condiciones de uso específicas.

### <a name="configure-authentication-contexts"></a>Configuración de contextos de autenticación

Los contextos de autenticación se administran en Azure Portal en **Azure Active Directory** > **Seguridad** > **Acceso condicional** > **Contexto de autenticación**.

![Administración del contexto de autenticación en Azure Portal](./media/concept-conditional-access-cloud-apps/conditional-access-authentication-context-get-started.png)

> [!WARNING]
> * No es posible eliminar las definiciones de contextos de autenticación durante la versión preliminar. 
> * La versión preliminar se limita a un total de 25 definiciones de contextos de autenticación en Azure Portal.

Para crear nuevas definiciones de contextos de autenticación, seleccione **Nuevo contexto de autenticación** en Azure Portal. Configure los siguientes atributos:

- **Nombre para mostrar** es el nombre que se usa para identificar el contexto de autenticación en Azure AD y entre aplicaciones que usan contextos de autenticación. Se recomiendan nombres que se puedan usar en varios recursos, como "dispositivos de confianza", para reducir el número de contextos de autenticación necesarios. Tener un conjunto reducido limita el número de redireccionamientos y proporciona una mejor experiencia para el usuario final.
- **Descripción** proporciona más información sobre las directivas que usan los administradores de Azure AD y las que aplican contextos de autenticación a los recursos.
- Cuando está activada la casilla **Publicar en aplicaciones**, anuncia el contexto de autenticación a las aplicaciones y hace que estén disponibles para asignarlas. Si no está activada, el contexto de autenticación no estará disponible para los recursos de nivel inferior. 
- **Identificador** es de solo lectura y se usa en tokens y aplicaciones para definiciones de contexto de autenticación de solicitudes específicas. Se muestra aquí para casos de uso de solución de problemas y desarrollo. 

A continuación, los administradores pueden seleccionar contextos de autenticación publicados en sus directivas de acceso condicional en **Asignaciones** > **Aplicaciones en la nube o acciones** > **Contexto de autenticación**.

### <a name="tag-resources-with-authentication-contexts"></a>Etiquetado de recursos con contextos de autenticación 

Para más información sobre el uso del contexto de autenticación en las aplicaciones, consulte los artículos siguientes.

- [SharePoint Online](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites?view=o365-worldwide#more-information-about-the-dependencies-for-the-authentication-context-option)
- [Microsoft Cloud App Security](/cloud-app-security/session-policy-aad?branch=pr-en-us-2082#require-step-up-authentication-authentication-context)
- Aplicaciones personalizadas

## <a name="next-steps"></a>Pasos siguientes

- [Acceso condicional: Condiciones](concept-conditional-access-conditions.md)
- [Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)
- [¿Cuáles son las dependencias de servicio del acceso condicional de Azure Active Directory?](service-dependencies.md)