---
title: Error inesperado al otorgar consentimiento a una aplicación | Microsoft Docs
description: Explica los errores que pueden producirse durante el proceso de otorgar su consentimiento a una aplicación y qué puede hacer al respecto
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: mtillman
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: a42801c723ea399ec8582e108c1a6e0f0baa2297
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080820"
---
# <a name="unexpected-error-when-performing-consent-to-an-application"></a>Error inesperado al otorgar consentimiento a una aplicación

En este artículo, se explican los errores que pueden producirse durante el proceso de otorgar su consentimiento a una aplicación. Si está solucionando solicitudes de consentimiento inesperadas que no contienen ningún mensaje de error, consulte [Escenarios de autenticación para Azure AD](../develop/authentication-vs-authorization.md).

Muchas aplicaciones que se integran con Azure Active Directory requieren permisos para acceder a otros recursos a fin de funcionar. Cuando estos recursos también se integran con Azure Active Directory, suelen solicitarse permisos para acceder a ellos mediante el marco de consentimiento común. Se muestra una petición de consentimiento, que generalmente ocurre la primera vez que se usa una aplicación, pero que también se puede producir en un uso posterior de la aplicación.

Determinadas condiciones deben cumplirse para que un usuario otorgue su consentimiento a los permisos que requiere una aplicación. Si no se cumplen estas condiciones, pueden producirse estos errores.

## <a name="requesting-not-authorized-permissions-error"></a>Error por solicitud de permisos no autorizados
* **AADSTS90093:** &lt;clientAppDisplayName&gt; solicita uno o varios permisos que no está autorizado a conceder. Póngase en contacto con el administrador, que puede dar su consentimiento para esta aplicación en su nombre.
* **AADSTS90094:** &lt;clientAppDisplayName&gt; necesita permiso para acceder a recursos de su organización que solo un administrador puede conceder. Pida a un administrador que conceda permiso a esta aplicación antes de poder usarla.

Este error se produce cuando un usuario que no es Administrador global intenta usar una aplicación que está solicitando permisos que solo un administrador puede conceder. Este error se puede solucionar si un administrador otorga el acceso a la aplicación en nombre de la organización.

Este error también puede producirse cuando se impide que un usuario otorgue consentimiento a una aplicación debido a que Microsoft detecta que la solicitud de permisos es arriesgada. En este caso, también se registrará un evento de auditoría con la categoría "ApplicationManagement", el tipo de actividad "Consentimiento a la aplicación" y el motivo de estado "Aplicación arriesgada detectada".

Otro escenario en el que se puede producir este error es cuando se requiere la asignación de usuario para la aplicación, pero no se proporcionó ningún consentimiento del administrador. En este caso, el administrador debe proporcionar primero el consentimiento del administrador.   

## <a name="policy-prevents-granting-permissions-error"></a>Error porque la directiva impide conceder permisos
* **AADSTS90093:** Un administrador de &lt;tenantDisplayName&gt; ha establecido una directiva que le impide otorgar a &lt;nombre de la aplicación&gt; los permisos que está solicitando. Póngase en contacto con un administrador de &lt;tenantDisplayName&gt;, que puede conceder permisos a esta aplicación en su nombre.

Este error se produce cuando un Administrador global desactiva la funcionalidad para que los usuarios otorguen consentimiento a aplicaciones y, a continuación, un usuario sin privilegios de administrador intenta usar una aplicación que requiere el consentimiento. Este error se puede solucionar si un administrador otorga el acceso a la aplicación en nombre de la organización.

## <a name="intermittent-problem-error"></a>Error de un problema intermitente
* **AADSTS90090:** Al parecer, el proceso de inicio de sesión detectó un problema intermitente para registrar los permisos que intenta conceder a &lt;clientAppDisplayName&gt;. Inténtelo de nuevo más tarde.

Este error indica que se ha producido un problema de servicio intermitente. Se puede resolver intentando otorgar su consentimiento a la aplicación de nuevo.

## <a name="resource-not-available-error"></a>Error de recurso no disponible
* **AADSTS65005:** La aplicación &lt;clientAppDisplayName&gt; solicitó permisos para acceder al recurso &lt;resourceAppDisplayName&gt; que no está disponible. 

Póngase en contacto con el desarrollador de aplicaciones.

##  <a name="resource-not-available-in-tenant-error"></a>Error de recurso no disponible en el inquilino
* **AADSTS65005:** &lt;clientAppDisplayName&gt; solicita permiso para acceder al recurso &lt;resourceAppDisplayName&gt; que no está disponible en su organización &lt;tenantDisplayName&gt;. 

Asegúrese de que este recurso esté disponible o póngase en contacto con un administrador de &lt;tenantDisplayName&gt;.

## <a name="permissions-mismatch-error"></a>Error de coincidencia de permisos
* **AADSTS65005:** La aplicación solicitó el consentimiento para acceder al recurso &lt;resourceAppDisplayName&gt;. Hay un error en esta solicitud porque no coincide con cómo se configuró anteriormente la aplicación durante el registro de aplicación. Póngase en contacto con el proveedor de la aplicación**.

Todos estos errores se producen cuando la aplicación a la que un usuario está intentando otorgar consentimiento solicita permisos para tener acceso a una aplicación de recursos que no se encuentra en el directorio de la organización (inquilino). Esta situación puede ocurrir por varios motivos:

-   El desarrollador de aplicaciones cliente configuró su aplicación incorrectamente, lo que provocó que solicitara acceso a un recurso no válido. En este caso, el desarrollador de aplicaciones debe actualizar la configuración de la aplicación cliente para resolver este problema.

-   Una entidad de servicio que representa la aplicación de recurso de destino no existe en la organización o existía en el pasado, pero se ha eliminado. Para resolver este problema, se debe haber aprovisionado a una entidad de servicio en la organización para la aplicación de recurso a fin de que la aplicación cliente pueda solicitar permisos a ella. La entidad de servicio se puede aprovisionar de varias maneras, en función del tipo de aplicación, entre las que se incluyen:

    -   Adquirir una suscripción para la aplicación del recurso (aplicaciones publicadas de Microsoft)

    -   Otorgar consentimiento a la aplicación del recurso

    -   Otorgar permisos a la aplicación en Azure Portal

    -   Agregar la aplicación desde la Galería de aplicaciones de Azure AD

## <a name="risky-app-error-and-warning"></a>Error y advertencia de aplicación de riesgo
* **AADSTS900941:** Se requiere consentimiento del administrador. La aplicación se considera de riesgo. (AdminConsentRequiredDueToRiskyApp)
* Esta aplicación puede ser peligrosa. Si confía en su origen, pídale al administrador que le conceda acceso.
* **AADSTS900981:** Se recibió una solicitud de consentimiento del administrador para una aplicación de riesgo. (AdminConsentRequestRiskyAppWarning)
* Esta aplicación puede ser peligrosa. Continúe solo si confía en su origen.

Ambos mensajes se mostrarán cuando Microsoft haya determinado que la solicitud de consentimiento puede ser peligrosa. Entre otros factores, esto puede ocurrir si no se ha agregado un [publicador comprobado](../develop/publisher-verification-overview.md) al registro de la aplicación. El código de error y mensaje primeros se mostrarán a los usuarios finales cuando se deshabilite el [flujo de trabajo de consentimiento del administrador](configure-admin-consent-workflow.md). El código y mensaje segundos se mostrarán a los usuarios finales cuando el flujo de trabajo de consentimiento del administrador esté habilitado y a los administradores. 

Los usuarios finales no podrán conceder el consentimiento a las aplicaciones que se hayan detectado como peligrosas. Los administradores pueden evaluar la aplicación, pero deben hacerlo muy atentamente, y continuar con precaución. Si la aplicación parece sospechosa tras una revisión más profunda, se puede notificar a Microsoft en la pantalla de consentimiento. 

## <a name="next-steps"></a>Pasos siguientes 

[Aplicaciones, permisos y consentimiento en Azure Active Directory (punto de conexión v1)](../develop/quickstart-register-app.md)<br>

[Ámbitos, permisos y consentimiento en Azure Active Directory (punto de conexión v2.0)](../develop/v2-permissions-and-consent.md)