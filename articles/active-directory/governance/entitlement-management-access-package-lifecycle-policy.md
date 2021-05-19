---
title: 'Modificación de la configuración del ciclo de vida para un paquete de acceso en la administración de derechos de Azure AD: Azure Active Directory'
description: Aprenda a modificar la información del solicitante y la configuración del ciclo de vida de un paquete de acceso en la administración de derechos de Azure Active Directory.
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
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 075e1a445ad1df82be245322871277038b14d641
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109713835"
---
# <a name="change-lifecycle-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Modificación de la configuración del ciclo de vida para un paquete de acceso en la administración de derechos de Azure AD

Como administrador de paquetes de acceso, puede cambiar la configuración del ciclo de vida de un paquete de acceso en cualquier momento si edita una directiva existente. Si cambia la fecha de expiración para una directiva, la fecha de expiración para las solicitudes que ya están aprobadas o en un estado de aprobación pendiente no cambiará.

En este artículo, se explica cómo se puede modificar la configuración del ciclo de vida de un paquete de acceso existente.

## <a name="open-requestor-information"></a>Apertura de la información del solicitante
Para asegurarse de que los usuarios tienen derecho de acceso a un paquete de acceso, se pueden configurar preguntas personalizadas para pedir a los usuarios que soliciten acceso a determinados paquetes de acceso. Entre las opciones de configuración se incluyen: localización, requerido u opcional y formatos de respuesta de texto y varias opciones. Los solicitantes verán las preguntas cuando solicitan el paquete y los aprobadores ven las respuestas a las preguntas para ayudarles a tomar su decisión. Use los pasos siguientes para configurar las preguntas de un paquete de acceso:

## <a name="open-lifecycle-settings"></a>Apertura de la configuración del ciclo de vida

Para cambiar la configuración del ciclo de vida de un paquete de acceso, debe abrir la directiva correspondiente. Siga estos pasos para abrir la configuración del ciclo de vida de un paquete de acceso.

**Requisitos previos de rol:** Administrador global, Administrador de Identity Governance, Administrador de usuarios, Propietario del catálogo o Administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Directivas** y, a continuación, haga clic en la directiva que contenga la configuración del ciclo de vida que desee editar.

    Se abre el panel Detalles de directiva en la parte inferior de la página.

    ![Paquete de acceso - Panel Detalles de directiva](./media/entitlement-management-shared/policy-details.png)

1. Haga clic en **Editar** para editar la directiva.

    ![Paquete de acceso - Editar directiva](./media/entitlement-management-shared/policy-edit.png)

1. Haga clic en la pestaña **Ciclo de vida** para abrir la configuración del ciclo de vida.

[!INCLUDE [Entitlement management lifecycle policy](../../../includes/active-directory-entitlement-management-lifecycle-policy.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Modificación de la configuración de solicitudes y aprobación de un paquete de acceso](entitlement-management-access-package-request-policy.md)