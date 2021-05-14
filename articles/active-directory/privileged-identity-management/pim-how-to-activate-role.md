---
title: 'Activación de mis roles de Azure AD en PIM: Azure Active Directory | Microsoft Docs'
description: Aprenda a activar los roles de Azure AD en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 03/15/2021
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99829d6afbec638f26ae91fcb73bd20220c54ffa
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109481902"
---
# <a name="activate-my-azure-ad-roles-in-pim"></a>Activación de mis roles de Azure AD en PIM

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) simplifica la forma en que las empresas administran el acceso con privilegios a los recursos de Azure AD y otros servicios en línea de Microsoft, como Microsoft 365 o Microsoft Intune.  

Si se le ha considerado *elegible* para un rol administrativo, debe *activar* la asignación de roles cuando necesite realizar acciones con privilegios. Por ejemplo, si administra de vez en cuando características de Microsoft 365, es posible que los administradores de roles con privilegios de su organización no le hayan asignado el rol de administrador global permanente, ya que ese rol afecta también a otros servicios. En su lugar, podrían haberle hecho elegible para roles de Azure AD, como administrador de Exchange Online. Puede solicitar la activación de ese rol cuando necesite sus privilegios y tendrá control de administrador durante un período predeterminado.

Este artículo está dirigido a los administradores que necesitan activar su rol de Azure AD en Privileged Identity Management.

## <a name="determine-your-version-of-pim"></a>Determinación de la versión de PIM

Desde noviembre de 2019, la parte de roles de Azure AD de Privileged Identity Management se está actualizando a una nueva versión que coincide con las experiencias de los roles de recursos de Azure. Esta actualización introduce características adicionales y [cambios en la API existente](azure-ad-roles-features.md#api-changes). Mientras se implementa la nueva versión, los procedimientos que seguirá en este artículo dependerán de la versión de Privileged Identity Management que tenga actualmente. Siga los pasos de esta sección para determinar la versión de Privileged Identity Management que tiene. Cuando averigüe la versión de Privileged Identity Management, puede seleccionar los procedimientos de este artículo que coincidan con esa versión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con el rol [Administrador de roles con privilegios](../roles/permissions-reference.md#privileged-role-administrator).
1. Abra **Azure AD Privileged Identity Management**. Si tiene un banner en la parte superior de la página de introducción, siga las instrucciones de la pestaña **Nueva versión** de este artículo. De lo contrario, siga las instrucciones de la pestaña **Versión anterior**.

    [![Selección de Azure AD > Privileged Identity Management](media/pim-how-to-add-role-to-user/pim-new-version.png)](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox)

# <a name="new-version"></a>[Nueva versión](#tab/new)

## <a name="activate-a-role-for-new-version"></a>Activación de un rol de nueva versión

Cuando necesite asumir un rol de Azure AD, puede solicitar la activación al abrir **Mis roles** en Privileged Identity Management.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra **Azure AD Privileged Identity Management**. Para información sobre cómo agregar el icono de Privileged Identity Management en el panel, consulte [Primer uso de PIM](pim-getting-started.md).

1. Seleccione **Mis roles** y, a continuación, seleccione **Roles de Azure AD** para ver una lista de los roles de Azure AD válidos.

    ![Mi página de roles en la que se muestran los roles que el usuario puede activar](./media/pim-how-to-activate-role/my-roles.png)

1. En la lista **Roles de Azure AD**, busque el rol que desea activar.

    ![Roles de Azure AD: mi lista de roles elegibles](./media/pim-how-to-activate-role/activate-link.png)

1. Seleccione **Activar** para abrir el Panel de activación.

    ![Roles de Azure AD: la página de activación contiene la duración y el ámbito](./media/pim-how-to-activate-role/activate-page.png)

1. Seleccione **Additional verification required**"** (Verificación adicional requerida) y siga las instrucciones para la verificación de seguridad adicional. Se le requiere que se autentique solo una vez por sesión.

    ![Pantalla para proporcionar una comprobación de seguridad, como un código PIN](./media/pim-resource-roles-activate-your-roles/resources-mfa-enter-code.png)

1. Después de la autenticación multifactor, seleccione **Activate before proceeding** (Activar antes de continuar).

    ![Comprobación de mi identidad mediante la autenticación multifactor antes de la activación del rol](./media/pim-how-to-activate-role/activate-role-mfa-banner.png)

1. Si desea especificar un ámbito reducido, seleccione **Ámbito** para abrir el panel de filtro. En el panel de filtro, puede especificar los recursos de Azure AD a los que necesita acceso. Se recomienda solicitar acceso al menor número de recursos que necesite.

1. Si es necesario, especifique una hora de inicio de activación personalizada. El rol de Azure AD se activaría después de la hora seleccionada.

1. En el campo **Razón**, escriba el motivo de la solicitud de activación.

1. Seleccione **Activar**.

    Si el [rol requiere aprobación](pim-resource-roles-approval-workflow.md) para activarse, aparecerá una notificación en la esquina superior del explorador que le informa de que la solicitud está pendiente de aprobación.

    ![La solicitud de activación está pendiente de notificación de aprobación](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate-notification.png)

## <a name="view-the-status-of-your-requests-for-new-version"></a>Visualización del estado de las solicitudes de nueva versión

Puede ver el estado de las solicitudes pendientes de activación.

1. Abra Azure AD Privileged Identity Management.

1. Seleccione **Mis solicitudes** para ver una lista de los roles de Azure AD y las solicitudes de rol de recursos de Azure.

    ![Mis solicitudes: página de Azure AD que muestra las solicitudes pendientes](./media/pim-how-to-activate-role/my-requests-page.png)

1. Desplácese a la derecha para ver la columna **Estado de solicitud**.

## <a name="cancel-a-pending-request-for-new-version"></a>Cancelación de una solicitud pendiente de nueva versión

Si no necesita activar un rol que requiera aprobación, puede cancelar una solicitud pendiente en cualquier momento.

1. Abra Azure AD Privileged Identity Management.

1. Seleccione **Mis solicitudes**.

1. En el rol que desea cancelar, seleccione el vínculo **Cancelar**.

    Al seleccionar cancelar, se cancelará la solicitud. Para volver a activar el rol, tendrá que enviar una nueva solicitud de activación.

   ![Mi lista de solicitudes con la acción Cancelar resaltada](./media/pim-resource-roles-activate-your-roles/resources-my-requests-cancel.png)

## <a name="troubleshoot-for-new-version"></a>Solución de problemas de nueva versión

### <a name="permissions-are-not-granted-after-activating-a-role"></a>Permisos no concedidos después de activar un rol

Al activar un rol en Privileged Identity Management, la activación podría no propagarse al instante a todos los portales que requieren el rol con privilegios. A veces, incluso si el cambio se propaga, el almacenamiento en caché web en un portal puede provocar que el cambio no surta efecto de inmediato. Si la activación se retrasa, cierre la sesión del portal en el que está intentando realizar la acción y vuelva a iniciarla. En Azure Portal, PIM cierra sesión y vuelva a iniciarla automáticamente.

# <a name="previous-version"></a>[Versión anterior](#tab/previous)

## <a name="activate-a-role-previous-version"></a>Activación de un rol (versión anterior)

Cuando necesite asumir un rol de Azure AD, puede solicitar la activación mediante la opción de navegación **Mis roles** de Privileged Identity Management.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra **Azure AD Privileged Identity Management**. Para información sobre cómo agregar el icono de Privileged Identity Management en el panel, consulte [Primer uso de PIM](pim-getting-started.md).

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Mis roles** para ver una lista de sus roles de Azure AD elegibles.

    ![Roles de Azure AD: mis roles en los que se muestra la lista de roles elegibles o activos](./media/pim-how-to-activate-role/directory-roles-my-roles.png)

1. Buscar el rol que desea activar.

    ![Roles de Azure AD: mi lista de roles elegibles que muestra el vínculo Activar](./media/pim-how-to-activate-role/directory-roles-my-roles-activate.png)

1. Seleccione **Activar** para abrir el panel de detalles de activación de rol.

1. Si el rol requiere la autenticación multifactor (MFA), seleccione **Compruebe su identidad antes de proceder**. Solo tiene que autenticarse una vez por sesión.

    ![Comprobación del panel mi identidad con MFA antes de la activación del rol](./media/pim-how-to-activate-role/directory-roles-my-roles-mfa.png)

1. Seleccione **Comprobar mi identidad** y siga las instrucciones para proporcionar comprobación de seguridad adicional.

    ![Página de comprobación de seguridad adicional que le pregunta cómo contactar con usted](./media/pim-how-to-activate-role/additional-security-verification.png)

1. Seleccione **Activar** para abrir el panel Activación.

    ![Panel de activación para especificar la hora de inicio, la duración, el vale y el motivo](./media/pim-how-to-activate-role/directory-roles-activate.png)

1. Si es necesario, especifique una hora de inicio de activación personalizada.

1. Especifique la duración de la activación.

1. En el campo **Motivo de la activación**, escriba el motivo de la solicitud de activación. Algunos roles requieren que proporcione un número de vale de problema.

    ![Panel de activación completado con un hora de inicio personalizada, la duración, el vale y el motivo](./media/pim-how-to-activate-role/directory-roles-activation-pane.png)

1. Seleccione **Activar**.

    Si el rol no requiere aprobación, aparecerá el panel **Estado de activación**, donde se muestra el estado de la activación.

    ![Página de estado de activación que muestra las tres fases de activación](./media/pim-how-to-activate-role/activation-status.png)

    Una vez que se hayan completado todas las fases, seleccione el vínculo **Cerrar sesión** para cerrar la sesión en Azure Portal. Ahora, cuando vuelva a iniciar sesión en el portal, podrá usar el rol.

    Si el [rol requiere aprobación](./azure-ad-pim-approval-workflow.md) para activarse, aparecerá una notificación de Azure en la esquina superior del explorador que le informa de que la solicitud está pendiente de aprobación.

## <a name="view-the-status-of-your-requests-previous-version"></a>Vista del estado de las solicitudes (versión anterior)

Puede ver el estado de las solicitudes pendientes de activación.

1. Abra Azure AD Privileged Identity Management.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Mis solicitudes** para ver una lista de sus solicitudes.

    ![Roles de Azure AD: lista Mis solicitudes](./media/pim-how-to-activate-role/directory-roles-my-requests.png)

## <a name="deactivate-a-role-previous-version"></a>Desactivación de un rol (versión anterior)

Cuando se ha activado un rol, si alcanza su límite de tiempo (de duración elegible), se desactiva automáticamente.

Si completa pronto sus tareas de administrador, también puede desactivar un rol de forma manual en Azure AD Privileged Identity Management.

1. Abra Azure AD Privileged Identity Management.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Mis roles**.

1. Seleccione **Roles activos** para ver su lista de roles activos.

1. Busque el rol que ha terminado de usar y haga clic en **Desactivar**.

## <a name="cancel-a-pending-request-previous-version"></a>Cancelación de una solicitud pendiente (versión anterior)

Si no necesita activar un rol que requiera aprobación, puede cancelar una solicitud pendiente en cualquier momento.

1. Abra Azure AD Privileged Identity Management.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Mis solicitudes**.

1. En el rol que desea cancelar, seleccione el botón **Cancelar**.

    Al seleccionar **Cancelar**, se cancelará la solicitud. Para volver a activar el rol, tendrá que enviar una nueva solicitud de activación.

   ![Lista Mis solicitudes con el botón Cancelar resaltado](./media/pim-how-to-activate-role/directory-role-cancel.png)

## <a name="troubleshoot-previous-version"></a>Solución de problemas (versión anterior)

### <a name="permissions-are-not-granted-after-activating-a-role"></a>Permisos no concedidos después de activar un rol

Al activar un rol en Privileged Identity Management, la activación se puede retrasar en portales de administración distintos de Azure Portal, como el portal de Office 365. Si la activación se retrasa, cierre la sesión del portal en el que se encuentra y, a continuación, vuelva a iniciarla. Use Privileged Identity Management para comprobar que aparece como miembro del rol.

 ---

## <a name="next-steps"></a>Pasos siguientes

- [Activar mis roles de Azure AD en Privileged Identity Management](pim-how-to-activate-role.md)
