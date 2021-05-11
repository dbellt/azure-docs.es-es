---
title: Configuración de Azure Active Directory para cumplir el nivel de alto impacto de FedRAMP High
description: información general sobre cómo puede cumplir el nivel de alto impacto de FedRAMP High para su organización mediante Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: barbaraselden
ms.author: baselden
manager: mtillman
ms.reviewer: martinco
ms.date: 4/26/2021
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 965fbe7ec1f46e006e30d721bf306a0debaf8473
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294159"
---
# <a name="configure-azure-active-directory-to-meet-fedramp-high-impact-level"></a>Configuración de Azure Active Directory para cumplir el nivel de alto impacto de FedRAMP High

El programa [Federal Risk and Authorization Management Program](https://www.fedramp.gov/) (FedRAMP) es un proceso de evaluación y autorización para proveedores de servicios en la nube (CSP) que crean ofertas de soluciones en la nube (OSO) para usarse con agencias federales. Azure y Azure Government han obtenido una aprobación inicial [Provisional Authority to Operate (P-ATO) en el nivel de alto impacto](https://docs.microsoft.com/compliance/regulatory/offering-fedramp) de la Junta de autorizaciones conjuntas, el grado más alto para la acreditación de FedRAMP.

Azure proporciona la capacidad de cumplir todos los requisitos de control para obtener una clasificación alta de FedRAMP, ya sea como director de seguridad o como una agencia federal. Es responsabilidad de su organización completar las configuraciones o procesos adicionales para el cumplimiento. Esta responsabilidad se aplica tanto a los proveedores de servicios criptográficos que buscan una autorización de alto nivel de FedRAMP para su director de seguridad, como a las agencias federales que buscan una autorización para operar (ATO). 

## <a name="microsoft-and-fedramp"></a>Microsoft y FedRAMP 

Microsoft Azure admite más servicios en el [nivel de alto impacto de FedRAMP High](https://docs.microsoft.com/azure/azure-government/compliance/azure-services-in-fedramp-auditscope) que cualquier otro proveedor de servicios criptográficos. Y aunque FedRAMP High en la nube pública de Azure cumplirá los requisitos de muchos clientes de la Administración Pública de Estados Unidos, las agencias con requisitos más estrictos podrán confiar en la nube de Azure Government. La nube de Azure Government proporciona medidas de seguridad adicionales, como una revisión más exhaustiva del personal. 

Microsoft tiene la obligación de volver a certificar sus servicios en la nube cada año para conservar sus autorizaciones. Para ello, Microsoft supervisa y evalúa continuamente sus controles de seguridad y demuestra que la seguridad de sus servicios sigue en cumplimiento.

* [Autorizaciones de FedRAMP para servicios en la nube de Microsoft](https://marketplace.fedramp.gov/)

* [Informes de auditoría de FedRAMP para Microsoft](https://aka.ms/MicrosoftFedRAMPAuditDocuments)

Para recibir otros informes de FedRAMP, envíe un correo electrónico a [Azure Federal Documentation](mailto:AzFedDoc@microsoft.com).

Hay varias rutas para lograr la autorización de FedRAMP. Puede reutilizar el paquete de autorización existente de Microsoft Azure y las instrucciones de este documento para reducir significativamente el tiempo y el esfuerzo necesarios para obtener una autorización ATO o P-ATO. Puede encontrar más información sobre FedRAMP en el [sitio web de FedRAMP](https://www.fedramp.gov/).

 ## <a name="scope-of-guidance"></a>Ámbito de la guía

La línea de base de FedRAMP High consta de 421 controles y mejoras de control del [catálogo de controles de seguridad 800-53 de NIST revisión 4](https://csrc.nist.gov/publications/detail/sp/800-53/rev-4/final). Si procede, se incluyó información aclaratoria acerca de [800-53 revisión 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final). Este conjunto de artículos cubre un subconjunto de estos controles, que están relacionados con la identidad y debe configurar. Proporcionamos instrucciones prescriptivas para ayudarle a lograr el cumplimiento en los controles que tiene la responsabilidad de configurar en Azure Active Directory (Azure AD). Para cumplir por completo algunos requisitos de control de identidad, es posible que necesite usar otros sistemas. Otros sistemas pueden incluir una herramienta de administración de eventos e información de seguridad (SIEM), como Azure Sentinel. Si usa servicios de Azure distintos de Azure Active Directory, habrá otros controles que deberá tener en cuenta, y podrá usar las funcionalidades que Azure ya tiene para cumplir con los controles.

Recursos de FedRAMP

* [Federal Risk and Authorization Management Program](https://www.fedramp.gov/) (Programa de administración federal de riesgos y autorizaciones)

* [Marco de evaluación de seguridad de FedRAMP](https://www.fedramp.gov/assets/resources/documents/FedRAMP_Security_Assessment_Framework.pdf)

* [Guía de agencias para autorizaciones de FedRAMP](https://www.fedramp.gov/assets/resources/documents/Agency_Guide_for_Reuse_of_FedRAMP_Authorizations.pdf)

* [Administración del cumplimiento en la nube de Microsoft](https://www.microsoft.com/trustcenter/common-controls-hub)

* [Nube de Microsoft Government](https://go.microsoft.com/fwlink/p/?linkid=2087246)

* [Ofertas de cumplimiento de Azure](https://aka.ms/azurecompliance)

* [Descripción general del ejemplo de plano técnico con un nivel de impacto alto de FedRAMP](https://docs.microsoft.com/azure/governance/blueprints/samples/fedramp-h/)

* [Centro de cumplimiento de Microsoft 365](https://docs.microsoft.com///microsoft-365/compliance/microsoft-365-compliance-center)

* [Administrador de cumplimiento de Microsoft](https://docs.microsoft.com///microsoft-365/compliance/compliance-manager)

 

## <a name="next-steps"></a>Pasos siguientes

[Configuración de controles de acceso](fedramp-access-controls.md)

[Configuración de los controles de identificación y autenticación](fedramp-identification-and-authentication-controls.md)

[Configuración de otros controles](fedramp-other-controls.md)

 
