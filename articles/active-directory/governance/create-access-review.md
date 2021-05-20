---
title: Creación de una revisión de acceso de grupos y aplicaciones - Azure AD
description: Obtenga información sobre cómo crear una revisión de acceso de los miembros del grupo o de la aplicación en las revisiones de acceso de Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 5/6/2021
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: abd0ff2b7330bdc459bf3775e92b376686ebfc83
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517194"
---
# <a name="create-an-access-review-of-groups-and-applications-in-azure-ad-access-reviews"></a>Creación de una revisión de acceso de los grupos y las aplicaciones en las revisiones de acceso de Azure AD

El acceso a los grupos y las aplicaciones para empleados e invitados cambia a lo largo del tiempo. Para reducir el riesgo derivado de las asignaciones de acceso obsoletas, los administradores pueden usar Azure Active Directory (Azure AD) para crear revisiones de acceso para los miembros de grupo o el acceso de aplicación. Microsoft 365 y los propietarios de grupos de seguridad también pueden usar Azure AD para crear revisiones de acceso para los miembros del grupo (versión preliminar) siempre y cuando el administrador global o de usuarios permita la configuración a través de la hoja Configuración de revisiones de acceso. Si necesita revisar el acceso de manera rutinaria, también puede crear revisiones de acceso periódicas. Para más información sobre estos escenarios, vea [Administración del acceso de los usuarios con las revisiones de acceso de Azure AD](manage-user-access-with-access-reviews.md) y [Administración del acceso de los invitados con las revisiones de acceso de Azure AD](manage-guest-access-with-access-reviews.md).

Puede ver un vídeo rápido en el que se habla sobre la habilitación de revisiones de acceso:

>[!VIDEO https://www.youtube.com/embed/X1SL2uubx9M]

En este artículo se describe cómo crear una o varias revisiones de acceso para el acceso de los miembros de un grupo o aplicación.

## <a name="prerequisites"></a>Requisitos previos

- Azure AD Premium P2
- administrador global o administrador de usuarios.
- Microsoft 365 y el propietario del grupo de seguridad (versión preliminar)

Para obtener más información, consulte [Requisitos de licencia](access-reviews-overview.md#license-requirements).

## <a name="create-one-or-more-access-reviews"></a>Creación de una o varias revisiones de acceso

1. Inicie sesión en Azure Portal y abra la [página de Identity Governance](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/).

2. En el menú izquierdo, haga clic en **Revisiones de acceso**.

3. Haga clic en **Nueva revisión de acceso** para crear una nueva revisión de acceso.

    ![Panel de revisión de acceso de Identity Governance](./media/create-access-review/access-reviews.png)

4. En **Paso 1: Selección de los elementos que se deben revisar**, seleccione el recurso que desea revisar.

    ![Creación de una revisión de acceso: nombre y descripción de la revisión](./media/create-access-review/select-what-review.png)

5. Si seleccionó **Grupos y equipos** en el paso 1, tiene dos opciones en el paso 2
   - **Todos los grupos de Microsoft 365 con usuarios invitados.** Seleccione esta opción si quiere crear revisiones periódicas de todos los usuarios invitados en todos los grupos de Microsoft Teams y Microsoft 365 de su organización. Puede elegir excluir determinados grupos haciendo clic en "Seleccionar grupos para excluir".
   - **Seleccione Grupos y equipos.** Seleccione esta opción si desea especificar un conjunto finito de equipos o grupos para revisar. Después de hacer clic en esta opción, verá una lista de grupos a la derecha para elegir.

     ![Grupos y equipos](./media/create-access-review/teams-groups.png)

     ![Grupos y equipos elegidos en la interfaz de usuario](./media/create-access-review/teams-groups-detailed.png)

6. Si seleccionó **Aplicaciones** en el paso 1, puede seleccionar una o más aplicaciones en el paso 2.

    >[!NOTE]
    > Si selecciona varios grupos o aplicaciones, se crearán varias revisiones de acceso. Por ejemplo, si selecciona 5 grupos para revisar, obtendrá 5 revisiones de acceso independientes

   ![La interfaz que se muestra si eligió aplicaciones en lugar de grupos](./media/create-access-review/select-application-detailed.png)

7. A continuación, en el paso 3 puede seleccionar un ámbito para la revisión. Tendrá las siguientes opciones
   - **Solo usuarios invitados.** Al seleccionar esta opción, se limita la revisión de acceso a solo los usuarios invitados de Azure AD B2B en el directorio.
   - **Todos.** Al seleccionar esta opción, se establece el ámbito de la revisión de acceso en todos los objetos de usuario asociados al recurso.

    >[!NOTE]
    > Si seleccionó todos los grupos de Microsoft 365 con usuarios invitados en el paso 2, su única opción es revisar los usuarios invitados en el paso 3

8. Haga clic en **Siguiente: Revisiones**.

9. En la sección **Selección de los revisores**, seleccione uno o más usuarios para realizar las revisiones de acceso. Puede elegir entre:
    - **Propietarios del grupo** (solo disponibles al realizar una revisión en un equipo o grupo)
    - **Grupos o usuarios seleccionados**
    - **Los usuarios revisan su propio acceso**
    - **Administradores de usuarios.**
    Si elige **Administradores de usuarios** o **Propietarios del grupo**  también tiene la opción de especificar un revisor de reserva. Se pide a los revisores de reserva que hagan una revisión cuando el usuario no tiene ningún administrador especificado en el directorio o el grupo no tiene un propietario.

    ![nueva revisión de acceso](./media/create-access-review/new-access-review.png)

10. En la sección **Especificación de la periodicidad de la revisión**, puede especificar una frecuencia como, por ejemplo, **semanal, mensual, trimestral, semestral p anual**. A continuación, especifique una **duración**, que define cuánto tiempo se abrirá una revisión para la entrada de los revisores. Por ejemplo, la duración máxima que puede establecer para una revisión mensual es 27 días, con el fin de evitar la superposición de revisiones. Es posible que quiera reducir la duración para asegurarse de que la entrada de revisores se aplica antes. A continuación, puede seleccionar una **fecha de inicio** y una **fecha de finalización**.

    ![Elija la frecuencia con la que debe producirse la revisión](./media/create-access-review/frequency.png)

11. Haga clic en el botón **Siguiente: Configuración** en la parte inferior de la página.

12. En la **Configuración de finalización** puede especificar lo que sucede una vez finalizada la revisión

    ![Creación de una revisión de acceso: configuración de finalización](./media/create-access-review/upon-completion-settings-new.png)

    Si desea quitar automáticamente el acceso para los usuarios que se han denegado, establezca Aplicar automáticamente los resultados al recurso en Habilitar. Si desea aplicar manualmente los resultados cuando se complete la revisión, establezca el conmutador en Deshabilitar.
    
    Use la lista Si los revisores no responden para especificar lo que ocurre con los usuarios a los que el revisor no ha revisado dentro del período de revisión. Este valor no afecta a los usuarios que los revisores revisaron manualmente. Si la decisión final del revisor es Denegar, se quitará el acceso del usuario.

    - **Sin cambios**: dejar el acceso del usuario sin cambios
    - **Quitar acceso**: quitar el acceso del usuario
    - **Aprobar acceso**: aprobar el acceso del usuario
    - **Aceptar recomendaciones**: aceptar la recomendación del sistema sobre la denegación o aprobación del acceso continuo del usuario

    ![Opciones de configuración de finalización](./media/create-access-review/upon-completion-settings-new.png)

    Use la opción Acción que se aplicará a los usuarios **invitados** denegados para especificar lo que ocurre con los usuarios invitados si se deniegan.
    - La eliminación de la pertenencia del usuario del recurso dará lugar a la eliminación del acceso del usuario denegado al grupo o a la aplicación que se está revisando, pero aún podrá iniciar sesión en el inquilino.
    - Impidir que el usuario inicie sesión durante 30 días y, a continuación, quitar al usuario del inquilino hará que se impida que los usuarios denegados inicien sesión en el inquilino, independientemente de si tienen acceso a otros recursos. Si se ha producido un error o si un administrador decide volver a habilitar el acceso de un usuario, puede hacerlo en un plazo de 30 días después de que se haya deshabilitado el usuario. Si no se realiza ninguna acción en los usuarios deshabilitados, se eliminarán del inquilino.

    Para más información sobre los procedimientos recomendados para eliminar los usuarios invitados que ya no tienen acceso a los recursos de su organización, consulte el artículo titulado [Uso de Azure AD Identity Governance para revisar y eliminar usuarios externos que ya no tienen acceso a los recursos.](access-reviews-external-users.md)


    > [!NOTE]
    > La acción que se aplicará a los usuarios invitados denegados no se puede configurar en las revisiones cuyo ámbito es superior al de los usuarios invitados. Tampoco se puede configurar en las revisiones de **todos los grupos de Microsoft 365 con usuarios invitados.** Cuando no se puede configurar, se usa la opción predeterminada de eliminación de la pertenencia del usuario del recurso en los usuarios denegados.

13. Puede enviar notificaciones a usuarios o grupos adicionales (versión preliminar) para recibir actualizaciones de finalización de las revisiones. Esta característica permite que partes interesadas que no sean el creador de la revisión reciban actualizaciones sobre el progreso de la revisión. Para usar esta característica, seleccione **Seleccionar usuarios o grupos** y agregue un usuario o grupo adicional cuando quiera recibir el estado de finalización.

14. En **Habilitar asistentes para la toma de decisiones de revisión**, elija si desea que el revisor reciba recomendaciones durante el proceso de revisión.

    ![Opciones de habilitación de asistentes para la toma de decisiones](./media/create-access-review/helpers.png)

15. En la sección **Configuración avanzada** puede elegir lo siguiente
    - Establezca **Justificación necesaria** en **Habilitar** para requerir un revisor para que proporcione un motivo de aprobación.
    - Establezca **Notificaciones de correo** en **Habilitar** para que Azure AD envíe notificaciones de correo electrónico a los revisores cuando se inicia una revisión de acceso y a los administradores cuando se complete.
    - Establezca **Avisos** en **Habilitar** para que Azure AD envíe recordatorios de revisiones de acceso en curso a los revisores que no hayan completado su revisión. Estos recordatorios serán la mitad de camino de la duración de la revisión.
    - El contenido del correo electrónico enviado a los revisores se genera automáticamente en función de los detalles de la revisión, como el nombre de la revisión, el nombre del recurso, la fecha de vencimiento, etc. Si necesita una forma de comunicar más información, como instrucciones adicionales o información de contacto, puede especificar estos detalles en la sección **Contenido adicional para el correo electrónico del revisor**. La información que escriba se incluirá en la invitación y los correos electrónicos de recordatorio enviados a los revisores asignados. En la sección resaltada en la imagen siguiente se muestra dónde aparece esta información.

      ![contenido adicional para el revisor](./media/create-access-review/additional-content-reviewer.png)

16. Haga clic en **Siguiente: Revisar y crear** para ir a la página siguiente

17. Ponga un nombre a la revisión de acceso. Opcionalmente, asigne a la revisión una descripción. El nombre y la descripción se muestran a los revisores.

18. Revise la información y seleccione **Crear**.

       ![pantalla revisar y crear](./media/create-access-review/create-review.png)

## <a name="allow--group-owners-to-create-and-manage-access-reviews-preview"></a>Permiso para que los propietarios de grupos creen y administren revisiones de acceso (versión preliminar)

Rol necesario: administrador global o administrador de usuarios.

1. Inicie sesión en Azure Portal y abra la [página de Identity Governance](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/).

1. En el menú de la izquierda, en **Revisiones de acceso**, **Configuración**.

1. En la página Delegado que puede crear y administrar revisiones de acceso, establezca la opción **(Versión preliminar) Los propietarios del grupo pueden crear y administrar las revisiones de acceso de los grupos que poseen** en **Sí**.

    ![Crear revisiones: permitir que los propietarios de grupos revisen](./media/create-access-review/group-owners-review-access.png)

    > [!NOTE]
    > De manera predeterminada, el valor se establece en **No**, por lo que debe actualizarse para permitir que los propietarios del grupo creen y administren las revisiones de acceso.

## <a name="start-the-access-review"></a>Inicio de la revisión de acceso

Una vez que haya especificado la configuración para una revisión de acceso, haga clic en **Iniciar**. La revisión de acceso aparecerá en la lista con un indicador de su estado.

![Lista de las revisiones de acceso y su estado](./media/create-access-review/access-reviews-list.png)

De forma predeterminada, Azure AD envía un correo electrónico a los revisores poco después de iniciar la revisión. Si decide no hacer que Azure AD envíe el correo electrónico, asegúrese de informar a los revisores de que hay una revisión de acceso esperando para que la lleven a cabo. Puede mostrarles las instrucciones sobre cómo [revisar el acceso a grupos o aplicaciones](perform-access-review.md). Si la revisión es para que los invitados revisen su propio acceso, muéstreles las instrucciones sobre cómo [revisar su propio acceso a los grupos o aplicaciones](review-your-access.md).

Si ha asignado como revisores a invitados y no han aceptado la invitación, no recibirán un correo electrónico de las revisiones de acceso, ya que, antes de proceder a la revisión, deben aceptar la invitación.

## <a name="access-review-status-table"></a>Tabla de estado de revisión de acceso

| Estado | Definición |
|--------|------------|
|NotStarted | Se ha creado la revisión y la detección de usuarios está en espera para empezar. |
|Inicializando   | La detección de usuarios está en curso para identificar a todos los usuarios que forman parte de la revisión. |
|Iniciando | La revisión se está iniciando. Si se habilitan las notificaciones por correo electrónico, se envían mensajes de correo electrónico a los revisores. |
|InProgress | La revisión se ha iniciado. Si se habilitan las notificaciones por correo electrónico, se habrán enviado mensajes de correo electrónico a los revisores. Los revisores pueden enviar decisiones hasta la fecha de vencimiento. |
|Completando | La revisión se ha completado y se están enviando mensajes de correo electrónico al propietario de la revisión. |
|Revisando automáticamente | La revisión se encuentra en una fase de revisión del sistema. El sistema está registrando decisiones para los usuarios que no se han revisado en función de las recomendaciones o decisiones previamente configuradas. |
|Revisado automáticamente | El sistema ha registrado decisiones para todos los usuarios que no se han revisado. La revisión está lista para continuar con **Aplicando** si está habilitada la opción de aplicación automática. |
|Aplicando | No habrá ningún cambio en el acceso para los usuarios que se han aprobado. |
|Aplicado | Los usuarios denegados, si los hay, se han quitado del recurso o directorio. |
|Con error | La revisión no pudo progresar. Este error puede estar relacionado con la eliminación del inquilino, un cambio en las licencias u otros cambios internos del inquilino. |

## <a name="create-reviews-via-apis"></a>Creación de revisiones mediante API

También puede crear las revisiones de acceso mediante API. Qué hacer para administrar las revisiones de acceso de grupos y usuarios de aplicación en Azure Portal también pueden realizarse mediante Microsoft Graph API. Para más información, consulte [Azure AD Access Reviews API reference](/graph/api/resources/accessreviewsv2-root?view=graph-rest-beta&preserve-view=true) (Referencia de API de revisiones de acceso de Azure AD). Para obtener un ejemplo de código, consulte [Example of retrieving Azure AD Access Reviews via Microsoft Graph](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/m-p/236096) (Ejemplo de recuperación de las revisiones de acceso de Azure AD a través de Microsoft Graph).

## <a name="next-steps"></a>Pasos siguientes

- [Revisión del acceso a grupos o aplicaciones](perform-access-review.md)
- [Revisión del acceso de uno mismo a grupos o aplicaciones](review-your-access.md)
- [Completar una revisión de acceso de grupos o aplicaciones](complete-access-review.md)
