---
title: 'Tutorial: Configuración de BrowserStack Single Sign-on para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y desaprovisionar de forma automática las cuentas de usuario de Azure AD para BrowserStack Single Sign-on.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 39999abc-e4a2-4058-81e0-bf88182f8864
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/22/2021
ms.author: Zhchia
ms.openlocfilehash: 200338ac50bceae6c3bf9730ee92a16c04a84b69
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108141476"
---
# <a name="tutorial-configure-browserstack-single-sign-on-for-automatic-user-provisioning"></a>Tutorial: Configuración de BrowserStack Single Sign-on para el aprovisionamiento automático de usuarios

En este tutorial se describen los pasos que debe realizar en BrowserStack Single Sign-on y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [BrowserStack Single Sign-on](https://www.browserstack.com) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en BrowserStack Single Sign-on
> * Eliminar usuarios de BrowserStack Single Sign-on cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y BrowserStack Single Sign-on
> * [Inicio de sesión único](./browserstack-single-sign-on-tutorial.md) en BrowserStack Single Sign-on (recomendado)

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario en Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una cuenta de usuario de BrowserStack con permisos de **propietario**.
* Un [plan Enterprise](https://www.browserstack.com/pricing) con BrowserStack. 
* Integración del [inicio de sesión](https://www.browserstack.com/docs/enterprise/single-sign-on/azure-ad) único con BrowserStack (obligatorio).

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
2. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Determine qué datos quiere [asignar entre Azure AD y BrowserStack Single Sign-on](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-browserstack-single-sign-on-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de BrowserStack Single Sign-on para admitir el aprovisionamiento con Azure AD

1. Inicie sesión en [BrowserStack](https://www.browserstack.com/users/sign_in) como usuario con permisos de **propietario**.

2. Vaya a **Cuenta** -> **Configuración y permisos**. Seleccione la pestaña **Seguridad** .

3. En **Aprovisionamiento automático de usuarios**, haga clic en **Configurar**.

    ![Configuración](media/browserstack-single-sign-on-provisioning-tutorial/configure.png)

4. Seleccione los atributos de usuario que desea controlar mediante Azure AD y haga clic en **Confirmar**.

    ![Usuario](media/browserstack-single-sign-on-provisioning-tutorial/attributes.png)

5. Copie los valores de **URL de inquilino** y **Token secreto**. Estos valores se escriben en el campo URL de inquilino y Token secreto de la pestaña Aprovisionamiento de la aplicación BrowserStack Single Sign-on en Azure Portal. Haga clic en **Done**(Listo).

    ![Authorization](media/browserstack-single-sign-on-provisioning-tutorial/credential.png)

6. La configuración de aprovisionamiento se ha guardado en BrowserStack. **Habilite** el aprovisionamiento de usuarios en BrowserStack una vez completada **la configuración de aprovisionamiento en Azure AD** para evitar el bloqueo de las invitaciones a nuevos usuarios desde la [cuenta](https://www.browserstack.com/accounts/manage-users) de BrowserStack. 

    ![Cuenta](media/browserstack-single-sign-on-provisioning-tutorial/enable.png)
    
## <a name="step-3-add-browserstack-single-sign-on-from-the-azure-ad-application-gallery"></a>Paso 3. Incorporación de BrowserStack Single Sign-on desde la galería de aplicaciones de Azure AD

Agregue BrowserStack Single Sign-on desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento en BrowserStack Single Sign-on. Si ha configurado previamente BrowserStack Single Sign-on para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario. Si elige delimitar quién se aprovisionará en la aplicación en función de la asignación, puede usar los [pasos](../manage-apps/assign-user-or-group-access-portal.md) siguientes para asignar usuarios a la aplicación. Si elige delimitar quién se aprovisionará en función únicamente de los atributos del usuario, puede usar un filtro de ámbito, tal como se describe [aquí](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Al asignar usuarios y grupos a BrowserStack Single Sign-on, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios asignados, puede controlarlo asignando uno o dos usuarios a la aplicación. Cuando el ámbito se establece en todos los usuarios, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-browserstack-single-sign-on"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en BrowserStack Single Sign-on 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD a fin de crear, actualizar y deshabilitar usuarios en función de las asignaciones de usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-browserstack-single-sign-on-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para BrowserStack Single Sign-on en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **BrowserStack Single Sign-on**.

    ![En la lista de aplicaciones, seleccione el vínculo BrowserStack Single Sign-on.](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña de aprovisionamiento automático](common/provisioning-automatic.png)

5. En la sección **Credenciales del administrador**, escriba la dirección URL del inquilino de BrowserStack Single Sign-on y el token secreto. Haga clic en **Probar conexión** para asegurarse de que Azure AD pueda conectarse a BrowserStack Single Sign-on. Si la conexión no se establece, asegúrese de que la cuenta de BrowserStack Single Sign-on tenga permisos de administrador y pruebe otra vez.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Sincronizar usuarios de Azure Active Directory con BrowserStack Single Sign-on**.

9. Revise los atributos de usuario que se sincronizan entre Azure AD y BrowserStack Single Sign-on en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de BrowserStack Single Sign-on con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de BrowserStack Single Sign-on admite el filtrado de usuarios en función de ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|--|
   |userName|String|&check;|
   |name.givenName|String|
   |name.familyName|String|
   |urn:ietf:params:scim:schemas:extension:Bstack:2.0:User:bstack_role|String|
   |urn:ietf:params:scim:schemas:extension:Bstack:2.0:User:bstack_team|String|
   |urn:ietf:params:scim:schemas:extension:Bstack:2.0:User:bstack_product|String|


10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Para habilitar el servicio de aprovisionamiento de Azure AD para BrowserStack Single Sign-on, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Elija los valores deseados en **Ámbito** en la sección **Configuración** para definir los usuarios que quiere que se aprovisionen en BrowserStack Single Sign-on.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

- Use los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
- Consulte la [barra de progreso](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
- Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="connector-limitations"></a>Limitaciones del conector

* BrowserStack Single Sign-on no admite el aprovisionamiento de grupos.
* BrowserStack Single Sign-on requiere que **emails[type eq "work"].value** y **userName** tengan el mismo valor de origen.

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

* Consulte la [guía de solución de problemas](https://www.browserstack.com/docs/enterprise/auto-user-provisioning/azure-ad#troubleshooting).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configuración de asignaciones de atributos en BrowserStack Single Sign-on](https://www.browserstack.com/docs/enterprise/auto-user-provisioning/azure-ad)
* [Configuración y habilitación del aprovisionamiento automático de usuarios en BrowserStack](https://www.browserstack.com/docs/enterprise/auto-user-provisioning/azure-ad#setup-and-enable-auto-user-provisioning)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)