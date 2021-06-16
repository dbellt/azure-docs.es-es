---
title: Configuración del flujo de trabajo de consentimiento del administrador en Azure Active Directory | Microsoft Docs
description: Aprenda a configurar una manera para que los usuarios finales soliciten acceso a las aplicaciones que requieren el consentimiento del administrador.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 10/29/2019
ms.author: mtillman
ms.reviewer: luleon
ms.collection: M365-identity-device-management
ms.openlocfilehash: 633505f0a0419bc595ecafc706d41aa6600c63c1
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112080676"
---
# <a name="configure-the-admin-consent-workflow"></a>Configuración del flujo de trabajo de consentimiento del administrador

En este artículo se describe cómo habilitar la característica del flujo de trabajo de consentimiento del administrador, que proporciona a los usuarios finales una manera de solicitar acceso a las aplicaciones que requieren el consentimiento del administrador.

Sin un flujo de trabajo de consentimiento del administrador, un usuario de un inquilino en el que esté deshabilitado el consentimiento del usuario se bloqueará cuando intente acceder a cualquier aplicación que requiera permisos para acceder a los datos de la organización. El usuario recibe un mensaje de error genérico que indica que no está autorizado para acceder a la aplicación y debe pedir ayuda a su administrador. Pero a menudo, el usuario no sabe con quién debe ponerse en contacto, por lo que terminan desistiendo o creando una cuenta local en la aplicación. Incluso cuando se notifica a un administrador, no siempre hay un proceso simplificado para ayudar al administrador a conceder acceso y notificar a sus usuarios.
 
El flujo de trabajo de consentimiento del administrador proporciona a los administradores una manera segura de conceder acceso a las aplicaciones que requieren la aprobación del administrador. Cuando un usuario intenta obtener acceso a una aplicación, pero no puede dar su consentimiento, puede enviar una solicitud de aprobación del administrador. La solicitud se envía por correo electrónico a los administradores que se han designado como revisores. Un revisor realiza una acción con respecto a la solicitud y el usuario recibe una notificación de la acción.

Para aprobar solicitudes, un revisor debe ser un administrador global, un administrador de aplicaciones en la nube o un administrador de aplicaciones. El revisor ya debe tener uno de estos roles de administrador asignados; designarlos como revisores no significa que se eleven sus privilegios.

## <a name="enable-the-admin-consent-workflow"></a>Habilitación del flujo de trabajo de consentimiento del administrador

Para habilitar el flujo de trabajo de consentimiento del administrador y elegir revisores, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global.
2. Haga clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo. Se abrirá la **extensión de Azure Active Directory**
3. En el cuadro de búsqueda del filtro, escriba "**Azure Active Directory**" y seleccione el elemento **Azure Active Directory**.
4. En el menú de navegación, haga clic en **Aplicaciones empresariales**. 
5. En **Administrar**, seleccione **Configuración del usuario**.
6. En **Solicitudes de consentimiento del administrador**, establezca **Los usuarios pueden solicitar el consentimiento del administrador para las aplicaciones en las que no puedan proporcionar un consentimiento** en **Sí**.

   ![Configuración del flujo de trabajo de consentimiento del administrador](media/configure-admin-consent-workflow/admin-consent-requests-settings.png)
 
6. Configure las siguientes opciones:

   * **Selección de usuarios que revisarán las solicitudes de consentimiento del administrador**. Seleccione revisores para este flujo de trabajo desde un conjunto de usuarios que tengan los roles de administrador global, administrador de aplicaciones en la nube y administrador de aplicaciones.
   * **Los usuarios seleccionados recibirán notificaciones por correo electrónico de las solicitudes**. Habilite o deshabilite las notificaciones por correo electrónico a los revisores cuando se realiza una solicitud.  
   * **Los usuarios seleccionados recibirán recordatorios de expiración de solicitudes**. Habilite o deshabilite las notificaciones de aviso por correo electrónico a los revisores cuando una solicitud está a punto de expirar.  
   * **La solicitud de consentimiento expira en (días)** . Especifique el tiempo durante en el que las solicitudes permanecen válidas.

7. Seleccione **Guardar**. La característica puede tardar hasta una hora en habilitarse.

> [!NOTE]
> Puede agregar o quitar revisores de este flujo de trabajo si modifica la lista de **selección de revisores de solicitudes de consentimiento del administrador**. Tenga en cuenta una limitación actual de esta característica, y es que los revisores pueden seguir revisando las solicitudes que se realizaron mientras se designaron como revisores.

## <a name="how-users-request-admin-consent"></a>Modo en que los usuarios solicitan el consentimiento del administrador

Una vez habilitado el flujo de trabajo de consentimiento del administrador, los usuarios pueden solicitar la aprobación del administrador para una aplicación de la que no tenga consentimiento para autorizar. En los pasos siguientes se describe la experiencia del usuario al solicitar la aprobación. 

1. El usuario intenta iniciar sesión en la aplicación.

2. Aparece el mensaje **Aprobación necesaria**. El usuario escribe una justificación de su necesidad de acceder a la aplicación y, a continuación, selecciona **Aprobación de solicitud**.

   ![Captura de pantalla en la que se muestra un cuadro de diálogo de aprobación necesaria en el que puede solicitar la aprobación.](media/configure-admin-consent-workflow/end-user-justification.png)

3. El mensaje **Solicitud enviada** confirma que la solicitud se envió al administrador. Si el usuario envía varias solicitudes, solo se envía la primera al administrador.

   ![Captura de pantalla en la que se muestra la confirmación de envío de la solicitud.](media/configure-admin-consent-workflow/end-user-sent-request.png)

 4. El usuario recibe una notificación por correo electrónico cuando se aprueba, se deniega o se bloquea su solicitud. 

## <a name="review-and-take-action-on-admin-consent-requests"></a>Revisión y realización de acciones con respecto a las solicitudes de consentimiento del administrador

Para revisar las solicitudes de consentimiento del administrador y tomar medidas, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como uno de los revisores registrados del flujo de trabajo de consentimiento del administrador.
2. Haga clic en **Todos los servicios** en la parte superior del menú de navegación izquierdo. Se abrirá la **extensión de Azure Active Directory**
3. En el cuadro de búsqueda del filtro, escriba "**Azure Active Directory**" y seleccione el elemento **Azure Active Directory**.
4. En el menú de navegación, haga clic en **Aplicaciones empresariales**.
5. En **Actividad**, seleccione **Solicitudes de consentimiento del administrador**.

   > [!NOTE]
   > Los revisores solo verán las solicitudes de administración que se crearon después de designarse como revisor.

1. Seleccione la aplicación que se solicita.
2. Revise los detalles de la solicitud:  

   * Para ver quién está solicitando acceso y por qué, seleccione la pestaña **Solicitado por**.
   * Para ver qué permisos solicita la aplicación, seleccione **Revisar permisos y consentimiento**.

8. Evalúe la solicitud y lleve a cabo la acción adecuada:

   * **Aprobar la solicitud**. Para aprobar una solicitud, conceda el consentimiento del administrador a la aplicación. Una vez que se aprueba una solicitud, se notifica a todos los solicitantes que se les ha concedido acceso.  
   * **Denegar la solicitud**. Para denegar una solicitud, debe proporcionar una justificación que se proporcionará a todos los solicitantes. Una vez denegada la solicitud, se notifica a todos los solicitantes que se les ha denegado el acceso a la aplicación. La denegación de una solicitud no impedirá que los usuarios soliciten el consentimiento del administrador a la aplicación en el futuro.  
   * **Bloquear la solicitud**. Para bloquear una solicitud, debe proporcionar una justificación que se proporcionará a todos los solicitantes. Una vez bloqueada una solicitud, se notifica a todos los solicitantes que se les ha denegado el acceso a la aplicación. Al bloquear una solicitud, se crea un objeto de entidad de servicio para la aplicación en el inquilino con un estado deshabilitado. Los usuarios no podrán solicitar el consentimiento del administrador a la aplicación en el futuro.
 
## <a name="email-notifications"></a>Notificaciones por correo electrónico
 
Si se configura, todos los revisores recibirán notificaciones por correo electrónico cuando se den las siguientes situaciones:

* Se ha creado una nueva solicitud.
* Ha expirado una solicitud.
* Una solicitud está cerca de su fecha de expiración.  
 
Los solicitantes recibirán notificaciones por correo electrónico en estos casos:

* Envían una nueva solicitud de acceso.
* Su solicitud ha expirado.
* Su solicitud se ha denegado o bloqueado.
* Su solicitud se ha aprobado.
 
## <a name="audit-logs"></a>Registros de auditoría 
 
En la tabla siguiente se describen los escenarios y los valores de auditoría disponibles para el flujo de trabajo de consentimiento del administrador.

|Escenario  |Servicio de auditoría  |Categoría de auditoría  |Actividad de auditoría  |Actor de auditoría  |Limitaciones del registro de auditoría  |
|---------|---------|---------|---------|---------|---------|
|El administrador habilita el flujo de trabajo de solicitud de consentimiento        |Revisiones de acceso           |UserManagement           |Crear plantilla de directiva de gobernanza          |Contexto de la aplicación            |Actualmente no se puede encontrar el contexto de usuario            |
|El administrador deshabilita el flujo de trabajo de solicitud de consentimiento       |Revisiones de acceso           |UserManagement           |Eliminar plantilla de directiva de gobernanza          |Contexto de la aplicación            |Actualmente no se puede encontrar el contexto de usuario           |
|El administrador actualiza las configuraciones de flujo de trabajo de consentimiento        |Revisiones de acceso           |UserManagement           |Actualizar plantilla de directiva de gobernanza          |Contexto de la aplicación            |Actualmente no se puede encontrar el contexto de usuario           |
|El usuario final crea una solicitud de consentimiento del administrador para una aplicación       |Revisiones de acceso           |Directiva         |Crear solicitud           |Contexto de la aplicación            |Actualmente no se puede encontrar el contexto de usuario           |
|Los revisores aprueban una solicitud de consentimiento del administrador       |Revisiones de acceso           |UserManagement           |Aprobar todas las solicitudes en el flujo de negocio          |Contexto de la aplicación            |Actualmente no se puede buscar el contexto de usuario ni el identificador de la aplicación a la que se concedió el consentimiento del administrador.           |
|Los revisores deniegan una solicitud de consentimiento del administrador       |Revisiones de acceso           |UserManagement           |Aprobar todas las solicitudes en el flujo de negocio          |Contexto de la aplicación            | Actualmente no se puede buscar el contexto de usuario del actor que denegó una solicitud de consentimiento del administrador          |

## <a name="faq"></a>Preguntas más frecuentes 

**He activado este flujo de trabajo, pero al probar la funcionalidad, ¿por qué no veo el nuevo mensaje "Aprobación necesaria" que me permita solicitar acceso?**

Después de activar la característica, los usuarios finales pueden tardar hasta 60 minutos en ver la actualización. Para comprobar que la configuración se ha aplicado correctamente, vea el valor **EnableAdminConsentRequests** en la API de `https://graph.microsoft.com/beta/settings`.

**Como revisor, ¿por qué no puedo ver todas las solicitudes pendientes?**

Los revisores solo pueden ver las solicitudes de administración que se crearon después de designarse como revisor. Por tanto, si se ha agregado recientemente como revisor, no verá las solicitudes que se crearon antes de la asignación.

**Como revisor, ¿por qué veo varias solicitudes para la misma aplicación?**
  
Si un desarrollador de aplicaciones ha configurado su aplicación para que use el consentimiento estático y dinámico para solicitar acceso a los datos de su usuario final, verá dos solicitudes de consentimiento del administrador. Una representa los permisos estáticos y la otra, los dinámicos.

**Como solicitante, ¿puedo comprobar el estado de mi solicitud?**  

No, por ahora los solicitantes solo pueden obtener actualizaciones a través de notificaciones por correo electrónico.

**Como revisor, ¿se puede aprobar la aplicación, pero no para todos?**
 
Si le preocupa conceder el consentimiento del administrador y permitir que todos los usuarios del inquilino utilicen la aplicación, recomendamos denegar la solicitud. Después, conceda manualmente el consentimiento del administrador restringiendo el acceso a la aplicación solicitando la asignación de usuarios y asignando usuarios o grupos a la aplicación. Para más información, consulte [Métodos para asignar usuarios y grupos](./assign-user-or-group-access-portal.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de cómo dar consentimiento a las aplicaciones, consulte [Marco de consentimiento de Azure Active Directory](../develop/consent-framework.md).

[Configuración del consentimiento de los usuarios finales a las aplicaciones](configure-user-consent.md)

[Concesión del consentimiento del administrador para todo el inquilino a una aplicación](grant-admin-consent.md)

[Permisos y consentimiento en la plataforma de identidad de Microsoft](../develop/v2-permissions-and-consent.md)

[Azure AD en Microsoft Q&A](/answers/topics/azure-active-directory.html)
