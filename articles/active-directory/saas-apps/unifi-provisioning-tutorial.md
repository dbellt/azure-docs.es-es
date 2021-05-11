---
title: 'Tutorial: Configuración de UNIFI para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y desaprovisionar de forma automática las cuentas de usuario de Azure AD para UNIFI.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 924c603f-574e-4e0a-9345-0cb0c7593dbb
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/20/2021
ms.author: Zhchia
ms.openlocfilehash: 7e9e9667c18375f9ec459f7e8c038d68f9b54299
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108187429"
---
# <a name="tutorial-configure-unifi-for-automatic-user-provisioning"></a>Tutorial: Configuración de UNIFI para aprovisionar usuarios automáticamente

En este tutorial, se describen los pasos que debe realizar en UNIFI y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y desaprovisiona usuarios y grupos de manera automática en [UNIFI](http://www.unifilabs.com/) mediante su servicio de aprovisionamiento correspondiente. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Crear usuarios en UNIFI
> * Quitar usuarios de UNIFI cuando ya no necesiten acceso
> * Mantener los atributos de usuario sincronizados entre Azure AD y UNIFI
> * Aprovisionar grupos y pertenencias a grupos en UNIFI
> * Realizar el [inicio de sesión único](unifi-tutorial.md) en UNIFI (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario en Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Un inquilino de [UNIFI](http://www.unifilabs.com/).
* Una cuenta de usuario de UNIFI con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
1. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Determine qué datos quiere [asignar entre Azure AD y UNIFI](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-unifi-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de UNIFI para admitir el aprovisionamiento con Azure AD

1. Asegúrese de que el inicio de sesión único esté habilitado correctamente en Enterprise Application en Azure. 
1. Encuentre la **Dirección URL de inicio de sesión** en Inicio de sesión único. En nuestro caso, esta es `https://login.microsoftonline.com/<guid>/saml2`.
1. Descargue el certificado (Base64) en la sección Certificado de firma de SAML.

    ![Vista del inicio de sesión único de Enterprise Application](media/unifi-provisioning-tutorial/enterprise-application-view.png)

1. Si el proveedor de identidades no se ha agregado a UNIFI, inicie sesión en el portal de UNIFI como **administrador de empresa**. Vaya a **Users (Usuarios) -> Configure SSO (Configurar inicio de sesión único) -> botón add provider (agregar proveedor)** .

    ![Vista para agregar un proveedor de identidades](media/unifi-provisioning-tutorial/add-identity-provider-view.png)

1. Se mostrará el cuadro de diálogo modal para agregar un proveedor de inicio de sesión único.

    ![Cuadro de diálogo modal para agregar proveedores de identidades](media/unifi-provisioning-tutorial/add-identity-provider-modal.png)

1. Proporcione un valor **Name** (Nombre) único de su preferencia. La dirección **URL** será la misma **Dirección URL de inicio de sesión** de la aplicación empresarial de Azure AD. Proporcione un valor cualquiera para **Token**. Coloque el valor de Certificado (Base64) en el campo **Certificado**. Si quiere que todos los usuarios creados a partir de este punto usen este proveedor de identidades, seleccione la casilla **Make this the default identity provider** (Establecer como predeterminado este proveedor de identidades).

    ![Cuadro de diálogo modal para añadir un proveedor de identidades relleno](media/unifi-provisioning-tutorial/add-identity-provider-modal-populated.png)

1. Haga clic en el botón SAVE (GUARDAR).

## <a name="step-3-add-unifi-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de UNIFI desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de UNIFI, agregue UNIFI desde la galería de aplicaciones de Azure AD. Si ya ha configurado previamente UNIFI para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Al asignar usuarios y grupos a UNIFI, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar roles adicionales. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-unifi"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en UNIFI 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en UNIFI en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-unifi-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para UNIFI en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **UNIFI**.

    ![Vínculo a UNIFI en la lista de aplicaciones](common/all-applications.png)

1. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

1. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña de aprovisionamiento automático](common/provisioning-automatic.png)

1. En la sección **Credenciales de administrador**, escriba los valores de **URL de inquilino** -`https://licensing.inviewlabs.com/api/scim/v2/` y **Token secreto** de UNIFI. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a UNIFI. Si la conexión no se establece, asegúrese de que la cuenta de UNIFI tiene permisos de administrador y pruebe de nuevo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

1. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que deba recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

1. Seleccione **Guardar**.

1. En la sección de **asignaciones**, seleccione **Synchronize Azure Active Directory Users to UNIFI** (Sincronizar usuarios de Azure Active Directory con UNIFI).

1. Revise los atributos de usuario que se sincronizan entre Azure AD y UNIFI en la sección **Attribute-Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades de **coincidencia** se usan para establecer coincidencias con las cuentas del usuario de UNIFI con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de UNIFI admita el filtrado de usuarios basado en ese atributo. Seleccione el botón **Guardar** para confirmar los cambios.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|---|
   |userName|String|&check;
   |active|Boolean|   
   |name.givenName|String|
   |name.familyName|String|
   |externalId|String|


1. En la sección de **asignaciones**, seleccione **Synchronize Azure Active Directory Groups to UNIFI** (Sincronizar grupos de Azure Active Directory con UNIFI).

1. Revise los atributos de grupo que se sincronizan entre Azure AD y UNIFI en la sección **Attribute-Mapping** (Asignación de atributos). Los atributos seleccionados como propiedades de **coincidencia** se usan para establecer correspondencia con los grupos de UNIFI para operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

      |Atributo|Tipo|Compatible con el filtrado|
      |---|---|---|
      |DisplayName|String|&check;
      |members|Referencia|
      |externalId|String|      

1. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Para habilitar el servicio de aprovisionamiento de Azure AD para UNIFI, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

1. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en UNIFI.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

1. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. 

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación
Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

* Use los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para determinar qué usuarios se han aprovisionado correctamente o sin éxito.
* Consulte la [barra de progreso](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
* Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Más información sobre los estados de cuarentena [aquí](../app-provisioning/application-provisioning-quarantine-status.md).  

## <a name="more-resources"></a>Más recursos

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)