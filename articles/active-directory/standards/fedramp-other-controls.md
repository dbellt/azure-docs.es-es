---
title: Configuración de controles adicionales para cumplir el nivel de impacto de FedRAMP High
description: Instrucciones detalladas sobre cómo configurar controles adicionales para cumplir los niveles de impacto de FedRAMP High.
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: barbaraselden
ms.author: baselden
manager: celested
ms.reviewer: martinco
ms.date: 4/26/2021
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 16b9842214b107760bf6e4a677099a7415bc6ff2
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108294249"
---
# <a name="configure-additional-controls-to-achieve-fedramp-high-impact-level"></a>Configuración de controles adicionales para cumplir el nivel de impacto de FedRAMP High

La siguiente lista de controles (y mejoras de control) de la familias siguientes puede requerir configuración en el inquilino de Azure AD.

Cada fila de las tablas siguientes proporciona instrucciones prescriptivas que le ayudan a desarrollar la respuesta de su organización a las responsabilidades compartidas respecto al control o la mejora de control.

## <a name="audit--accountability"></a>Auditoría y rendición de cuentas

* AU-02 Auditoría de eventos

* AU-03 Contenido de la auditoría
* AU-06 Revisión, análisis e informes de auditoría


| Identificador y subparte del control| Responsabilidades e instrucciones del cliente |
| - | - |
| AU-02 <br>AU-03 <br>AU-03(1)<br>AU-03(2)| **Asegurarse de que el sistema sea capaz de auditar los eventos definidos en la parte a de AU-02 y coordinarse con otras entidades dentro del subconjunto de eventos auditables de la organización para apoyar las investigaciones a posteriori. Implementar la gestión centralizada de los registros de auditoría**.<p>Todas las operaciones de ciclo de vida de la cuenta (creación, modificación, habilitación, deshabilitación y eliminación de cuentas) se auditan dentro de los registros de auditoría de Azure AD. Todos los eventos de autenticación y autorización se auditan en los registros de inicio de sesión de Azure AD y los riesgos detectados se auditan en los registros de Identity Protection. Cada uno de estos registros se puede transmitir directamente a una solución de Administración de eventos e información de seguridad (SIEM), como Azure Sentinel. También puede usar Azure Event Hubs para integrar registros con soluciones SIEM de terceros.<p>Eventos de auditoría<li> [Informes de actividad de auditoría en el portal de Azure Active Directory](https://docs.microsoft.com///azure/active-directory/reports-monitoring/concept-audit-logs)<li> [Informes de actividad de inicio de sesión en el portal de Azure Active Directory](https://docs.microsoft.com///azure/active-directory/reports-monitoring/concept-sign-ins)<li>[Cómo: Investigar los riesgos](https://docs.microsoft.com///azure/active-directory/identity-protection/howto-identity-protection-investigate-risk)<p>Integraciones de SIEM<li> [Azure Sentinel: conexión de datos desde Azure Active Directory (Azure AD)](https://docs.microsoft.com///azure/sentinel/connect-azure-active-directory)<li>[Transmisión a un centro de eventos de Azure y a otros SIEM](https://docs.microsoft.com///azure/active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub) |
| AU-06<br>AU-06(1)<br>AU-06(3)<br>AU-06(4)<br>AU-06(5)<br>AU-06(6)<br>AU-06(7)<br>AU-06(10)<br>| **Revisar y analizar los registros de auditoría al menos una vez a la semana para identificar actividades inapropiadas o inusuales e informar de los resultados al personal adecuado**. <p>Las instrucciones proporcionadas anteriormente para AU-02 y AU-03 permiten revisar semanalmente los registros de auditoría y los informes al personal adecuado. No puede cumplir estos requisitos solo con Azure AD. También debe usar una solución SIEM como Azure Sentinel.<p>[¿Qué es Azure Sentinel?](https://docs.microsoft.com///azure/sentinel/overview) |

## <a name="incident-response"></a>Respuesta a los incidentes

* IR-04 Tratamiento de incidentes

* IR-05 Supervisión de incidentes

| Identificador y subparte del control| Responsabilidades e instrucciones del cliente |
| - | - |
| IR-04<br>IR-04(1)<br>IR-04(2)<br>IR-04(3)<br>IR-04(4)<br>IR-04(6)<br>IR-04(8)<br>IR-05<br>IR-05(1)| **Implementar las funcionalidades de control y supervisión de incidentes, incluida la gestión automatizada de incidentes, la reconfiguración dinámica, la continuidad de las operaciones, la correlación de la información, las amenazas internas, la correlación con organizaciones externas, la supervisión de incidentes y el seguimiento automatizado**. <p>Todos los cambios de configuración se registran en los registros de auditoría. Los eventos de autenticación y autorización se auditan en los registros de inicio de sesión, y los riesgos detectados se auditan en los registros de Identity Protection. Cada uno de estos registros se puede transmitir directamente a una solución de Administración de eventos e información de seguridad (SIEM), como Azure Sentinel. También puede usar Azure Event Hubs para integrar registros con soluciones SIEM de terceros. Automatice la reconfiguración dinámica en función de los eventos dentro de SIEM mediante MSGraph o Azure AD PowerShell.<p>Eventos de auditoría<br><li>[Informes de actividad de auditoría en el portal de Azure Active Directory](https://docs.microsoft.com///azure/active-directory/reports-monitoring/concept-audit-logs)<li>[Informes de actividad de inicio de sesión en el portal de Azure Active Directory](https://docs.microsoft.com///azure/active-directory/reports-monitoring/concept-sign-ins)<li>[Cómo: Investigar los riesgos](https://docs.microsoft.com///azure/active-directory/identity-protection/howto-identity-protection-investigate-risk)<p>Integraciones de SIEM<li>[Azure Sentinel: conexión de datos desde Azure Active Directory (Azure AD)](https://docs.microsoft.com///azure/sentinel/connect-azure-active-directory)<li>[Transmisión a un centro de eventos de Azure y a otros SIEM](https://docs.microsoft.com///azure/active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub)<p>Reconfiguración dinámica<li>[Módulo AzureAD](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0)<li>[Overview of Microsoft Graph (Información general de Microsoft Graph)](https://docs.microsoft.com/graph/overview?view=graph-rest-1.0) |


  
## <a name="personnel-security"></a>Seguridad del personal

* PS-04 Finalización del contrato del personal

| Identificador y subparte del control| Responsabilidades e instrucciones del cliente |
| - | - |
| PS-04<br>PS-04(2)| **Informar automáticamente al personal responsable de deshabilitar el acceso al sistema.** <p>Deshabilite las cuentas y revoque todos los autenticadores y credenciales asociados en un plazo de 8 horas. <p>Configure el aprovisionamiento (incluida la deshabilitación tras la finalización) de las cuentas de Azure AD desde sistemas de RR. UU. externos, Active Directory local o directamente en la nube. Ponga fin a todo el acceso al sistema revocando las sesiones existentes. <p>Account Provisioning (Aprovisionamiento de cuentas)<li> Consulte instrucciones detalladas en AC-02. <p>Revocación de todos los autenticadores asociados <li> [Revocación del acceso de usuario en un emergencia de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/enterprise-users/users-revoke-access) |


## <a name="system--information-integrity"></a>Integridad del sistema y de la información

* SI-04 Supervisión del sistema de información

 Identificador y subparte del control| Responsabilidades e instrucciones del cliente |
| - | - |
| SI-04<br>SI-04(1)| **Implementar un sistema de supervisión y detección de intrusos en todo el sistema de información**<p>Incluya todos los registros de Azure AD (auditoría, inicio de sesión, Identity Protection) en la solución de supervisión del sistema de información. <p>Transmita registros de Azure AD a una solución SIEM (consulte IA-04). |

## <a name="next-steps"></a>Pasos siguientes

[Configuración de controles de acceso](fedramp-access-controls.md)

[Configuración de los controles de identificación y autenticación](fedramp-identification-and-authentication-controls.md)

[Configuración de otros controles](fedramp-other-controls.md)
 
