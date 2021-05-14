---
title: 'Proceso de solicitud y notificaciones: administración de derechos de Azure AD'
description: Información acerca del proceso de solicitud para un paquete de acceso y de cuándo se envían notificaciones por correo electrónico en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: mamtakumar
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: ajburnle
ms.reviewer: mamkumar
ms.collection: M365-identity-device-management
ms.openlocfilehash: 95d48a3787f3dd54b5713ad0c9358e329e74aecc
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786346"
---
# <a name="request-process-and-email-notifications-in-azure-ad-entitlement-management"></a>Proceso de solicitud y notificaciones por correo electrónico en la administración de derechos de Azure AD

Cuando un usuario envía una solicitud a un paquete de acceso, se inicia un proceso para proporcionar esa solicitud de acceso. La administración de derechos de Azure AD envía notificaciones por correo electrónico a los aprobadores y solicitantes cuando, durante el proceso, se producen eventos clave. En este artículo se describen el proceso de solicitud y las notificaciones por correo electrónico que se envían.

## <a name="request-process"></a>Proceso de solicitud

Un usuario que necesite tener acceso a un paquete de acceso puede enviar una solicitud de acceso. Según la configuración de la directiva, la solicitud puede requerir una aprobación. Cuando se aprueba una solicitud, comienza un proceso para asignar al usuario acceso a cada recurso del paquete de acceso. El siguiente diagrama muestra una visión general del proceso y de los diferentes estados:

![Diagrama del proceso de aprobación](./media/entitlement-management-process/request-process.png)

| State | Descripción |
| --- | --- |
| Enviado | El usuario envía una solicitud. |
| Pendiente de aprobación | Si la directiva para un paquete de acceso requiere aprobación, la solicitud queda pendiente de aprobación. |
| Expirada | Si ningún aprobador aprueba una solicitud dentro del tiempo de espera de la solicitud de aprobación, la solicitud expira. Para intentarlo de nuevo, el usuario tendrá que volver a enviar la solicitud. |
| Denegado | El aprobador rechaza una solicitud. |
| Aprobado | El aprobador aprueba una solicitud. |
| Entrega | **No** se ha asignado al usuario acceso a todos los recursos del paquete de acceso. Si se trata de un usuario externo, puede que el usuario aún no haya accedido al directorio de recursos. También es posible que no haya aceptado la solicitud de consentimiento. |
| Delivered (Entregado) | Se ha asignado al usuario acceso a todos los recursos del paquete de acceso. |
| Acceso extendido | Si se permiten extensiones en la directiva, el usuario extiende la asignación. |
| Acceso expirado | El acceso del usuario al paquete de acceso ha expirado. Para obtener acceso de nuevo, el usuario tendrá que enviar una solicitud. |

## <a name="email-notifications"></a>Notificaciones por correo electrónico

Si es un aprobador, recibirá notificaciones por correo electrónico cuando necesite aprobar una solicitud de acceso. También recibirá notificaciones cuando se haya completado una solicitud de acceso. Si es solicitante, también se le envían notificaciones por correo electrónico que indiquen el estado de su solicitud.

En los diagramas siguientes se muestra cuándo se envían estas notificaciones por correo electrónico a los aprobadores o al solicitante. Haga referencia a la tabla [notificaciones por correo electrónico](entitlement-management-process.md#email-notifications-table) para buscar el número correspondiente a las notificaciones de correo electrónico que se muestran en los diagramas.

### <a name="first-approvers-and-alternate-approvers"></a>Primeros aprobadores y aprobadores alternativos
En el diagrama siguiente se muestra la experiencia de los primeros aprobadores y los aprobadores alternativos, así como las notificaciones por correo electrónico que reciben durante el proceso de solicitud:

![Flujo de proceso de primeros aprobadores y alternativos](./media/entitlement-management-process/first-approvers-and-alternate-with-escalation-flow.png)

### <a name="requestors"></a>Solicitantes
En el diagrama siguiente se muestra la experiencia de los aprobadores, así como las notificaciones por correo electrónico que reciben durante el proceso de solicitud:

![Flujo del proceso del solicitante](./media/entitlement-management-process/requestor-approval-request-flow.png)

### <a name="2-stage-approval"></a>Aprobación en 2 fases
En el diagrama siguiente se muestra la experiencia de los aprobadores de fase 1 y fase 2, así como las notificaciones por correo electrónico que reciben durante el proceso de solicitud:

![Flujo del proceso de aprobación en 2 fases](./media/entitlement-management-process/2stage-approval-with-request-timeout-flow.png)

### <a name="email-notifications-table"></a>Tabla de notificaciones por correo electrónico
En la tabla siguiente se proporcionan más detalles sobre cada una de estas notificaciones por correo electrónico. Para administrar estos mensajes de correo electrónico, puede usar reglas. En Outlook, por ejemplo, puede crear reglas para trasladar los correos electrónicos a una carpeta si el asunto contiene palabras de esta tabla:

| # | Asunto del correo electrónico | Cuándo se envía | Se envía a |
| --- | --- | --- | --- |
| 1 | Acción requerida: Aprobar o denegar la solicitud reenviada por *[fecha]* | Este correo electrónico se enviará a los aprobadores alternativos de la fase 1 (después de escalar la solicitud) para realizar una acción. | Aprobadores alternativos de la fase 1 |
| 2 | Acción requerida: Aprobar o denegar la solicitud por *[fecha]* | Este correo electrónico se enviará al primer aprobador, si la escalación está deshabilitada, para realizar una acción. | Primer aprobador |
| 3 | Recordatorio: Aprobar o denegar la solicitud por *[fecha]* para *[solicitante]* | Este correo electrónico de recordatorio se enviará al primer aprobador, si la escalación está deshabilitada. El correo electrónico le pide que realice alguna acción si no lo ha hecho. | Primer aprobador |
| 4 | Aprobar o denegar la solicitud por *[hora]* el *[fecha]* | Este correo electrónico se enviará al primer aprobador, si la escalación está habilitada, para realizar una acción. | Primer aprobador |
| 5 | Recordatorio de acción requerida: Aprobar o denegar la solicitud por *[fecha]* para *[solicitante]* | Este correo electrónico de recordatorio se enviará al primer aprobador, si la escalación está habilitada. El correo electrónico le pide que realice alguna acción si no lo ha hecho. | Primer aprobador |
| 6 | La solicitud expiró para *[access_package]* | Este correo electrónico se enviará al primer aprobador y a los aprobadores alternativos de la fase 1 después de que la solicitud ha expirado. | Primer aprobador, aprobadores alternativos de la fase 1 |
| 7 | Solicitud aprobada para el *[solicitante]* para *[access_package]* | Este correo electrónico se enviará al primer aprobador y a los aprobadores alternativos de la fase 1 al completarse la solicitud. | Primer aprobador, aprobadores alternativos de la fase 1 |
| 8 | Solicitud aprobada para el *[solicitante]* para *[access_package]* | Este correo electrónico se enviará al primer aprobador y a los aprobadores alternativos de la fase 1, de una solicitud en 2 fases, cuando la solicitud de la fase 1 esté aprobada. | Primer aprobador, aprobadores alternativos de la fase 1 |
| 9 | Se denegó la solicitud a *[access_package]* | Este correo electrónico se enviará al solicitante cuando se deniegue su solicitud. | Solicitante |
| 10 | Expiró su solicitud a *[access_package]* | Este correo electrónico se enviará al solicitante al final de una solicitud en 1 o 2 fases. El correo electrónico notifica al solicitante que la solicitud ha expirado. | Solicitante |
| 11 | Acción requerida: Aprobar o denegar la solicitud por *[fecha]* | Este correo electrónico se enviará al segundo aprobador, si la escalación está deshabilitada, para realizar una acción. | Segundo aprobador |
| 12 | Recordatorio de acción requerida: Aprobar o denegar la solicitud por *[fecha]* | Este correo electrónico de recordatorio se enviará al segundo aprobador, si la escalación está deshabilitada. La notificación le pide que realice alguna acción si no lo ha hecho. | Segundo aprobador |
| 13 | Acción requerida: Aprobar o denegar la solicitud por *[fecha]* para *[solicitante]* | Este correo electrónico se enviará al segundo aprobador, si la escalación está habilitada, para realizar una acción. | Segundo aprobador |
| 14 | Recordatorio de acción requerida: Aprobar o denegar la solicitud por *[fecha]* para *[solicitante]* | Este correo electrónico de recordatorio se enviará al segundo aprobador, si la escalación está habilitada. La notificación le pide que realice alguna acción si no lo ha hecho. | Segundo aprobador |
| 15 | Acción requerida: Aprobar o denegar la solicitud reenviada por *[fecha]* | Este correo electrónico se enviará a los aprobadores alternativos de la fase 2, si la escalación está habilitada, para realizar una acción. | Aprobadores alternativos de la fase 2 |
| 16 | Solicitud aprobada para el *[solicitante]* para *[access_package]* | Este correo electrónico se enviará al segundo aprobador y a los aprobadores alternativos de la fase 2 al aprobar la solicitud. | Segundo aprobador, aprobadores alternativos de la fase 2 |
| 17 | Una solicitud expiró para *[access_package]* | Este correo electrónico se enviará al segundo aprobador y a los aprobadores alternativos después de que la solicitud ha expirado. | Segundo aprobador, aprobadores alternativos de la fase 2 |
| 18 | Ahora tiene acceso a *[access_package]* | Este correo electrónico se enviará a los usuarios finales para que empiecen a usar su acceso. | Solicitante |
| 19 | Extender el acceso a *[access_package]* por *[fecha]* | Este correo electrónico se enviará a los usuarios finales antes de que expire su acceso. | Solicitante |
| 20 | Ha finalizado el acceso a *[access_package]* | Este correo electrónico se enviará a los usuarios finales después de que expire su acceso. | Solicitante |

### <a name="access-request-emails"></a>Mensajes de correo electrónico de solicitud de acceso

Cuando un solicitante envía una solicitud de acceso para un paquete de acceso que está configurado para requerir aprobación, todos los aprobadores agregados a la directiva reciben una notificación por correo electrónico con los detalles de la solicitud. Los detalles del correo electrónico incluyen: organización del nombre del solicitante y justificación empresarial; y la fecha de inicio y finalización de acceso solicitada (si se proporcionan). Los detalles también incluirán cuándo se envió la solicitud y cuándo expirará.

El correo electrónico incluye un vínculo a Mi acceso donde los aprobadores pueden aprobar o denegar la solicitud de acceso. Esta es una notificación de correo electrónico de ejemplo que se envía al primer aprobador o al segundo aprobador (si está habilitada la aprobación en 2 fases) para completar una solicitud de acceso:

![Correo electrónico de solicitud de aprobación para un paquete de acceso](./media/entitlement-management-shared/approver-request-email.png)

Los aprobadores también pueden recibir un correo electrónico de recordatorio. El correo electrónico pide al aprobador que tome una decisión sobre la solicitud. Este es un ejemplo de notificación por correo electrónico que recibe el aprobador para recordarle tomar medidas:

![Mensaje de correo electrónico del recordatorio de la solicitud de acceso](./media/entitlement-management-process/approver-access-request-reminder-email.png)

### <a name="alternate-approvers-request-emails"></a>Correos electrónicos de solicitud para los aprobadores alternativos

Si está habilitada la configuración de aprobadores alternativos y la solicitud todavía está pendiente, se reenviará. Los aprobadores alternativos recibirán un correo electrónico para aprobar o denegar la solicitud. Puede habilitar los aprobadores alternativos en las fases 1 y 2. Este es un ejemplo de correo electrónico de la notificación que reciben los aprobadores alternativos:

![Correo electrónico de solicitud para los aprobadores alternativos](./media/entitlement-management-process/alternate-approver-email-fwd-request.png)

Tanto el aprobador como los aprobadores alternativos pueden aprobar o denegar la solicitud.

### <a name="approved-or-denied-emails"></a>Mensajes de correo electrónico de aprobación o denegación

 Cuando un aprobador reciba una solicitud de acceso enviada por un solicitante, puede aprobar o denegar la solicitud de acceso. El aprobador debe agregar una justificación comercial que apoye su decisión. Este es un ejemplo de correo electrónico que se envía a los aprobadores y aprobadores alternativos una vez aprobada una solicitud:

![Correo electrónico de solicitud aprobada para un paquete de acceso](./media/entitlement-management-process/approver-request-email-approved.png)

Cuando se aprueba una solicitud de acceso y se aprovisiona su acceso, se envía una notificación por correo electrónico al solicitante de que ahora tiene acceso al paquete de acceso. Este es un ejemplo de notificación por correo electrónico que se envía a un solicitante cuando se le concede acceso al paquete de acceso:

![Correo electrónico de solicitud de acceso aprobada para el solicitante](./media/entitlement-management-process/requestor-email-approved.png)

Cuando se deniega una solicitud de acceso, se envía una notificación por correo electrónico al solicitante. Este es un ejemplo de una notificación por correo electrónico que se envía a un solicitante cuando se le deniega la solicitud de acceso:

![Correo electrónico de solicitud denegada al solicitante](./media/entitlement-management-process/requestor-email-denied.png)

### <a name="2-stage-approval-access-request-emails"></a>Correos electrónicos de solicitud de acceso de aprobación en 2 fases

Si la aprobación en 2 fases está habilitada, al menos dos aprobadores deben aprobar la solicitud, uno de cada fase, antes de que el solicitante pueda recibir acceso.

Durante la fase 1, el primer aprobador recibirá el correo electrónico de solicitud de acceso y tomará una decisión. Si aprueba la solicitud, todos los primeros aprobadores y aprobadores alternativos de la fase 1 (si está habilitada la escalación) recibirán una notificación de que se ha completado la fase 1. Este es un correo electrónico de ejemplo de la notificación que se envía cuando se completa la fase 1:

![Correo electrónico de solicitud de acceso en 2 fases](./media/entitlement-management-process/approver-request-email-2stage.png)

Una vez que el primer aprobador o los aprobadores alternativos aprueban la solicitud en la fase 1, comienza la fase 2. Durante la fase 2, el segundo aprobador recibirá el correo electrónico de notificación de solicitud de acceso. Después de que el segundo aprobador o los aprobadores alternativos en la fase 2 (si está habilitada la escalación) decidan aprobar o denegar la solicitud, los correos electrónicos de notificación se envían al primer y segundo aprobadores, y a todos los aprobadores alternativos de la fase 1 y la fase 2, así como el solicitante.

### <a name="expired-access-request-emails"></a>Mensajes de correo electrónico de solicitud de acceso expirada

Las solicitudes de acceso pueden expirar si ningún aprobador ha aprobado o denegado la solicitud. 

La solicitud expira cuando se llega la fecha de expiración configurada; los aprobadores ya no la pueden aprobar ni rechazar. Este es un ejemplo de correo electrónico de la notificación que se envía a todos los aprobadores, primero, segundo y alternativos (si la aprobación en 2 fases está habilitada):

![Mensaje de correo electrónico de solicitud de acceso expirada para los aprobadores](./media/entitlement-management-process/approver-request-email-expired.png)

Se envía al solicitante una notificación por correo electrónico que le indica que su solicitud de acceso ha expirado y que debe volver a enviarla. En el diagrama siguiente se muestra la experiencia del solicitante y las notificaciones por correo electrónico que recibe cuando solicita ampliar el acceso:

![Flujo del proceso para ampliar el acceso del solicitante](./media/entitlement-management-process/requestor-expiration-request-flow.png) 

Este es un ejemplo de una notificación por correo electrónico que se envía a un solicitante cuando su solicitud de acceso ha expirado:

![Mensaje de correo electrónico de solicitud de acceso expirada para el aprobador](./media/entitlement-management-process/requestor-email-request-expired.png)

## <a name="next-steps"></a>Pasos siguientes

- [Solicitud de acceso a un paquete de acceso](entitlement-management-request-access.md)
- [Aprobación o denegación de solicitudes de acceso](entitlement-management-request-approve.md)
