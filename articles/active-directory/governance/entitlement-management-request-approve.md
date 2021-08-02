---
title: 'Aprobación o denegación de solicitudes de acceso: administración de derechos de Azure AD'
description: Obtenga información sobre cómo usar el portal Mi acceso para aprobar o denegar solicitudes para un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: mamtakumar
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mamkumar
ms.collection: M365-identity-device-management
ms.openlocfilehash: dd47a623700c495a571bc4ca6b949d1a2229178a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110469167"
---
# <a name="approve-or-deny-access-requests-in-azure-ad-entitlement-management"></a>Aprobación o denegación de solicitudes de acceso en la administración de derechos de Azure AD

Con la administración de derechos de Azure AD, puede configurar directivas para requerir aprobación para los paquetes de acceso y elegir uno o varios aprobadores. En este artículo se describe cómo los aprobadores designados pueden aprobar o denegar solicitudes para paquetes de acceso.

## <a name="open-request"></a>Abrir una solicitud

El primer paso para aprobar o denegar solicitudes de acceso es buscar y abrir la solicitud de acceso pendiente de aprobación. Hay dos maneras de abrir la solicitud de acceso.

**Rol necesario:** Aprobador

1. Buscar un correo electrónico de Microsoft Azure que le pida que apruebe o deniegue una solicitud. Este es un ejemplo de correo electrónico:

    ![Correo electrónico de solicitud de aprobación para un paquete de acceso](./media/entitlement-management-shared/approver-request-email.png)

1. Haga clic en el vínculo **Aprobar o denegar una solicitud** para abrir la solicitud de acceso.

1. Inicie sesión en el portal Mi acceso.

Si no tiene el correo electrónico, puede encontrar las solicitudes de acceso pendientes de aprobación mediante estos pasos.

1. Inicie sesión en el portal Mi acceso en [https://myaccess.microsoft.com](https://myaccess.microsoft.com).  (Para el Gobierno de EE. UU., el dominio del vínculo del portal Mi acceso será `myaccess.microsoft.us`).

1. En el menú izquierdo, haga clic en **Aprobaciones** para ver una lista de las solicitudes de acceso pendientes de aprobación.

1. En la pestaña **Pendientes**, busque la solicitud.

## <a name="view-requestors-answers-to-questions-preview"></a>Visualización de las respuestas del solicitante a las preguntas (versión preliminar)

1. Vaya a la pestaña **Aprobaciones** en Mi acceso.

1. Vaya a la solicitud que le gustaría aprobar y haga clic en **Detalles**. También puede hacer clic en **Aprobar** o **Denegar** si está listo para tomar una decisión.

1. Haga clic en **Detalles de la solicitud**.

    ![Portal Mi acceso: solicitud de acceso, hacer clic en Detalles de la solicitud](./media/entitlement-management-request-approve/requestor-information-request-details.png)

1. La información proporcionada por el solicitante estará en la parte inferior del panel.

    ![Captura de pantalla que muestra los detalles de la solicitud.](./media/entitlement-management-request-approve/requestor-information-requestor-answers.png)

1. En función de la información proporcionada por el solicitante, puede aprobar o denegar la solicitud. Consulte los pasos de aprobación o denegación de una solicitud para obtener instrucciones.

## <a name="approve-or-deny-request"></a>Aprobar o denegar una solicitud

Después de abrir una solicitud de acceso pendiente de aprobación, puede ver los detalles que le ayudarán a tomar una decisión de aprobación o denegación.

**Rol necesario:** Aprobador

1. Haga clic en el vínculo **Ver** para abrir el panel Solicitud de acceso.

1. Haga clic en **Detalles** para ver más información sobre la solicitud de acceso.

    Los detalles incluyen el nombre de usuario, la organización, las fechas de inicio y fin del acceso (si se proporcionan), la justificación comercial, cuándo se envió la solicitud y cuándo expirará.

1. Haga clic en **Aprobar** o **Denegar**.

1. Si es necesario, escriba un motivo.

    ![Captura de pantalla que muestra la página en la que acepta o rechaza la solicitud.](./media/entitlement-management-request-approve/my-access-approve-request.png)

1. Haga clic en **Enviar** para enviar su decisión.

    Si una directiva se configura con varios aprobadores en una fase, solo un aprobador debe tomar una decisión sobre la aprobación pendiente. Una vez que un aprobador haya enviado su decisión sobre la solicitud de acceso, la solicitud se completa y ya no está disponible para que los otros aprobadores la aprueben o denieguen. Los otros aprobadores pueden consultar la decisión sobre la solicitud y el aprobador que tomó la decisión en su portal Mi acceso.

    Si ninguno de los aprobadores configurados en una fase puede aprobar o denegar la solicitud de acceso, la solicitud expira después de la duración de la solicitud configurada. El usuario recibe una notificación que le indica que su solicitud de acceso ha expirado y que debe volver a enviar la solicitud de acceso.

## <a name="next-steps"></a>Pasos siguientes

- [Solicitud de acceso a un paquete de acceso](entitlement-management-request-access.md)
- [Solicitud de notificaciones de proceso y correo electrónico](entitlement-management-process.md)
