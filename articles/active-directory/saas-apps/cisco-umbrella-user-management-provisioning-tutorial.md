---
title: 'Tutorial: Configuración de Cisco Umbrella User Management para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo aprovisionar y desaprovisionar automáticamente las cuentas de usuario de Azure AD para Cisco Umbrella User Management.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 1aa20f40-19ec-4213-9a3b-5eb2bcdd9bbd
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/20/2021
ms.author: Zhchia
ms.openlocfilehash: becc5010908c810688cfbd969bb12f236b2f127f
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111896097"
---
# <a name="tutorial-configure-cisco-umbrella-user-management-for-automatic-user-provisioning"></a>Tutorial: Configuración de Cisco Umbrella User Management para aprovisionar usuarios automáticamente

En este tutorial, se describen los pasos que debe realizar en Cisco Umbrella User Management y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [Cisco Umbrella User Management](https://umbrella.cisco.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en Cisco Umbrella User Management
> * Quitar usuarios de Cisco Umbrella User Management cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y Cisco Umbrella User Management
> * Aprovisionar grupos y pertenencias a grupos en Cisco Umbrella User Management

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario en Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una [suscripción a Cisco Umbrella](https://signup.umbrella.com).
* Una cuenta de usuario de Cisco Umbrella, con plenos permisos de administración.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
2. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Determine qué datos se van a [asignar entre Azure AD y Cisco Umbrella User Management](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-import-objectguid-attribute-via-azure-ad-connect-optional"></a>Paso 2. Importación del atributo ObjectGUID mediante Azure AD Connect (opcional)
Si ya ha aprovisionado identidades de usuario y grupo de AD local en Cisco Umbrella anteriormente y ahora quiere aprovisionar los mismos usuarios y grupos de Azure AD, debe sincronizar el atributo ObjectGUID para que las identidades aprovisionadas previamente se mantengan en la directiva de Umbrella.

> [!NOTE]
> El conector de AD de Umbrella local debe desactivarse antes de importar el atributo ObjectGUID.
  
Al usar Microsoft Azure AD Connect, el atributo ObjectGUID de los usuarios y grupos no se sincroniza de forma predeterminada desde AD en el entorno local a Azure AD. Para sincronizar este atributo, habilite el valor **Sincronización de atributos de las extensiones de directorios** opcional y seleccione los atributos objectGUID para los grupos y usuarios.  

   ![Página de características opcionales del asistente de Azure Active Directory Connect](./media/cisco-umbrella-user-management-provisioning-tutorial/active-directory-connect-directory-extension-attribute-sync.png)

> [!NOTE]
> La búsqueda en **Atributos disponibles** distingue mayúsculas de minúsculas.

   ![Captura de pantalla en la que se muestra la página de selección "Extensiones de directorio".](./media/cisco-umbrella-user-management-provisioning-tutorial/active-directory-connect-directory-extensions.png)

## <a name="step-3-configure-cisco-umbrella-user-management-to-support-provisioning-with-azure-ad"></a>Paso 3. Configuración de Cisco Umbrella User Management para admitir el aprovisionamiento con Azure AD

1. Inicie sesión en el [panel de Cisco Umbrella](https://login.umbrella.com ). Vaya a **Deployments** >  (Implementaciones)**Core Identities** >  (Identidades principales) **Users and Groups** (Usuarios y grupos).

2. Si el mecanismo de importación está establecido como manual, haga clic en **Import from IdP** (Importar desde IdP) para cambiarlo. 
  
3. Expanda la tarjeta de Azure Active Directory y haga clic en el vínculo de **API Keys page** (Página de claves de API).

   ![API](./media/cisco-umbrella-user-management-provisioning-tutorial/keys.png)

4. Expanda la tarjeta de Azure Active Directory en la página de claves de API y haga clic en **Generate Token** (Generar token).

   ![Generate](./media/cisco-umbrella-user-management-provisioning-tutorial/token.png)

5. El token generado se mostrará solo una vez. Copie y guarde la dirección URL y el token. Estos valores se escribirán en los campos **URL de inquilino** y **Token secreto** respectivamente de la pestaña Aprovisionamiento de la aplicación Cisco Umbrella User Management en Azure Portal.  


## <a name="step-4-add-cisco-umbrella-user-management-from-the-azure-ad-application-gallery"></a>Paso 4. Adición de Cisco Umbrella User Management desde la galería de aplicaciones de Azure AD

Agregue Cisco Umbrella User Management desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento en Cisco Umbrella User Management. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](../manage-apps/add-application-portal.md). 

## <a name="step-5-define-who-will-be-in-scope-for-provisioning"></a>Paso 5. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Al asignar usuarios y grupos a Cisco Umbrella User Management, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

## <a name="step-6-configure-automatic-user-provisioning-to-cisco-umbrella-user-management"></a>Paso 6. Configuración del aprovisionamiento automático de usuarios en Cisco Umbrella User Management 

Esta sección le guiará por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en Cisco Umbrella User Management en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-cisco-umbrella-user-management-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios de Cisco Umbrella User Management en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Cisco Umbrella User Management**.

    ![Vínculo a Cisco Umbrella User Management en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña de aprovisionamiento automático](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba los valores de dirección URL de inquilino y token de secreto de Cisco Umbrella User Management. Haga clic en **Probar conexión** para asegurarse de que Azure AD pueda conectarse a Cisco Umbrella User Management. Si se produce un error de conexión, asegúrese de que la cuenta de Cisco Umbrella User Management tenga permisos de administrador y vuelva a intentarlo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección de **asignaciones**, seleccione **Synchronize Azure Active Directory Users to Cisco Umbrella User Management** (Sincronizar usuarios de Azure Active Directory con Cisco Umbrella User Management).

9. Revise los atributos de usuario que se sincronizan entre Azure AD y Cisco Umbrella User Management en la sección **Attribute-Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades de **coincidencia** se usan para hacer coincidir las cuentas de usuario de Cisco Umbrella User Management a fin de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de Cisco Umbrella User Management admita el filtrado de usuarios en función de ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|---|
   |userName|String|&check;|
   |externalId|String|
   |active|Boolean|
   |DisplayName|String|
   |name.givenName|String|
   |name.familyName|String|
   |name.formatted|String|
   |urn:ietf:params:scim:schemas:extension:ciscoumbrella:2.0:User:nativeObjectId|String|

> [!NOTE]
> Si ha importado el atributo objectGUID para los usuarios a través de Azure AD Connect (consulte el paso 2), agregue una asignación de objectGUID a urn:ietf:params:scim:schemas:extension:ciscoumbrella:2.0:User:nativeObjectId.

10. En la sección de **asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Cisco Umbrella User Management** (Sincronizar grupos de Azure Active Directory con Cisco Umbrella User Management).

11. Revise los atributos de grupo que se sincronizan entre Azure AD y Cisco Umbrella User Management en la sección **Attribute-Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades de **coincidencia** se usan para hacer coincidir los grupos de Cisco Umbrella User Management a fin de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|Compatible con el filtrado|
      |---|---|---|
      |DisplayName|String|&check;|
      |externalId|String|
      |members|Referencia|
      |urn:ietf:params:scim:schemas:extension:ciscoumbrella:2.0:Group:nativeObjectId|String|

> [!NOTE]
> Si ha importado el atributo objectGUID para los grupos a través de Azure AD Connect (consulte el paso 2), agregue una asignación de objectGUID a urn:ietf:params:scim:schemas:extension:ciscoumbrella:2.0:Group:nativeObjectId.

12. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD para Cisco Umbrella User Management, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

14. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en Cisco Umbrella User Management.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-7-monitor-your-deployment"></a>Paso 7. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

1. Use los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
2. Consulte la [barra de progreso](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
3. Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="connector-limitations"></a>Limitaciones del conector
* Cisco Umbrella User Management admite el aprovisionamiento de un máximo de 200 grupos. Si se supera este número, puede que los grupos restantes que estén en el ámbito no se aprovisionen en Cisco Umbrella. 

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)