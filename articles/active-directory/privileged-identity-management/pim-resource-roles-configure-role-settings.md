---
title: Configure las opciones del rol de recursos de Azure en PIM - Azure AD | Microsoft Docs
description: Aprenda a configurar los roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 01/01/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 10242d867c5d9676b4f6833151863b820b2f11d1
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109481812"
---
# <a name="configure-azure-resource-role-settings-in-privileged-identity-management"></a>Configurar las opciones del rol de recursos de Azure en Privileged Identity Management

Cuando establece la configuración de roles de recursos de Azure, se define la configuración predeterminada que se aplica a las asignaciones de roles de recursos de Azure en Azure Active Directory (Azure AD) Privileged Identity Management (PIM). Para configurar el flujo de trabajo de aprobación y especificar quién puede aprobar o denegar solicitudes, use los siguientes procedimientos.

## <a name="open-role-settings"></a>Apertura de la configuración de roles

Siga estos pasos para abrir la configuración de un rol de recursos de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario en el rol [Administrador de roles con privilegios](../roles/permissions-reference.md#privileged-role-administrator).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Seleccione el recurso que desea administrar, por ejemplo, una suscripción o un grupo de administración.

    ![Página de recursos de Azure que muestra los recursos que se pueden administrar](./media/pim-resource-roles-configure-role-settings/resources-list.png)

1. Seleccione **Configuración**.

    ![Página de configuración de roles que muestra los roles de recursos de Azure](./media/pim-resource-roles-configure-role-settings/resources-role-settings.png)

1. Seleccione el rol cuya configuración desea configurar.

    ![Página de detalles de configuración de roles que muestra varias opciones de asignación y activación](./media/pim-resource-roles-configure-role-settings/resources-role-setting-details.png)

1. Seleccione **Editar** para abrir el panel **Edición de la configuración de roles**. La primera pestaña le permite actualizar la configuración de la activación del rol en Privileged Identity Management.

    ![Página Editar configuración de rol con la pestaña Activación abierta](./media/pim-resource-roles-configure-role-settings/role-settings-activation-tab.png)

1. Seleccione la pestaña **Asignación** o el botón **Siguiente: Asignación** en la parte inferior de la página para abrir la pestaña Configuración de asignación. Estas opciones controlan las asignaciones de roles realizadas dentro de la interfaz Privileged Identity Management.

    ![Pestaña Asignación de roles en la página Configuración de rol](./media/pim-resource-roles-configure-role-settings/role-settings-assignment-tab.png)

1. Use la pestaña **Notificación** o el botón **Siguiente: Activación** situado en la parte inferior de la página para ir a la pestaña Configuración de notificaciones de este rol. Estas opciones controlan todas las notificaciones de correo electrónico relacionadas con este rol.

    ![Pestaña Notificaciones de rol en la página Configuración de rol](./media/pim-resource-roles-configure-role-settings/role-settings-notification-tab.png)

    En la pestaña **Notificaciones** de la página Configuración de roles, Privileged Identity Management permite realizar un control pormenorizado sobre quién recibe las notificaciones y qué notificaciones reciben.

    - **Desactivación de un correo electrónico**<br>Si desactiva la casilla de destinatario predeterminado y elimina los destinatarios adicionales, puede desactivar correos electrónicos específicos.  

    - **Limite de correos electrónicos a direcciones especificadas**<br>Si desactiva la casilla destinatario predeterminado, puede desactivar los mensajes de correo electrónico enviados a destinatarios predeterminados. Después puede agregar direcciones de correo electrónico adicionales como destinatarios adicionales. Si desea agregar más de una dirección de correo electrónico, sepárelas con un punto y coma (;).

    - **Envío de correos electrónicos a destinatarios predeterminados y a destinatarios adicionales**<br>Si selecciona la casilla de destinatario predeterminado y agrega direcciones de correo electrónico para destinatarios adicionales, puede enviar correos electrónicos al destinatario predeterminado y a destinatarios adicionales.

    - **Solo correo electrónico crítico**<br>Para cada tipo de correo electrónico, puede activar la casilla para recibir solo correos electrónicos críticos. Esto significa que Privileged Identity Management continuará enviando mensajes de correo electrónico a los destinatarios configurados solo cuando el correo electrónico requiera una acción inmediata. Por ejemplo, no se desencadenarán los mensajes de correo electrónico que pidan a los usuarios que amplíen su asignación de roles, pero sí los que requieran que los administradores aprueben una solicitud de ampliación.

1. Seleccione el botón **Actualizar** en cualquier momento para actualizar la configuración del rol.

## <a name="assignment-duration"></a>Duración de la asignación

Puede elegir entre dos opciones de duración de asignación para cada tipo de asignación (Apto y Activo) cuando se configuran las opciones para un rol. Estas opciones se convierten en la duración máxima predeterminada cuando se asigna un usuario al rol en Privileged Identity Management.

Puede elegir uno de estas opciones de duración de asignación tipo **Apto**:

| | Descripción |
| --- | --- |
| **Permitir asignación elegible permanente** | Los administradores de recursos pueden asignar una asignación válida permanente. |
| **Hacer que las asignaciones elegibles expiren después de** | Los administradores de recursos pueden requerir que todas las asignaciones elegibles tengan una fecha de inicio y finalización especificada. |

Además, puede elegir una de estas opciones de duración de asignación tipo **Activo**:

| | Descripción |
| --- | --- |
| **Permitir asignaciones activas permanentes** | Los administradores de recursos pueden asignar una asignación activa permanente. |
| **Hacer que las asignaciones activas expiren después de** | Los administradores de recursos pueden requerir que todas las asignaciones activas tengan una fecha de inicio y finalización especificada. |

> [!NOTE]
> Los administradores de recursos pueden renovar todas las asignaciones que tienen una fecha de finalización específica. Además, los usuarios pueden iniciar solicitudes de autoservicio para [ampliar o renovar las asignaciones de roles](pim-resource-roles-renew-extend.md).

## <a name="require-multi-factor-authentication"></a>Requerir autenticación multifactor

Privileged Identity Management proporciona el cumplimiento opcional de Azure AD Multi-Factor Authentication en dos escenarios distintos.

### <a name="require-multi-factor-authentication-on-active-assignment"></a>Requerir autenticación multifactor para las asignaciones activas

En algunos casos, es posible que quiera asignar un usuario o un grupo a un rol durante un breve período de tiempo (por ejemplo, un día). En este caso, no es necesario que los usuarios asignados soliciten la activación. En este escenario, Privileged Identity Management no puede exigir la autenticación multifactor cuando el usuario usa su asignación de roles, porque ya está activa en el rol desde el momento en que se asignan.

Para asegurarse de que el administrador de recursos que realiza la asignación es quien dicen ser, puede exigir la autenticación multifactor en la asignación activa; para ello, active la casilla **Requerir autenticación multifactor en la asignación activa**.

### <a name="require-multi-factor-authentication-on-activation"></a>Requerir Multi-Factor Authentication durante la activación

Puede exigir que los usuarios que sean elegibles para un rol demuestren quiénes están usando Azure AD Multi-Factor Authentication antes de que se puedan activar. La autenticación multifactor garantiza que el usuario sea quien dice ser con certeza razonable. Aplicar esta opción protege los recursos críticos en situaciones en las que es posible que la cuenta de usuario se haya puesto en peligro.

Para requerir la autenticación multifactor antes de la activación, active la casilla **Requerir autenticación multifactor en activación**.

Para obtener más información, consulte [Autenticación multifactor y Privileged Identity Management](pim-how-to-require-mfa.md).

## <a name="activation-maximum-duration"></a>Duración máxima de la activación

Use control deslizante **Duración máxima de la activación (horas)** para establecer el tiempo máximo, en horas, que un rol permanece activo antes de expirar. Este valor puede oscilar entre una y 24 horas.

## <a name="require-justification"></a>Requerir justificación

Puede requerir que los usuarios escriban una justificación empresarial cuando se activen. Para requerir justificación, active la casilla **Requerir justificación para las asignaciones activas** o la casilla **Requerir justificación en las activaciones**.

## <a name="require-approval-to-activate"></a>Solicitud de aprobación para activar

Si desea solicitar aprobación para activar un rol, siga estos pasos.

1. Active la casilla **Se requiere aprobación para activar**.

1. Elija **Seleccionar aprobadores** para abrir la página **Seleccionar un miembro o grupo**.

    ![Seleccionar un panel de usuarios o grupos para seleccionar aprobadores](./media/pim-resource-roles-configure-role-settings/resources-role-settings-select-approvers.png)

1. Seleccione al menos un usuario o grupo y, luego, haga clic en **Seleccionar**. Puede agregar cualquier combinación de usuarios y grupos. Debe seleccionar al menos un aprobador. No hay aprobadores predeterminados.

    Su elección aparecerá en la lista de aprobadores seleccionados.

1. Una vez que haya especificado todos los valores de rol, seleccione **Actualizar** para guardar los cambios.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de recursos de Azure en Privileged Identity Management](pim-resource-roles-assign-roles.md)
- [Configuración de alertas de seguridad para roles de recursos de Azure en Privileged Identity Management](pim-resource-roles-configure-alerts.md)
