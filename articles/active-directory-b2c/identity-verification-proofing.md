---
title: Prueba y verificación de identidades para Azure AD B2C
titleSuffix: Azure AD B2C
description: Obtenga información acerca de los asociados que se integran con Azure AD B2C para ofrecer soluciones de prueba y verificación de identidades
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/23/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 536c63fec9958823191f3b01ad2142022832ef23
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124390"
---
# <a name="identity-verification-and-proofing-partners"></a>Asociados de pruebas y verificación de identidades

Con los asociados de Azure AD B2C, los clientes pueden habilitar la verificación y revisión de identidades de sus usuarios finales antes de permitir el registro de la cuenta o el acceso a esta. En la verificación y revisión de identidades se puede comprobar la información y ejecución de documentos basadas en el conocimiento.

Un diagrama de arquitectura de alto nivel explica el flujo.

![Diagrama que muestra el flujo de revisión de identidades](./media/partner-gallery/third-party-identity-proofing.png)

Microsoft colabora con los siguientes asociados ISV.

| Asociado ISV | Descripción y tutoriales de integración |
|:-------------------------|:--------------|
|![Captura de pantalla de un logotipo de Experian.](./media/partner-gallery/experian-logo.png) | [Experian](./partner-experian.md) es un proveedor de corrección y verificación de identidad que realiza evaluaciones de riesgos basadas en atributos de usuario para evitar fraudes. |
|![Captura de pantalla de un logotipo de IDology.](./media/partner-gallery/idology-logo.png) | [IDology](./partner-idology.md) es un proveedor de verificación y revisión de identidades con soluciones de verificación de identidad, soluciones de prevención de fraudes y soluciones de cumplimiento, entre otras.|
|![Captura de pantalla de un logotipo de Jumio.](./media/partner-gallery/jumio-logo.png) | [Jumio](./partner-jumio.md) es un servicio de comprobación de identificadores que habilita la comprobación automatizada de identificadores en tiempo real, de modo que protege los datos de los clientes. |
| ![Captura de pantalla de un logotipo de LexisNexis.](./media/partner-gallery/lexisnexis-logo.png) | [LexisNexis](./partner-lexisnexis.md) es un proveedor validación de identidades y generación de perfiles que comprueba la identificación del usuario y proporciona una evaluación de riesgos completa basada en el dispositivo del usuario. |
| ![Captura de pantalla de un logotipo de Onfido](./media/partner-gallery/onfido-logo.png) | [Onfido](./partner-onfido.md) es un id. de documento y solución de verificación biométrica facial que permite a las empresas satisfacer sus requisitos de identidad y *Reconocimiento del cliente* en tiempo real.  |

## <a name="additional-information"></a>Información adicional

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](./tutorial-create-user-flows.md?pivots=b2c-custom-policy&tabs=applications)

## <a name="next-steps"></a>Pasos siguientes

Seleccione un asociado de la tabla mencionada para aprender a integrar su solución con Azure AD B2C.