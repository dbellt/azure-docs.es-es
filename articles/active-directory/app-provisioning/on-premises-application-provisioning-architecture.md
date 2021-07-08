---
title: Arquitectura de aprovisionamiento de aplicaciones locales de Azure AD | Microsoft Docs
description: Describe la información general de la arquitectura de aprovisionamiento de aplicaciones locales.
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 05/28/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 696f2f9e54e350b2cce952cbec8d745779e1d00a
ms.sourcegitcommit: ff1aa951f5d81381811246ac2380bcddc7e0c2b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111570365"
---
# <a name="azure-ad-on-premises-application-provisioning-architecture"></a>Arquitectura de aprovisionamiento de aplicaciones locales de Azure AD

>[!IMPORTANT]
> La versión preliminar de aprovisionamiento locales se encuentra actualmente en una versión preliminar solo por invitación. Puede solicitar acceso a la funcionalidad [aquí](https://aka.ms/onpremprovisioningpublicpreviewaccess). Abriremos la versión preliminar para más clientes y conectores durante los próximos meses a medida que nos preparamos para la disponibilidad general.

## <a name="overview"></a>Información general

En el diagrama siguiente se muestra información general de cómo funciona el aprovisionamiento de aplicaciones local.

![Architecture](.\media\on-premises-application-provisioning-architecture\arch-3.png)

Existen tres componentes principales para el aprovisionamiento de usuarios en una aplicación local.

- El agente de aprovisionamiento proporciona conectividad entre Azure AD y el entorno local.
- El host ECMA convierte las solicitudes de aprovisionamiento de Azure AD en solicitudes realizadas a la aplicación de destino. Actúa como puerta de enlace entre Azure AD y la aplicación. Permite importar los conectores ECMA2 existentes que se usan con Microsoft Identity Manager. Tenga en cuenta que el host ECMA no es necesario si ha creado una aplicación SCIM o una puerta de enlace SCIM.
- El servicio de aprovisionamiento de Azure AD actúa como motor de sincronización.

>[!NOTE]
> No se requiere la sincronización de MIM. Sin embargo, puede usar la sincronización de MIM para crear y probar el conector ECMA antes de importarlo en el host ECMA.


### <a name="firewall-requirements"></a>Requisitos de firewall

No es necesario abrir conexiones entrantes a la red corporativa. Los agentes de aprovisionamiento solo usan conexiones salientes al servicio de aprovisionamiento, lo que significa que no es necesario abrir los puertos de firewall para las conexiones entrantes. Tampoco necesita una red perimetral (DMZ), porque todas las conexiones son salientes y tienen lugar a través de un canal seguro. 

## <a name="agent-best-practices"></a>Procedimientos recomendados del agente
- Asegúrese de que el servicio Auto Update del agente de aprovisionamiento de Azure AD Connect se esté ejecutando. Se habilita de forma predeterminada al instalar el agente. La actualización automática es necesaria para que Microsoft admita su implementación.
- Evite todo tipo de inspección insertada en las comunicaciones TLS salientes entre los agentes y Azure. Este tipo de inspección insertada provoca la degradación en el flujo de la comunicación.
- El agente tiene que comunicarse con Azure y la aplicación; por eso, la ubicación del agente afecta a la latencia de esas dos conexiones. Puede probar a minimizar la latencia del tráfico de extremo a extremo optimizando cada conexión de red. Cada conexión se puede optimizar mediante uno de estos modos:
- Reducir la distancia entre los dos extremos del salto.
- Elegir la red adecuada por la que pasar. Por ejemplo, puede ser más rápido pasar por una red privada en lugar de la red pública de Internet debido a los vínculos dedicados.

## <a name="provisioning-agent-questions"></a>Preguntas sobre el agente de aprovisionamiento
**¿Cuál es la versión de disponibilidad general del agente de aprovisionamiento?**

Consulte [Agente de aprovisionamiento de Azure AD Connect: Historial de lanzamiento de versiones](provisioning-agent-release-version-history.md) para ver la versión más reciente en GA del agente de aprovisionamiento.

**¿Cómo puedo saber la versión de mi agente de aprovisionamiento?**

 1. Inicie sesión en el servidor de Windows en el que está instalado el agente de aprovisionamiento.
 2. Vaya al menú Panel de control -> Desinstalar o cambiar un programa.
 3. Busque la versión correspondiente a la entrada Microsoft Azure AD Connect Provisioning Agent.

**¿Microsoft inserta automáticamente las actualizaciones del agente de aprovisionamiento?**

Sí, Microsoft actualiza automáticamente el agente de aprovisionamiento si el servicio de Windows Agent Updater de Microsoft Azure AD Connect está en ejecución. Es necesario asegurarse de que el agente está actualizado para obtener soporte técnico para la solución de problemas.

**¿Puedo instalar el agente de aprovisionamiento en el mismo servidor que ejecuta Azure AD Connect o Microsoft Identity Manager (MIM)?**

Sí, puede instalar el agente de aprovisionamiento en el mismo servidor que ejecuta Azure AD Connect o MIM, pero no son necesarios.

**¿Cómo se puede configurar el agente de aprovisionamiento para usar un servidor proxy para la comunicación HTTP saliente?**

El agente de aprovisionamiento admite el uso de un proxy de salida. Puede configurarlo al editar el archivo de configuración del agente **C:\Archivos de programa\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config**. Agregue al archivo las siguientes líneas, hacia el final del archivo, justo antes de la etiqueta de cierre </configuration>. Reemplace las variables [proxy-server] y [proxy-port] con los valores del puerto y el nombre del servidor proxy.
```
    <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
            <proxy
                usesystemdefault="true"
                proxyaddress="http://[proxy-server]:[proxy-port]"
                bypassonlocal="true"
            />
        </defaultProxy>
    </system.net>
```
**¿Cómo se puede garantizar que el agente de aprovisionamiento sea capaz de comunicarse con el inquilino de Azure AD y que ningún firewall bloquee los puertos requeridos por el agente?**

También puede comprobar si todos los puertos necesarios están abiertos.

**¿Cómo puedo desinstalar el agente de aprovisionamiento?**
1. Inicie sesión en el servidor de Windows en el que está instalado el agente de aprovisionamiento.
2. Vaya al menú Panel de control -> Desinstalar o cambiar un programa.
3. Desinstale los programas siguientes:
     - Microsoft Azure AD Connect Provisioning Agent
     - Agent Updater de Microsoft Azure AD Connect
     - Microsoft Azure AD Connect Provisioning Agent Package


## <a name="next-steps"></a>Pasos siguientes

- [Aprovisionamiento de aplicaciones](user-provisioning.md)
- [Requisitos previos del host del conector ECMA de Azure AD](on-premises-ecma-prerequisites.md)
- [Instalación del host del conector ECMA de Azure AD](on-premises-ecma-install.md)
- [Configuración del host del conector ECMA de Azure AD](on-premises-ecma-configure.md)
