---
title: 'Creación de una revisión de acceso de los roles de recursos de Azure en PIM: Azure AD | Microsoft Docs'
description: Aprenda a crear una revisión de acceso para los roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: pim
ms.date: 04/05/2021
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 87c0ce72348f67c22759915a3a15c69193ad2f60
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552804"
---
# <a name="create-an-access-review-of-azure-resource-roles-in-privileged-identity-management"></a>Creación de una revisión de acceso de los roles de recursos de Azure en Privileged Identity Management

La necesidad de acceso a los roles de recursos de Azure con privilegios para los empleados cambia con el tiempo. Para reducir el riesgo asociado con las asignaciones de roles obsoletas, debe revisar el acceso periódicamente. Puede usar Azure Active Directory (Azure AD) Privileged Identity Management (PIM) para crear revisiones de acceso para obtener acceso con privilegios a los roles de recursos de Azure. También puede configurar revisiones de acceso periódicas que se produzcan automáticamente. En este artículo se describe cómo crear una o varias revisiones de acceso.

## <a name="prerequisite-license"></a>Licencia de requisitos previos

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]. Para más información sobre las licencias para PIM, consulte [Requisitos de licencia para usar Privileged Identity Management](subscription-requirements.md).

> [!Note]
>  Actualmente, se puede limitar el ámbito de una revisión de acceso a las entidades de servicio con acceso a Azure AD y roles de recursos de Azure (versión preliminar) con una edición de Azure Active Directory Premium P2 activa en el inquilino. El modelo de licencia de las entidades de servicio finalizará con la disponibilidad general de esta característica y es posible que se requieran licencias adicionales.

## <a name="prerequisite-role"></a>Rol necesario

 Para crear revisiones de acceso, debe tener asignado el rol de Azure [Propietario](../../role-based-access-control/built-in-roles.md#owner) o [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) para el recurso.

## <a name="open-access-reviews"></a>Abrir las revisiones de acceso

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que esté asignado a uno de los roles necesarios.

1. Seleccione **Identity Governance**.
 
1. En el menú de la izquierda, seleccione **Recursos de Azure** en **Azure AD Privileged Identity Management**.

1. Seleccione el recurso que quiere administrar, por ejemplo, una suscripción.

1. En Administrar, seleccione **Revisiones de acceso**.

    ![Recursos de Azure: lista de revisiones de acceso que muestra el estado de todas las revisiones](./media/pim-resource-roles-start-access-review/access-reviews.png)

1. Haga clic en **Nuevo** para crear una revisión de acceso nueva.

1. Ponga un nombre a la revisión de acceso. Opcionalmente, asigne a la revisión una descripción. El nombre y la descripción se muestran a los revisores.

    ![Creación de una revisión de acceso: nombre y descripción de la revisión](./media/pim-resource-roles-start-access-review/name-description.png)

1. Establezca un valor para **Fecha de inicio**. De forma predeterminada, una revisión de acceso ocurre una vez, se inicia a la misma hora en que se crea y finaliza en un mes. Puede cambiar las fechas de inicio y de finalización para hacer que una revisión de acceso se inicie en el futuro, transcurridos tantos días como desee.

    ![Fecha de inicio, frecuencia, duración, finalización, número de veces y fecha de finalización](./media/pim-resource-roles-start-access-review/start-end-dates.png)

1. Para realizar que la revisión de acceso sea periódica, cambie la opción **Frecuencia** de **Una vez** a **Semanal**, **Mensual**,  **Trimestral**, **Anual** o **Semestral**. Use el control deslizante o el cuadro de texto **Duración** para definir cuántos días se abrirá cada revisión de la serie periódica para que los revisores escriban datos. Por ejemplo, la duración máxima que puede establecer para una revisión mensual es 27 días, con el fin de evitar la superposición de revisiones.

1. Use el valor **Fin** para especificar cómo finalizar la serie de revisión de acceso periódica. La serie puede terminar de tres formas: se ejecuta continuamente para iniciar revisiones indefinidamente, hasta una fecha concreta, o hasta que se haya completado un número definido de veces. Un administrador de usuarios o un administrador de empresa puede detener la serie después de su creación cambiando la fecha en **Configuración**, de manera que termine en esa fecha.

1. En la sección **Usuarios**, seleccione el ámbito de la revisión. Para revisar los usuarios, seleccione **Usuarios, o bien seleccione (versión preliminar) Entidades de servicio** para revisar las cuentas de equipo con acceso al rol de Azure.   

    ![Ámbito de los usuarios para revisar la pertenencia a rol de](./media/pim-resource-roles-start-access-review/users.png)


1. En **Revisar la pertenencia al rol**, seleccione los roles de Azure con privilegios que se deben revisar. 

    > [!NOTE]
    > - Los roles seleccionados aquí incluyen [roles permanentes y elegibles](../privileged-identity-management/pim-how-to-add-role-to-user.md).
    > - Si selecciona más de un rol, se crearán varias revisiones de acceso. Por ejemplo, al seleccionar cinco roles, se crearán cinco revisiones de acceso independientes.
    Si va a crear una revisión de acceso de los **roles de Azure AD**, a continuación se muestra un ejemplo de la lista de revisión de pertenencia.

    ![Panel para la revisión de pertenencia que muestra los roles de Azure AD que puede seleccionar](./media/pim-resource-roles-start-access-review/review-membership.png)

    Si va a crear una revisión de acceso de los **roles de recursos de Azure**, la imagen a continuación muestra un ejemplo de la lista de revisión de pertenencia.

    ![Panel para la revisión de pertenencia que muestra los recursos de Azure que puede seleccionar](./media/pim-resource-roles-start-access-review/review-membership-azure-resource-roles.png)

1. En la sección **Revisores**, seleccione una o más personas para que revisen a todos los usuarios. También puede seleccionar que los miembros revisen su propio acceso.

    ![Lista de los revisores de los usuarios o miembros (por sí mismos) seleccionados](./media/pim-resource-roles-start-access-review/reviewers.png)

    - **Usuarios seleccionados**: use esta opción para designar un usuario específico para completar la revisión. Esta opción está disponible independientemente del ámbito de la revisión, y los revisores seleccionados pueden revisar los usuarios y las entidades de servicio. 
    - **Miembros (por sí mismos)** : use esta opción para hacer que los usuarios revisen sus propias asignaciones de roles. Esta opción solo está disponible si la revisión está en el ámbito de **Usuarios**.
    - **Administrador**: use esta opción para que el administrador del usuario revise su asignación de roles. Esta opción solo está disponible si la revisión está en el ámbito de **Usuarios**. Tras seleccionar Administrador, también tendrá la opción de especificar un revisor de reserva. Se pide a los revisores de reserva que revisen a un usuario cuando este no tiene ningún administrador especificado en el directorio. 

### <a name="upon-completion-settings"></a>Configuración de finalización

1. Para especificar lo que sucede una vez finalizada una revisión, expanda la sección **Configuración de finalización**.

    ![Al terminar, las opciones Aplicar automáticamente y Debe revisar no responden.](./media/pim-resource-roles-start-access-review/upon-completion-settings.png)

1. Si desea quitar automáticamente el acceso para los usuarios que se han denegado, establezca **Aplicar automáticamente los resultados al recurso** en **Habilitar**. Si desea aplicar manualmente los resultados cuando se complete la revisión, establezca el conmutador en **Deshabilitar**.

1. Use la lista **Si el revisor no responde** para especificar lo que ocurre con los usuarios a los que el revisor no ha revisado dentro del período de revisión. Este valor no afecta a los usuarios que los revisores revisaron manualmente. Si la decisión final del revisor es Denegar, se quitará el acceso del usuario.

    - **Sin cambios**: dejar el acceso del usuario sin cambios
    - **Quitar acceso**: quitar el acceso del usuario
    - **Aprobar acceso**: aprobar el acceso del usuario
    - **Aceptar recomendaciones**: aceptar la recomendación del sistema sobre la denegación o aprobación del acceso continuo del usuario

### <a name="advanced-settings"></a>Configuración avanzada

1. Para especificar configuraciones adicionales, expanda la sección **Configuración avanzada**.

    ![Opciones avanzadas para mostrar recomendaciones, requerir el motivo de la aprobación, notificaciones por correo y recordatorios.](./media/pim-resource-roles-start-access-review/advanced-settings.png)

1. Establezca **Mostrar recomendaciones** en **Habilitar** para mostrar las recomendaciones del sistema de los revisores según la información del acceso del usuario.

1. Establezca **Requerir motivo de la aprobación** en **Habilitar** para requerir que el revisor proporcione un motivo para la aprobación.

1. Establezca **Notificaciones de correo** en **Habilitar** para que Azure AD envíe notificaciones de correo electrónico a los revisores cuando se inicia una revisión de acceso y a los administradores cuando se complete.

1. Establezca **Avisos** en **Habilitar** para que Azure AD envíe recordatorios de revisiones de acceso en curso a los revisores que no hayan completado su revisión.
1. El contenido del correo electrónico enviado a los revisores se genera automáticamente en función de los detalles de la revisión, como el nombre de esta, el nombre del recurso, la fecha de vencimiento, etc. Si necesita una forma de comunicar más información, como instrucciones adicionales o información de contacto, puede especificar estos detalles en el **correo electrónico de contenido adicional para el revisor** que se incluirá en la invitación y en los correos electrónicos de recordatorio enviados a los revisores asignados. La sección resaltada a continuación es donde se mostrará esta información.

    ![Contenido del correo electrónico enviado a los revisores con aspectos destacados](./media/pim-resource-roles-start-access-review/email-info.png)

## <a name="start-the-access-review"></a>Inicio de la revisión de acceso

Una vez que haya especificado la configuración para una revisión de acceso, haga clic en **Iniciar**. La revisión de acceso aparecerá en la lista con un indicador de su estado.

![Lista de revisiones de acceso que muestra el estado de la revisión iniciada](./media/pim-resource-roles-start-access-review/access-reviews-list.png)

De forma predeterminada, Azure AD envía un correo electrónico a los revisores poco después de iniciar la revisión. Si decide no hacer que Azure AD envíe el correo electrónico, asegúrese de informar a los revisores de que hay una revisión de acceso esperando para que la lleven a cabo. Puede mostrarles las instrucciones sobre cómo [revisar el acceso a los roles de recursos de Azure](pim-resource-roles-perform-access-review.md).

## <a name="manage-the-access-review"></a>Administración de la revisión de acceso

En la página **Información general** de la revisión de acceso, puede seguir el progreso de los revisores a medida que completan las revisiones. Los derechos de acceso no se cambian en el directorio hasta que [la revisión finaliza](pim-resource-roles-complete-access-review.md).

![Página de información general de revisiones de acceso que muestra los detalles de la revisión](./media/pim-resource-roles-start-access-review/access-review-overview.png)

Si se trata de una revisión puntual, una vez finalizado el período de revisión de acceso o cuando el administrador detenga la revisión de acceso, siga los pasos sobre [cómo realizar una revisión de acceso de los roles de recursos de Azure](pim-resource-roles-complete-access-review.md) para ver los resultados y aplicarlos.  

Para administrar una serie de revisiones de acceso, vaya a la revisión de acceso y verá los próximos eventos en Revisiones programadas; ahí podrá editar la fecha de finalización o agregar o quitar revisores según corresponda.

Según las selecciones de la **Configuración de finalización**, la aplicación automática se ejecutará después de la fecha de finalización de la revisión o cuando se detenga manualmente la revisión. El estado de la revisión cambiará de **Completado** a estados intermedios como **Aplicando** y, por último, a **Aplicado**. Debería ver que los usuarios denegados, si es que los hay, se eliminan de los roles en unos minutos.

## <a name="next-steps"></a>Pasos siguientes

- [Revisar el acceso a los roles de recursos de Azure](pim-resource-roles-perform-access-review.md)
- [Revisión de acceso de roles de recursos de Azure](pim-resource-roles-complete-access-review.md)
- [Creación de una revisión de acceso de roles de Azure AD](pim-how-to-start-security-review.md)
