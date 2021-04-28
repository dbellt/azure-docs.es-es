---
title: Seguridad integral en Azure | Microsoft Docs
description: En este artículo se proporciona un mapa de los servicios de Azure que le ayudan a proteger los recursos en la nube, así como a detectar e investigar las amenazas.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
ms.assetid: a5a7f60a-97e2-49b4-a8c5-7c010ff27ef8
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 4/07/2021
ms.author: terrylan
ms.openlocfilehash: 7612732f01075759d76c2bce5e8c710afc676642
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108132870"
---
# <a name="end-to-end-security-in-azure"></a>Seguridad integral en Azure
Una de las mejores razones para usar Azure en sus aplicaciones y servicios es poder aprovechar su amplia gama de funcionalidades y herramientas de seguridad. Estas herramientas y funcionalidades permiten crear soluciones seguras en la plataforma Azure segura. Microsoft Azure proporciona confidencialidad, integridad y disponibilidad para los datos del cliente, al mismo tiempo que hace posible una responsabilidad transparente.

En el diagrama y la documentación siguientes se presentan los servicios de seguridad de Azure. Estos servicios le ayudan a satisfacer las necesidades de seguridad de su empresa y a proteger a los usuarios, los dispositivos, los recursos, los datos y las aplicaciones en la nube.

## <a name="microsoft-security-services-map"></a>Mapa de servicios de seguridad de Microsoft

El mapa de servicios de seguridad organiza los servicios por los recursos que protegen (columna). El diagrama también agrupa los servicios en las siguientes categorías (fila):

- Seguridad y protección: servicios que le permiten implementar una estrategia de defensa en profundidad por niveles que abarca la identidad, los hosts, las redes y los datos. Esta colección de funcionalidades y servicios de seguridad proporciona una manera de comprender y mejorar la posición de seguridad en el entorno de Azure.
- Detección de amenazas: servicios que identifican actividades sospechosas y facilitan la mitigación de la amenaza.
- Investigación y respuesta: servicios que extraen datos de registro para que pueda evaluar las actividades sospechosas y responder a ellas.

El diagrama incluye el programa Azure Security Benchmark, una colección de recomendaciones de seguridad de gran impacto que puede usar para ayudar a proteger los servicios que usa en Azure.

:::image type="content" source="media/end-to-end/security-diagram.svg" alt-text="Diagrama que muestra los servicios de seguridad integral de Azure." border="false":::

## <a name="security-controls-and-baselines"></a>Controles de seguridad y líneas de base
El programa [Azure Security Benchmark](../benchmarks/introduction.md) incluye una colección de recomendaciones de seguridad de gran impacto que puede usar para ayudar a proteger los servicios que usa en Azure:

- Controles de seguridad: estas recomendaciones se suelen aplicar tanto al inquilino como a los servicios de Azure. Cada recomendación señala una lista de partes interesadas que suelen estar implicadas en el planeamiento, la aprobación o la implementación de la prueba comparativa.
- Líneas de base del servicio: aplican los controles a servicios individuales de Azure para proporcionar recomendaciones sobre la configuración de seguridad de dicho servicio.

## <a name="secure-and-protect"></a>Seguridad y protección

:::image type="content" source="media/end-to-end/secure-and-protect.svg" alt-text="Diagrama que muestra los servicios de Azure que le ayudan a proteger los recursos en la nube." border="false":::

| Servicio | Descripción |
|------|--------|
| [Azure Security Center](../../security-center/security-center-introduction.md)| Un sistema unificado de administración de seguridad de la infraestructura que fortalece la posición de seguridad de los centros de datos y proporciona protección avanzada contra amenazas en todas las cargas de trabajo híbridas que se encuentran en la nube, ya sea que estén en Azure o no, así como también en el entorno local. |
| **Administración de&nbsp;&&nbsp;identidades&nbsp;y acceso** | |
| [Azure Active Directory (AD)](../../active-directory/fundamentals/active-directory-whatis.md)| El servicio de administración de identidades y acceso basado en la nube de Microsoft.  |
|  | [Acceso condicional](../../active-directory/conditional-access/overview.md) es la herramienta que usa Azure AD para reunir las señales, tomar decisiones y aplicar las directivas de la organización. |
|  | [Servicios de dominio](../../active-directory-domain-services/overview.md) es la herramienta que usa Azure AD para proporcionar servicios de dominio administrados, como la unión a un dominio, la directiva de grupo, el protocolo ligero de acceso a directorios (LDAP) y la autenticación Kerberos/NTLM. |
|  | [Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md) es un servicio de Azure AD que permite administrar, controlar y supervisar el acceso a recursos importantes de la organización. |
|  | La [autenticación multifactor](../../active-directory/authentication/concept-mfa-howitworks.md) es la herramienta que usa Azure AD para ayudar a proteger el acceso a los datos y las aplicaciones mediante la solicitud de una segunda forma de autenticación. |
| [Azure AD Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md) | Herramienta que permite a las organizaciones automatizar la detección y corrección de riesgos basados en identidades, investigar riesgos mediante datos del portal y exportar datos de detección de riesgos a utilidades de terceros para analizarlos más a fondo. |
| **Red de&nbsp;&&nbsp;infraestructura** |  |
| [VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md) | Una puerta de enlace de red virtual que se usa para enviar tráfico cifrado entre una red virtual de Azure y una ubicación local a través de la red pública de Internet y para enviar tráfico cifrado entre redes virtuales de Azure a través de la red de Microsoft. |
| [Azure DDoS Protection Estándar](../../ddos-protection/ddos-protection-overview.md) | Proporciona características de mitigación de DDoS mejoradas para la defensa contra los ataques DDoS. Se ajusta de forma automática para proteger los recursos específicos de Azure de una red virtual. |
| [Azure Front Door](../../frontdoor/front-door-overview.md) | Un punto de entrada global y escalable que usa la red perimetral global de Microsoft para crear aplicaciones web rápidas, seguras y muy escalables. |
| [Azure Firewall](../../firewall/overview.md) | Se trata de un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones. |
| [Azure Key Vault](../../key-vault/general/overview.md) | Un almacén de secretos seguro para tokens, contraseñas, certificados, claves de API y otros secretos. Key Vault también se puede usar para crear y controlar las claves de cifrado utilizadas para cifrar los datos. |
| [HSM administrado por Key Vault (versión preliminar)](../../key-vault/managed-hsm/overview.md) | Un servicio en la nube que cumple los estándares, totalmente administrado, de alta disponibilidad y de un solo inquilino que le permite proteger las claves criptográficas de las aplicaciones de nube mediante dispositivos HSM validados de FIPS 140-2 nivel 3. |
| [Azure Private Link](../../private-link/private-link-overview.md) | Permite acceder a los servicios PaaS de Azure (por ejemplo, Azure Storage y SQL Database) y a los servicios hospedados en Azure que son propiedad de los clientes, o a los servicios de asociados, a través de un punto de conexión privado de la red virtual. |
| [Azure Application Gateway](../../application-gateway/overview.md) | Un equilibrador de carga avanzado para administrar el tráfico a las aplicaciones web. Application Gateway puede tomar decisiones de enrutamiento basadas en atributos adicionales de una solicitud HTTP, por ejemplo los encabezados de host o la ruta de acceso del URI. |
| [Azure Service Bus](../../service-bus-messaging/service-bus-messaging-overview.md) | Un agente de mensajes empresarial totalmente administrado que incluye colas de mensajes y temas que se pueden publicar y a los que es posible suscribirse. Service Bus se usa para desacoplar aplicaciones de servicios. |
| [Firewall de aplicaciones web](../../web-application-firewall/overview.md) | Ofrece una protección centralizada de las aplicaciones web contra vulnerabilidades de seguridad comunes. WAF se puede implementar con Azure Application Gateway y Azure Front Door. |
| **Datos y aplicaciones** |  |
| [Azure Backup](../../backup/backup-overview.md) | Proporciona soluciones sencillas, seguras y rentables tanto para realizar copias de seguridad de datos de la nube de Microsoft Azure como para recuperarlos. |
| [Cifrado del servicio Azure Storage](../../storage/common/storage-service-encryption.md) | Cifra automáticamente los datos antes de que se almacenen y los descifra automáticamente cuando los recupera. |
| [Azure Information Protection](/azure/information-protection/what-is-information-protection) | Una solución basada en la nube que permite a las organizaciones descubrir, clasificar y proteger los documentos y correos electrónicos mediante la aplicación de etiquetas. |
| [API Management](../../api-management/api-management-key-concepts.md) | Una manera de crear puertas de enlace de API coherentes y modernas para servicios de back-end existentes. |
| [Azure confidential computing](../../confidential-computing/overview.md) (Computación confidencial de Azure) | Permite aislar los datos confidenciales mientras se procesan en la nube. |
| [Azure DevOps](/azure/devops/user-guide/what-is-azure-devops) | Los proyectos de desarrollo se benefician de varias capas de tecnologías de seguridad y gobernanza, prácticas operativas y directivas de cumplimiento cuando se almacenan en Azure DevOps. |
| **Acceso al cliente** |  |
| [Azure AD External Identities](../../active-directory/external-identities/compare-with-b2c.md) | Mediante External Identities de Azure AD, puede facilitar a personas ajenas a una organización el acceso a sus aplicaciones y recursos, además de permitirles iniciar sesión con la identidad que prefieran. |
|  | Puede compartir sus aplicaciones y recursos con usuarios externos mediante la colaboración de [Azure AD B2B.](../../active-directory/external-identities/what-is-b2b.md) |
|  | [Azure AD B2C](../../active-directory-b2c/overview.md) tiene cabida para millones de usuarios y miles de millones de autenticaciones al día y permite supervisar y controlar automáticamente amenazas como la denegación de servicio, la aplicación de la contraseña o los ataques por fuerza bruta. |

## <a name="detect-threats"></a>Detectar amenazas

:::image type="content" source="media/end-to-end/detect-threats.svg" alt-text="Diagrama que muestra los servicios de Azure que detectan amenazas." border="false":::

| Servicio | Descripción |
|------|--------|
| [Azure Defender](../../security-center/azure-defender.md) | Ofrece protección inteligente y avanzada para las cargas de trabajo y recursos híbridos y de Azure. El panel de Azure Defender en Security Center proporciona visibilidad y control de las características de protección de cargas de trabajo en la nube en el entorno. |
| [Azure Sentinel](../../sentinel/overview.md) | Una solución nativa de nube y escalable para la administración de eventos de información de seguridad (SIEM) y la respuesta automatizada de orquestación de seguridad (SOAR). Sentinel ofrece análisis de seguridad inteligente e inteligencia frente a amenazas en toda la empresa, de forma que proporciona una única solución para la detección de alertas, la visibilidad de amenazas, la búsqueda proactiva y la respuesta a amenazas. |
| **Administración de&nbsp;&&nbsp;identidades&nbsp;y acceso** |  |
| [Microsoft 365 Defender](/microsoft-365/security/defender/microsoft-365-defender) | Un conjunto unificado de defensa de la empresa que coordina de forma nativa la detección, la prevención, la investigación y la respuesta en relación con los puntos de conexión, las identidades, el correo electrónico y las aplicaciones para proporcionar protección integrada frente a ataques sofisticados antes y después de una infracción. |
|  | [Microsoft Defender para punto de conexión](/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint) es una plataforma de seguridad empresarial para puntos de conexión diseñada para evitar, detectar, investigar y responder a amenazas avanzadas. |
|  | [Microsoft Defender for Identity](/defender-for-identity/what-is) es una solución de seguridad basada en la nube que aprovecha las señales de Active Directory local para identificar, detectar e investigar amenazas avanzadas, identidades puestas en peligro y acciones malintencionadas dirigidas a la organización efectuadas por usuarios internos. |
| [Azure AD Identity Protection](../../active-directory/identity-protection/howto-identity-protection-configure-notifications.md) | Envía dos tipos de correos electrónicos de notificación automatizados para ayudarle a administrar el riesgo de los usuarios y las detecciones de riesgos: correo electrónico de usuarios en riesgo detectados y correo electrónico de resumen semanal. |
| **Infraestructura y red** |  |
| [Azure Defender para IoT](../../defender-for-iot/overview.md) | Una solución de seguridad unificada para identificar dispositivos, vulnerabilidades y amenazas de IoT y OT. Permite proteger todo el entorno de IoT/OT, independientemente de que necesite proteger dispositivos IoT/OT existentes o incorporar seguridad a nuevas innovaciones de IoT. |
| [Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) | Proporciona herramientas para supervisar, diagnosticar, ver métricas y habilitar o deshabilitar registros de recursos en una red virtual de Azure. Network Watcher está diseñado para supervisar y reparar el estado de la red de los productos de IaaS, lo que incluye máquinas virtuales, redes virtuales, puertas de enlace de aplicaciones y equilibradores de carga. |
| [Registro de auditoría de Azure Policy](../../governance/policy/overview.md) | Ayuda a aplicar los estándares de la organización y a evaluar el cumplimiento a gran escala. Azure Policy utiliza registros de actividad, que están habilitados automáticamente para incluir el origen del evento, la fecha, el usuario, la marca de tiempo, las direcciones de origen y de destino, y otros elementos útiles. |
| **Datos y aplicaciones** |  |
| [Azure Defender para registros de contenedor](../../security-center/defender-for-container-registries-introduction.md) | Incluye un detector de vulnerabilidades para examinar las imágenes de los registros de Azure Container Registry basados en Azure Resource Manager y proporciona una mayor visibilidad de las vulnerabilidades de las imágenes. |
| [Azure Defender para Kubernetes](../../security-center/defender-for-kubernetes-introduction.md) | Proporciona protección contra amenazas de nivel de clúster al supervisar los servicios administrados por AKS mediante los registros recuperados por Azure Kubernetes Service (AKS). |
| [Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) | Un agente de seguridad de acceso a la nube (CASB) que funciona en varias nubes. Proporciona visibilidad enriquecida, control sobre el viaje de los datos y análisis sofisticados para identificar y combatir las ciberamenazas en todos los servicios en la nube. |

## <a name="investigate-and-respond"></a>Investigación y respuesta

:::image type="content" source="media/end-to-end/investigate-and-respond.svg" alt-text="Diagrama que muestra los servicios de Azure que le ayudan a investigar y responder a las amenazas." border="false":::

| Servicio | Descripción |
|------|--------|
| [Azure Sentinel](../../sentinel/hunting.md) | Eficaces herramientas de búsqueda y consulta para buscar amenazas de seguridad en los orígenes de datos de la organización. |
| [Registros &nbsp; y &nbsp; métricas &nbsp; de &nbsp; Azure Monitor](../../azure-monitor/overview.md) | Ofrece una solución completa para recopilar y analizar la telemetría del entorno local y en la nube y realizar acciones en función de estos datos. Azure Monitor [recopila datos de diversos orígenes y los agrega](../../azure-monitor/data-platform.md#observability-data-in-azure-monitor) a una plataforma de datos común donde se pueden usar en el análisis, la visualización y la generación de alertas. |
| **Administración de&nbsp;&&nbsp;identidades&nbsp;y acceso** |  |
| [Informes &nbsp; y &nbsp; supervisión &nbsp; de &nbsp; Azure AD](../../active-directory/reports-monitoring/index.yml) | Los [informes de Azure AD](../../active-directory/reports-monitoring/overview-reports.md) proporcionan una vista completa de la actividad del entorno. |
|  | La [supervisión de Azure AD](../../active-directory/reports-monitoring/overview-monitoring.md) le permite enrutar los registros de actividad de Azure AD a diferentes puntos de conexión.|
| [Historial de auditoría de PIM de Azure AD](../../active-directory/privileged-identity-management/pim-how-to-use-audit-log.md) | Muestra todas las asignaciones de roles y activaciones en los últimos 30 días de todos los roles con privilegios. |
| **Datos y aplicaciones** |  |
| [Microsoft Cloud App Security](/cloud-app-security/investigate) | Proporciona herramientas para mejorar su comprensión de lo que sucede en el entorno de nube. |

## <a name="next-steps"></a>Pasos siguientes

- Comprenda la [responsabilidad compartida en la nube](shared-responsibility.md).

- Conozca las [opciones de aislamiento en la nube de Azure ](isolation-choices.md) contra usuarios malintencionados y no malintencionados.