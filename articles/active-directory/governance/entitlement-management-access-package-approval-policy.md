---
title: 'Cambio de la configuración de aprobación para un paquete de acceso en la administración de derechos de Azure AD: Azure Active Directory'
description: Aprenda a modificar la configuración de información de aprobación y del solicitante de un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 09/16/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c857de5c9182d49e2527c9cbf416d7012c46370
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109713961"
---
# <a name="change-approval-and-requestor-information-preview-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Cambio de la configuración de información de aprobación y del solicitante (versión preliminar) para un paquete de acceso de administración de derechos de Azure AD

Como administrador de paquetes de acceso, puede cambiar la configuración de información de aprobación y del solicitante de un paquete de acceso en cualquier momento si edita una directiva existente o agrega una nueva directiva.

En este artículo, se explica cómo se puede modificar la configuración de información de aprobación y del solicitante de un paquete de acceso existente.

## <a name="approval"></a>Aprobación

En la sección Aprobación, especifique si se requiere una aprobación cuando los usuarios soliciten este paquete de acceso. La configuración de aprobación funciona de la siguiente manera:

- Una solicitud para una aprobación de una sola fase solo debe aprobarla uno de los aprobadores seleccionados o un aprobador de reserva. 
- Una solicitud para una aprobación de dos fases solo debe aprobarla uno de los aprobadores seleccionados de cada fase.
- El aprobador puede ser un administrador, un patrocinador interno o un patrocinador externo, dependiendo de quién rige el acceso a la directiva.
- Para una aprobación de una sola fase o de dos fases no se necesita la aprobación de cada uno de los aprobadores seleccionados.
- La decisión de aprobación se basa en el aprobador que revise primero la solicitud.

Para ver una demostración de cómo agregar aprobadores a una directiva de solicitud, vea el vídeo siguiente:

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4cZfg]

Para ver una demostración de cómo agregar una aprobación de varias fases a una directiva de solicitud, vea el vídeo siguiente:

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4d1Jw]


## <a name="change-approval-settings-of-an-existing-access-package"></a>Cambio de la configuración de aprobación de un paquete de acceso existente

Siga estos pasos para especificar la configuración de aprobación de las solicitudes para el paquete de acceso:

**Requisitos previos de rol:** administrador global, administrador de Identity Governance, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Seleccione una directiva para editar o agregar una nueva directiva al paquete de acceso.
    1. Haga clic en **Directivas** y, a continuación, en **Agregar directiva** si desea crear una nueva directiva.
    1. Haga clic en la directiva que desee editar y después haga clic en **Editar**.

1. Vaya a la pestaña **Solicitud**.

1. Para requerir la aprobación de las solicitudes de los usuarios seleccionados, establezca el botón de alternancia **Requerir aprobación** en **Sí**. O bien, para que las solicitudes se aprueban automáticamente, establezca el botón de alternancia en **No**.

1. Para requerir que los usuarios proporcionen una justificación para solicitar el paquete de acceso, establezca la opción **R** en **Sí**.
    
1. Ahora determine si las solicitudes requerirán la aprobación de una sola fase o de dos fases. Establezca el botón de alternancia **Número de fases** en **1** para una aprobación de una sola fase o en **2**, para la aprobación de dos fases.

    ![Acceder a paquete - Solicitudes - Configuración de aprobación](./media/entitlement-management-access-package-approval-policy/approval.png)


Siga los pasos que se indican a continuación para agregar aprobadores después de seleccionar el número de fases que requiere: 

### <a name="single-stage-approval"></a>Aprobación de una sola fase

1. Agregue el **Primer aprobador**:
    
    Si la directiva está configurada para regir el acceso de los usuarios en el directorio, puede seleccionar **Administrador como aprobador**. O bien agregue un usuario específico con un clic en **Agregar aprobadores** después de seleccionar Elegir aprobadores específicos en el menú desplegable.
    
    ![Paquete de acceso - Solicitudes - Para usuarios en el directorio - Primer aprobador](./media/entitlement-management-access-package-approval-policy/approval-single-stage-first-approver-manager.png)

    Si esta directiva está configurada para regir el acceso de usuarios que no están en el directorio, puede seleccionar **Patrocinador externo** o **Patrocinador interno**. O bien agregue un usuario específico con un clic en **Agregar aprobadores** o grupos en Elegir aprobadores específicos.
    
    ![Paquete de acceso - Solicitudes - Para usuarios fuera del directorio - Primer aprobador](./media/entitlement-management-access-package-approval-policy/out-directory-first-approver.png)
    
1. Si seleccionó **Administrador** como el primer aprobador, haga clic en **Agregar reserva** para seleccionar uno o más usuarios o grupos del directorio como aprobador de reserva. Los aprobadores de reserva reciben la solicitud si la administración de derechos no encuentra al administrador del usuario que solicita el acceso.

    La administración de derechos encuentra al administrador mediante el atributo **Manager**. El atributo está en el perfil del usuario en Azure AD. Para más información, consulte [Incorporación o actualización de la información de perfil de un usuario mediante Azure Active Directory](../fundamentals/active-directory-users-profile-azure-portal.md).

1. Si seleccionó **Elegir aprobadores específicos**, haga clic en **Agregar aprobadores** para seleccionar uno o varios usuarios o grupos del directorio para que sean aprobadores.

1. En el cuadro bajo **¿Dentro de cuántos días es necesario tomar una decisión?** , especifique el número de días que un aprobador tiene que revisar una solicitud para este paquete de acceso.

    Si no se aprueba una solicitud durante este período de tiempo, se rechazará automáticamente. El usuario tendrá que enviar otra solicitud para el paquete de acceso.

1. Para requerir que los aprobadores justifiquen su decisión, establezca la opción Requerir justificación del aprobador en **Sí**.

    La justificación es visible para otros aprobadores y el solicitante.

### <a name="2-stage-approval"></a>Aprobación en 2 fases

Si seleccionó una aprobación de dos fases, deberá agregar otro aprobador.

1. Agregue el **Segundo aprobador**: 
    
    Si los usuarios están en el directorio, agregue un usuario específico como el segundo aprobador mediante un clic en **Agregar aprobadores** en Elegir aprobadores específicos.

    ![Paquete de acceso - Solicitudes - Para usuarios en el directorio - Segundo aprobador](./media/entitlement-management-access-package-approval-policy/in-directory-second-approver.png)

    Si los usuarios no están en el directorio, seleccione **Patrocinador interno** o **Patrocinador externo** como el segundo aprobador. Después de seleccionar el aprobador, agregue los aprobadores de reserva.

    ![Paquete de acceso - Solicitudes - Para usuarios fuera del directorio - Segundo aprobador](./media/entitlement-management-access-package-approval-policy/out-directory-second-approver.png) 

1. Especifique el número de días que tiene el segundo aprobador para aprobar la solicitud en el cuadro bajo **¿Dentro de cuántos días es necesario tomar una decisión?** 

1. Establezca el botón de alternancia Requerir justificación del aprobador en **Sí** o en **No**.

### <a name="alternate-approvers"></a>Aprobadores alternativos

Puede especificar aprobadores alternativos de manera similar a cómo se especifican el primer y el segundo aprobador que pueden aprobar solicitudes. Tener aprobadores alternativos lo ayudará a asegurarse de que las solicitudes se aprueban o deniegan antes de que expiren (tiempo de expiración). Puede enumerar como aprobadores alternativos el primer aprobador y el segundo aprobador para la aprobación de dos fases. 

Al especificar aprobadores alternativos, en caso de que el primer o el segundo aprobador no pueda aprobar ni rechazar la solicitud, la solicitud pendiente se reenvía a los aprobadores alternativos según la programación de reenvío que haya especificado durante la configuración de la directiva. Reciben un correo electrónico para aprobar o denegar la solicitud pendiente.

Después de reenviar la solicitud a los aprobadores alternativos, el primer o el segundo aprobador puede seguir aprobando o denegando la solicitud. Los aprobadores alternativos usan el mismo sitio Mi acceso para aprobar o denegar la solicitud pendiente.

Podemos enumerar personas o grupos de personas como aprobadores y aprobadores alternativos. Asegúrese de que se muestran distintos conjuntos de personas como el primer aprobador, el segundo aprobador y los aprobadores alternativos.
Por ejemplo, si especificó que Alice y Bob sean los primeros aprobadores, indique que Carol y Dave serán los aprobadores alternativos. Siga estos pasos para agregar aprobadores alternativos a un paquete de acceso:

1. En el Primer aprobador, el Segundo aprobador o en ambos, haga clic en **Mostrar configuración de solicitud avanzada**.

    :::image type="content" source="media/entitlement-management-access-package-approval-policy/alternate-approvers-click-advanced-request.png" alt-text="Paquete de acceso - Directiva - Mostrar configuración de solicitud avanzada":::

1. Establezca la opción **Si no se toman medidas, ¿quiere realizar el reenvío a los aprobadores alternativos?** en **Sí**.

1. Haga clic en **Agregar aprobadores alternativos** y seleccione los aprobadores alternativos de la lista.

    ![Paquete de acceso - Directiva - Agregar aprobadores alternativos](./media/entitlement-management-access-package-approval-policy/alternate-approvers-add.png)

    Si selecciona administrador como aprobador para el primer aprobador, tendrá una opción adicional, **Administrador de segundo nivel como aprobador alternativo**, disponible para elegir en el campo de aprobador alternativo. Si selecciona esta opción, debe agregar un aprobador de reserva al que reenviar la solicitud en caso de que el sistema no encuentre el administrador de segundo nivel.

1. En el cuadro **¿Dentro de cuántos días quiere realizar el reenvío a los aprobadores alternativos?** , escriba el número de días durante los cuales los aprobadores pueden aprobar o denegar una solicitud. Si ningún aprobador ha aprobado o denegado la solicitud en ese período, la solicitud expirará (tiempo de expiración) y el usuario deberá enviar otra solicitud para el paquete de acceso. 

    Las solicitudes solo se pueden reenviar a aprobadores alternativos un día después de que la duración de la solicitud llega a la mitad y la decisión de los aprobadores principales debe ser agotar el tiempo de espera después de 4 días como mínimo. Si el tiempo de espera de la solicitud es menor o igual que 3, no hay tiempo suficiente para reenviar la solicitud a los aprobadores alternativos. En este ejemplo, la duración de la solicitud es de 14 días. De este modo, la mitad de la duración de la solicitud es el día 7. Por lo tanto, la solicitud no se puede reenviar antes del día 8. Además, las solicitudes no se pueden reenviar el último día de la duración de la solicitud. Por lo tanto, en el ejemplo, la solicitud se puede reenviar a más tardar el día 13.

## <a name="enable-requests"></a>Habilitación de solicitudes

1. Si quiere que el paquete de acceso esté disponible de inmediato para los usuarios de la directiva de solicitud para que puedan solicitarlo, cambie el conmutador de alternancia Habilitar a **Sí**.

    Podrá habilitarla en todo momento cuando haya terminado de crear el paquete de acceso.

    Si seleccionó **Ninguno (solo para las asignaciones directas del administrador)** y establece la habilitación en **No**, los administradores no podrán asignar directamente este paquete de acceso.

    ![Paquete de acceso: directiva; configuración de habilitación de la directiva](./media/entitlement-management-access-package-approval-policy/enable-requests.png)

1. Haga clic en **Next**.

## <a name="collect-additional-requestor-information-for-approval-preview"></a>Recopilación de información del solicitante adicional para su aprobación (versión preliminar)

Para asegurarse de que los usuarios obtienen acceso a los paquetes de acceso adecuados, puede requerir que los solicitantes respondan a preguntas del campo de texto personalizado o de tipo test en el momento de la solicitud. Existe un límite de 20 preguntas por directiva y un límite de 25 respuestas para preguntas de tipo test. A continuación, se mostrarán a las preguntas a los aprobadores para ayudarles a tomar una decisión.

1. Vaya a la pestaña **Requestor information** (Información del solicitante) y haga clic en la subpestaña **Preguntas**.
 
1. Escriba lo que desea pedir al solicitante, lo cual también se conoce como cadena que se va a mostrar, para la pregunta del cuadro **Pregunta**.

    ![Paquete de acceso: Directiva Habilitar la configuración de la información del solicitante](./media/entitlement-management-access-package-approval-policy/add-requestor-info-question.png)

1. Si la comunidad de usuarios que requerirán acceso al paquete de acceso no tienen un idioma preferido común, puede mejorar la experiencia de los usuarios que solicitan acceso en myaccess.microsoft.com. Para mejorar la experiencia, puede proporcionar cadenas de visualización alternativas para diferentes idiomas. Por ejemplo, si el explorador web de un usuario está establecido en español y tiene configuradas cadenas de visualización en español, dichas cadenas se mostrarán al usuario solicitante. Para configurar la localización de las solicitudes, haga clic en **agregar localización**.
    1. Una vez en el panel **Add localizations for question** (Agregar traducciones para la pregunta), seleccione el **código de idioma** para el idioma en el que va a traducir la pregunta.
    1. En el idioma configurado, escriba la pregunta en el cuadro **Localized Text** (Texto traducido).
    1. Una vez que haya agregado todas las traducciones necesarias, haga clic en **Guardar**.

    ![Paquete de acceso: Directiva Configurar texto traducido](./media/entitlement-management-access-package-approval-policy/add-localization-question.png)

1. Seleccione el **formato de respuesta** en el que desea que los solicitantes respondan. Entre los formatos de respuesta se incluyen: *texto breve*, *opciones múltiples* y *texto largo*.
 
    ![Paquete de acceso: Directiva Seleccionar ver y editar el formato de respuesta de opciones múltiples](./media/entitlement-management-access-package-approval-policy/answer-format-view-edit.png)
 
1. Si selecciona varias opciones, haga clic en el botón **Ver y editar** para configurar las opciones de respuesta.
    1. Después de seleccionar Ver y editar, se abrirá el panel **View/edit question** (Ver y editar pregunta).
    1. Escriba las opciones de respuesta que desea dar al solicitante al responder a la pregunta de los cuadros **Answer values** (Valores de respuesta).
    1. Escriba tantas respuestas como sea necesario y haga clic en **Save** (Guardar).
    
    ![Paquete de acceso: Directiva Escribir opciones múltiples](./media/entitlement-management-access-package-approval-policy/answer-multiple-choice.png)
  
1. Para requerir que los solicitantes respondan a esta pregunta al solicitar acceso a un paquete de acceso, haga clic en la casilla situada debajo de **Required** (Obligatorio).

1. Rellene las pestañas restantes (p. ej., Ciclo de vida) en función de sus necesidades.

Después de haber configurado la información del solicitante en su directiva de paquete de acceso, puede ver las respuestas del solicitante a las preguntas. Para obtener instrucciones sobre cómo ver la información del solicitante, consulte [Visualización de las respuestas del solicitante a las preguntas (versión preliminar)](entitlement-management-request-approve.md#view-requestors-answers-to-questions-preview).

## <a name="next-steps"></a>Pasos siguientes
- [Cambiar la configuración del ciclo de vida de un paquete de acceso](entitlement-management-access-package-lifecycle-policy.md)
- [Ver las solicitudes de un paquete de acceso](entitlement-management-access-package-requests.md)
