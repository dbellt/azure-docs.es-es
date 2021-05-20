---
title: 'Tutorial: Configuración de ServiceNow para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y desaprovisionar automáticamente cuentas de usuario de Azure AD en ServiceNow.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/10/2021
ms.author: jeedes
ms.openlocfilehash: afad903734fded062681f18683cb74492993fe1c
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684224"
---
# <a name="tutorial-configure-servicenow-for-automatic-user-provisioning"></a>Tutorial: Configuración de ServiceNow para el aprovisionamiento automático de usuarios

En este tutorial, se describen los pasos que debe realizar en ServiceNow y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando Azure AD está configurado, aprovisiona y desaprovisiona automáticamente usuarios y grupos en [ServiceNow](https://www.servicenow.com/) utilizando su servicio de aprovisionamiento. 

Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en ServiceNow
> * Eliminación de usuarios de ServiceNow cuando ya no necesiten tener acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y ServiceNow
> * Aprovisionamiento de grupos y pertenencias a grupos en ServiceNow
> * Permiso para utilizar el [inicio de sesión único](servicenow-tutorial.md) en ServiceNow (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario de Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (como administrador de aplicaciones, administrador de aplicaciones en la nube, propietario de la aplicación o administrador global).
* Una [instancia de ServiceNow](https://www.servicenow.com/) de Calgary o superior
* Una [instancia de ServiceNow Express](https://www.servicenow.com/) de Helsinki o superior
* Una cuenta de usuario de ServiceNow con el rol de administrador

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1: Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
2. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Determine qué datos [se asignarán entre Azure AD y ServiceNow](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-servicenow-to-support-provisioning-with-azure-ad"></a>Paso 2: Configuración de ServiceNow para admitir el aprovisionamiento con Azure AD

1. Identifique el nombre de la instancia de ServiceNow. Puede encontrar el nombre de instancia en la dirección URL que usa para acceder a ServiceNow. En el ejemplo siguiente, el nombre de la instancia es **dev35214**.

   ![Captura de pantalla de una instancia de ServiceNow.](media/servicenow-provisioning-tutorial/servicenow-instance.png)

2. Obtenga las credenciales de un administrador de ServiceNow. Vaya al perfil del usuario de ServiceNow y compruebe que tiene el rol de administrador. 

   ![Captura de pantalla de un rol de administrador de ServiceNow.](media/servicenow-provisioning-tutorial/servicenow-admin-role.png)


## <a name="step-3-add-servicenow-from-the-azure-ad-application-gallery"></a>Paso 3: Agregar ServiceNow desde la galería de aplicaciones de Azure AD

Agregue ServiceNow desde la galería de aplicaciones de Azure AD para empezar a administrar el aprovisionamiento en ServiceNow. Si previamente configuró el inicio de sesión único (SSO) en ServiceNow, podrá utilizar la misma aplicación. No obstante, es recomendable que cree una aplicación independiente para probar la integración. [Más información sobre cómo agregar una aplicación desde la galería](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4: Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se va a aprovisionar en función de las asignaciones de la aplicación o de los atributos del usuario o grupo. Si elige determinar el ámbito de los usuarios a los que se aprovisionará la aplicación en función de la asignación, puede usar los [pasos para asignar usuarios y grupos a la aplicación](../manage-apps/assign-user-or-group-access-portal.md). Si elige determinar el ámbito de los usuarios a los que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede [usar un filtro de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

Tenga en cuenta estas sugerencias:

* Al asignar usuarios y grupos a ServiceNow, debe seleccionar un rol que no sea el rol de acceso predeterminado. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de la aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar más roles. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando se establece el ámbito de aprovisionamiento en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-servicenow"></a>Paso 5: Configurar el aprovisionamiento automático de usuarios en ServiceNow 

En esta sección, encontrará directrices para configurar el servicio de aprovisionamiento de Azure AD, así como para crear, actualizar y deshabilitar usuarios y grupos en TestApp. Puede basar la configuración en las asignaciones de usuarios y grupos de Azure AD.

Para configurar el aprovisionamiento automático de usuarios para ServiceNow en Azure AD, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Captura de pantalla que muestra el panel de aplicaciones empresariales.](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **ServiceNow**.

    ![Captura de pantalla que muestra una lista de aplicaciones.](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Captura de pantalla de las opciones de administración con la opción Aprovisionamiento seleccionada.](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Captura de pantalla de la lista desplegable Modo de aprovisionamiento con la opción Automático seleccionada.](common/provisioning-automatic.png)

5. En la sección **Credenciales de administrador**, escriba las credenciales del administrador y el nombre de usuario de ServiceNow. Para asegurarse de que Azure AD puede conectarse a ServiceNow, seleccione **Probar conexión**. Si la conexión no se establece, asegúrese de que la cuenta de ServiceNow tiene permisos de administrador e inténtelo de nuevo.

    ![Captura de pantalla de la página de aprovisionamiento del servicio, donde puede especificar las credenciales de administrador.](./media/servicenow-provisioning-tutorial/servicenow-provisioning.png)

6. En el cuadro **Notification Email** (Dirección de correo electrónico para notificaciones), escriba la dirección de correo electrónico de la persona o grupo que deben recibir las notificaciones de error de aprovisionamiento. A continuación, active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Captura de pantalla que muestra los cuadros del correo electrónico de notificación.](common/provisioning-notification-email.png)

7. Seleccione **Guardar**.

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to** (Sincronizar usuarios de Azure Active Directory con ServiceNow).

9. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y ServiceNow. Los atributos seleccionados como propiedades **Matching** se usan para buscar coincidencias con las cuentas de usuario de ServiceNow con el objetivo de realizar operaciones de actualización. 

   Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de ServiceNow admite el filtrado de usuarios en función de ese atributo. 
   
   Seleccione el botón **Guardar** para confirmar los cambios.

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to ServiceNow** (Sincronizar grupos de Azure Active Directory con ServiceNow).

11. En la sección **Asignaciones de atributos**, revise los atributos de grupo que se sincronizan entre Azure AD y ServiceNow. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de ServiceNow con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

12. Para configurar los filtros de ámbito, consulte las instrucciones que se describen en [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Para habilitar el servicio de aprovisionamiento de Azure AD en ServiceNow, cambie **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Captura de pantalla que muestra el estado de aprovisionamiento activado.](common/provisioning-toggle-on.png)

14. En la sección **Configuración**, elija los valores de **Ámbito** que correspondan para definir los usuarios o grupos que desea que se aprovisionen en ServiceNow.

    ![Captura de pantalla de las opciones del ámbito de aprovisionamiento.](common/provisioning-scope.png)

15. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Captura de pantalla del botón para guardar una configuración de aprovisionamiento.](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo inicial tarda más tiempo en completarse que los siguientes. Los ciclos posteriores tendrán lugar aproximadamente cada 40 minutos, siempre que el servicio de aprovisionamiento de Azure AD esté en ejecución. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6: Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

- Use los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para determinar qué usuarios se han aprovisionado correctamente y los que no.
- Consulte la [barra de progreso](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
- Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. [Más información sobre los estados de cuarentena](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas
* Cuando aprovisione ciertos atributos (como **Department** y **Location**) en ServiceNow, es necesario que los valores existan en una tabla de referencia de ServiceNow. De lo contrario, aparecerá el error **InvalidLookupReference**. 

   Por ejemplo, imagine que tiene dos ubicaciones (Seattle, Los Angeles) y tres departamentos (ventas, finanzas, marketing) en una tabla de ServiceNow. Si intenta aprovisionar un usuario cuyo departamento sea "ventas" y cuya ubicación sea "Seattle", la operación se realizará correctamente. Si intenta aprovisionar un usuario cuyo departamento sea "ventas" y cuya ubicación sea "LA", no se realizará el aprovisionamiento. Es necesario agregar la ubicación "LA" a la tabla de referencia de ServiceNow o actualizar el atributo del usuario en Azure AD para que coincida con el formato en ServiceNow. 
* Si se produce el error **EntryJoiningPropertyValueIsMissing**, revise las [asignaciones de atributos](../app-provisioning/customize-application-attributes.md) para identificar el atributo que corresponda. Este valor debe estar presente en el usuario o grupo que esté intentando aprovisionar. 
* Revise la [API SOAP de ServiceNow](https://docs.servicenow.com/bundle/newyork-application-development/page/integrate/web-services-apis/reference/r_DirectWebServiceAPIFunctions.html) para conocer los requisitos o las limitaciones (por ejemplo, el formato que debe utilizarse para especificar el código de país de un usuario).
* Las solicitudes de aprovisionamiento se envían de forma predeterminada a https://{nombre-de-instancia}.service-now.com/{nombre-de-tabla}. Si necesita una dirección URL de inquilino personalizada, puede proporcionar la dirección URL completa como nombre de la instancia.
* El error **ServiceNowInstanceInvalid** indica que se ha producido un problema al comunicarse con la instancia de ServiceNow. Este es el texto del mensaje de error:
  
  `Details: Your ServiceNow instance name appears to be invalid.  Please provide a current ServiceNow administrative user name and          password along with the name of a valid ServiceNow instance.`                                                              

  Si tiene problemas para probar la conexión, intente seleccionar **No** en las siguientes opciones de ServiceNow:
   
  - **Seguridad del sistema** > **High security settings** (Configuración de alta seguridad) > **Require basic authentication for incoming SCHEMA requests** (Requerir autenticación básica para las solicitudes de SCHEMA entrantes)
  - **Propiedades del sistema** > **Servicios web** > **Require basic authorization for incoming SOAP requests** (Requerir autenticación básica para las solicitudes de SOAP entrantes)

     ![Captura de pantalla de la opción para autorizar las solicitudes de SOAP.](media/servicenow-provisioning-tutorial/servicenow-webservice.png)

  Si el problema sigue sin resolverse, póngase en contacto con el equipo de soporte de ServiceNow y pídales que activen la depuración de SOAP. 

* El servicio de aprovisionamiento de Azure AD funciona actualmente en [intervalos IP](../app-provisioning/use-scim-to-provision-users-and-groups.md#ip-ranges) concretos. Si es necesario, puede restringir otros intervalos IP y agregar estos intervalos IP concretos a la lista de permitidos de la aplicación. Esta técnica permitirá el flujo de tráfico desde el servicio de aprovisionamiento de Azure AD a la aplicación.

* No se admiten las instancias de ServiceNow autohospedadas. 

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
