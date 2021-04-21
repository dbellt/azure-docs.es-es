---
title: 'Tutorial: Configuración de Jostle para aprovisionar usuarios automáticamente con Azure Active Directory | Microsoft Docs'
description: Aprenda a aprovisionar y desaprovisionar de forma automática las cuentas de usuario de Azure AD en Jostle.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 6dbb744f-8b8e-4988-b293-ebe079c8c5c5
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/05/2021
ms.author: Zhchia
ms.openlocfilehash: d2ab0009f036afa38dc9e401223854a034d45e42
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107368553"
---
# <a name="tutorial-configure-jostle-for-automatic-user-provisioning"></a>Tutorial: Configuración de Jostle para aprovisionar usuarios automáticamente

En este tutorial, se describen los pasos que debe realizar en Jostle y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando está configurado, Azure AD aprovisiona y desaprovisiona automáticamente usuarios y grupos en [Jostle](https://www.jostle.me/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../manage-apps/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en Jostle
> * Eliminación de usuarios de Jostle cuando ya no necesiten acceso
> * Mantenimiento de los atributos de usuario sincronizados entre Azure AD y Jostle
> * [Inicio de sesión único](jostle-tutorial.md) en Jostle (recomendado)

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario en Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Un [inquilino de Jostle](https://www.jostle.me/).
* Una cuenta de usuario de Jostle con permisos de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento

1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
1. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Determine qué datos se van a [asignar entre Azure AD y Jostle](../app-provisioning/customize-application-attributes.md).

## <a name="step-2-configure-jostle-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Jostle para que admita el aprovisionamiento con Azure AD

### <a name="automation-account"></a>Cuenta de Automation

Antes de empezar, deberá crear un **usuario de Automation** en la intranet de Jostle. Esta será la cuenta que usará para configurar con Azure. Los usuarios de Automation se pueden crear como administrador en **Configuración > User accounts and data (Cuentas de usuario y datos) > Manage Automation users (Administrar usuarios de Automation)** .

Para más información sobre los usuarios de Automation y cómo crear uno, consulte [este artículo](https://forum.jostle.us/hc/en-us/articles/360057364073).

Una vez creada, **se debe activar** la cuenta de usuario de Automation (es decir, iniciar sesión en la intranet al menos una vez) para poder usarse para configurar Azure.

### <a name="manage-user-provisioning"></a>Administración del aprovisionamiento de usuarios

Antes de empezar, asegúrese de que la suscripción de la cuenta **incluye las características de inicio de sesión único y de aprovisionamiento de usuarios**. Si no es así, puede ponerse en contacto con el administrador de atención al cliente en <success@jostle.me> y este le ayudará a agregarlas a la cuenta.

El siguiente paso consiste en obtener la **dirección URL de API** y la **clave de API** de Jostle:

1. Vaya a Navegación principal y haga clic en **Configuración de administración**.
1. En **User data to/from other systems** (Datos de usuario hacia otros sistemas y desde estos) haga clic en **Manage user provisioning** (Administrar aprovisionamiento de usuarios). Si esta opción no aparece aquí y ha comprobado que su cuenta ya incluye el inicio de sesión único y el aprovisionamiento de usuarios, póngase en contacto con el equipo de soporte técnico en <support@jostle.me> para que le ayuden a habilitar esta página en la Configuración de administración.
1. En la sección **User Provisioning API details** (Detalles de la API de aprovisionamiento de usuarios), vaya al campo **Your Base URL** (Su dirección URL base), haga clic en el botón Copy (Copiar) y guarde la dirección URL en una ubicación a la que pueda acceder más adelante con facilidad.                                                           

   ![Aprovisionamiento](media/jostle-provisioning-tutorial/manage-user-provisioning.png)
                
1. A continuación, haga clic en el botón **Add a new key** (Agregar una nueva clave).
1. En la siguiente pantalla, vaya al campo **Automation User** (Usuario de Automation) y use el menú desplegable para seleccionar la cuenta de usuario de Automation. 

   ![Integration Account](media/jostle-provisioning-tutorial/select-integration-account.png)                                                                                                                                     
1. En el campo **Provisioning API key description** (Descripción de la clave de API de aprovisionamiento) asigne un nombre a la clave (por ejemplo, "Azure") y, a continuación, haga clic en el botón **Add** (Agregar).

1. Una vez generada la clave, **asegúrese de copiarla inmediatamente** y guárdela donde guardó la dirección URL (ya que esta será la única vez que aparezca la clave).                                                               
1. A continuación, usará la **dirección URL de API** y la **clave de API** para configurar la integración en Azure.
## <a name="step-3-add-jostle-from-the-azure-ad-application-gallery"></a>Paso 3. Adición de Jostle desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Jostle, agregue Jostle desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Jostle para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda que cree una aplicación independiente al probar la integración inicialmente. Puede encontrar más información sobre cómo agregar una aplicación desde la galería [aquí](../manage-apps/add-gallery-app.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento 

El servicio de aprovisionamiento de Azure AD le permite definir quién se aprovisionará, en función de la asignación a la aplicación y de los atributos del usuario o grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [aquí](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* Al asignar usuarios y grupos a Jostle, debe seleccionar un rol que no sea **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar más roles. 

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo en todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios o grupos a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-jostle"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en Jostle 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios y grupos en Jostle en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-jostle-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para Jostle en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Jostle**.

    ![Vínculo a Jostle en la lista de aplicaciones](common/all-applications.png)

1. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

1.  Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña de aprovisionamiento automático](common/provisioning-automatic.png)

1. En la sección **Credenciales de administrador**, especifique la información de **URL de inquilino** y de **Token secreto** de Jostle. Para asegurarse de que Azure AD puede conectarse a Jostle, seleccione **Probar conexión**. Si la conexión no se establece, asegúrese de que la cuenta de Jostle tenga permisos de administrador e inténtelo de nuevo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

1. En el campo **Dirección de correo electrónico para notificaciones**, escriba la dirección de correo electrónico de la persona o grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

1. Seleccione **Guardar**.

1. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Jostle** (Sincronizar usuarios de Azure Active Directory con Jostle).

1. Examine los atributos de usuario que se sincronizan entre Azure AD y Jostle en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Jostle con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de Jostle admite el filtrado de usuarios basado en ese atributo. Para confirmar los cambios, seleccione **Guardar**.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|---|
   |userName|String|&check;|
   |active|Boolean|
   |name.givenName|String|
   |name.familyName|String|
   |emails[type eq "work"].value|String|
   |emails[type eq "personal"].value|String|
   |emails[type eq "alternate1"].value|String|
   |emails[type eq "alternate2"].value|String|  
   |urn:ietf:params:scim:schemas:extension:jostle:2.0:User:alternateEmail1Label|String|
   |urn:ietf:params:scim:schemas:extension:jostle:2.0:User:alternateEmail2Label |String|  

1. Para configurar los filtros de ámbito, consulte las instrucciones que se proporcionan en [Tutorial de los filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Jostle, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

1. Seleccione los valores que quiera usar en **Ámbito** en la sección **Configuración** para definir los usuarios o grupos que quiere aprovisionar en Jostle.

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