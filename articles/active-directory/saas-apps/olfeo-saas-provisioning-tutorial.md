---
title: 'Tutorial: Configuración de Olfeo SAAS para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo aprovisionar y desaprovisionar automáticamente cuentas de usuario de Azure AD para Olfeo SAAS.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 5f6b0320-dfe7-451c-8cd8-6ba7f2e40434
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/26/2021
ms.author: Zhchia
ms.openlocfilehash: 0afd2e2420868e40847ff1243fc61fb5b15c4853
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144122"
---
# <a name="tutorial-configure-olfeo-saas-for-automatic-user-provisioning"></a>Tutorial: Configuración de Olfeo SAAS para el aprovisionamiento automático de usuarios

En este tutorial, se describen los pasos que debe realizar en Olfeo SAAS y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y desaprovisiona usuarios y grupos de manera automática en [Olfeo SAAS](https://www.olfeo.com) mediante su servicio de aprovisionamiento. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en Olfeo SAAS
> * Eliminación de usuarios de Olfeo SAAS cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Olfeo SAAS
> * Aprovisionar grupos y pertenencias a grupos en Olfeo SAAS
> * [Inicio de sesión único](olfeo-saas-tutorial.md) en Olfeo SaaS (recomendado)

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario en Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Un [inquilino de OLFEO SaaS](https://www.olfeo.com/).
* Una cuenta de usuario de Olfeo SAAS con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento

1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
1. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Determine qué datos quiere [asignar entre Azure AD y Olfeo SAAS](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-olfeo-saas-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Olfeo SAAS para admitir el aprovisionamiento con Azure AD

1. Inicie sesión en la consola de administración de Olfeo SAAS. 
1. Navegue a **Configuración > Annuaires**.
1. Cree un nuevo directorio y asígnele un nombre.
1. Seleccione un proveedor de **Azure** y haga clic en **Cr�er** para guardar el nuevo directorio. 
1. Vaya a la pestaña **Sincronización** para ver la **dirección URL del inquilino** y el **secreto de Jeton**. Estos valores se escriben en los campos **URL de inquilino** y **Token de secreto** de la pestaña Aprovisionamiento de la aplicación Olfeo SAAS en Azure Portal.

## <a name="step-3-add-olfeo-saas-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Olfeo SAAS desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Olfeo SAAS, agregue esta aplicación desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Olfeo SAAS para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Al asignar usuarios y grupos a Olfeo SAAS, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar más roles. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-olfeo-saas"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en Olfeo SAAS 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios y grupos en Olfeo SAAS en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-olfeo-saas-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Olfeo SAAS en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Olfeo SAAS**.

    ![Vínculo a Olfeo SAAS en la lista de aplicaciones](common/all-applications.png)

1. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

1.  Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña de aprovisionamiento automático](common/provisioning-automatic.png)

1. En la sección **Credenciales de administrador**, especifique la información de **URL de inquilino** y de **Token secreto** de Olfeo SAAS. Para asegurarse de que Azure AD puede conectarse a Olfeo SAAS, seleccione **Probar conexión**. Si la conexión no se establece, asegúrese de que la cuenta de Olfeo SAAS tiene permisos de administrador e inténtelo de nuevo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

1. En el campo **Dirección de correo electrónico para notificaciones**, escriba la dirección de correo electrónico de la persona o grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

1. Seleccione **Guardar**.

1. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Olfeo SAAS** (Sincronizar usuarios de Azure Active Directory con Olfeo SAAS).

1. Examine los atributos de grupo que se sincronizan entre Azure AD y Olfeo SAAS en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Olfeo SAAS con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de Olfeo SAAS admite el filtrado de usuarios basado en ese atributo. Para confirmar los cambios, seleccione **Guardar**.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|---|
   |userName|String|&check;|
   |DisplayName|String|
   |active|Boolean|
   |emails[type eq "work"].value|String|
   |preferredLanguage|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |externalId|String|  

1. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Olfeo SAAS** (Sincronizar grupos de Azure Active Directory con Olfeo SAAS).

1. Examine los atributos de grupo que se sincronizan entre Azure AD y Olfeo SAAS en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para establecer coincidencias con los grupos de Olfeo SAAS con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|Compatible con el filtrado|
      |---|---|---|
      |DisplayName|String|&check;|
      |externalId|String|
      |members|Referencia|

1. Para configurar los filtros de ámbito, vea las instrucciones que se proporcionan en [Tutorial de los filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Olfeo SAAS, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

1. Seleccione los valores que quiera usar en **Ámbito** en la sección **Configuración** para definir los usuarios o grupos que quiere aprovisionar en Olfeo SAAS.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

1. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose.

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación

Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

* Use los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para determinar los usuarios que se han aprovisionado correctamente y los que no.
* Consulte la [barra de progreso](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
* Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Para obtener más información sobre los estados de cuarentena, vea [Estado de cuarentena del aprovisionamiento de aplicaciones](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="more-resources"></a>Más recursos

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)