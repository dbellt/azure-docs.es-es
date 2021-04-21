---
title: Administración del acceso externo con la Administración de derechos de Azure Active Directory
description: Uso de la Administración de derechos de Azure Active Directory como parte del plan de seguridad de acceso externo general.
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 12/18/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 89744b63a555cc02d35815b4066ce572b7f77e38
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107531897"
---
# <a name="manage-external-access-with-entitlement-management"></a>Administración del acceso externo con la Administración de derechos 


La [Administración de derechos](../governance/entitlement-management-overview.md) es una funcionalidad de control de identidad que permite a las organizaciones administrar el ciclo de vida de identidad y acceso a escala, mediante la automatización de los flujos de trabajo de solicitud de acceso, las asignaciones de acceso, las revisiones y la expiración. La Administración de derechos permite que los usuarios no administradores delegados creen [paquetes de acceso](../governance/entitlement-management-overview.md) a los que los usuarios externos de otras organizaciones puedan solicitar acceso. Los flujos de trabajo de aprobación única o de varias etapas se pueden configurar para evaluar solicitudes y [aprovisionar](../governance/what-is-provisioning.md) a los usuarios con acceso limitado en el tiempo con revisiones periódicas. La Administración de derechos permite el aprovisionamiento y desaprovisionamiento basados en directivas de cuentas externas.

## <a name="key-concepts-for-enabling-entitlement-management"></a>Conceptos clave para habilitar la Administración de derechos

Los conceptos clave siguientes son importantes para comprender la Administración de derechos.

### <a name="access-packages"></a>Paquetes de acceso

Un [paquete de acceso](../governance/entitlement-management-overview.md) es la base de la administración de derechos. Los paquetes de acceso son agrupaciones de recursos regidos por directivas que un usuario necesita para colaborar en un proyecto o realizar otras tareas. Por ejemplo, un paquete de acceso podría incluir:

* Acceso a sitios específicos de SharePoint.

* Aplicaciones empresariales, incluidas las aplicaciones internas y SaaS personalizadas, como Salesforce.

* Canales de Microsoft Teams.

* Microsoft 365 Groups. 

### <a name="catalogs"></a>Catalogs

Los paquetes de acceso residen en [catálogos](../governance/entitlement-management-catalog-create.md). Creará un catálogo cuando quiera agrupar recursos relacionados y paquetes de acceso y delegar la capacidad de administrarlos. En primer lugar, agregará recursos a un catálogo y luego podrá agregar dichos recursos para acceder a los paquetes. Por ejemplo, puede que desee crear un catálogo “Finanzas” y [delegar su administración](../governance/entitlement-management-delegate.md) a un miembro del equipo de finanzas. A continuación, esa persona puede [agregar recursos](../governance/entitlement-management-catalog-create.md), crear paquetes de acceso y administrar la aprobación de acceso a esos paquetes.

En el diagrama siguiente se muestra un ciclo de vida de gobernanza típico para un usuario externo que obtiene acceso a un paquete de acceso que tiene una expiración.

![Diagrama del ciclo de gobernanza de un usuario externo.](media/secure-external-access/6-governance-lifecycle.png)

### <a name="self-service-external-access"></a>Acceso externo de autoservicio

Puede exponer los paquetes de acceso a través del [portal Mi acceso de Azure AD](../governance/entitlement-management-request-access.md) para permitir que los usuarios externos soliciten el acceso. Las directivas determinan quién puede solicitar un paquete de acceso. Especifique quién tiene permiso para solicitar el paquete de acceso:

* [Organizaciones conectadas](../governance/entitlement-management-organization.md) específicas

* Todas las organizaciones conectadas configuradas

* Todos los usuarios de cualquier organización

* Usuarios miembros o invitados que ya están en el inquilino

### <a name="approvals"></a>Aprobaciones   
Los paquetes de acceso pueden incluir la aprobación obligatoria para el acceso. **Implemente siempre procesos de aprobación para usuarios externos**. Las aprobaciones pueden ser únicas o de varias etapas. Las aprobaciones se determinan mediante directivas. Si los usuarios internos y externos necesitan acceder al mismo paquete, es probable que configure diferentes directivas de acceso para las distintas categorías de organizaciones conectadas y para los usuarios internos.

### <a name="expiration"></a>Expiration  
Los paquetes de acceso pueden incluir una fecha de expiración. La expiración se puede establecer en un día concreto o asignar al usuario un número específico de días para el acceso. Cuando el paquete de acceso expira y el usuario no tiene otro acceso, el objeto de usuario invitado B2B que representa al usuario puede eliminarse o bloquearse para iniciar sesión. Le recomendamos que aplique la expiración de los paquetes de acceso para los usuarios externos. No todos los paquetes de acceso tienen expiraciones. Para aquellos que no lo las tienen, asegúrese de realizar revisiones de acceso.

### <a name="access-reviews"></a>Revisiones de acceso

Los paquetes de acceso pueden requerir [revisiones de acceso](../governance/manage-guest-access-with-access-reviews.md) periódicas, que requieren que el propietario del paquete o una persona designada certifique la necesidad continua de acceso de los usuarios. 

Antes de configurar la revisión, determine lo siguiente.

* Quién

   * ¿Cuáles son los criterios para el acceso continuado?

   * ¿Quiénes son los revisores especificados?

* ¿Con qué frecuencia deben realizarse las revisiones programadas?

   * Las opciones integradas incluyen mensualmente, trimestralmente, cada dos años o anualmente. 

   * Se recomiendan trimestralmente o más frecuentemente para los paquetes que admiten el acceso externo. 

 

> [!IMPORTANT]
> Las revisiones de acceso de los paquetes de acceso solo revisan el acceso concedido a través de la Administración de derechos. Por lo tanto, debe configurar otros procesos para revisar cualquier acceso proporcionado a usuarios externos fuera de la Administración de derechos.

Para más información sobre las revisiones de acceso, consulte [Planeamiento de una implementación de revisiones de acceso de Azure Active Directory](../governance/deploy-access-reviews.md).

## <a name="using-automation-in-entitlement-management"></a>Uso de la automatización en la Administración de derechos

Puede realizar [funciones de Administración de derechos mediante Microsoft Graph](/graph/tutorial-access-package-api), incluidas las siguientes:

* [Administración de paquetes de acceso](/graph/api/resources/accesspackage?view=graph-rest-beta&preserve-view=true)

* [Administración de revisiones del acceso](/graph/api/resources/accessreviewsv2-root?view=graph-rest-beta&preserve-view=true)

* [Administración de organizaciones conectadas](/graph/api/resources/connectedorganization?view=graph-rest-beta&preserve-view=true)

* [Administración de la configuración de Administración de derechos](/graph/api/resources/entitlementmanagementsettings?view=graph-rest-beta&preserve-view=true)

## <a name="recommendations"></a>Recomendaciones 

Se recomiendan los procedimientos para controlar el acceso externo con la Administración de derechos.

**En el caso de los proyectos con uno o más asociados comerciales, [cree y use paquetes de acceso](../governance/entitlement-management-access-package-create.md) para incorporar usuarios de esos asociados y proporcionarles acceso a los recursos.** . 

* Si ya tiene usuarios B2B en el directorio, también puede asignarlos directamente a los paquetes de acceso adecuados.

* Puede asignar acceso en [Azure Portal](../governance/entitlement-management-access-package-assignments.md) o a través de [Microsoft Graph](/graph/api/resources/accesspackageassignmentrequest?view=graph-rest-beta&preserve-view=true).

**Use la configuración de Gobierno de identidades para quitar usuarios del directorio cuando los paquetes de acceso expiren**.

![Captura de pantalla de la configuración de la administración del ciclo de vida de los usuarios externos.](media/secure-external-access/6-manage-external-lifecycle.png)

Esta configuración solo se aplica a los usuarios que se incorporaron a través de la Administración de derechos.

**[Delegue la administración de catálogos y paquetes de acceso](../governance/entitlement-management-delegate.md) a los propietarios del negocio, que tienen más información sobre quién debe acceder**.

![Captura de pantalla de la configuración de un catálogo.](media/secure-external-access/6-catalog-management.png)

**‎[Exija la expiración de los paquetes de acceso](../governance/entitlement-management-access-package-lifecycle-policy.md) a los que los usuarios externos tienen acceso.**


![Captura de pantalla de la configuración de la expiración del paquete de acceso.](media/secure-external-access/6-access-package-expiration.png)

* Si conoce la fecha de finalización de un paquete de acceso basado en proyecto, use el valor de En la fecha para establecer la fecha concreta. 

* En caso contrario, se recomienda que la expiración no sea superior a 365 días, a menos que se sepa que es una involucración de varios años.

* Permita a los usuarios extender el acceso.

* Requiera la aprobación para conceder la extensión.

**[Exija revisiones de acceso de los paquetes](../governance/manage-guest-access-with-access-reviews.md) para evitar el acceso inadecuado a los invitados.**

![Captura de pantalla de la creación de un nuevo paquete de acceso.](media/secure-external-access/6-new-access-package.png)

* Exija revisiones trimestralmente.

* En el caso de proyectos que tengan en cuenta el cumplimiento, establezca los revisores para que sean revisores específicos, en lugar de la autorrevisión para usuarios externos. Los usuarios que son administradores de paquetes de acceso son un buen punto de partida para los revisores. 

* En el caso de los proyectos menos confidenciales, la autorrevisión de los usuarios reducirá la carga de la organización para quitar el acceso de los usuarios que ya no están en su organización de origen.

Para más información, consulte [Gobernanza del acceso de los usuarios externos en la administración de derechos de Azure AD](../governance/entitlement-management-external-users.md). 

### <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre cómo proteger el acceso externo a los recursos. Se recomienda realizar las acciones en el orden indicado.

1. [Determinación de la posición de seguridad para el acceso externo](1-secure-access-posture.md)

2. [Detección del estado actual](2-secure-access-current-state.md)

3. [Creación de un plan de gobernanza](3-secure-access-plan.md)

4. [Uso de grupos con fines de seguridad](4-secure-access-groups.md)

5. [Transición a B2B de Azure AD](5-secure-access-b2b.md)

6. [Acceso seguro mediante la Administración de derechos](6-secure-access-entitlement-managment.md) (usted está en este punto).

7. [Acceso seguro mediante directivas de acceso condicional](7-secure-access-conditional-access.md)

8. [Acceso seguro mediante etiquetas de confidencialidad](8-secure-access-sensitivity-labels.md)

9. [Acceso seguro a Microsoft Teams, OneDrive y SharePoint](9-secure-access-teams-sharepoint.md)

 

