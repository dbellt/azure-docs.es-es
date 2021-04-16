---
title: Visibilidad y control de aplicaciones con Microsoft Cloud App Security
description: Conozca los métodos para identificar los niveles de riesgo de las aplicaciones, detener vulneraciones y fugas en tiempo real y usar conectores de aplicaciones para aprovechar las ventajas de las API de proveedores para la visibilidad y el gobierno.
services: active-directory
author: iantheninja
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 02/03/2020
ms.author: iangithinji
ms.collection: M365-identity-device-management
ms.openlocfilehash: d9a9f56d70e049200cee0c3655a9998feccfa301
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377843"
---
# <a name="cloud-app-visibility-and-control"></a>Control y visibilidad de aplicaciones en la nube

Para sacar el máximo partido de las aplicaciones y los servicios en la nube, un equipo de TI debe encontrar el equilibrio adecuado entre respaldar el acceso y conservar el control para proteger datos críticos. Microsoft Cloud App Security proporciona visibilidad enriquecida, control durante el trayecto de los datos y análisis sofisticados para identificar y combatir las ciberamenazas en todos los servicios en la nube de Microsoft y de terceros.

## <a name="discover-and-manage-shadow-it-in-your-network"></a>Detección y administración de shadow IT en la red

Cuando se pregunta a los administradores de TI cuántas aplicaciones en la nube creen que usan sus empleados, la media es de 30 o 40. En realidad, la media de aplicaciones que usan los empleados de su organización es de más de 1000 aplicaciones distintas. Shadow IT le permite conocer e identificar qué aplicaciones se usan y cuál es el nivel de riesgo que suponen. El 80 % de los empleados usan aplicaciones no autorizadas que nadie ha revisado y que puede que no sean compatibles con las directivas de seguridad y cumplimiento. Además, debido a que los clientes pueden acceder a sus recursos y aplicaciones desde fuera de la red corporativa, ya no es suficiente disponer de reglas y directivas en los firewalls.

Use Microsoft Cloud App Discovery (una característica de Azure Active Directory Premium P1) para detectar las aplicaciones que se usan, explorar el riesgo de estas aplicaciones, configurar directivas para identificar nuevas aplicaciones de riesgo y no autorizar estas aplicaciones para bloquearlas de forma nativa mediante el proxy o dispositivo de firewall.

- Detección e identificación de shadow IT
- Evaluación y análisis
- Administración de las aplicaciones
- Creación avanzada de informes sobre detección de Shadow IT
- Control de aplicaciones autorizadas
 
### <a name="learn-more"></a>Más información

- [Detección y administración de shadow IT en la red](/cloud-app-security/tutorial-shadow-it)
- [Aplicaciones detectadas con Cloud App Security ](/cloud-app-security/discovered-apps)
 
## <a name="user-session-visibility-and-control"></a>Visibilidad y control de la sesión del usuario 

En el área de trabajo actual, a menudo no basta con saber lo que ocurre en su entorno en la nube a posteriori. Desea detener las vulneraciones y fugas en tiempo real antes de que los empleados pongan en peligro los datos y la organización de forma intencionada o accidental. Junto con Azure Active Directory (Azure AD), Microsoft Cloud App Security ofrece estas funcionalidades en una experiencia completa que se integra con Control de aplicaciones de acceso condicional. 

El control de sesión usa una arquitectura de proxy inverso y se integra de forma exclusiva con el acceso condicional de Azure AD. El acceso condicional de Azure AD permite exigir controles de acceso en las aplicaciones de la organización, en función de ciertas condiciones. Las condiciones definen quién (usuario o grupo de usuarios), qué (aplicaciones en la nube) y dónde (ubicaciones y redes) se aplica una directiva de acceso condicional. Después de determinar las condiciones, puede enrutar a los usuarios a Cloud App Security donde puede proteger los datos en tiempo real.  

Con este control, puede:  
- Controlar las descargas de archivos
- Supervisar escenarios de B2B  
- Controlar el acceso a los archivos  
- Proteger documentos durante la descarga  
 
### <a name="learn-more"></a>Más información

- [Protección de aplicaciones con el control de sesión en Cloud App Security](/cloud-app-security/proxy-intro-aad)
 
## <a name="advanced-app-visibility-and-controls"></a>Visibilidad y controles avanzados de la aplicación 

Los conectores de aplicaciones usan las API de los proveedores de aplicaciones para permitir una mayor visibilidad y control de las aplicaciones a las que se conecta mediante Microsoft Cloud App Security. Cloud App Security aprovecha las API que proporciona el proveedor de nube. Cada servicio tiene su propia plataforma y limitaciones de API, como la limitación de peticiones, los límites de API, las ventanas de API dinámicas de cambio de tiempo, etc. El equipo de producto de Cloud App Security trabajó con estos servicios para optimizar el uso de las API y proporcionar el mejor rendimiento. Teniendo en cuenta las diferentes limitaciones que los servicios imponen en sus API, los motores de Cloud App Security usan la capacidad máxima permitida. Algunas operaciones, como el análisis de todos los archivos del inquilino, requieren numerosas llamadas API, por lo que se reparten durante un período más largo. Es previsible que algunas directivas se ejecuten durante varias horas o días. 
 
### <a name="learn-more"></a>Más información  

- [Conexión de aplicaciones en Cloud App Security ](/cloud-app-security/enable-instant-visibility-protection-and-governance-actions-for-your-apps)

## <a name="next-steps"></a>Pasos siguientes

- [Detección y administración de shadow IT en la red](/cloud-app-security/tutorial-shadow-it)
- [Aplicaciones detectadas con Cloud App Security ](/cloud-app-security/discovered-apps)
- [Protección de aplicaciones con el control de sesión en Cloud App Security](/cloud-app-security/proxy-intro-aad)
- [Conexión de aplicaciones en Cloud App Security ](/cloud-app-security/enable-instant-visibility-protection-and-governance-actions-for-your-apps)