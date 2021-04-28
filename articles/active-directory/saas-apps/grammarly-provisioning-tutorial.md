---
title: 'Tutorial: Configuración de Grammarly para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo aprovisionar y desaprovisionar automáticamente cuentas de usuario de Azure AD para Grammarly.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: cd2dd9d7-4901-40c8-8888-98850557b072
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/16/2021
ms.author: Zhchia
ms.openlocfilehash: 39d1e9e439ad1b14b6f6517868c21c682c45a7a3
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108130600"
---
# <a name="tutorial-configure-grammarly-for-automatic-user-provisioning"></a>Tutorial: Configuración de Grammarly para el aprovisionamiento automático de usuarios

En este tutorial se describen los pasos que debe realizar en Grammarly y Azure Active Directory (Azure AD) para configurar el aprovisionamiento automático de usuarios. Cuando se configura, Azure AD aprovisiona y cancela el aprovisionamiento de usuarios y grupos de manera automática en [Grammarly](https://www.grammarly.com/) mediante el servicio de aprovisionamiento de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Funcionalidades admitidas
> [!div class="checklist"]
> * Creación de usuarios en Grammarly
> * Eliminación de usuarios de Grammarly cuando ya no necesiten acceso
> * Sincronización de los atributos de usuario entre Azure AD y Grammarly

## <a name="prerequisites"></a>Requisitos previos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* [Un inquilino de Azure AD](../develop/quickstart-create-new-tenant.md) 
* Una cuenta de usuario en Azure AD con [permiso](../roles/permissions-reference.md) para configurar el aprovisionamiento (por ejemplo, Administrador de aplicaciones, Administrador de aplicaciones en la nube, Propietario de la aplicación o Administrador global). 
* Una cuenta de negocios de Grammarly con acceso de administrador.

## <a name="step-1-plan-your-provisioning-deployment"></a>Paso 1. Planeación de la implementación de aprovisionamiento
1. Obtenga información sobre [cómo funciona el servicio de aprovisionamiento](../app-provisioning/user-provisioning.md).
1. Determine quién estará en el [ámbito de aprovisionamiento](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
1. Determine qué datos se van a [asignar entre Azure AD y Grammarly](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-grammarly-to-support-provisioning-with-azure-ad"></a>Paso 2. Configuración de Grammarly para admitir el aprovisionamiento con Azure AD

Póngase en contacto con su representante de Grammarly o escriba en <support@grammarly.com> para solicitar el token de aprovisionamiento.

## <a name="step-3-add-grammarly-from-the-azure-ad-application-gallery"></a>Paso 3. Incorporación de Grammarly desde la galería de aplicaciones de Azure AD

Para empezar a administrar el aprovisionamiento de Grammarly, agregue Grammarly desde la galería de aplicaciones de Azure AD. Si ha configurado previamente Grammarly para el inicio de sesión único, puede usar la misma aplicación. Sin embargo, se recomienda crear una aplicación independiente al probar inicialmente la integración. Para obtener más información sobre cómo agregar una aplicación desde la galería, vea [este inicio rápido](../manage-apps/add-application-portal.md).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Paso 4. Determinar quién estará en el ámbito de aprovisionamiento

Puede usar el servicio de aprovisionamiento de Azure AD para definir quién se aprovisionará en función de la asignación a la aplicación o en función de los atributos del usuario o el grupo. Si elige el ámbito del que se aprovisionará en la aplicación en función de la asignación, puede usar los pasos [siguientes](../manage-apps/assign-user-or-group-access-portal.md) para asignar usuarios y grupos a la aplicación. Si elige el ámbito del que se aprovisionará en función únicamente de los atributos del usuario o grupo, puede usar un filtro de ámbito, tal como se describe [Aprovisionamiento de aplicaciones con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

* Al asignar usuarios y grupos a Grammarly, debe seleccionar un rol que no sea el de **Acceso predeterminado**. Los usuarios con el rol de acceso predeterminado se excluyen del aprovisionamiento y se marcarán como no autorizados en los registros de aprovisionamiento. Si el único rol disponible en la aplicación es el rol de acceso predeterminado, puede [actualizar el manifiesto de aplicación](../develop/howto-add-app-roles-in-azure-ad-apps.md) para agregar más roles.

* Empiece por algo pequeño. Pruebe con un pequeño conjunto de usuarios y grupos antes de implementarlo para todos. Cuando el ámbito del aprovisionamiento se define en los usuarios y grupos asignados, puede controlarlo asignando uno o dos usuarios, o grupos, a la aplicación. Cuando el ámbito se establece en todos los usuarios y grupos, puede especificar un [filtro de ámbito basado en atributos](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).


## <a name="step-5-configure-automatic-user-provisioning-to-grammarly"></a>Paso 5. Configuración del aprovisionamiento automático de usuarios en Grammarly

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en una aplicación de prueba en función de las asignaciones de grupos y usuarios de Azure AD.

### <a name="configure-automatic-user-provisioning-for-grammarly-in-azure-ad"></a>Configuración del aprovisionamiento automático de usuarios para Grammarly en Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Captura de pantalla que muestra el panel de aplicaciones empresariales.](common/enterprise-applications.png)

1. En la lista de aplicaciones, seleccione **Grammarly**.

    ![Captura de pantalla que muestra el vínculo de Grammarly en la lista de aplicaciones.](common/all-applications.png)

1. Seleccione la pestaña **Aprovisionamiento**.

    ![Captura de pantalla que muestra la pestaña Aprovisionamiento.](common/provisioning.png)

1. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Captura de pantalla que muestra Modo de aprovisionamiento establecido en Automático.](common/provisioning-automatic.png)

1. En la sección **Credenciales de administrador**, en el campo para especificar la **dirección URL de inquilino**, escriba `https://sso.grammarly.com/scim/v2` y, en el campo **Token secreto**, escriba el token que proporcionó Grammarly (consulte el paso 2 anterior). Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a Grammarly. Si la conexión no se establece, asegúrese de que la cuenta de Grammarly tiene permisos de administrador y pruebe de nuevo.

    ![Captura de pantalla que muestra los cuadros URL de inquilino y token secreto.](common/provisioning-testconnection-tenanturltoken.png)

1. En el cuadro **Dirección de correo electrónico para notificaciones**, escriba la dirección de correo electrónico de la persona o grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Captura de pantalla que muestra el cuadro de texto de la dirección de correo electrónico de notificación.](common/provisioning-notification-email.png)

1. Seleccione **Guardar**.

1. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Grammarly** (Sincronizar usuarios de Azure Active Directory con Grammarly).

1. Revise los atributos de usuario que se sincronizan entre Azure AD y Grammarly en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Grammarly con el objetivo de realizar operaciones de actualización. Si decide cambiar el [atributo de destino coincidente](../app-provisioning/customize-application-attributes.md), deberá asegurarse de que la API de Grammarly admite el filtrado de usuarios basado en ese atributo. Para confirmar los cambios, seleccione **Guardar**.

   |Atributo|Tipo|Compatible con el filtrado|
   |---|---|---|
   |userName|String|&check;|
   |externalId|String|
   |active|Boolean|
   |DisplayName|String|
   |emails[type eq "work"].value|String|


1. Para configurar los filtros de ámbito, consulte las instrucciones que se proporcionan en [Tutorial de los filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Para habilitar el servicio de aprovisionamiento de Azure AD para Grammarly, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Captura de pantalla que muestra el estado de aprovisionamiento activado.](common/provisioning-toggle-on.png)

1. Seleccione los valores que quiera usar en **Ámbito** en la sección **Configuración** para definir los usuarios o grupos que quiere aprovisionar en Grammarly.

    ![Captura de pantalla que muestra el ámbito de aprovisionamiento.](common/provisioning-scope.png)

1. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Captura de pantalla en la que se muestra el botón Guardar.](common/provisioning-configuration-save.png)

Esta operación inicia el ciclo de sincronización inicial de todos los usuarios y grupos definidos en **Ámbito** en la sección **Configuración**. El ciclo de sincronización inicial tarda más tiempo en realizarse que los ciclos posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose.

## <a name="step-6-monitor-your-deployment"></a>Paso 6. Supervisión de la implementación

Una vez configurado el aprovisionamiento, use los recursos siguientes para supervisar la implementación:

* Use los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para determinar los usuarios que se han aprovisionado correctamente y los que no.
* Consulte la [barra de progreso](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md) para ver el estado del ciclo de aprovisionamiento y cuánto falta para que finalice.
* Si la configuración de aprovisionamiento parece estar en mal estado, la aplicación pasará a estar en cuarentena. Para obtener más información sobre los estados de cuarentena, vea [Estado de cuarentena del aprovisionamiento de aplicaciones](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)