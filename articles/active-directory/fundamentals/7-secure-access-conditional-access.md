---
title: Administración del acceso externo mediante el acceso condicional de Azure Active Directory
description: Cómo usar las directivas de acceso condicional de Azure Active Directory para proteger el acceso externo a los recursos.
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
ms.openlocfilehash: 64209a4d9ca200c69783a4ae317b38beef8ded89
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "98222316"
---
# <a name="manage-external-access-with-conditional-access-policies"></a>Administración del acceso externo mediante directivas de acceso condicional 

El [acceso condicional](../conditional-access/overview.md) es la herramienta que usa Azure AD para agrupar señales, aplicar directivas y determinar si se debe permitir que un usuario acceda a los recursos. Para más información sobre cómo crear y usar directivas de acceso condicional, consulte [Planeamiento de la implementación del acceso condicional](../conditional-access/plan-conditional-access.md). 

![Diagrama de señales y decisiones relativas al acceso condicional](media/secure-external-access//7-conditional-access-signals.png)



Este artículo aborda la aplicación de directivas de acceso condicional a usuarios externos y da por supuesto que no tiene acceso a la funcionalidad de [administración de derechos](../governance/entitlement-management-overview.md). Las directivas de acceso condicional se pueden usar en combinación con la administración de derechos.

Anteriormente en esta documentación, [creó un plan de seguridad](3-secure-access-plan.md) que describe lo siguiente:

* Las aplicaciones y los recursos tienen los mismos requisitos de seguridad y se pueden agrupar con fines de acceso.

* Los requisitos de inicio de sesión para usuarios externos.

Usará ese plan para crear las directivas de acceso condicional para el acceso externo. 

> [!IMPORTANT]
> Cree algunas cuentas de prueba de usuarios externos para las directivas que cree antes de aplicarlas a todos los usuarios externos.

## <a name="conditional-access-policies-for-external-access"></a>Directivas de acceso condicional para el acceso externo

A continuación, se muestran los procedimientos recomendados en relación con la regulación del acceso externo mediante directivas de acceso condicional.

* Si no puede usar organizaciones conectadas en la administración de derechos, cree un grupo de seguridad de Azure AD o un grupo de Microsoft 365 para cada organización asociada con la que trabaja. Asigne todos los usuarios de ese asociado al grupo. Después, puede usar esos grupos en las directivas de acceso condicional.

* Cree el menor número posible de directivas de acceso condicional. En el caso de las aplicaciones que tengan las mismas necesidades de acceso, agréguelas a la misma directiva.  
‎ 
   > [!NOTE]
   > Se pueden aplicar directivas de acceso condicional a 250 aplicaciones como máximo. Si hay más de 250 aplicaciones que tienen las mismas necesidades de acceso, duplique las directivas. La directiva A se aplicará a las aplicaciones que van de 1 a 250, la directiva B se aplicará a las aplicaciones que van de 251 a 500, etc.

* Utilice una convención de nomenclatura para identificar claramente las directivas específicas del acceso externo. Una convención de nomenclatura es ‎*ExternalAccess_actiontaken_AppGroup*. Por ejemplo, ExternalAccess_Block_FinanceApps.

## <a name="block-all-external-users-from-resources"></a>Bloqueo del acceso a recursos para todos los usuarios externos

Puede impedir que los usuarios externos accedan a conjuntos específicos de recursos por medio de directivas de acceso condicional. Una vez que haya identificado el conjunto de recursos a los que desea impedir el acceso, cree una directiva.

Para crear una directiva que bloquee el acceso de los usuarios externos a un conjunto de aplicaciones:

1. Acceda a **Azure Portal**, seleccione **Azure Active Directory**, **Seguridad** y **Acceso condicional**.

2. Seleccione **Nueva directiva** y escriba un **nombre**. Por ejemplo, ExternalAccess_Block_FinanceApps.

3. Seleccione **Usuarios y grupos**. En la pestaña Incluir, elija **Seleccionar usuarios y grupos** y, a continuación, seleccione **Todos los usuarios externos e invitados**. 

4. Seleccione **Excluir** y especifique los grupos de administradores y las cuentas de acceso de emergencia.

5. Seleccione **Aplicaciones o acciones en la nube**, elija **Seleccionar aplicaciones**, seleccione todas las aplicaciones para las que desea bloquear el acceso externo y, por último, elija **Seleccionar**.

6. Seleccione **Condiciones** y **Ubicaciones**. En Configurar, elija **Sí** y **Cualquier ubicación**.

7. En Controles de acceso, seleccione **Conceder**, cambie el botón de alternancia a **Bloquear** y elija **Seleccionar**.

8. Asegúrese de que la opción Habilitar directiva se haya establecido en **solo informe** y, a continuación, seleccione **Crear**.

## <a name="block-external-access-to-all-except-specific-external-users"></a>Bloqueo del acceso externo para todos los usuarios externos, salvo un grupo específico

Puede haber ocasiones en las que desee bloquear a todos los usuarios externos, salvo un grupo específico. Por ejemplo, tal vez desee bloquear el acceso de todos los usuarios externos a las aplicaciones financieras, salvo los usuarios que trabajan para el equipo financiero. Para hacerlo:

1. Cree un grupo de seguridad que contenga los usuarios externos que deben tener acceso al grupo financiero.

2. Siga los pasos 1 a 3 de la sección anterior para bloquear el acceso externo a los recursos.

3. En el paso 4, agregue el grupo de seguridad que desea excluir del bloqueo de acceso a las aplicaciones financieras.

4. Siga el resto de los pasos.

## <a name="implement-conditional-access"></a>Implementación del acceso condicional

Muchas directivas comunes de acceso condicional están documentadas. Las siguientes se pueden adaptar a los usuarios externos:

* [Exigir la autenticación multifactor para todos los usuarios](../conditional-access/howto-conditional-access-policy-all-users-mfa.md)

* [Acceso condicional basado en riesgos de usuario](../conditional-access/howto-conditional-access-policy-risk-user.md)

* [Exigir la autenticación multifactor para el acceso desde redes que no son de confianza](../conditional-access/untrusted-networks.md) 

* [Exigir los términos de uso](../conditional-access/terms-of-use.md)

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre cómo proteger el acceso externo a los recursos. Se recomienda realizar las acciones en el orden indicado.

1. [Determinación de la posición de seguridad deseada para el acceso externo](1-secure-access-posture.md)

2. [Detección del estado actual](2-secure-access-current-state.md)

3. [Creación de un plan de gobernanza](3-secure-access-plan.md)

4. [Uso de grupos con fines de seguridad](4-secure-access-groups.md)

5. [Transición a B2B de Azure AD](5-secure-access-b2b.md)

6. [Acceso seguro mediante la administración de derechos](6-secure-access-entitlement-managment.md)

7. [Acceso seguro mediante directivas de acceso condicional](7-secure-access-conditional-access.md) (usted está aquí)

8. [Acceso seguro mediante etiquetas de confidencialidad](8-secure-access-sensitivity-labels.md)

9. [Acceso seguro a Microsoft Teams, OneDrive y SharePoint](9-secure-access-teams-sharepoint.md)
