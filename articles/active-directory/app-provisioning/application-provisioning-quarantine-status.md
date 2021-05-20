---
title: Estado de cuarentena en el aprovisionamiento de aplicaciones de Azure Active Directory
description: Cuando haya configurado una aplicación para el aprovisionamiento automático de usuarios, obtenga información sobre el estado de aprovisionamiento de los medios de cuarentena y cómo borrarlo.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: c3cdf8ab99506afd644d59fa8e74ed59fa1af853
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109783178"
---
# <a name="application-provisioning-in-quarantine-status"></a>Aprovisionamiento de aplicaciones en el estado de cuarentena

El servicio de aprovisionamiento de Azure AD supervisa el estado de la configuración. También pone las aplicaciones incorrectas en un estado de "cuarentena". Si la mayoría de las llamadas realizadas en el sistema de destino, o todas, tienen un error de manera uniforme, el trabajo de aprovisionamiento se marca como en cuarentena. Un ejemplo de error es un error recibido debido a credenciales de administrador no válidas.

En cuarentena:
- la frecuencia de los ciclos incrementales se reduce gradualmente a una vez al día.
- El trabajo de aprovisionamiento se quita de la cuarentena después de que se hayan resuelto todos los errores y se inicie el siguiente ciclo de sincronización. 
- Si el trabajo de aprovisionamiento permanece en cuarentena durante más de cuatro semanas, se deshabilita (deja de ejecutarse).

## <a name="how-do-i-know-if-my-application-is-in-quarantine"></a>¿Cómo puedo saber si mi aplicación está en cuarentena?

Hay tres maneras de comprobar si una aplicación está en cuarentena:
  
- En Azure Portal, vaya a **Azure Active Directory** > **Aplicaciones empresariales** > &lt;*nombre de la aplicación*&gt; > **Aprovisionamiento** y compruebe si la barra de progreso tienen algún mensaje de cuarentena.   

  ![Barra de estado de aprovisionamiento que muestra el estado de cuarentena](./media/application-provisioning-quarantine-status/progress-bar-quarantined.png)

- En Azure Portal, vaya a **Azure Active Directory** > **Registros de auditoría** > filtro en **Actividad: cuarentena** y revise el historial de cuarentena. La vista de la barra de progreso como se describió anteriormente muestra si el aprovisionamiento está actualmente en cuarentena. Los registros de auditoría muestran el historial de cuarentena de una aplicación. 

- Utilice la solicitud de Microsoft Graph [Get synchronizationJob](/graph/api/synchronization-synchronizationjob-get?tabs=http&view=graph-rest-beta&preserve-view=true) (Obtener trabajo de sincronización) para obtener, mediante programación, el estado del trabajo de aprovisionamiento:

```microsoft-graph
        GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/
```

- Compruebe su correo electrónico. Cuando una aplicación se pone en cuarentena, se envía un correo electrónico de notificación de una sola vez. Si el motivo de la cuarentena cambia, se envía un correo electrónico actualizado que muestra la nueva razón para la cuarentena. Si no ve un correo electrónico:

  - Asegúrese de haber especificado un **correo electrónico de notificación** válido en la configuración de aprovisionamiento de la aplicación.
  - Asegúrese de que no haya ningún filtrado de correo no deseado en la bandeja de entrada del correo electrónico de notificación.
  - Asegúrese de que no ha cancelado la suscripción a los mensajes de correo electrónico.
  - Buscar correos electrónicos desde `azure-noreply@microsoft.com`

## <a name="why-is-my-application-in-quarantine"></a>¿Por qué mi aplicación está en cuarentena?

|Descripción|Acción recomendada|
|---|---|
|**Problema de cumplimiento de SCIM:** se devolvió una respuesta HTTP/404 No encontrado en lugar de la respuesta HTTP/200 Correcto esperada. En este caso, el servicio de aprovisionamiento de Azure AD realizó una solicitud a la aplicación de destino y recibió una respuesta inesperada.|Compruebe la sección de credenciales de administrador. Vea si la aplicación requiere especificar la dirección URL del inquilino y si la dirección URL es correcta. Si no experimenta ningún problema, póngase en contacto con el desarrollador de la aplicación para asegurarse de que su servicio sea conforme con SCIM. https://tools.ietf.org/html/rfc7644#section-3.4.2 |
|**Credenciales no válidas:** al intentar autorizar el acceso a la aplicación de destino, se recibió una respuesta de la aplicación de destino que indica que las credenciales proporcionadas no son válidas.|Vaya a la sección Credenciales de administrador de la interfaz de usuario de la configuración de aprovisionamiento y autorice el acceso de nuevo con credenciales válidas. Si la aplicación está en la galería, revise el tutorial de configuración correspondiente para ver los pasos adicionales necesarios.|
|**Roles duplicados:** los roles importados de ciertas aplicaciones como Salesforce y Zendesk deben ser únicos. |Vaya al [manifiesto](../develop/reference-app-manifest.md) de la aplicación en Azure Portal y quite el rol duplicado.|

 Una solicitud de Microsoft Graph para obtener el estado del trabajo de aprovisionamiento muestra la siguiente razón para la cuarentena:
- `EncounteredQuarantineException` indica que se proporcionaron credenciales no válidas. El servicio de aprovisionamiento no puede establecer una conexión entre el sistema de origen y el de destino.
- `EncounteredEscrowProportionThreshold` indica que el aprovisionamiento superó el umbral de custodia. Esta condición se crea cuando se produce un error en más del 40 % de los eventos de aprovisionamiento. Para obtener más información, consulte los detalles del umbral de custodia.
- `QuarantineOnDemand` significa que hemos detectado un problema con la aplicación y la hemos establecido manualmente en cuarentena.

**Umbrales de custodia**

Si se cumple el umbral de custodia proporcional, el trabajo de aprovisionamiento pasará a cuarentena. Esta lógica está sujeta a cambios, pero funciona aproximadamente como se describe a continuación: 

Un trabajo puede entrar en cuarentena independientemente del recuento de errores por problemas como las credenciales de administrador o el cumplimiento de SCIM. Sin embargo, en general, la cantidad mínima para comenzar a evaluar si se debe poner en cuarentena por el elevado número de errores es de 5000. Por ejemplo, un trabajo con 4000 errores no entrará en cuarentena. Sin embargo, un trabajo con 5000 errores desencadenaría una evaluación. Una evaluación usa los siguientes criterios:  
- Si se produce un error en más del 40 % de los eventos de aprovisionamiento o hay más de 40 000 errores, el trabajo de aprovisionamiento pasará a cuarentena. Los errores de referencia no se contarán como parte del umbral del 40 % o de 40 000. Por ejemplo, si no se actualiza un administrador o un miembro del grupo, se produce un error de referencia.
- Un trabajo en el que 45 000 usuarios no se aprovisionaron correctamente entraría en cuarentena, ya que supera el umbral de 40 000.
- Un trabajo en el que 30 000 no se pudieron aprovisionar pero sí 5000 entraría en cuarentena, ya que supera el umbral del 40 % y el mínimo de 5000.
- Un trabajo con 20 000 errores pero 100 000 completados correctamente no entraría en cuarentena porque no supera el umbral de errores del 40 % o el máximo de 40 000 errores.  
- Existe un umbral absoluto de 60 000 errores que tiene en cuenta los errores de referencia y de no referencia. Por ejemplo, no se pudieron aprovisionar 40 000 usuarios y no se completaron 21 000 actualizaciones de administrador. El total es 61 000 errores y supera el límite de 60 000.


## <a name="how-do-i-get-my-application-out-of-quarantine"></a>¿Cómo puedo sacar la aplicación de la cuarentena?

En primer lugar, resuelva el problema que provocó que la aplicación se pusiera en cuarentena.

- Compruebe la configuración de aprovisionamiento de la aplicación para asegurarse de que ha [escrito las credenciales de administrador válidas](../app-provisioning/configure-automatic-user-provisioning-portal.md#configuring-automatic-user-account-provisioning). Azure AD debe establecer una relación de confianza con la aplicación de destino. Asegúrese de que ha especificado las credenciales válidas y de que su cuenta tiene los permisos necesarios.

- Revise los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para investigar más a fondo qué errores están causando la cuarentena y así poder solucionarlos. Seleccione **Azure Active Directory** &gt; **Aplicaciones empresariales** &gt; **Registros de aprovisionamiento (versión preliminar)** en la sección **Actividad**.

Una vez resuelto el problema, reinicie el trabajo de aprovisionamiento. Algunos cambios en la configuración de aprovisionamiento de la aplicación, como asignaciones de atributos o filtros de ámbito, reiniciarán automáticamente el proceso aprovisionamiento. La barra de progreso en la página **Provisioning** (Aprovisionamiento) de la aplicación indica cuándo comenzó el aprovisionamiento por última vez. Si necesita reiniciar el trabajo de aprovisionamiento manualmente, use uno de los métodos siguientes:  

- Use Azure Portal para reiniciar el trabajo de aprovisionamiento. En la página **Aprovisionamiento** de la aplicación, seleccione **Reiniciar aprovisionamiento**. Esta acción reinicia completamente el servicio de aprovisionamiento, lo que puede tardar cierto tiempo. Se volverá a ejecutar un ciclo inicial completo, que borrará los elementos en custodia, quitará la aplicación de la cuarentena y borrará las marcas de agua. Después, el servicio evaluará de nuevo todos los usuarios del sistema de origen y determinará si están en el ámbito del aprovisionamiento. Esto puede ser útil si la aplicación está en cuarentena actualmente, como se explica en este artículo, o si necesita realizar un cambio en las asignaciones de atributos. Tenga en cuenta que el ciclo inicial tarda más tiempo en completarse que el ciclo incremental típico debido al número de objetos que deben evaluarse. Se puede obtener más información sobre el rendimiento de los ciclos inicial e incremental [aquí](application-provisioning-when-will-provisioning-finish-specific-user.md).

- Use Microsoft Graph para [reiniciar el trabajo de aprovisionamiento](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta&preserve-view=true). Así tendrá control total sobre lo que reinicie. Puede optar por borrar los elementos en custodia (para reiniciar el contador de custodia que se acumula en el estado de cuarentena), borrar la cuarentena (para quitar la aplicación de la cuarentena) o borrar las marcas de agua. Use la siguiente solicitud:
 
```microsoft-graph
        POST /servicePrincipals/{id}/synchronization/jobs/{jobId}/restart
```

Reemplace "{ID}" por el valor del identificador la aplicación y reemplace "{jobId}" por el [identificador del trabajo de sincronización](/graph/api/resources/synchronization-configure-with-directory-extension-attributes?tabs=http&view=graph-rest-beta&preserve-view=true#list-synchronization-jobs-in-the-context-of-the-service-principal).
